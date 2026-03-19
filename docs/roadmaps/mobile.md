# Mobile Security Roadmap

**Goal:** Understand, test, and secure Android and iOS applications and devices. This covers both research (finding vulnerabilities) and forensics (investigating compromised devices).

!!! warning "Authorization Required"
    Test only on apps and devices you own or have explicit written permission to test. Bug bounty programs exist for authorized mobile research — use them.

---

## The Mobile Landscape

Mobile security is split across two distinct ecosystems:

| Factor | Android | iOS |
| :--- | :--- | :--- |
| OS base | Linux kernel | XNU (Darwin/BSD) |
| App distribution | Play Store + sideloading | App Store only (+ jailbreak) |
| Sandboxing | SELinux + app sandbox | iOS sandbox (strict) |
| Reverse engineering | APK = ZIP, easily decompiled | IPA requires more effort |
| Forensics | Easier to extract data | Apple Secure Enclave complicates extraction |
| Primary test platform | **Start here** | After Android foundation |

**Start with Android.** The open ecosystem, accessible file formats (APK), and available tooling make it far easier to learn fundamentals before moving to iOS.

---

## Phase 0 — Prerequisites

- Linux CLI basics (see [Linux page](../knowledge/systems/linux.md))
- Networking fundamentals (HTTP/S, TLS, DNS)
- Basic programming — Java or Kotlin for Android; Swift/Objective-C for iOS (reading, not writing)
- Python — for scripting analysis

---

## Phase 1 — Android Security Fundamentals

### How Android Works (Security Perspective)

- **APK structure:** An APK is a ZIP. Unzip one and explore.
    - `AndroidManifest.xml` — permissions, activities, exported components
    - `classes.dex` — compiled bytecode (Dalvik/ART)
    - `res/` — resources, sometimes hardcoded secrets here
    - `lib/` — native libraries (.so files)
- **Permissions model:** Dangerous vs Normal permissions; runtime permissions (Android 6+)
- **IPC mechanisms:** Intents (implicit/explicit), Content Providers, Services, Broadcast Receivers
- **Sandboxing:** Each app runs as a unique UID; inter-app communication must go through IPC

### Setting Up an Android Lab

```bash
# Option 1: Physical device (recommended)
# Enable Developer Options → USB Debugging → connect via adb

# Option 2: Android Emulator (AVD in Android Studio)
# Create an AVD without Google Play (easier to root)
# Download Android Studio: https://developer.android.com/studio

# Install ADB (Android Debug Bridge)
sudo apt install adb          # Debian/Kali
brew install android-platform-tools   # macOS

# Basic ADB commands
adb devices                   # List connected devices
adb shell                     # Interactive shell on device
adb pull /sdcard/file.txt .   # Copy file from device
adb install app.apk           # Install APK
adb logcat                    # View device logs (real-time)
```

### Static Analysis

**Step 1: Get the APK**

```bash
# From device (if app is installed)
adb shell pm path com.example.app
# Returns: package:/data/app/com.example.app-1.apk
adb pull /data/app/com.example.app-1.apk ./target.apk
```

**Step 2: Decompile**

```bash
# APKTool — decodes resources and smali bytecode
apktool d target.apk -o output/

# JADX — converts .dex to readable Java
jadx -d output_java/ target.apk
# Or use the GUI: jadx-gui target.apk
```

**Step 3: Hunt for Vulnerabilities**

```bash
# Search for hardcoded secrets
grep -r "password\|api_key\|secret\|token\|AWS\|BEGIN RSA" output/ --include="*.java" -l

# Check exported components in Manifest
cat output/AndroidManifest.xml | grep -i "exported=\"true\""

# Look for SQL queries (potential SQLi)
grep -r "rawQuery\|execSQL" output/ --include="*.java"
```

**Tools:**

