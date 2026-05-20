# Android Root Detection Bypass with Objection

> Mobile Instrumentation Laboratory — Mobile Application Security  
> Analyst: Omar Haouani  
> Target APK: `com.pwnsec.firestorm`  
> Tool: Objection (Frida-based)

---

# Overview

This lab demonstrates how to use **Objection**, a mobile instrumentation framework built on top of **Frida**, to bypass Android root detection mechanisms and SSL pinning protections inside a target application.

The project covers:

- Frida environment verification
- Objection installation and setup
- Hooking Android applications
- Root detection bypass
- SSL pinning bypass
- Startup automation
- Native (C/C++) check handling

---

# Objectives

- Install and configure Objection
- Connect to a target Android application
- Use `android root disable`
- Validate root detection bypass
- Automate injection at startup
- Handle native checks and protections

---

# Technical Environment

| Component | Specification |
|---|---|
| Android Device/Emulator | Android 16 Emulator |
| Frida Server | 16.x |
| Objection | v1.12.4 |
| Target APK | `com.pwnsec.firestorm` |

---

# Prerequisites

- Python 3.8+
- pip
- ADB (Android Platform Tools)
- Rooted Android device/emulator
- Frida & frida-server (matching versions)

---

# Phase 1 — Verifying the Frida Environment

Before starting, verify that Frida is correctly running on the target device.

## Command

```bash
frida-ps -U
```

## Result

The Android emulator responds correctly and active Android processes are listed successfully.

<img width="350" height="477" alt="1 (2)" src="https://github.com/user-attachments/assets/83871256-bc0d-4a9b-96a1-d4f14400f63f" />


---

# Phase 2 — Installing Objection

Objection can be installed directly using pip.

## Command

```bash
pip install objection
```

---

# Phase 3 — Launching Objection on the Target Application

## Command

```bash
objection -n com.pwnsec.firestorm start
```

## Result

Objection successfully attaches to the application and opens its interactive console.

| Information | Value |
|---|---|
| Tool | objection v1.12.4 |
| Author | @leonjza (SensePost) |
| Device | Android 16 (usb) |
| Application | com.pwnsec.firestorm |

<img width="332" height="148" alt="2 (2)" src="https://github.com/user-attachments/assets/a09ff7fd-2ba9-4ea0-b529-65d4fc1a31c7" />

---

# Phase 4 — Searching for Root Detection Methods

The following commands help identify classes and methods related to root detection.

## Commands

```bash
android hooking search classes root
android hooking search methods isRoot
```

## Goal

Identify the application's root detection logic and related methods.

<img width="515" height="400" alt="3" src="https://github.com/user-attachments/assets/02beacf0-676d-4088-9bff-42486c3ae448" />


---

# Phase 5 — Bypassing Root Detection

## Commands

```bash
android root disable
android sslpinning disable
```

<img width="464" height="136" alt="4" src="https://github.com/user-attachments/assets/27520dc4-83a1-4ff5-8420-dc2ca2a5be4c" />


---

# Understanding `android root disable`

The command automatically enables several hooks to bypass common root detection mechanisms:

- Hooks `File.exists()` for suspicious paths:
  - `/system/bin/su`
  - `/system/xbin/su`
- Overrides `Build.TAGS`
  - Replaces `test-keys` with `release-keys`
- Hooks `Runtime.exec()`
  - Blocks shell execution attempts
- Hooks `System.getenv()`
  - Hides environment variables containing `su`

---

# Understanding `android sslpinning disable`

This command disables SSL certificate pinning by:

- Replacing TrustManagers
- Overriding `TrustManagerImpl.verifyChain()`
- Allowing HTTPS traffic interception using tools like:
  - Burp Suite
  - MITM proxies

---

# Phase 6 — Bypass Validation

After executing the bypass commands:

- The application no longer detects root
- "Root detected" messages disappear
- All application features become accessible

| Verification | Status |
|---|---|
| Root Detection | Bypassed |
| SSL Pinning | Disabled |
| Application Features | Accessible |

---

# Phase 7 — Startup Automation

Objection commands can be executed automatically during startup.

## Direct Command Injection

```bash
objection -g com.pwnsec.firestorm explore --startup-command "
android root disable;
android sslpinning disable
"
```

## Using a Script

### `bypass_root.obj`

```text
android root disable
android sslpinning disable
```

### Execute Script

```bash
objection -g com.pwnsec.firestorm explore --startup-script bypass_root.obj
```

<!-- IMAGE 5 HERE -->

---

# Phase 8 — Handling Native Checks (C/C++)

Some applications implement root detection using native `.so` libraries.

## Native Hooking Commands

```bash
android native hooking list exports
```

```bash
android native hooking watch --method name_of_function
```

---

# Advanced Native Bypass Using Frida

```javascript
Java.perform(function () {

    var nativeFunc = Module.findExportByName(
        "libnative.so",
        "check_root"
    );

    Interceptor.replace(
        nativeFunc,
        new NativeCallback(function () {

            return 0; // false

        }, 'int', [])
    );

});
```

---

# Objection Command Summary

| Command | Function |
|---|---|
| `objection -g APP start` | Attach to a running application |
| `objection -n APP start` | Start and attach to an application |
| `android root disable` | Bypass root detection |
| `android sslpinning disable` | Disable SSL pinning |
| `android hooking search classes KEYWORD` | Search classes |
| `android hooking search methods KEYWORD` | Search methods |
| `android hooking list classes` | List all classes |
| `android hooking watch class METHOD` | Monitor method execution |

---

# Exploitation Workflow

```text
Rooted Android Device
        ↓
Frida Server Running
        ↓
Objection Connection to App
        ↓
android root disable → Hooks Enabled
        ↓
android sslpinning disable → SSL Disabled
        ↓
Application Successfully Bypassed
```

<!-- IMAGE 6 HERE -->

---

# Developer Recommendations

To protect Android applications against Frida/Objection bypass techniques:

1. Detect Frida and Objection processes
2. Use native code checks (C/C++)
3. Perform runtime integrity verification
4. Delay security checks during execution
5. Detect hooks and tampering
6. Use code obfuscation and virtualization

---

# Checklist

- [x] Python & pip installed
- [x] ADB configured
- [x] Rooted Android device/emulator ready
- [x] Frida Server installed
- [x] Objection installed
- [x] Connected to target application
- [x] `android root disable` executed
- [x] Root bypass validated
- [x] Automation tested

---

# Conclusion

This lab demonstrated:

- The simplicity of Objection for Android instrumentation
- The effectiveness of `android root disable`
- SSL pinning bypass techniques
- Automated injection workflows
- Native bypass possibilities using Frida

Objection significantly simplifies Android application instrumentation and makes advanced bypass techniques accessible to both beginners and experienced security researchers.

---

# Resources

- Objection GitHub Repository
- Frida Documentation
- OWASP MASVS — Anti-Tampering

---

# Author

**Omar Haouani**  
Mobile Application Security Laboratory  
