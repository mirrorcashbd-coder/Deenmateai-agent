# AnyClaw (Android)

Android APK that embeds a Termux-style Linux bootstrap environment, installs
the latest [OpenCode](https://opencode.ai) release, and presents the built-in
OpenCode web UI inside a WebView. No accounts or login required to start the
app.

## Architecture

```
┌─────────────────────────────────────────┐
│              Android APK                │
│                                         │
│  ┌──────────────┐  ┌────────────────┐   │
│  │   WebView    │  │  Bootstrap     │   │
│  │              │  │  Installer     │   │
│  │  localhost:  │  │                │   │
│  │   18923      │  │  Extracts      │   │
│  │              │  │  Termux env    │   │
│  └──────┬───────┘  └───────┬────────┘   │
│         │                  │            │
│         ▼                  ▼            │
│  ┌──────────────────────────────────┐   │
│  │   /data/data/com.codex.mobile/   │   │
│  │   files/usr/  (Termux prefix)    │   │
│  │                                  │   │
│  │    ├── bin/node                  │   │
│  │    ├── bin/opencode (latest)     │   │
│  │    └── lib/node_modules/         │   │
│  │        └── opencode-ai/          │   │
│  │                                  │   │
│  └──────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

OpenCode is started with `opencode web --port 18923 --hostname 127.0.0.1`,
which serves the OpenCode web UI directly to the embedded WebView.

## Prerequisites

- Android Studio (or just the Android SDK command-line tools)
- Java 17+
- curl (for downloading bootstrap)

## Build Instructions

### 1. Download the Termux bootstrap

```bash
cd android
./scripts/download-bootstrap.sh
```

This downloads `bootstrap-aarch64.zip` (~30 MB) from Termux releases into `app/src/main/assets/`.

### 2. Build the APK

```bash
./gradlew assembleDebug
```

The APK will be at `app/build/outputs/apk/debug/app-debug.apk`.

For a release build:

```bash
./gradlew assembleRelease
```

## First Run

On first launch, the app will:

1. Extract the bootstrap environment (~30 MB compressed, ~100 MB extracted)
2. Install proot and Node.js from the Termux repository
3. Run `npm install -g opencode-ai@latest` (always the latest OpenCode release)
4. Start the OpenCode web server and load the WebView
5. Every launch after that skips straight to step 4

No login, account, or API key is required to open the app. When the user is
ready to use an AI model provider, they can authenticate later with
`opencode auth login`.

## Minimum Requirements

- Android 7.0 (API 24) or higher
- arm64-v8a device (most modern Android phones)
- ~500 MB free storage for bootstrap + Node.js + OpenCode
- Internet connection (for first-run package installs and model API calls)