| Tool | Purpose | Link |
| :--- | :--- | :--- |
| APKTool | APK decompilation | [ibotpeaches.github.io](https://apktool.org/) |
| JADX | Dex → Java decompilation | [github.com/skylot/jadx](https://github.com/skylot/jadx) |
| MobSF | Automated static + dynamic analysis | [github.com/MobSF/Mobile-Security-Framework-MobSF](https://github.com/MobSF/Mobile-Security-Framework-MobSF) |
| apkleaks | Secret scanning in APKs | [github.com/dwisiswant0/apkleaks](https://github.com/dwisiswant0/apkleaks) |

### Dynamic Analysis & Traffic Interception

Intercept HTTPS traffic between the app and its server:

```bash
# 1. Install Burp Suite Community (free): https://portswigger.net/burp/communitydownload
# 2. Configure proxy: Burp → 127.0.0.1:8080
# 3. On Android (or emulator), set WiFi proxy to your Burp IP:8080
# 4. Install Burp CA cert on device:
#    Go to http://burp → download cert → install as trusted CA

# For apps with certificate pinning, use:
# Frida + objection to bypass pinning at runtime
pip install frida-tools objection
objection -g com.example.app explore
# Then: android sslpinning disable
```

### Common Android Vulnerabilities

| Vulnerability | What to Look For |
| :--- | :--- |
| Insecure data storage | SQLite DBs, SharedPreferences, logs, external storage |
| Exported components | Activities/services/providers with `exported=true` |
| Hardcoded secrets | API keys, credentials in source code or resources |
| Weak cryptography | MD5, SHA1, ECB mode, hardcoded IVs/keys |
| Certificate pinning bypass | Apps that trust user-installed CAs |
| Deep link abuse | Unvalidated intent data from deep links |
| SQL injection | Raw queries in Content Providers |

---

## Phase 2 — iOS Security Fundamentals

### How iOS Works (Security Perspective)

- **Secure Boot Chain:** Every component verified by Apple's signature
- **Secure Enclave:** Separate coprocessor handling keys and biometrics
- **App Sandbox:** Strict isolation; apps cannot access each other's data
- **Code Signing:** All code must be signed; jailbreaking breaks this chain
- **IPA structure:** Like APK, it's a ZIP — but binaries are compiled ARM code (harder to reverse)

### iOS Lab Setup

You need a **jailbroken device** for serious iOS security research (or a simulator for limited testing).

!!! info "Legality"
    Jailbreaking your own device for security research is legal in the US under DMCA exemptions (renewed periodically by the Library of Congress). Legality varies by jurisdiction. It may void your device warranty. Only jailbreak devices you own and use for authorized research.

- **Checkra1n** — jailbreak for A5–A11 devices (works on iOS up to 14.x)
- **Palera1n** — jailbreak for A8+ devices on newer iOS versions
- **Corellium** — virtual iOS devices (paid, used by professionals)

```bash
# After jailbreak, install Frida via Cydia/Sileo
# Then on your Mac/Linux:
pip install frida-tools

# List running apps
frida-ps -U

# Attach to an app
frida -U -n "TargetApp" -l script.js
```

### iOS Static Analysis

```bash
# Extract and decrypt IPA (requires jailbroken device with frida-ios-dump or bfdecrypt)
# Then analyze with:
# - class-dump → extract Objective-C headers
# - Ghidra or IDA Free → reverse the ARM binary
# - Hopper Disassembler → iOS-friendly disassembler (paid, free trial)

# Check for common issues:
# - Keychain misuse (data stored with kSecAttrAccessibleAlways)
# - Logging sensitive data (NSLog in production builds)
# - Insecure URLSession config (disabling TLS validation)
```

---

## Phase 3 — Mobile Forensics

### Android Forensics

```bash
# Logical extraction (no root needed)
adb backup -all -shared backup.ab   # Create backup
# Convert .ab to .tar for analysis:
dd if=backup.ab bs=24 skip=1 | zlib-flate -uncompress | tar xf -

# Physical extraction (needs root or forensic tool)
# Tools: Cellebrite UFED (paid), AXIOM (paid), Andriller (free/freemium)

# Key artifact locations (rooted device or image)
# /data/data/<package>/ — app private data, SQLite DBs
# /data/system/accounts.db — synced accounts
# /data/misc/wifi/WifiConfigStore.xml — stored WiFi passwords
# /sdcard/DCIM/ — camera photos (EXIF metadata!)
```

### iOS Forensics

- **Non-jailbroken:** iTunes/Finder backup (if not encrypted) → use **iBackupBot** or **iPhone Backup Extractor**
- **Encrypted backup:** Requires the backup password — no bypass exists legitimately
- **Jailbroken / chip-off:** Full filesystem extraction possible

**Key iOS artifacts:**

| Artifact | Location | Contains |
| :--- | :--- | :--- |
| SMS/iMessage | `Library/SMS/sms.db` | All messages |
| Call history | `Library/CallHistoryDB/CallHistory.storedata` | Calls |
| Browser history | `Library/Safari/` | Visited URLs |
| Location data | `Library/Caches/locationd/` | Location history |
| App data | `private/var/mobile/Containers/Data/Application/` | Per-app data |

---

## Practice Platforms

| Platform | Focus | Cost |
| :--- | :--- | :--- |
| [DVIA-v2](https://github.com/prateek147/DVIA-v2) | Damn Vulnerable iOS App | Free |
| [DIVA Android](https://github.com/payatu/diva-android) | Damn Vulnerable Android App | Free |
| [HackTheBox — Mobile Challenges](https://academy.hackthebox.com/) | CTF mobile challenges | Free tier |
| [InjuredAndroid](https://github.com/B3nac/InjuredAndroid) | Android security challenges | Free |

## Key Reading

| Resource | Type |
| :--- | :--- |
| [OWASP Mobile Security Testing Guide (MSTG)](https://mas.owasp.org/MASTG/) | Free book — the bible for mobile pentesting |
| [OWASP Mobile Application Security Checklist](https://mas.owasp.org/checklists/) | Testing checklist |
| [Android Security Internals (book)](https://nostarch.com/androidsecurity) | Deep dive, paid |
| [iOS App Reverse Engineering](https://github.com/iosre/iOSAppReverseEngineering) | Free book |
