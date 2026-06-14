# PIP-QUEST — APK build project

This folder turns the PIP-QUEST web app into a real, installable Android **.apk**
using [Capacitor](https://capacitorjs.com/). The app's actual code lives in `www/`
(it's the same single-file app you already have). Capacitor just wraps it in a
native Android shell so it can be compiled and signed like any other app.

You have two ways to produce the APK. Pick one.

---

## Option A — Cloud build (no Android tools on your computer)

If you don't want to install Android Studio, let GitHub build it for you. The
included workflow (`.github/workflows/build-apk.yml`) runs on GitHub's servers,
which already have the Android SDK.

1. Create a new repository on GitHub.
2. Upload the **entire contents of this `apk-build` folder** to the repo
   (so `package.json`, `capacitor.config.json`, `www/`, and `.github/` all sit
   at the root of the repo).
3. Go to the repo's **Actions** tab. If prompted, enable workflows.
4. The build starts on push automatically. To run it manually, open
   **Build PIP-QUEST APK → Run workflow**.
5. When it finishes (green check), open the run and download the
   **pip-quest-debug-apk** artifact at the bottom. Inside is `app-debug.apk`.
6. Copy that APK to your phone and install it (see `INSTALL.md` for enabling
   "install unknown apps").

That's the whole loop — every push rebuilds the APK.

---

## Option B — Local build (Android Studio on your computer)

Prerequisites: **Node.js 18+**, **Java JDK 17**, and **Android Studio**
(which installs the Android SDK).

From inside this folder:

```bash
npm install            # install Capacitor
npm run android:add    # create the native android/ project (first time only)
npm run sync           # copy www/ into the native project
npm run open           # open it in Android Studio
```

In Android Studio, press **Run** to launch on a connected phone/emulator, or use
**Build → Build Bundle(s) / APK(s) → Build APK(s)** to get an installable file.

Prefer the command line? After `npm run android:add`:

```bash
npm run apk
```

This runs `cap sync` then `gradlew assembleDebug`, and the APK lands at:

```
android/app/build/outputs/apk/debug/app-debug.apk
```

---

## Updating the app later

The app lives entirely in `www/`. If you change `www/index.html` (or any asset),
just re-run `npm run sync` (local) or push again (cloud) and rebuild. No other
steps.

---

## A note on voice transcription inside the APK

The app records voice logs with `MediaRecorder` and transcribes them live using
the browser's **Web Speech API** (`SpeechRecognition`). That works great in
Chrome and in an installed PWA, but inside a packaged Android WebView the Web
Speech API is **often unavailable** — Google doesn't guarantee it there. The app
already detects this and degrades gracefully: you can still record and play back
audio and type the transcript yourself; it just won't auto-transcribe.

If you want rock-solid **native** speech-to-text in the compiled app, add a
Capacitor speech plugin and wire it to the record button:

```bash
npm install @capacitor-community/speech-recognition
npm run sync
```

Then call the plugin's `startListening()` from the recording handler in
`www/index.html` (search for `SpeechRecognition` — that's where to branch to the
native plugin when it's present). This step is optional; everything else in the
app — tasks, objectives, notes, photos, audio recording, XP/levels, themes,
offline storage — works in the APK with no changes.

> Reminder: microphone and camera need permission. Capacitor will prompt on
> first use. Photos and tasks/notes work regardless; only auto-transcription has
> the WebView caveat above.
