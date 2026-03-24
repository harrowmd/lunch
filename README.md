# 🥗 Lunch — What's for lunch?

A progressive web app (PWA) that suggests a random **keto-friendly lunch idea** based on your current location and live weather. Available as a browser PWA and as a native Android APK.

**Live app:** https://harrowmd.github.io/lunch/
**Latest Android APK:** https://github.com/harrowmd/lunch/releases/latest/download/lunch.apk

---

## Table of Contents

- [Features](#features)
- [How It Works](#how-it-works)
- [Gestures](#gestures)
- [Weather Emojis](#weather-emojis)
- [Using the PWA](#using-the-pwa)
- [Android APK](#android-apk)
- [Project Structure](#project-structure)
- [APIs Used](#apis-used)
- [Building the Android APK](#building-the-android-apk)
- [GitHub Pages Setup](#github-pages-setup)

---

## Features

| Feature | Detail |
|---|---|
| 📍 Location detection | Uses the browser Geolocation API to find your current position |
| 🌤️ Live weather | Fetches real-time weather from Open-Meteo (free, no API key) |
| 🌡️ Weather emoji | Displays a contextual emoji and temperature in the top-right corner |
| 🍗 Keto lunch ideas | Picks a random meal from TheMealDB across keto-friendly categories |
| 📸 Meal card | Shows a photo, dish name, and a link to the full recipe |
| 👈 Swipe left | Dismiss the current suggestion and load a new one |
| 👉 Swipe right | React with "Yummy! 😋" if the suggestion looks good |
| ❓ Help | Tap the `?` button (bottom-left) for a full feature guide |
| 📱 Installable | Full PWA with manifest and service worker — add to home screen |
| 🤖 Android APK | Native WebView wrapper built automatically via GitHub Actions |

---

## How It Works

### 1. Location
On load, the app requests the device's GPS location via the browser [Geolocation API](https://developer.mozilla.org/en-US/docs/Web/API/Geolocation_API). If permission is denied, the weather step is skipped and a meal is still shown.

### 2. Weather
Latitude and longitude are sent to the [Open-Meteo](https://open-meteo.com) API:

```
GET https://api.open-meteo.com/v1/forecast
    ?latitude={lat}&longitude={lon}
    &current=temperature_2m,weather_code
    &timezone=auto
```

The response provides a [WMO weather code](https://open-meteo.com/en/docs#weathervariables) and temperature in °C, which are mapped to an emoji and a short label (see [Weather Emojis](#weather-emojis) below).

### 3. Meal suggestion
A keto-friendly category is chosen at random from: **Chicken, Beef, Seafood, Lamb, Pork**. The [TheMealDB](https://www.themealdb.com/api.php) free API is called twice:

1. `filter.php?c={category}` — returns all meals in the category
2. `lookup.php?i={id}` — fetches full details (photo, name, source URL) for a randomly selected meal

No user data is stored or transmitted anywhere beyond these two API calls.

### 4. Display
- **Top-left:** `Lunch for {weekday}` — updates automatically each day
- **Top-right:** weather emoji + temperature and condition
- **Centre:** meal card with photo, dish name, and recipe link
- **Bottom-left:** `?` help button

---

## Gestures

| Gesture | Action |
|---|---|
| **Swipe left** on the card (> 80 px) | Card slides off screen; next random meal loads |
| **Swipe right** on the card (> 80 px) | Card snaps back; `Yummy... 😋` appears below the recipe link |
| Short swipe either direction | Card snaps back — no action |

The card follows your finger in real time and fades as you drag left, giving immediate visual feedback.

---

## Weather Emojis

| Emoji | Condition | Rule |
|---|---|---|
| 🥵 | Hot | ≥ 30 °C (overrides weather code) |
| 🥶 | Freezing | ≤ 0 °C (overrides weather code) |
| ☀️ | Clear sky | WMO code 0 |
| 🌤️ | Partly cloudy | WMO codes 1–2 |
| ☁️ | Overcast | WMO code 3 |
| 🌫️ | Fog | WMO codes 45–48 |
| 🌦️ | Drizzle | WMO codes 51–57 |
| 🌧️ | Rain | WMO codes 61–67 |
| ❄️ | Snow | WMO codes 71–77 |
| 🌧️ | Rain showers | WMO codes 80–82 |
| 🌨️ | Snow showers | WMO codes 85–86 |
| ⛈️ | Thunderstorm | WMO codes 95–99 |

The page background colour changes to match the weather (pale yellow for sun, light blue for rain, white-blue for snow).

---

## Using the PWA

### In a browser
Visit **https://harrowmd.github.io/lunch/** in any modern browser. Grant location permission when prompted.

### Install to home screen (iOS)
1. Open the URL in Safari
2. Tap the **Share** button → **Add to Home Screen**
3. The app opens full-screen with no browser chrome

### Install to home screen (Android)
1. Open the URL in Chrome
2. Tap the **⋮ menu** → **Add to Home screen** (or accept the install banner)
3. The app opens full-screen — the address bar is hidden

> The web manifest sets `"display": "fullscreen"`. The Fullscreen API is also requested on first touch to hide the browser address bar during in-browser use.

---

## Android APK

A native Android APK is available for side-loading (installation outside the Play Store). It wraps the live PWA URL in a full-screen WebView with native geolocation permission handling.

### Download and install
1. Download [`lunch.apk`](https://github.com/harrowmd/lunch/releases/latest/download/lunch.apk) from the latest GitHub Release
2. On your Android device go to **Settings → Security** (or **Apps → Special app access**) and enable **Install unknown apps** for your browser or file manager
3. Open the downloaded `.apk` file and tap **Install**
4. Launch **Lunch** from your app drawer

> The `?` help button inside the PWA also shows a **Download Android App** link when viewed on an Android browser (the link is hidden when already running inside the APK).

### APK behaviour vs browser PWA

| Feature | Browser PWA | Android APK |
|---|---|---|
| Weather & meals | ✅ | ✅ (loads live from GitHub Pages) |
| Geolocation | ✅ browser prompt | ✅ native Android permission dialog |
| Fullscreen | Requested on first touch | ✅ always full-screen |
| Download button | Shown on Android browsers | Hidden (already installed) |
| Updates | Instant (service worker) | APK auto-rebuilt by CI on every push |

### How the APK detects it is running inside itself
`MainActivity.java` appends `LunchAPK/1` to the WebView user-agent string. The PWA checks for this token:

```javascript
const ua = navigator.userAgent;
if (/android/i.test(ua) && !/LunchAPK\//.test(ua)) {
  // show download link — Android browser only
}
```

---

## Project Structure

```
lunch/
├── docs/                        # GitHub Pages root (served as the PWA)
│   ├── index.html               # Main app — weather, meals, gestures, help overlay
│   ├── manifest.json            # Web app manifest (name, icons, display mode)
│   ├── sw.js                    # Service worker (cache-first assets, network-first APIs)
│   ├── icon-192.png             # PWA icon (192 × 192)
│   └── icon-512.png             # PWA icon (512 × 512)
│
├── android/                     # Native Android WebView wrapper
│   ├── app/
│   │   ├── build.gradle         # App module (AGP 8.2.2, minSdk 21, targetSdk 34)
│   │   └── src/main/
│   │       ├── AndroidManifest.xml
│   │       ├── java/com/harrowmd/lunch/
│   │       │   └── MainActivity.java   # WebView + geolocation + custom UA token
│   │       └── res/
│   │           ├── layout/activity_main.xml
│   │           ├── values/strings.xml
│   │           ├── values/styles.xml
│   │           └── mipmap-*/ic_launcher.png   # Launcher icons (5 densities)
│   ├── build.gradle             # Root build config
│   ├── settings.gradle
│   └── gradle.properties
│
├── .github/
│   └── workflows/
│       └── build-apk.yml        # CI: build → sign → publish APK to GitHub Releases
│
└── README.md
```

---

## APIs Used

### Open-Meteo
- **URL:** https://open-meteo.com
- **Auth:** None — completely free, no API key required
- **Endpoint used:** `/v1/forecast` with `current=temperature_2m,weather_code`
- **Docs:** https://open-meteo.com/en/docs

### TheMealDB
- **URL:** https://www.themealdb.com
- **Auth:** None — free tier uses public API key `1`
- **Endpoints used:**
  - `/api/json/v1/1/filter.php?c={category}` — list meals by category
  - `/api/json/v1/1/lookup.php?i={id}` — full meal details
- **Docs:** https://www.themealdb.com/api.php

---

## Building the Android APK

The APK is built automatically by **GitHub Actions** on every push to `main`. You do not need to build it manually.

### Workflow: `.github/workflows/build-apk.yml`

| Step | Detail |
|---|---|
| Checkout | `actions/checkout@v4` |
| Java 17 | `actions/setup-java@v4` (Temurin distribution) |
| Android SDK | `android-actions/setup-android@v3` — installs `platforms;android-34` and `build-tools;34.0.0` |
| Generate keystore | `keytool` creates a fresh RSA-2048 signing keystore each run |
| Gradle wrapper | `gradle wrapper --gradle-version 8.2` — generated in CI, not committed |
| Build APK | `./gradlew assembleRelease` with keystore path/password as project properties |
| Publish | `softprops/action-gh-release@v2` — creates/updates the `apk-latest` release with `lunch.apk` |

> **Note:** A new keystore is generated on each CI run. This is fine for side-loading but not suitable for Play Store distribution (which requires a stable signing key).

### Build locally

```bash
# Prerequisites: JDK 17, Android SDK (ANDROID_HOME set), Gradle 8.x

cd android

# Generate a keystore (one-time)
keytool -genkey -v \
  -keystore ../keystore.jks -alias lunch \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -dname "CN=Lunch, O=HarrowMD, C=GB" \
  -storepass yourpassword -keypass yourpassword

# Generate Gradle wrapper
gradle wrapper --gradle-version 8.2

# Build
./gradlew assembleRelease \
  -PKEYSTORE_PATH=../keystore.jks \
  -PKEYSTORE_PASSWORD=yourpassword \
  -PKEY_ALIAS=lunch \
  -PKEY_PASSWORD=yourpassword

# Output: android/app/build/outputs/apk/release/app-release.apk
```

---

## GitHub Pages Setup

The PWA is served from the `main` branch, `/docs` folder.

To configure (already set up for this repo):

1. Go to **Settings → Pages**
2. Source: **Deploy from a branch**
3. Branch: `main` / folder: `/docs`
4. Save — the app is live at `https://{username}.github.io/{repo}/`

The service worker caches `index.html` and `manifest.json` for offline use. API calls (Open-Meteo, TheMealDB) are always fetched live.

---

*Built with [Claude Code](https://claude.ai/code) · Weather by [Open-Meteo](https://open-meteo.com) · Recipes by [TheMealDB](https://www.themealdb.com)*
