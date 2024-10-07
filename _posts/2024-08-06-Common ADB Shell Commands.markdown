---
title: Common ADB Shell Commands
date: 2024-09-06 14:10:00 +0800
categories: [Android App Security]
tags: [Android, App Security, ADB, Android App Security]
---

# Common ADB Shell Commands

# For package mangement

## pm

```bash
adb shell pm
```

**Purpose:** Provides various package management functionalities, including listing installed packages, installing new apps, and uninstalling apps.

## List packages

```bash
adb shell pm list packages
```

- **Purpose:** Lists all installed packages on the device. You can filter results by a specific keyword.

## Path of apk of installed application

```bash
adb shell pm path <packagename>
```

- **Purpose:** It gives the path of apk for the packages installed in a system. So we can use that to pull to host machine for static analsyis.

## package info

```bash
adb shell pm dump <package_name>
```

- **Purpose:** Provides detailed information about a specific package, including permissions, activities, services, and more.

## install

```bash
adb shell pm install /path/to/app.apk
```

- **Purpose:** Installs an APK file on the device. You can replace an existing app with the `r` flag

## install APK with permissions

```bash
adb install -g /path/to/app.apk
```

- **Purpose:** Installs an APK and grants all runtime permissions specified in the manifest automatically.

## uninstall

```bash
adb shell pm uninstall <package_name>
```

- **Purpose:** Uninstalls the specified application from the device.

## clear

```bash
adb shell pm clear <package_name>
```

- **Purpose:** Clears all data associated with the specified application, effectively resetting it to its initial state.

## enable

```bash
adb shell pm enable <package_name>
```

- **Purpose:** Enables a previously disabled application, allowing it to run again.

## disable

```bash
adb shell pm disable-user <package_name>
```

- **Purpose:** Disables a package, preventing it from running and removing it from the launcher.

## check install

```bash
adb shell pm path <package_name>
```

- **Purpose:** Displays the path to the APK file of the specified package if it is installed; otherwise, it returns an error.

## grant permission

```bash
adb shell pm grant <package_name> <permission>
```

- **Purpose:** Grants a specific permission to the application, useful for testing.

## revoke permission

```bash
adb shell pm revoke <package_name> <permission>
```

- **Purpose:** Revokes a specific permission from the application.

## list permissions

```bash
adb shell pm list permissions
```

- **Purpose:** Lists all permissions available in the Android system.

## force stop

```bash
adb shell pm force-stop <package_name>
```

- **Purpose:** Forces the specified application to stop, which can be useful for troubleshooting.

## list permissions for a package

```bash
adb shell pm list packages -f
```

- **Purpose:** Lists all the permissions that the specified package has requested.

## find apps with specific permission

```bash
adb shell pm list packages -g <permission>
```

- **Purpose:** Lists all packages that have been granted a specific permission.

## Extract APK of installed application from the android system

```bash
$ adb shellvbox86p:/ pm list packages
vbox86p:/ pm path packagename
$ adb pull outputofabovecommand_excluding_package
```

#
# For Activity Management

## Start specific activity of an application

```bash
adb shell am start -n <package_name>/<activity_name>
```

- **Purpose:** Starts a specified activity within a given package. Useful for launching specific components of an app.
- **Example:**
    
    ```bash
    adb shell am start -n com.example.app/.MainActivity
    ```
    

## Start Activity with Parameters

### 1. extra string (es and esa)

```bash
adb shell am start -n <package_name>/<activity_name> --es <key> <value>
```

- **Purpose:** Starts a specified activity within a given package and passes extra parameters to it via key-value pairs.
- **Example:**
    
    ```bash
    adb shell am start -n com.example.app/.MainActivity --es user_id "12345"
    ```
    
- **Explanation:** In this example, `user_id` is the key, and `"12345"` is the value being passed to the `MainActivity` of `com.example.app`. The receiving activity can access this value using the `getStringExtra` method.
    
    ![Untitled](../images/android%20app/Untitled.png)
    
    ![Untitled](../images/android%20app/Untitled%201.png)
    

String might not be always the case so for we can use following flag for other data types.

```bash
Datatype      flag              array flag
extra integer --ei              --eia
exta float    --ef              --efa
exra bolllean --ez #in value it takes true and false
```

## start service

```bash
adb shell am startservice -n <package_name>/<service_name>
```

- **Purpose:** Starts a specified service within a given package, allowing you to interact with background processes.
- **Example:**
    
    ```bash
    adb shell am startservice -n com.example.app/.MyService
    ```
    

## stop service

```bash
adb shell am stopservice -n <package_name>/<service_name>
```

- **Purpose:** Stops a specified service within a given package.
- **Example:**
    
    ```bash
    adb shell am stopservice -n com.example.app/.MyService
    ```
    

