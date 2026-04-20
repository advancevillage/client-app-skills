# app-ios

iOS-specific constraints for Capacitor + React projects.

---

## Push Notifications (APNs)

- Use native APNs — no third-party push SDK on iOS
- Enable `Push Notifications` + `Background Modes → Remote notifications` in Xcode Capabilities
- Register via `@capacitor/push-notifications` plugin
- Handle foreground / background / terminated states separately
- Device token must be sent to backend on every app launch (token may rotate)

---

## Version Update

- Update prompt links to **App Store** product page
- Use `@capacitor/app` to get current version; compare against API response
- Force-update: block UI, no dismiss. Soft-update: dismissible dialog.

---

## Class / Component Structure (Green Frame)

For any native Swift/ObjC files added inside `app-ios/`:

Group methods in this order:

```
1. life cycle       (viewDidLoad → viewWillAppear → viewDidAppear → viewWillDisappear → viewDidDisappear → dealloc)
2. init             (setup, configure methods)
3. IBAction         (all action methods, suffix with "Action": tapLoginAction)
4. getter / setter  (lazy properties)
5. gesture          (gesture recognizer handlers)
6. notification     (NSNotification observers and handlers)
7. net request      (API call methods)
8. present VC       (navigation / modal presentation)
9. delegates        (one block per protocol implemented)
```

- Lifecycle methods must follow **actual execution order**
- IBAction methods must end with `Action` suffix
- `.m` method order must match `.h` declaration order
- Use `#pragma mark - Block Name` to separate blocks

---

## Category / Extension Naming

```objc
// Feature extension on a base class:
PLVBaseMediaViewController+PPT.h
PLVBaseMediaViewController+Live.h
```

- Format: `BaseClass+FeatureName`
- One category per feature concern — do not create catch-all categories

---

## Module / Asset Organization

```
app-ios/App/
├── Modules/
│   ├── Login/
│   │   ├── PLVLoginViewController.h/.m
│   │   └── Assets/
│   └── Watch/
│       ├── PLVLiveViewController.h/.m
│       ├── PLVVodViewController.h/.m
│       ├── Media/
│       │   ├── Base/
│       │   │   ├── PLVBaseMediaViewController.h/.m
│       │   │   ├── PPT/PLVBaseMediaViewController+PPT.h/.m
│       │   │   └── Live/PLVBaseMediaViewController+Live.h/.m
│       │   └── ...
│       └── Chatroom/
└── Common/
```

- Resources (images, fonts, xibs) live **inside** the module folder, not in a global `Assets.xcassets`
- Shared resources go to `Common/Assets.xcassets`

---

## Capacitor iOS Config (`capacitor.config.ts`)

```ts
ios: {
  contentInset: 'always',
  allowsLinkPreview: false,
  scrollEnabled: false,        // disable if app uses custom scroll
}
```

- Set `NSCameraUsageDescription`, `NSMicrophoneUsageDescription`, `NSLocationWhenInUseUsageDescription`, `NSPhotoLibraryUsageDescription` in `Info.plist` before requesting any permission
- All `Info.plist` permission strings must be user-friendly Chinese descriptions (per App Store review guidelines for CN market)

---

## Build & Release

- Scheme: `Debug` for development, `Release` for TestFlight / App Store
- Version (`CFBundleShortVersionString`) and build number (`CFBundleVersion`) must be bumped before each TestFlight upload
- Use `capacitor sync` before every Xcode build after web changes
