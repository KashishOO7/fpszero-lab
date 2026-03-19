# Mobile Security

> See the [Mobile Security Roadmap →](../roadmaps/mobile.md) for the full learning path.
> This page is the condensed reference for mobile security concepts and quick-reference tooling.

---

## Mobile Threat Model

Mobile devices are full computers that happen to be in your pocket, location-tracked 24/7, with a microphone, camera, and payment system attached.

### What Makes Mobile Unique

| Factor | Security Implication |
| :--- | :--- |
| Physical access risk | Device lost/stolen → full data exposure if not encrypted |
| Persistent sensors | Camera, mic, GPS can be abused by malicious apps |
| Third-party app ecosystem | App store gatekeeping is the primary sandbox |
| Cellular network | IMSI catchers, SS7 attacks, rogue base stations |
| Mobile-specific APIs | SMS OTP interception, contact access, location |
| App stores as trust anchors | One compromised developer account affects all users |

---

## Android Security Model

### Architecture Stack

```
┌─────────────────────────────────────┐
│  Applications (APK layer)           │
├─────────────────────────────────────┤
│  Android Framework (Java APIs)      │
├─────────────────────────────────────┤
│  ART / Dalvik Runtime               │
├─────────────────────────────────────┤
│  Native Libraries (C/C++)           │
├─────────────────────────────────────┤
│  Hardware Abstraction Layer (HAL)   │
├─────────────────────────────────────┤
│  Linux Kernel + SELinux             │
└─────────────────────────────────────┘
```

### Key Security Mechanisms

**Sandboxing:** Each app runs as a unique UID. By default, apps cannot read each other's data.

**Permissions:** Apps must declare permissions in `AndroidManifest.xml`. Dangerous permissions require runtime user approval (Android 6+).

```xml
<!-- AndroidManifest.xml — security-relevant declarations -->
<manifest>
    <!-- Permissions the app requests -->
    <uses-permission android:name="android.permission.CAMERA"/>
    <uses-permission android:name="android.permission.READ_CONTACTS"/>
    
    <!-- exported=true means OTHER apps can interact with this component -->
    <activity android:name=".MainActivity" android:exported="true">
        <intent-filter>
            <action android:name="android.intent.action.MAIN"/>
        </intent-filter>
    </activity>
    
    <!-- exported=false (default) = only this app can reach it -->
    <activity android:name=".InternalActivity" android:exported="false"/>
    
    <!-- Content provider — can expose data to other apps -->
    <provider
        android:name=".DataProvider"
        android:authorities="com.example.provider"
        android:exported="true"   <!-- ← Dangerous if not protected -->
        android:readPermission="com.example.READ"/>
</manifest>
```

**SELinux:** Mandatory access control at the kernel level — even root processes have constrained permissions.

**Verified Boot:** Cryptographic verification of the boot chain — detect if system partition is tampered.

---

## iOS Security Model

### Key Mechanisms

**Secure Boot Chain:** Every component signed by Apple. Jailbreaking disrupts this.

**App Sandbox:** More restrictive than Android. Apps live in isolated containers.

```
/var/mobile/Containers/Data/Application/<UUID>/
├── Documents/         ← User data, backed up by iTunes
├── Library/           ← App settings, caches
│   ├── Preferences/   ← NSUserDefaults (plist files)
│   ├── Caches/        ← Not backed up
│   └── Application Support/
└── tmp/               ← Temporary, not backed up
```

**Keychain:** Secure credential storage, backed by Secure Enclave.

```swift
// Correct Keychain usage (Swift)
let query: [String: Any] = [
    kSecClass as String: kSecClassGenericPassword,
    kSecAttrAccount as String: "userPassword",
    kSecValueData as String: password.data(using: .utf8)!,
    // Critical: don't use kSecAttrAccessibleAlways
    kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
]
SecItemAdd(query as CFDictionary, nil)
```

---

## Security Testing Quick Reference

### ADB Commands (Android)

```bash
adb devices                          # List connected devices
adb shell                            # Interactive shell
adb install app.apk                  # Install APK
adb pull /data/data/com.app/ .       # Extract app data (root needed)
adb logcat | grep "com.target.app"   # Filter logs for target app
adb shell pm list packages           # List installed apps
adb shell dumpsys activity           # Activity manager state
adb shell am start -n com.app/.MainActivity  # Launch activity
adb shell content query --uri content://com.app.provider/table  # Query content provider
```

### Static Analysis Quick Commands

