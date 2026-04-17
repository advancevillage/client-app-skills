# app-android

Android-specific constraints for Capacitor + React projects.

---

## Push Notifications (JPush 极光)

- Use **JPush (极光推送)** SDK — register app in JPush console first
- Add JPush SDK to `app-android/` via Gradle dependency
- Configure `AndroidManifest.xml` with JPush required permissions and receivers
- Use `@capacitor-community/fcm` or a custom Capacitor plugin to bridge JPush events to the web layer
- Send Registration ID to backend on every launch

---

## Version Update

- Update prompt links to the **app market** (channel-specific)
- For multi-channel builds: each channel flavor defines its own market URL
- Force-update: block UI, no dismiss. Soft-update: dismissible dialog.
- Use `@capacitor/app` to get current version; compare against API response

---

## Class / Component Structure (Green Frame)

For any native Kotlin/Java files added inside `app-android/`:

Group methods in this order:

```
1. life cycle             (onCreate → onStart → onResume → onPause → onStop → onDestroy)
2. init                   (setup, configure, inject)
3. onClick                (View.OnClickListener implementations)
4. onActivityResult       (deprecated but still used in some flows)
5. onRequestPermissionsResult  (permission callback)
6. net request            (API call methods)
7. startActivity          (navigation methods)
8. interfaces             (one block per interface implemented)
```

- Lifecycle methods must follow **actual execution order**
- Each interface implementation gets its own `// region Interface: XxxListener` block
- Use `// region Block Name` / `// endregion` to separate blocks in Android Studio

---

## Module / Asset Organization

```
app-android/app/src/main/java/.../
├── login/
│   └── PolyvCloudClassLoginActivity.kt
├── watch/
│   ├── PolyvCloudClassHomeActivity.kt
│   ├── chat/
│   │   ├── PolyvChatBaseFragment.kt
│   │   ├── PolyvChatGroupFragment.kt
│   │   ├── PolyvChatPrivateFragment.kt
│   │   ├── PolyvChatFragmentAdapter.kt
│   │   ├── adapter/
│   │   └── imageScan/
│   ├── danmu/
│   ├── linkMic/
│   └── player/
```

**Naming rules:**
- Activity/Fragment class names carry the module prefix: `PolyvLoginActivity`, not `LoginActivity`
- Adapters named after their parent: `PolyvChatFragmentAdapter`

**Shared resources → `commonui` library:**
- If a module has reusable UI components + drawable/layout resources, extract to a separate Android library module (`commonui`)
- This isolates shared resources from business modules and enables independent versioning

---

## Permissions

Declare all required permissions in `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

- Runtime permissions (API 23+): always use the unified `permissions/` wrapper from `common/`
- Check permission state before any feature that requires it — do not assume granted

---

## Capacitor Android Config

- Run `npx cap sync android` after every web build before opening Android Studio
- `minSdkVersion`: 22 minimum (covers ~99% of active Android devices)
- `targetSdkVersion`: keep up to date with latest stable Android release
- Enable `android:usesCleartextTraffic="true"` only for dev builds, not release

---

## Build & Release

- Use product **flavors** for multi-channel distribution (各应用市场)
- `versionCode` must increment on every release build
- `versionName` follows semver: `major.minor.patch`
- Run `./gradlew assembleRelease` (not debug) for QA and production builds
- Sign with release keystore — never commit keystore or passwords to git