## broadcast

```bash
adb shell am broadcast -a <action>
```

- **Purpose:** Sends a broadcast intent to the system, which can be used to communicate with other applications or system components.
- **Example:**
    
    ```bash
    adb shell am broadcast -a android.intent.action.BOOT_COMPLETED
    ```
    

## get tasks

```bash
adb shell am get-tasks
```

- **Purpose:** Lists the current tasks and their states, providing insights into the app's activity stack.


#
#
# For System Level Interaction

## Dumpsys

```bash
adb shell dumpsys
```

### Additional Flags

- **`-proto`**: Outputs in a protocol buffer format for easier parsing.
- **`-verbose`**: Provides more detailed output.

### Uses

1. **Activity Management**:
    - **Command**: `adb shell dumpsys activity`
    - **Purpose**: Displays the current activities and their states, including the activity stack, running tasks, and more.
2. **Package Management**:
    - **Command**: `adb shell dumpsys package <package_name>`
    - **Keynote**: We can use it for whole system as well as for specific packages
    - **Purpose**: Provides details about installed packages, their permissions, and the components (activities, services, receivers) they contain.
3. **Memory Usage**:
    - **Command**: `adb shell dumpsys meminfo <package_name>`
    - **Keynote**: We can use it for whole system as well as for specific packages
    - **Purpose**: Displays memory usage details for a specific application, including the amount of memory used by different components.
4. **Battery Information**:
    - **Command**: `adb shell dumpsys battery`
    - **Purpose**: Provides detailed battery statistics, including current charge, health, status, voltage, and temperature.
5. **Window Manager State**:
    - **Command**: `adb shell dumpsys window`
    - **Purpose**: Gives information about the current state of windows, such as visibility, focus, and window attributes.
6. **Connectivity Information**:
    - **Command**: `adb shell dumpsys connectivity`
    - **Purpose**: Displays network connectivity status and details about connected networks (Wi-Fi, mobile data).
7. **Sensor Information**:
    - **Command**: `adb shell dumpsys sensorservice`
    - **Purpose**: Lists all registered sensors and their current states.
8. **Notification Service**:
    - **Command**: `adb shell dumpsys notification`
    - **Purpose**: Shows the current state of the notification system, including active notifications.
9. **Telephony Service**:
    - **Command**: `adb shell dumpsys telephony.registry`
    - **Purpose**: Provides information about telephony services, such as SIM card status, call states, and network types.
10. **System Performance**:
    - **Command**: `adb shell dumpsys cpuinfo`
    - **Purpose**: Displays CPU usage information for each process.
11. **Cpu Information**
    - Command: `adb shell dumpsys cpuinfo`
    - Purpose: Displays CPU usage information for each process currently running on the device.
12. **Process Statistics**
    - **Command:** `adb shell dumpsys procstats`
    - **Purpose:** Displays statistics about processes running on the device, including memory usage and CPU time.
13. **Media Playback**
    - **Command:** `adb shell dumpsys media_session`
    - **Purpose:** Provides information about media sessions and their states.

13. **Power Management**

- **Command:** `adb shell dumpsys power`
- **Purpose:** Displays information about the power management state of the device.

15. **Performance Stats**

- **Command:** `adb shell dumpsys perf`
- **Purpose:** Provides performance metrics and stats for various system components.
1. **Input** **Devices**
    - **Command**: `adb shell dumpsys input`
    - **Purpose:** Provides information about input devices and their current state.

## getprop

```bash
adb shell getprop
```

- **Purpose:** Retrieves system properties. This command gives you access to various system settings and configurations.
- A list of system properties, including device model, Android version, build number, etc. You can also query specific properties:
    
    ```bash
    adb shell getprop ro.product.model
    ```
    

## ps

```bash
adb shell ps
```

- **Purpose:** Lists the processes running on the device, similar to the Unix `ps` command.

## logcat

```bash
adb shell logcat
```

- **Purpose:** Displays the log output from the system and apps. This is useful for debugging and monitoring app behavior in real-time.

## service

```bash
adb shell service
```

- **Purpose:** Interacts with system services directly, allowing you to start, stop, or query the state of various services.

## strace

```bash
adb shell strace -p <PID>
```

- **Purpose:** Traces system calls and signals for a specific process, providing insights into its operations and interactions with the kernel.

# For settings

```bash
adb shell settings <other> <flags>
```

- **Purpose:** Manages system settings and retrieves information about the system configuration.
- **Example:**
    - Retrieve a setting:
        
        ```bash
        adb shell settings get system screen_brightness
        ```
        
    - Set a setting:
        
        ```bash
        adb shell settings put system screen_brightness 100
        ```
        