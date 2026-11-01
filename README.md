# ganjuko-updates

OTA (over-the-air) update endpoint for the **Ganjuko** Android app
(`com.cloudcorprecords.ganjuko`, a Capacitor wrap of an RPG Maker MZ game).

This repo is a static update server — no code runs here:

- **`manifest.json`** (this repo's `main` branch) is the manifest the app
  fetches on launch/resume when it is on Wi-Fi. It names the latest bundle
  version and where to download it.
- **Releases** host the bundle zips. A bundle is the game's web payload
  (engine JS, species data, images, audio, maps — `index.html` at the zip
  root), i.e. exactly the content the APK already carries. No game source
  repository, no secrets — the same compiled payload that ships in the APK.

## How updates flow

1. App starts (or resumes) on Wi-Fi → fetches `manifest.json`.
2. If `latest` differs from the running bundle, the app downloads the zip
   in the background (SHA-256 verified) and stages it.
3. The staged bundle applies at the **next** app start. A running session
   is never interrupted.
4. Every bundle must confirm its own boot health; a bundle that fails to
   boot is automatically rolled back to the previous bundle by the
   in-app updater (@capgo/capacitor-updater, manual/self-hosted mode).

Saves live in WebView local storage and survive bundle swaps.

## Publishing a new bundle

1. Copy the current game project into the staging `www` (see the build
   report in the main Ganjuko workspace), bump `www/ota-version.json`.
2. Zip the **contents** of `www` (index.html at the zip root) as
   `bundle-<version>.zip`; compute its SHA-256.
3. Create a GitHub Release tagged `<version>` with the zip attached.
4. Add the entry to `manifest.json` and set `latest` to the new version.
   Commit + push. Done — installed apps pick it up on their next
   Wi-Fi launch.

Native shell changes (new plugins, permissions, icon, min SDK) still
require a fresh APK install; this endpoint updates game content only.
