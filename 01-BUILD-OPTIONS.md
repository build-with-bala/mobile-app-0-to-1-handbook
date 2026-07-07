# Build & Ship Options — 0→1 Mobile App (iOS + Android)

> Scope: framework, build toolchain, CI/CD, code signing, OTA, test/distribution channels — **this doc is the source of truth for these shared build tables.** **Store accounts, push setup, IAP, and Apple/Google review gauntlets live in Document 2**, which references these tables rather than restating them.

## 1. Framework decision matrix

| Framework | License / cost | Rendering | Web-code reuse | iOS App Store | Native push + IAP | Pick when |
|---|---|---|---|---|---|---|
| **Capacitor (+Ionic UI)** | OSS MIT, free | System WebView (WK/Android WebView) | ~100% | Yes | Yes (`@capacitor/push-notifications`; RevenueCat / `@capgo/native-purchases`) | **Web-first team, reuse existing web app — DEFAULT** |
| React Native + Expo | OSS MIT; EAS free tier+paid | Native widgets (JSI) | UI rewrite in RN | Yes | Yes (`expo-notifications`; `react-native-iap`/RevenueCat) | Want native feel + managed builds |
| React Native (bare) | OSS MIT, free | Native widgets | UI rewrite | Yes | Yes | Max native config control, no Expo layer |
| Flutter | OSS BSD, free | Own canvas (Skia/Impeller) | 0% (Dart rewrite) | Yes | Yes (`firebase_messaging`; official `in_app_purchase`) | Native perf/consistency, no web reuse needed |
| **Kotlin Multiplatform / Compose MP** | OSS Apache-2.0, free | Compose MP = own canvas (Skia) on iOS / native on Android; or native UI per platform | 0% (Kotlin rewrite) | Yes | Yes (native interop via expect/actual; KMP push/IAP libs, RevenueCat KMP) | Kotlin/Android team sharing logic across native |
| **.NET MAUI** | OSS MIT, free | Native widgets (handlers) | 0% (C#/XAML rewrite) | Yes | Yes (`Plugin.Firebase` push; `Plugin.InAppBilling`/.NET IAP) | C#/.NET shop |
| PWA (installed) | Web standards, free | WebView (browser) | ~100% | **No listing, no native IAP** | iOS web push only after Add-to-Home-Screen (16.4+) | Can skip App Store entirely |
| TWA + Bubblewrap | OSS Google CLI, free | WebView | ~100% | **Android-only, no iOS** | Play Billing via Digital Goods API | Ship a PWA to Play cheaply |
| PWABuilder | Free (MS, hosted) | WebView | ~100% | iOS WKWebView output **risks 4.2 reject** | — | Fast Android TWA path |
| Native (Swift/Kotlin) | Free SDKs | Native | 0% (2 codebases) | Yes | Yes | Bleeding-edge platform APIs only |

**Recommended default = Capacitor.** Wraps existing web app unchanged → both stores, official push + IAP plugins, no rewrite. TWA(+installed PWA) = zero-native Android fallback; **iOS has no TWA equivalent** and a bare WKWebView wrapper draws Apple **Guideline 4.2** rejection (thin website wrapper) → *see Doc 2*. Add real native features (push, offline, native nav, device APIs) to clear 4.2.

**Rendering split:** Capacitor/PWA/TWA = system WebView (max reuse); RN + MAUI = native widgets (UI rewrite); Flutter + Compose MP = own pixels (full rewrite). Preserving web code ⇒ Capacitor/PWA/TWA; KMP/MAUI/Flutter/RN = rewrites chosen for native feel.

## 2. Build toolchain — the Mac problem

- **iOS: a Mac is mandatory.** Xcode + `codesign`/Keychain are macOS-only; the final `.ipa` build/sign/submit MUST run on macOS. Cross-platform frameworks let you *write* anywhere; they do not remove this. Options: own Mac, rented cloud Mac, or macOS CI runner.
- **Android: no Mac needed.** Android Studio runs on macOS/Win/Linux; wraps Gradle (`./gradlew bundleRelease` → `.aab`).
- Google Play: **`.aab` mandatory** (new apps since Aug 2021, updates since Nov 2021); **target API 35 / Android 15** for new apps+updates since **2025-08-31** (Wear/Auto/TV: API 34). **Next gate: API 36 / Android 16 target bump expected ~2026-08-31** ⚠︎verify-at-submit — API 35 stales within weeks. `.apk` still fine for sideload/other stores.
- **Size limits** (force "download heavy media post-install" — Doc 2 §5.2): iOS app bundle **≤ 4 GB**; **cellular download cap ~200 MB** (over → Wi-Fi-only prompt) ⚠︎verify-at-submit. Android **`.aab` base module ≤ 200 MB** install; above that use **Play Asset Delivery / dynamic feature modules** (asset packs to 512 MB install-time, ~2 GB on-demand). Oversized binaries are rejected/undeliverable at upload.

**Rented Mac (persistent box, not per-build):**

| Service | Pricing | Note |
|---|---|---|
| MacStadium | Mac mini M4 from ~$119/mo ⚠︎verify-at-submit | Dedicated; month-to-month +15–30% vs annual; M4 may need 3+ annual |
| MacinCloud | Hourly / PAYG / managed | Sporadic one-off builds |
| AWS EC2 mac | Per-hour, **24h min host allocation** | Elastic but costly for a single build |

## 3. CI/CD build services

| Service | OSS / self-host | Free tier | Paid | Notes |
|---|---|---|---|---|
| **GitHub Actions** (hosted) | Self-host runners possible | 2,000 min/mo private (Team 3,000, Ent 50,000); **public = unlimited** | usage | Linux 1×/Win 2×/**macOS 10×** → 2,000 free = ~200 macOS min. **macOS $0.062/min** (Jan 1 2026, −39%), Linux $0.006, Win $0.010 ⚠︎verify-at-submit |
| **Xcode Cloud** (Apple, hosted) | No | **25 compute-hrs/mo** | $49.99/mo (100 hrs) · $99.99 (250) · $399.99 (1,000) ⚠︎verify-at-submit | Apple-native; auto-signs, TestFlight/ASC integrated; macOS-only, needs $99/yr membership — **sidesteps the GH 10× / Mac-rental problem** |
| **EAS Build** (Expo) | No (self-host N/A) | **15 iOS + 15 Android/mo** (30 total), slow queue, 45-min cap | Starter $19/mo ($45 credit, priority, 2h, 1 concurrency); Production $199/mo ($225 credit, 2); Ent custom ($1,000+); +$50/mo/concurrency | credits **don't roll over** |
| **Codemagic** | No | **500 macOS-M2 min/mo** | PAYG M2 $0.095/min, M4 $0.114/min, Linux/Win $0.045; annual M2 $3,990/yr, M4 $5,400/yr (3 concur.); Ent from $12,000/yr; +$49/mo concurrency | best free macOS allowance |
| **Bitrise** | No | Hobby 300 credits/mo (1cr = 1 Linux min **OR 30s macOS**), 5 concur., 1 private app, 90-min | Starter $99/mo ($89 annual, 3 concur., ~100 builds); Pro $218/mo ($200 annual, up to 10 macOS concur.) | mobile-DevOps visual builder |
| **CircleCI** | Self-host runners | 30,000 credits/mo (no rollover); macOS **runs on free** (m4pro.medium 200 cr/min ≈$0.12) → ~150 macOS min/mo | Performance from $15/mo | **M1/M2 classes EOL Feb 16 2026** → M4 Pro ⚠︎verify-at-submit |
| **Jenkins** | **OSS free** | your Mac hardware | — | Mac mini agents + Fastlane; you own all ops |
| **GitLab CI** | **OSS self-managed free** | register own macOS runner | SaaS macOS runners paid | integrated if on GitLab |
| **Fastlane** | **OSS MIT free** | runs in any CI or local | — | `match`/`gym`/`pilot`/`deliver`/`supply`/`scan` — the free signing+upload glue |

> ⚠ GitHub's announced **$0.002/min self-hosted-runner charge (slated Mar 1 2026) was POSTPONED** after backlash — **not currently billed**.
> ⚠ GitHub multipliers (1×/2×/10×) govern **free included-minute drawdown**, NOT the PAYG list rates ($0.006/$0.010/$0.062per min), which are priced independently — don't derive one from the other (that's why 2× and 10× don't cleanly map to the $ figures).

## 4. Code signing

**iOS**
- **App Store Connect API key (`.p8`)** — the CI path. Generate in ASC → Users & Access → Keys. Need **`.p8` (downloadable ONCE) + Key ID + Issuer ID**; store as CI secrets; **up to 50 Team keys/account** (the *2-key* limit is the APNs `.p8` push key, a different credential — *see Doc 2 §3.1*). Enables automatic signing + TestFlight/App Store uploads **without 2FA**. Signing scope (certs/profiles) needs an **Admin-role** key; a Developer-role key can only upload.
- Manual alt: create Distribution certificate + App Store provisioning profile in the Apple portal.
- **Fastlane `match`** = syncs certs+private keys via an **encrypted private git repo** (team code-signing). Repo MUST be private — a leak = leaked signing identity.

**Android**
- **Play App Signing** (recommended, default for new apps): you hold a **resettable UPLOAD key**; **Google holds the immutable APP SIGNING key** (RSA 4096). Losing a self-managed signing key permanently blocks updates — let Google hold it.
- Generate upload keystore: `keytool -genkeypair`. Enroll Play App Signing on first upload.
- **TWA gotcha:** `assetlinks.json` must list the **Play-managed signing-cert SHA-256**, not your upload key — else silent verification fail → URL bar appears.

## 5. OTA updates (JS-asset only)

**Legality:** allowed ONLY for interpreted JS/HTML/CSS/assets. Legal basis = **Apple DPLA §3.3.2** (WebKit/JavaScriptCore interpreted code that does **not change the app's advertised primary purpose**) — *not* Guideline 2.5.2, which is the prohibition. Google Play **Device & Network Abuse** bars downloaded executable code (dex/JAR/.so) but permits JS in a webview. **Native/plugin/Capacitor-version bumps MUST ship through the store.**

| Tool | Framework | OSS / self-host | Pricing |
|---|---|---|---|
| **Capgo** | Capacitor | **Yes** (plugin LGPL-3.0, backend AGPL-3.0) | Cloud headline (annual-effective/mo): **$12 / $33 / $83 / $208+** — Solo 2k MAU·100GiB·20GiB / Maker 10k·1TB / Team 100k·10TB / Ent 1M+ (from $2,490/yr). 14-day trial, **no perpetual free** ⚠︎verify-at-submit |
| **Capawesome Cloud** | Capacitor | No | from **$9/mo**, free tier, unlimited live updates ⚠︎verify-at-submit |
| **EAS Update** | RN/Expo | Yes (open `expo-updates` protocol) | **Free 1k MAU**/100GiB/20GiB; Starter $19/mo 3k; Production $199/mo 50k/1TiB; Ent 1M+. Overage BW $0.10/GiB, storage $0.05/GiB. **MAU = downloaded ≥1 update** ⚠︎verify-at-submit |
| **code-push-server** (MS) | RN only | **Yes, MIT** (you pay infra) | Free — but repo **archived read-only 2025-05-20**, unmaintained |
| Microsoft App Center / CodePush | — | — | **DEAD — retired 2025-03-31. Do not build new.** |
| Ionic Appflow | Capacitor/Cordova | No | **CLOSED to new customers, sunsets 2027-12-31 — AVOID** |

## 6. Distribution & test channels

| Channel | Capacity | Review | Note |
|---|---|---|---|
| **TestFlight internal** | ≤100 (ASC-roled users) | None, instant | fastest team loop |
| **TestFlight external** | ≤10,000/app | **Beta App Review on first build of a version** (later builds may skip); ≤6 submissions/24h | builds **expire 90 days** after upload |
| **Play internal testing** | ≤100 | None, near-instant | fastest Play track |
| **Play closed testing** | 2,000/list × 50 lists | — | new **personal** accounts: **≥12 testers, 14 continuous days** before production → *see Doc 2* |
| **Play open testing** | Unlimited (or cap ≥1,000) | — | after closed-test gate met |
| **Firebase App Distribution** | Free; 500/project, 200/group | — | iOS needs registered **UDIDs** (ad-hoc); pre-store QA only |
| **Play Internal App Sharing** | link, any signing key, reusable version codes | — | link **expires 60 days**, ≤100 users/link |

**Rollout control (updates only; first publish always 100%):**
- **Apple phased release:** 1/2/5/10/20/50/100% over 7 days (one step/24h), auto-update users only; pausable up to 30 days (unlimited pauses). ⚠︎verify-at-submit
- **Google staged rollout:** pick %, **raise manually** (no auto-increase), halt anytime.

## 7. Build-commands cheat-sheet

```bash
# Capacitor
npm run build && npx cap sync
npx cap open ios              # Xcode → Product > Archive > Distribute
cd android && ./gradlew bundleRelease      # → app-release.aab

# React Native (bare)
xcodebuild ... / Xcode Archive             # iOS (needs Mac)
cd android && ./gradlew bundleRelease      # .aab
cd android && ./gradlew assembleRelease    # .apk

# Expo
eas build -p ios         # / -p android
eas submit -p ios        # / -p android

# Flutter
flutter build ipa        # needs Mac
flutter build appbundle  # .aab
flutter build apk        # .apk

# Kotlin Multiplatform (Compose MP)
./gradlew :composeApp:assembleRelease      # Android .aab/.apk
./gradlew :composeApp:iosArm64 ...         # then Xcode Archive (needs Mac)

# .NET MAUI
dotnet publish -f net8.0-ios               # needs Mac
dotnet publish -f net8.0-android           # .aab

# TWA (Android-only, PWA → Play)
bubblewrap build         # or PWABuilder GUI  → signed .aab

# Fastlane lanes
# iOS:     match  → gym → pilot (TestFlight) / deliver (App Store)
# Android: gradle → supply (Play track upload)
```

## 8. Recommended stack (web-first 0→1)

```
Framework   Capacitor (reuse existing web app; add native push+IAP+offline to clear Apple 4.2)
Android alt TWA + Bubblewrap for a zero-native Play build (assetlinks.json = Play signing SHA-256)
Build host  Codemagic (500 free macOS-M2 min/mo) to start; GitHub Actions if code already on GH;
            or Xcode Cloud (25 free compute-hrs/mo) if all-in on Apple tooling
Automation  Fastlane (match/gym/pilot/deliver/supply) — free, portable across any CI
iOS signing App Store Connect API key (.p8 + Key ID + Issuer ID, Admin role) → automatic signing
Android sig Play App Signing (Google holds signing key; you guard a resettable upload keystore) → .aab
OTA         Capgo (JS-asset only); EAS Update instead if the app is RN/Expo
Test        TestFlight internal (iOS) + Play internal/Internal App Sharing (Android); Firebase App
            Distribution for one cross-platform QA pipe
Ship        Apple phased release (7-day auto) + Google staged rollout (manual %); start ~1–5%
```

**Non-negotiable gotchas:** iOS has **no Mac-free path**; GitHub free macOS ≈ ~200 min/mo (10× multiplier) — Xcode Cloud's 25 free hrs is the managed escape; EAS free queue is slow + 45-min cap; `.p8` and any self-managed Android signing key are **one-mistake-fatal** — save them; target **API 35** now → **API 36 ~Aug 2026** ⚠︎verify and ship **`.aab`** or Play rejects at upload; keep binaries under **iOS 4 GB / Android 200 MB base** — stream heavy media post-install; OTA **cannot** change native code or the app's primary purpose.

*Store accounts (Apple $99/yr, Google $25 one-time), push (FCM/APNs) setup, IAP/StoreKit/Play Billing + commissions, and the full Apple/Google review gauntlets → **Document 2**.*