```bash
# Decompile APK
apktool d app.apk -o decompiled/
jadx -d java_output/ app.apk

# Quick secrets hunt
grep -r "password\|passwd\|secret\|api_key\|apikey\|token\|auth" decompiled/ \
  --include="*.java" --include="*.xml" -l

# Check for debug mode
grep -r "debuggable" decompiled/AndroidManifest.xml

# Find exported components
grep -r "exported=\"true\"" decompiled/AndroidManifest.xml

# Look for insecure data storage
grep -r "MODE_WORLD_READABLE\|MODE_WORLD_WRITEABLE" decompiled/ --include="*.java"

# Check for HTTP (not HTTPS)
grep -r "http://" decompiled/ --include="*.java" --include="*.xml"
```

### Dynamic Analysis Setup

```bash
# Burp Suite proxy setup for Android
# 1. Set Burp proxy to listen on all interfaces: 0.0.0.0:8080
# 2. Configure Android WiFi proxy → your IP:8080
# 3. Visit http://burp → download cert
# 4. Install as trusted CA: Settings → Security → Install Certificate

# For apps with certificate pinning:
# Install Frida on the device/emulator
adb push frida-server /data/local/tmp/
adb shell "chmod 755 /data/local/tmp/frida-server"
adb shell "/data/local/tmp/frida-server &"

# Use objection to bypass pinning
pip install objection
objection -g com.target.app explore
# In objection console:
android sslpinning disable

# Or use Frida directly
frida -U -n com.target.app -l ssl_bypass.js
```

### Frida Snippets

```javascript
// Intercept a method call
Java.perform(function() {
    var TargetClass = Java.use("com.target.ClassName");
    
    TargetClass.checkPassword.implementation = function(password) {
        console.log("[*] checkPassword called with: " + password);
        var result = this.checkPassword(password);
        console.log("[*] Result: " + result);
        return result;
        // Or force return true: return true;
    };
});

// Hook and log all HTTP requests (OkHttp)
Java.perform(function() {
    var Request = Java.use("okhttp3.Request");
    Request.url.overload().implementation = function() {
        var url = this.url();
        console.log("[HTTP] " + url.toString());
        return url;
    };
});
```

---

## Common Mobile Vulnerabilities

| Vulnerability | Platform | How to Find |
| :--- | :--- | :--- |
| Insecure data storage | Both | Grep for SQLite, SharedPrefs, logs with sensitive data |
| Hardcoded credentials | Both | Strings analysis, grep in source |
| Cleartext traffic | Both | Proxy + check HTTP URLs in manifest/code |
| Exported component abuse | Android | Enumerate exported components, test with ADB |
| Insecure deeplinks | Both | Find URL schemes, test parameter injection |
| Weak cryptography | Both | Search for ECB mode, MD5, hardcoded keys |
| Keychain misuse | iOS | Check `kSecAttrAccessible` values |
| Certificate pinning bypass | Both | Frida/objection |
| Path traversal in content provider | Android | Test `../` in content provider URIs |
| WebView XSS/JS interface | Android | Check `addJavascriptInterface` usage |

---

## Tools Quick Reference

| Tool | Platform | Purpose | Link |
| :--- | :--- | :--- | :--- |
| Frida | Both | Dynamic instrumentation | [frida.re](https://frida.re) |
| objection | Both | Runtime mobile exploration | [github](https://github.com/sensepost/objection) |
| MobSF | Both | Automated static + dynamic | [github](https://github.com/MobSF/Mobile-Security-Framework-MobSF) |
| JADX | Android | APK decompilation | [github](https://github.com/skylot/jadx) |
| APKTool | Android | APK decoding | [ibotpeaches](https://apktool.org/) |
| apkleaks | Android | Secret scanning | [github](https://github.com/dwisiswant0/apkleaks) |
| Ghidra | Both | Binary reverse engineering | [ghidra-sre.org](https://ghidra-sre.org/) |
| Hopper | iOS | iOS binary disassembler | [hopperapp.com](https://www.hopperapp.com/) |
| class-dump | iOS | ObjC header extraction | [GitHub](https://github.com/nygard/class-dump) |

## Key References

| Resource | Type |
| :--- | :--- |
| [OWASP MASTG](https://mas.owasp.org/MASTG/) | Mobile Security Testing Guide — the bible |
| [OWASP MASVS](https://mas.owasp.org/MASVS/) | Mobile App Security Verification Standard |
| [Android Security Bulletin](https://source.android.com/docs/security/bulletin) | Monthly Android CVEs |
| [iOS Security Research](https://security.apple.com/) | Apple security documentation |
| [Frida Documentation](https://frida.re/docs/) | Frida API reference |
