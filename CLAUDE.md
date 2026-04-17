# App Development Design Spec

## Tech Stack

| Layer | Choice |
|-------|--------|
| Cross-platform | Capacitor |
| UI Framework | React + Vite |
| State | Zustand |
| HTTP | Axios + interceptors |
| Routing | React Router v6 |
| UI Components | Self-built (no third-party UI lib) |

---

## Directory Structure

```
src/
├── modules/                  # Business modules (one folder per feature)
│   ├── splash/               # Splash screen: version display, ad, update check
│   ├── login/
│   └── home/
│       ├── chat/
│       ├── player/
│       └── ...
├── common/
│   ├── components/           # Shared UI components (no business logic)
│   ├── hooks/                # Shared hooks
│   ├── store/                # Zustand stores
│   ├── http/                 # Axios instance + interceptors
│   ├── analytics/            # Analytics SDK wrapper
│   └── permissions/          # Unified permission wrapper
├── router/                   # React Router v6 config
└── main.tsx
```

**Rules:**
- Each `modules/xxx/` contains: `index.tsx`, `components/`, `hooks/`, `store.ts`, `api.ts`
- Module assets (images, styles) live inside the module folder
- `common/components/` must be pure UI — no API calls, no business store dependencies

---

## Naming Conventions

| Target | Convention | Example |
|--------|-----------|---------|
| Component / Class | PascalCase | `LoginPage`, `UserCard` |
| Function / Variable | camelCase | `getUserInfo`, `isLoading` |
| Member variable prefix | `m` | `mWidth`, `mHeight` |
| Constant | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| Enum type | PascalCase + `s` suffix | `UIControlEvents` |
| Enum value | PascalCase, no underscore | `UIControlEventTouchDown` |
| Image file | `module_feature_desc.png` | `uc_user_icon.png` |
| CSS class | kebab-case | `user-avatar`, `nav-bar` |

- Use full words; avoid abbreviations unless universally known (msg, init, img, nav, btn, bg)
- No single-letter variables outside loop indices

---

## Code Quality

- Single file: max **800 lines**
- Single method/function: max **80 lines**
- Methods between blank lines: **1 blank line**
- Code blocks separated by: **1 blank line**
- Delete unused: resources, methods, commented-out code, meaningless comments
- Add comments at: class properties, complex methods, large code blocks

---

## Module Structure (Red Frame)

Organize by **business function**, not by file type.

```
# Good
modules/
  login/
    LoginPage.tsx
    components/
    assets/

# Bad
components/
  LoginForm.tsx
pages/
  Login.tsx
```

- Shared code/assets across modules → `common/`
- A module that grows into a reusable library → extract to `packages/`

---

## Class / Component Structure (Green Frame)

Group methods by function block in this order:

**React Component:**
```
1. Types / Props interface
2. Constants inside component
3. State (useState / useReducer)
4. Refs
5. Derived values / useMemo
6. Effects (useEffect) — in lifecycle order
7. Event handlers (handle*)
8. Async/API calls
9. Render helpers
10. Return (JSX)
```

- Lifecycle effects ordered by **actual execution sequence**
- Event handlers prefixed with `handle`: `handleSubmit`, `handlePress`
- Keep render logic thin — extract sub-components if JSX exceeds ~50 lines

---

## HTTP Layer (`common/http/`)

```
Request interceptor:  inject token → add common headers (appId, channelId)
Response interceptor: parse error code → handle 401 (redirect login)
                      → detect network error → show toast
```

- All API calls go through the shared Axios instance — never create raw `axios.create()` in modules
- Error messages displayed via a single global toast, not per-component alerts

---

## Analytics (`common/analytics/`)

```ts
Analytics.init(appId, userId, channelId)       // call on app launch
Analytics.sendEvent(eventId, attr)              // user action events
Analytics.sendPageBegin(eventId, attr)          // page enter
Analytics.sendPageEnd(eventId, attr)            // page leave
```

- Offline queue: cache events locally when network unavailable, batch-upload on reconnect
- Backend provides both single-event and batch-event endpoints
- Underlying implementation must be swappable (adapter pattern)

**Required events (minimum):**

| Event | Priority |
|-------|----------|
| App launch | High |
| User registered | High |
| Payment started | High |
| Payment completed | High |
| User logged in | Medium |
| Promotion clicked | Medium |
| App exit | Low |
| Page enter/leave | Low |

---

## Permissions (`common/permissions/`)

Handle three states for every permission request:

1. **Never asked** → request permission
2. **Denied** → show dialog explaining why → link to system settings
3. **Granted** → proceed

Permissions to cover: camera, microphone, location, storage, network.

---

## Required App Features

| Feature | Priority | Notes |
|---------|----------|-------|
| Version check on launch | High | API call on splash, dialog to update |
| Push notifications | High | See platform skills |
| Splash version label | Medium | Display current app version |
| Splash ad | Low | Configurable via API |

---

## Test Coverage Requirements

Every feature must cover:

- **Network**: normal, offline, slow, no permission
- **Permissions**: first request, denied, re-request after denial
- **Memory**: repeated navigation, repeated data refresh, image/video loops
- **Background/foreground**: repeated switching, long background → resume
- **Input**: max length, overflow display (ellipsis), special characters
- **Share**: all platforms, link tap, QR code scan

---

## Branch Strategy

```
master          production only, never commit directly
  └── hotfix/*  cut from master, merge back to master + develop
  └── develop   integration branch, never commit directly
        └── function/*  feature branches, merge to develop when done
```

- Developer works on `function/your-name-feature` or personal branch
- Develop → Master only after QA sign-off
