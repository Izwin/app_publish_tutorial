# 📱 Publishing a Flutter App to Google Play and App Store

This README contains all essential commands and configuration snippets required to prepare and publish a Flutter application.  
Use it as a **reference guide** for your release process.

---

## 🔧 Project Preparation

### Check `pubspec.yaml`
```yaml
name: my_app
description: My first Flutter app
version: 1.0.0+1   # (user-facing version + build number)

dependencies:
  flutter:
    sdk: flutter

dev_dependencies:
  flutter_launcher_icons: ^0.13.1
```

---

## 🎨 App Icon Setup

### Add `flutter_launcher_icons` configuration to `pubspec.yaml`
```yaml
flutter_icons:
  android: true
  ios: true
  image_path: "icon.png"   # 1024x1024 PNG
  adaptive_icon_background: "#ffffff"
  adaptive_icon_foreground: "icon.png"
```

### Generate icons
```bash
flutter pub get
dart run flutter_launcher_icons:main
```

---

## 🚀 Test in Release Mode

```bash
flutter run --release
```

---

## 🤖 Publishing to Google Play

### 1. Create a signing key
```bash
keytool -genkey -v -keystore mykey.keystore -alias my-key-alias   -keyalg RSA -keysize 2048 -validity 10000
```

### 2. Create `key.properties` (inside `android/`)
```properties
storePassword=YOUR_STORE_PASSWORD
keyPassword=YOUR_KEY_PASSWORD
keyAlias=my-key-alias
storeFile=../mykey.keystore
```

---

### 3A. Update `android/app/build.gradle` (Groovy DSL)
```gradle
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file("key.properties")
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
    signingConfigs {
        release {
            keyAlias keystoreProperties["keyAlias"]
            keyPassword keystoreProperties["keyPassword"]
            storeFile keystoreProperties["storeFile"] ? file(keystoreProperties["storeFile"]) : null
            storePassword keystoreProperties["storePassword"]
        }
    }
    buildTypes {
        release {
            signingConfig signingConfigs.release
            shrinkResources true
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

---

### 3B. Update `android/app/build.gradle.kts` (Kotlin DSL)
```kotlin
import java.util.Properties
import java.io.FileInputStream

val keystoreProperties = Properties()
val keystorePropertiesFile = rootProject.file("key.properties")
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(FileInputStream(keystorePropertiesFile))
}

android {
    signingConfigs {
        create("release") {
            keyAlias = keystoreProperties["keyAlias"] as String
            keyPassword = keystoreProperties["keyPassword"] as String
            storeFile = keystoreProperties["storeFile"]?.let { file(it as String) }
            storePassword = keystoreProperties["storePassword"] as String
        }
    }
    buildTypes {
        getByName("release") {
            signingConfig = signingConfigs.getByName("release")
            isShrinkResources = true
            isMinifyEnabled = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
}
```

---

### 4. Build release App Bundle
```bash
flutter build appbundle
```
Generated file:  
`build/app/outputs/bundle/release/app-release.aab`

---

## 🍏 Publishing to App Store

### 1. Open project in Xcode
```bash
open ios/Runner.xcworkspace
```

### 2. Configure signing in Xcode
- Go to **Signing & Capabilities**
  - ✅ Enable *Automatically manage signing*  
  - ✅ Select your Developer Team  
  - ✅ Ensure `Bundle Identifier` is unique  

### 3. Archive the app
In Xcode:  
`Product > Archive`

### 4. Distribute via App Store Connect
- After building the archive → **Distribute App** → **App Store Connect** → **Upload**  

### 5. Fill metadata in App Store Connect
- App Name  
- Description  
- Keywords (up to 100 chars)  
- Screenshots (for all screen sizes)  
- Age rating  

---

## 🛠️ Useful Commands

### Clean project
```bash
flutter clean
```

### Reinstall dependencies
```bash
flutter pub get
```

### Build APK (if needed)
```bash
flutter build apk --release
```

### Build iOS release locally
```bash
flutter build ios --release
```

### Install release build on device
```bash
flutter install
```

---

## 📌 Best Practices

1. **Keep your keystore and passwords safe** – without them, future updates in Google Play are impossible.  
2. **Always test in release mode** to avoid surprises.  
3. **High-quality screenshots & descriptions = more downloads.**  
4. **Use soft-launch** (release in limited regions first).  
5. **Iterate regularly** – listen to user feedback and update your app often.  
