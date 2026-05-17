# LAB 11 ANDROID ROOT DETECTION BYPASS – FRIDA STRIKE OPERATION

> *EL YAMANI OMAYMA*  

---

##  EXECUTIVE SUMMARY

**Target Application:** OWASP Uncrackable Level 1  
**Attack Vector:** Frida Dynamic Instrumentation  
**Objective:** Neutralize all root detection mechanisms (Java + Native layers)  
**Status:** ✅ **FULLY COMPROMISED**

---

##  PHASE 1 – INFRASTRUCTURE RECONNAISSANCE

### 1.1 Device Validation

Initial connectivity check confirms the Android emulator is responsive and ready for instrumentation.

```bash
adb devices
Output: Emulator emulator-5554 detected in device mode.
```

<img width="861" height="106" alt="image" src="https://github.com/user-attachments/assets/01dc45ab-fdc1-4b8d-bbdf-3d71a75c333c" />

### 1.2 Hardware Profiling
CPU architecture extraction determines the correct frida-server binary variant.

```bash
adb shell getprop ro.product.cpu.abi
```
Result: x86_64 – 64-bit Intel architecture confirmed.

<img width="555" height="40"  src="https://github.com/user-attachments/assets/3afdfd3d-9776-4c8d-bcbf-f4feaaac708d" />

---
## PHASE 2 – FRIDA DEPLOYMENT
### 2.1 Server Process Verification

The frida-server daemon must be operational before any hooking attempt.

```bash
adb shell ps | findstr frida
```
Discovery: Process frida-server running with PID 5758 – injection ready.

<img width="490" height="28"  src="https://github.com/user-attachments/assets/29ca2e8f-e560-4710-b7b7-cb584ab2ad6a" />

### 2.2 Port Forwarding Configuration
Network tunnels established for bidirectional communication between host and device.

```bash
adb forward --list
```
Tunnels Active:

```text
tcp:27042 → tcp:27042

tcp:27043 → tcp:27043
```

<img width="278" height="59"  src="https://github.com/user-attachments/assets/efc73917-986e-4d23-ac2d-ebab243d3091" />

### 2.3 Process Enumeration

Live process audit reveals all running applications on the target device.

```bash
frida-ps -U
```
Notable Processes: Camera, Chrome, Google Play Store, Phone, Photos, Settings, and the target application.

<img width="533" height="212"  src="https://github.com/user-attachments/assets/0a28b557-d83d-4c8d-baf1-7350c9acebf8" />

### 2.4 Version Integrity Check
Version consistency between client and server is critical for stable hooking.

```bash
frida --version
Result: Version 17.9.1 matched across all components.
```

<img width="438" height="50"  src="https://github.com/user-attachments/assets/ae858501-73fa-42fb-8cfc-d7b9a4341bb4" />

---
## PHASE 3 – ROOT DETECTION MECHANISMS IDENTIFIED

### 3.1 Java Layer Protections

| Check Method | Detection Logic | Bypass Strategy |
|---|---|---|
| Build.TAGS | Looks for "test-keys" | Hook getter → return "release-keys" |
| File.exists() | Checks /system/bin/su, /system/xbin/su | Intercept → return false |
| Runtime.exec() | Executes su, which su, busybox | Block execution → substitute harmless command |
| RootBeer.isRooted() | Third-party library check | Hook method → return false |
| AlertDialog.show() | Displays "Root detected" popup | Suppress dialog entirely |
| System.exit() | Terminates application on detection | Block call → prevent crash |

### 3.2 Native Layer Protections

| Check Method | Detection Logic | Bypass Strategy |
|---|---|---|
| open() / openat() | Accesses /proc/mounts, /proc/self/maps | Hook → return -1 |
| access() | Checks file permissions on suspicious paths | Hook → hide existence |
| stat() / lstat() | Retrieves file metadata for su binaries | Hook → spoof absence |

---
## PHASE 4 – INJECTION & HOOK EXECUTION
### 4.1 Single Script Deployment
Initial injection with JavaScript-based root detection bypass.

```bash
frida -U -f owasp.mstg.uncrackable1 -l bypass_root.js --no-pause
```
Console Output Analysis:

```text
[+] Hooks a/b/c installés -> root detection désactivée
[+] Hook Build.TAGS -> release-keys
[+] Hooks Runtime.exec installés
[+] Hook System.exit installé
[+] Hook AlertDialog installé
[+] Java layer bypass installed
[+] sg.vantagepoint.a.c.a() -> false
[+] Blocked AlertDialog.show()
```
Interpretation: All Java-layer root detection hooks successfully registered. The method sg.vantagepoint.a.c.a() which previously returned true (root detected) now returns false.

<img width="584" height="101"  src="https://github.com/user-attachments/assets/2cffb81c-e9db-42d9-88b4-648b34de7d8d" />

### 4.2 Multi‑Script Aggressive Bypass

Enhanced attack combining multiple instrumentation scripts for maximum coverage.

```bash
frida -U -f owasp.mstg.uncrackable1 -l hook_abc.js -l bypass_root.js
Result: Both Java and native detection vectors neutralized simultaneously.
```

<img width="1600" height="679" alt="image" src="https://github.com/user-attachments/assets/2dbc6450-bb65-4e00-8d0f-699f07582f60" />

---
## PHASE 5 – COMPROMISE ASSESSMENT

### 5.1 Before vs After Comparison

| Security Control | Pre‑Injection State | Post‑Injection State |
|---|---|---|
| Root Status Flag | true (detected) | false (hidden) |
| Build.TAGS | test-keys | release-keys (spoofed) |
| Su Binary Detection | Positive | Negative (blocked) |
| Runtime Execution | su command runs | Command blocked |
| Alert Dialog | Visible | Suppressed |
| Application Termination | Immediate exit | Normal operation |

### 5.2 Hook Coverage Summary

| Hooked Component | Intercepted Method | Outcome |
|---|---|---|
| android.os.Build | TAGS getter | Return spoofed value |
| java.io.File | exists() | Hide suspicious files |
| java.lang.Runtime | exec() | Block root commands |
| com.scottyab.rootbeer.RootBeer | isRooted() | Force false |
| android.app.AlertDialog | show() | Suppress popup |
| java.lang.System | exit() | Prevent termination |

---
## PHASE 6 – TECHNICAL CONCLUSION
### 6.1 Attack Success Criteria

✅ Frida-server deployed and operational

✅ Port forwarding configured correctly

✅ Target process identified and attached

✅ Java-layer hooks registered successfully

✅ Native-layer hooks intercepting system calls

✅ Application no longer detects root environment

✅ No termination or crash observed

### 6.2 Security Implications
This demonstration proves that client‑side root detection is insufficient for protecting sensitive applications. A determined attacker with physical access and Frida can:

- Spoof system properties

- Hide file existence

- Block command execution

- Suppress warning dialogs

- Force false return values from security checks

### 6.3 Defense Recommendations

| Weakness | Recommended Mitigation |
|---|---|
| Client‑side only checks | Implement server‑side attestation |
| Hardcoded path strings | Obfuscate or encrypt detection logic |
| Single‑point hooks | Use multiple independent detection layers |
| No anti‑tampering | Add Frida detection and integrity checks |
| Predictable behavior | Introduce randomization in check timing |
