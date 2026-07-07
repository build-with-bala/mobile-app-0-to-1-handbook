# Complete 0→1 App Build — iOS + Android (Notifications + IAP)

A checkable, ordered handbook. All prices/dates from verified mid-2026 corpus; corrected values used. `⚠︎verify-at-submit` = volatile/in-flux. Companion to the **build-options** doc (**Doc1**): shared build/CI/OTA/distribution/rollout tables live there — referenced here, not duplicated.

---

## 1 · Accounts & Prerequisites

### 1.1 Store accounts — costs & identity

| Item | Apple | Google Play |
|---|---|---|
| Fee | **$99/yr** recurring (Enterprise $299/yr) | **$25 one-time**, unlimited apps |
| Individual verify | Personal legal name, no D-U-N-S; 2FA Apple Account; ~24–48h | Govt ID; email+phone OTP; device verify via Play Console mobile app |
| Org verify | **D-U-N-S** + legal entity + **work email on org domain** + **live functional public website on that domain** + binding-authority; days–2 wks | **D-U-N-S** + org document + rep govt ID + org website; email+phone OTP |
| Fee waiver | Nonprofit / accredited-edu / gov | — |

- **D-U-N-S** = free from Dun & Bradstreet. Time-to-obtain: **~5–7 business days via Apple's link** (no paid expedite there) vs **up to 30 days via D&B general request** (Google's path). Gov entities exempt. Check if entity already has one.
- **Google Sept 2026 developer verification** = FIRST WAVE ONLY (Brazil, Indonesia, Singapore, Thailand); all-dev enrollment ~Mar 2026, broader/global 2027+; extends to sideload/3rd-party stores. **D-U-N-S required only for ORG-type devs**; individuals verify name/address/ID; free hobbyist tier (~20 devices) skips govt-ID. `⚠︎verify-at-submit`

### 1.2 New-personal-Google-account gate (plan ~3 wks runway)

- Personal accounts **created after 2023-11-13** must run **closed testing: ≥12 testers opted-in 14 CONTINUOUS days** before applying for production. Was 20, cut to 12 on **2024-12-11**. Only the CLOSED track counts (internal doesn't). Opt-out resets the streak; Google checks real engagement (recruit 20–25 buffer). Production-access review **≤7 days**. **Org accounts + pre-2023-11-13 personal = EXEMPT.**

### 1.3 Identifiers — permanent, decide up front

| | iOS | Android |
|---|---|---|
| Name | Bundle ID | package name = `applicationId` |
| Format | reverse-DNS `com.company.app`, globally unique | same |
| Mutable? | **NO after first publish**; must match certs/profiles | **NO after first publish**; user-visible in Play URL |

### 1.4 Legal / compliance prereqs (both stores)

- [ ] **Privacy-policy URL** — publicly reachable, loads without error on review day (dead link = instant rejection). Required even for zero-data apps.
- [ ] **In-app account deletion** — Apple **5.1.1(v)** in force since **2022-06-30**; deactivate ≠ delete; if using Sign in with Apple, call **SiwA REST API to revoke tokens** on delete. Google requires **in-app path AND public web deletion URL** usable by uninstalled users + disclosed in Data safety form.
- [ ] **Age ratings** — Apple new tiers **4+/9+/13+/16+/18+**, questionnaire was due **2026-01-31** (now enforced — un-answered apps can't submit updates). Google **IARC** questionnaire mandatory. Answer AI/chatbot + medical/wellness honestly.
- [ ] **Data disclosure** — Apple App Privacy nutrition labels + Google Data safety form (must match actual SDK behavior + privacy manifest).

### 1.5 Tax & banking (only if selling / IAP)

| | Apple | Google |
|---|---|---|
| Agreement | Account Holder signs **Paid Applications Agreement** (Agreements, Tax & Banking) | **Google payments/merchant profile** (auto-links to Console) |
| Bank | One bank account, no splitting | Bank in profile's country, **no PO box** |
| Tax | US tax form required for ALL devs | US W-9 / non-US W-8 |
| Payout | ~45 days after fiscal-month-end | — |
| 1099-K (US) | — | Filed when **>$20k gross AND >200 tx/yr** (OBBBA-2025 restored; some states lower e.g. MA/MD $600) |

- Free-only apps skip banking (Apple) / merchant profile.

---

## 2 · Project Scaffold & Native Capabilities

### 2.1 Framework choice

**Default: Capacitor** (web-first, MIT, free) — wraps your existing web app in a native iOS+Android shell, ~100% web reuse, official push + IAP plugins. Alternatives: **RN+Expo / Flutter** (native-feel rewrite), **installed PWA** (cheapest reach; **no iOS App Store, no iOS native IAP**), **TWA+Bubblewrap** (Android-only PWA wrap; **iOS has no TWA equivalent**).

- All of **Capacitor / RN-Expo / Flutter** reach true native StoreKit 2 + Play Billing + APNs/FCM.
- → Full 9-option OSS/pricing matrix (Capacitor, Ionic, RN bare, Flutter, PWA, TWA, PWABuilder, native Swift/Kotlin) in **Doc1**.

### 2.2 Clear Apple 4.2 (thin-wrapper = #1 rejection)

Add REAL native features before submitting: **push + native navigation + offline cache + device APIs (camera/mic/geo) + IAP**. Bare WKWebView = rejected as "repackaged website."

### 2.3 Scaffold (Capacitor)

```
npm i @capacitor/core @capacitor/cli
npx cap init                       # set appName + bundle ID
# set webDir → build output
npx cap add ios android
npm run build && npx cap sync
```

### 2.4 Build commands

Capacitor: `npm run build && npx cap sync` → iOS `npx cap open ios` (Xcode Archive) / Android `cd android && ./gradlew bundleRelease` (`.aab`). RN / Expo (`eas build`) / Flutter (`flutter build ipa|appbundle`) cheat sheet → **Doc1**.

### 2.5 Build/CI & signing

- **Mac MANDATORY for iOS** — Xcode+codesign are macOS-only. Own / rent / cloud CI / **Xcode Cloud**.
- Google requires **.aab** (not .apk) since Aug 2021; **target API 35** (Android 15) since 2025-08-31 — **API 36 / Android 16 expected required for new apps+updates ~Aug 31 2026 `⚠︎verify-at-submit`**.
- **Play App Signing**: Google holds the immutable app-signing key; you keep a resettable **upload** key (guard it; losing app key = locked out forever).
- **App Store Connect API key (.p8)** = 2FA-free CI signing/upload; **Key ID + Issuer ID + .p8** (downloadable ONCE; Admin role for signing scope).
- **Xcode Cloud** (Apple first-party, built into Xcode/App Store Connect): **25 compute-hrs/mo free**, then paid (100h ~$50/mo `⚠︎verify`) — managed macOS CI with **no Mac to own/rent and no GitHub 10× macOS-minute multiplier**.
- → Full CI/Mac pricing table (Fastlane, GitHub Actions, EAS, Codemagic, Bitrise, CircleCI, MacStadium/MacinCloud/EC2-mac) in **Doc1**.

---

## 3 · Push Notifications

### 3.1 Transports (free under everything)

- **APNs** (iOS only path): **.p8 token auth** — one key never expires, all apps under Team ID, covers dev+prod, **max 2 active keys**, downloadable once.
- **FCM** (Android native, iOS-via-APNs, web): free unlimited. **Legacy HTTP/XMPP + "server key" DEAD since 2024-06-20** → server sends MUST use **FCM HTTP v1 + OAuth2 service-account JSON**.

### 3.2 iOS setup

1. Firebase project → add iOS app → drag `GoogleService-Info.plist` into Xcode target.
2. Apple Developer → Keys → create **APNs Auth Key (.p8)**; record **Key ID + Team ID**; enable Push Notifications on App ID.
3. Xcode → Signing & Capabilities → **Push Notifications** (writes `aps-environment`: development/production) + **Background Modes → Remote notifications**.
4. Firebase → Cloud Messaging → upload `.p8` + Key ID + Team ID (one key = dev+prod).
5. Request permission: `UNUserNotificationCenter.requestAuthorization([.alert,.badge,.sound])`; `.provisional` = quiet (Notification Center only, no prompt). Then `registerForRemoteNotifications`.

### 3.3 Android setup

- `google-services.json` into `app/` + google-services Gradle plugin.
- **Android 13+ (API 33): request `POST_NOTIFICATIONS` runtime permission** — OFF by default on fresh installs (auto-pre-granted only on OS upgrade). No request = zero delivery.

### 3.4 Web/VAPID (optional)

- iOS Web Push: **iOS 16.4+, Home-Screen-installed PWA ONLY** (never Safari tab); needs manifest `display:standalone`, service worker (Push+Notifications+SW APIs), self-gen **VAPID** keypair, permission from a **direct user tap**.

### 3.5 Rules & mechanics

- **Guideline 4.5.4**: push must NOT be required to run app; marketing push needs in-app **opt-in consent UI + in-app opt-out**.
- **Rich media (iOS)**: `"mutable-content":1` + **Notification Service Extension** downloads/attaches media within ~30s; **4 KB** APNs payload cap.
- **Deep links**: route/URL in `data` payload; handle tap in response delegate (iOS) / intent extras (Android); wire Universal Links / App Links for https.
- **Tokens rotate** (reinstall/restore/update) — fetch every launch, sync to backend, prune stale.
- Local (device-scheduled) notifications need no server; exact alarms on Android 12+/13+ need `SCHEDULE_EXACT_ALARM`/`USE_EXACT_ALARM`.

### 3.6 Delivery-tool matrix

| Tool | OSS/self-host | Pricing |
|---|---|---|
| FCM / APNs / Expo Push | proprietary-free | **Free** (Expo Push = RN/Expo only) |
| **Novu** | **OSS (MIT), self-host** | Self-host free; cloud free 10k runs/mo + 20 workflows; Pro $30/mo (30k, $1.20/1k overage); Team $250/mo |
| ntfy / Gotify | **OSS self-host** | Free (ops/homelab-grade, NOT consumer marketing push) |
| OneSignal | freemium | Free **unlimited mobile push** (web ~10k); Growth $19/mo + $0.012/mobile-MAU + $0.004/web-sub |
| Knock | freemium | Free 10k msgs/mo; Starter $250/mo (50k, $0.005/msg) |
| Courier | freemium | Free 10k sends/mo; PAYG **$0.005/send** (old $20/mo plan gone) |
| Pusher Beams | freemium | Free sandbox 1,000 subs; paid from $29/mo (Startup 10k) |
| WonderPush | freemium | €1/mo + €1 per 1,000 subs, unlimited sends |
| Airship | paid | Enterprise only (~$30k+/yr est.) |
| Amazon SNS | paid | 1M mobile push/mo free (permanent) then $0.50/M; raw fan-out, no UI |

**Metering differs**: OneSignal = mobile MAU, Novu = workflow runs, Knock/Courier = messages. Model real volume.

---

## 4 · In-App Purchases

### 4.1 When IAP is mandatory / forbidden

| Content | iOS rule |
|---|---|
| In-app digital goods, subs, currency, unlocks | **MUST use StoreKit IAP (3.1.1)** — no keys/QR/crypto/external unlock |
| Physical goods / real-world services | **MUST NOT use IAP (3.1.3/3.1.5)** — use other payment |
| Reader apps / multiplatform | carve-outs (3.1.3(a)/(b)) |

### 4.2 Commissions (mid-2026)

**Apple**: 30% standard; **15% Small Business Program** (≤$1M prior-yr proceeds, new devs auto-qualify, cliff to 30% on crossing $1M, effective 15 days after fiscal-month-end of approval). Non-SBP auto-renew subs: 30% first paid year → **15% after 12 continuous paid months** (free-trial excluded). China storefront post-yr-1 sub = **12% since 2026-03-15**.

**Google** — MAJOR SHIFT (US/UK/EEA live **2026-06-30**, unbundled):
| | US/UK/EEA new model | Rest-of-world (legacy) |
|---|---|---|
| First $1M + all subs | 10% service + **5% billing** = ~15% effective | 15% |
| >$1M non-recurring | 20% (new install) / 25% (existing) + billing | 30% |
| Alt-billing / external web link | **0% billing fee** (service fee only) | user-choice = −4pp |
| Program (Apps Experience/Games Level Up) | lowers >$1M non-recurring to 15% | — |

### 4.3 US external-purchase links & EU DMA

- **US external links: Apple charges $0 commission as of July 2026.** Guidelines 3.1.1(a) (updated 2025-05-01) let US-storefront apps link out **without the entitlement**. Kagan denied Apple's stay **2026-05-06**; **SCOTUS granted cert 2026-06-30** (Apple v Epic No. 25-1311, argued ~Oct 2026 term); district court hasn't set a "reasonable" fee. **Do NOT hard-code a US external-link fee** — architect a toggle. Non-US storefronts still require entitlements + commission. `⚠︎verify-at-submit`
- **EU DMA (June-2025 terms)**: **5% Core Technology Commission** on digital-goods revenue (replaced €0.50 CTF, sunset 2026-01-01) + store fee Tier 1 5% / Tier 2 13% (10% SBP) + 2% initial-acquisition fee. Dispute open. `⚠︎verify-at-submit`

### 4.4 Product setup

- **Apple** (App Store Connect → Monetization): consumable / non-consumable / **auto-renewable sub** / non-renewing sub. Subs live in a **subscription group** (max 100/group, one active per group, levels = up/down/crossgrade). **Intro offer** = free trial / pay-up-front / pay-as-you-go, **one per user per group**.
- **Google** (Play Console): in-app products + subscriptions = **base plans + offers**. Enroll reduced-fee tier + Account Group.

### 4.5 Client tech & version floors

- iOS: **StoreKit 2 (iOS 15+)** — set deployment target ≥15 or plugins may silently fall back to StoreKit 1.
- Android: **Play Billing Library v7+ floor (Aug 31 2025)**; **v8 GA June 30 2025**, **v9.0.0 shipped May 19 2026**. **v8 becomes mandatory min for new apps/updates Aug 31 2026** (ext to Nov 1 2026; same window likely bumps target to **API 36 / Android 16**). Alt/user-choice needs **9.1+**. Ship on v8 or v9. `⚠︎verify`

### 4.6 Server-side validation (required)

- **Apple**: `verifyReceipt` + ASSN V1 **DEPRECATED** → **App Store Server API** (signed-JWT auth → Apple-signed JWS) + **App Store Server Notifications V2**. `beginRefundRequest(in:)` shows Apple's refund sheet (Apple decides). OSS **App Store Server Library** (Swift/Java/Python/Node) verifies JWS self-hosted.
- **Google**: **Play Developer API** (`purchases.subscriptionsv2.get`) via service account + **RTDN over Pub/Sub**. **Acknowledge within 3 days or auto-refund.**
- **Restore-purchases** is REQUIRED (3.1.1): StoreKit 2 auto-sync (`Transaction.currentEntitlements`) + a **visible Restore button** (`AppStore.sync()`) — missing button is a common rejection.

### 4.7 Testing

- iOS: local **`.storekit` config file** (no network) → then **Sandbox Testers** (ASC → Users & Access) + Developer Mode on device. Sandbox subs renew on accelerated timers (~5 min/month), auto-cancel after ~6 renewals.
- Android: license testers + closed testing track.

### 4.8 Cross-platform SDK matrix

| SDK | OSS/self-host | Pricing |
|---|---|---|
| StoreKit 2 / Play Billing | native, free | Free (15–30% store cut) |
| App Store Server Library | **OSS self-host** | Free (DIY entitlement logic) |
| **@capgo/native-purchases** (Capacitor) | **OSS (MIT)** | Free; DIY receipt validation |
| **react-native-iap** | **OSS** | Free; DIY validation |
| **in_app_purchase** (Flutter) | **OSS** | Free; DIY server verify |
| cordova-plugin-purchase (Iaptic) | **OSS** | Free; Iaptic validator optional paid |
| RevenueCat | freemium | Free ≤$2,500 MTR, then **1% of MTR** (gross, pre-cut) |
| Adapty | freemium | Free ≤**$5,000 MTR**, then 1% ($99/mo floor removed 2026-02-16) |
| Qonversion | freemium | Free ≤$10,000 MTR, then 0.6% (Starter)/0.8% (Growth), **no monthly minimum** |
| Superwall | freemium | Free ≤$10,000 **MAR**, then 1% of attributed revenue |
| Glassfy | **DEFUNCT ~end-2024** | Do NOT use; migrate to RevenueCat |

**MTR ≠ MAR ≠ MRR** — SDK fees are on GROSS, on top of the 15–30% store cut.

### 4.9 Offers, win-back & dunning (retention)

**Acquisition/retention offers:**

| | Apple | Google |
|---|---|---|
| Intro offer | free trial / pay-up-front / pay-as-you-go — 1 per user per group | Base-plan offers (new-customer eligibility) |
| Offer/promo codes | **Offer Codes** (custom/one-time promo & intro pricing; redeem in-app via `presentCodeRedemptionSheet` or App Store URL) + **promo codes** (≤100/app/version, free access, reviewers/press) | **Promo codes** (≤500/quarter, free/discounted, one-time) |
| Win-back | **Win-back offers** for lapsed subs (surfaced on App Store + in-app since 2024) | **Offers with upgrade/win-back eligibility** for lapsed/existing users (base plans + offers) |

**Dunning (failed-renewal recovery — biggest involuntary-churn source):**
- **Apple**: enable **Billing Grace Period** (6d weekly / 16d monthly+, keeps access) + automatic **billing retry (≤60 days)**. Failure/recovery states via **ASSN V2**: `DID_FAIL_TO_RENEW` (subtype `GRACE_PERIOD`), `GRACE_PERIOD_EXPIRED`, `EXPIRED`, `DID_RENEW`/recover.
- **Google**: configure **grace period** (billing retry, keeps access) → **Account Hold** (≤30 days, access revoked, sub still recoverable) → canceled/expired. States via **RTDN**: `SUBSCRIPTION_IN_GRACE_PERIOD`, `SUBSCRIPTION_ON_HOLD`, `SUBSCRIPTION_RECOVERED`, `SUBSCRIPTION_CANCELED`/`EXPIRED`.
- Handle grace/hold server-side (don't cut access on first failed charge); prompt in-app to fix payment.

---

## 5 · Supporting-Services Matrices (OSS-vs-paid)

### 5.1 Realtime / video / voice / chat

| Tool | OSS/self-host | Pricing |
|---|---|---|
| **LiveKit** (best OSS for AI voice/video agents) | **OSS Apache-2.0** | Self-host free (Redis+TURN+egress). Cloud: Build free 5k WebRTC+1k agent min **HARD CAP**; Ship $50/mo (150k); Scale $500/mo (1.5M) |
| **Jitsi** | **OSS Apache-2.0** | Self-host free; JaaS free ≤25 MAU then $0.35/MAU |
| mediasoup / ion-sfu / Socket.IO | **OSS (ISC/MIT)** | Free; **no turnkey mobile SDK** (build signaling+WebRTC yourself) |
| Janus | **OSS GPLv3** | Free; copyleft (Meetecho commercial license) |
| Daily.co | freemium | 10k free participant-min/mo then **$0.004/min PAYG (no cap)** |
| 100ms | freemium | 10k conferencing + 10k streaming free min; $0.004/participant-min; recording $0.016/min (no free) |
| Agora | freemium | 10k free min/mo; HD video ~$3.99/1k min; **accounts after 2025-08-29 lose free 10k on paid package** |
| Vonage (ex-OpenTok) | freemium | 100k free trial min; $0.0041/participant-min (≤25 publishers) |
| Twilio Video | paid | Usage-based; **Dec-2026 EOL was REVERSED (Oct 2024)**, still supported |
| Stream Chat | freemium | Build free 1,000 MAU/100 conn; Maker $100 credit; Start $399/mo (annual, 10k MAU); Elevate $599/mo. **MAU=anyone who connected** |
| Sendbird | freemium | 30-day trial→free Dev 100 MAU; Starter ~$399/mo |
| **Supabase Realtime** | **OSS self-host** | Free 200 conn + 2M msg/mo; Pro $25/mo (500 conn) |
| Ably | freemium | Free 6M msg + 200 conn; Standard $29/mo |
| Pusher Channels | freemium | Free sandbox **100 conn** + 200k msg/day; Startup $49/mo; Pro $99/mo |
| PubNub | freemium | Free 200 MAU + ~1M tx/mo |

- **HARD CAP (service stops)**: LiveKit Build, Pusher. **PAYG (bill grows)**: Daily/100ms/Agora. Know which = launch-day billing risk.
- Don't run WebRTC in WKWebView (camera/mic/background unreliable) — use native SDK. iOS incoming calls need **PushKit + CallKit**.

### 5.2 Media/CDN + object storage

| Tool | OSS/self-host | Pricing |
|---|---|---|
| **ImageKit** | managed | Free **20 GB BW (HARD cap)** + 3 GB storage, unlimited transforms; Lite $9/mo; Pro $89/mo |
| Cloudinary | managed | Free 25 credits/mo (**suspends on overage**); Plus $99/mo; Advanced $249/mo |
| Cloudflare Images | managed | $5/100k stored + $1/100k delivered; ~5k free transforms; **no egress** |
| Cloudflare Stream | managed | $5/1k min stored + $1/1k min delivered; encoding+player free |
| Bunny (Optimizer/Stream/Storage) | managed | Optimizer $9.50/mo/site; Stream $0.01/GB store + $0.005/GB deliver; Storage $0.01/GB; $1/mo min |
| Gumlet | freemium | Free 30 GB BW unlimited transforms; paid ~$25–32/mo |
| imgix | paid | Prepaid **expiring** credit bundles |
| **imgproxy / Thumbor** | **OSS (MIT) self-host** | Free (libvips) in front of R2/S3; **require signed URLs** (SSRF/DoS) |
| **Cloudflare R2** (best zero-egress origin) | managed | Free 10 GB; $0.015/GB-mo; **$0 egress** |
| **Backblaze B2** | managed | **$6.95/TB-mo**; free egress via CDN partners (Cloudflare/Bunny/Fastly) |
| AWS S3 | managed | $0.023/GB-mo; **egress $0.09/GB** (front with CDN) |
| Wasabi | managed | **$7.99/TB-mo** (raised 2026-07-01); 1TB min, 90-day min; cold/backup only |
| DO Spaces | managed | $5/mo = 250 GiB + 1 TB egress bundled + CDN |
| **MinIO** | OSS (AGPLv3) | Server code still runs (existing deploys OK); **Community edition archived read-only ~2026-04-25, UI stripped, no new binaries/patches `⚠︎verify`** — for NEW self-hosts prefer **Garage** (AGPLv3) / **SeaweedFS** (Apache-2.0) |

- Split ORIGIN (R2/B2) from OPTIMIZE+CDN layer; never serve originals; deliver device-sized AVIF/WebP via `f_auto`; lock origin to CDN; long `Cache-Control` immutable on hashed URLs. Capacitor needs **absolute CDN URLs**.
- **Binary size caps force post-install media delivery**: Apple **4 GB** app max + **~200 MB cellular OTA-download** limit (Wi-Fi above); Android base+config-APK **200 MB** download cap → **Play Asset Delivery / dynamic feature modules** above that (AAB total to 4 GB via asset packs). **Oversized binaries are rejected/undeliverable at upload** — ship heavy media from the CDN post-install, don't bundle in the binary.

### 5.3 Auth + BaaS

| Tool | OSS/self-host | Pricing |
|---|---|---|
| Firebase Auth | managed | Spark free 50k MAU + **3k DAU cap**; Blaze from $0.0055/MAU; **Phone/SMS never free** |
| **Supabase** (Auth=GoTrue) | **OSS self-host** | Free 50k MAU (**pauses after 7d idle**); Pro $25/mo (100k); Team $599/mo |
| Clerk | managed | Free 50k **MRU** (retained users); Pro $25/mo + $0.02/MRU; Business $300/mo |
| Auth0 (Okta) | managed | Free 25k MAU; Essentials $35/mo; Professional $240/mo (~$0.07/MAU overage; ~50k≈$3,500/mo) |
| **Keycloak** | **OSS Apache-2.0** | Free, no MAU caps (Java+DB ops) |
| **Ory** (Kratos/Hydra) | **OSS Apache-2.0** | Self-host free; Network from $29/mo |
| **Appwrite** | **OSS self-host** | Free 75k MAU; Pro **$25/mo PER PROJECT** (200k MAU); self-host free |
| **Nhost** | **OSS self-host** | Free (pauses); Pro $25/mo; Team $599/mo |
| **PocketBase** | **OSS (MIT) self-host** | Free single Go binary; single-node SQLite (no cloud) |
| Stack Auth / **Hexclave** | **OSS** (client MIT / server AGPLv3) | Managed free tier + self-host |

- **Free BaaS projects that pause after 7d idle (Supabase/Nhost) must be upgraded to paid BEFORE production.**
- **Sign in with Apple (4.8)** required only if a THIRD-PARTY/social login is the primary account method; email/password-only exempt. Android: legacy `GoogleSignInClient` deprecated → **Credential Manager**.

### 5.4 Analytics

| Tool | OSS/self-host | Pricing |
|---|---|---|
| **PostHog** (analytics+replay+flags+crash) | **OSS self-host** | Free 1M events + 5k replays + 1M flags + 100k exceptions/mo; then $0.00005/event; no per-seat |
| GA4 / Firebase Analytics | managed | Free unlimited (weak funnels/cohorts) |
| Amplitude | managed | Free 10k MTU/2M events; Plus $49/mo |
| Mixpanel | managed | Free 1M events + 10k replays; Growth $0.28/1k events |
| Segment (CDP) | managed | Free 1k MTU/2 dest; Team $120/mo (10k MTU, **2 sources**, unlimited destinations) |

- Metering: Amplitude/Segment = MTU (users); Mixpanel/PostHog = events. Instrument through a thin wrapper (or Segment) to swap tools.

### 5.5 Crash / monitoring

| Tool | OSS/self-host | Pricing |
|---|---|---|
| **Sentry** (best cross-platform) | **OSS self-host (FSL)** | Free 5k errors/1 user/30d; Team $26/mo (50k, unlimited users); Business $80/mo; Seer AI ~$40/mo/contributor |
| Firebase Crashlytics | managed | Free unlimited (native-first; weaker for web/JS) |
| Bugsnag / SmartBear Insight Hub | managed | Free 7.5k events + 1M spans; Select ~$20/mo; Preferred ~$33/mo |
| Instabug / **Luciq** | paid | Sales-led on DAU+seats; legacy ~$249/mo+ |

Upload source maps / dSYMs so stack traces symbolicate.

---

## 6 · Pre-Submission Checklist

- [ ] Build with **Xcode 26 + iOS 26 SDK** (mandatory to upload since 2026-04-28; SDK-only, keep low deployment target). Liquid Glass on by default unless opted out. `⚠︎verify-at-submit`
- [ ] Android targets **API 35** (Android 15) + 64-bit (arm64-v8a) + `.aab` + Play App Signing — **plan API 36 / Android 16 for the ~Aug 31 2026 bump `⚠︎verify-at-submit`**.
- [ ] Binary under caps (Apple 4 GB / ~200 MB cellular; Android 200 MB base download) — heavy media loaded from CDN post-install.
- [ ] Info.plist **purpose string for EVERY permission** (specific, non-generic — missing = runtime crash + 5.1.1 reject).
- [ ] `PrivacyInfo.xcprivacy` = collected data types + **Required-Reason-API codes**; every 3rd-party SDK ships a **signed** manifest (audit via Xcode Privacy Report).
- [ ] `ITSAppUsesNonExemptEncryption = NO` (standard TLS) — unblocks TestFlight.
- [ ] App Privacy nutrition label + Google Data safety form + Privacy Manifest **all agree**.
- [ ] Privacy-policy URL live; in-app account deletion + Google web deletion URL working.
- [ ] Age-rating questionnaires answered (Apple + IARC).
- [ ] IAP visible + functional to reviewer; **Restore button** present.
- [ ] Sign in with Apple wired (if social login) incl. working hide-my-email relay.
- [ ] Demo account (2FA OFF, permanent, backend live) + reviewer notes in App Review Information.
- [ ] Real native features present (push/offline/native nav) → clears 4.2.
- [ ] TestFlight internal build clean of crashes; Google pre-launch report clean.

---

## 7 · Apple Review Gauntlet

### 7.1 Top rejection reasons

| # | Cause | Fix |
|---|---|---|
| 4.2 | Thin web wrapper | Add push/offline/native nav/device APIs |
| 2.1 | Broken demo account / down backend / expired creds / 2FA | Permanent monitored reviewer account, 2FA off |
| 5.1.1 | Vague/missing purpose string (crashes) | Specific string per permission |
| upload | Missing/invalid Privacy Manifest / unsigned SDK | Add signed `.xcprivacy` for app + every SDK |
| 3.1.1 | Digital goods bypass IAP / physical goods via IAP | Route correctly |
| 4.3 | Duplicate/multi-bundle white-labels | One app + account/IAP switching |
| 4.8 | Social-login-only without SiwA/own login | Add SiwA |

### 7.2 Requirements

- **Privacy nutrition labels** (mandatory since 2020-12-08) — ASC questionnaire, must match SDK+manifest.
- **Privacy Manifest + Required-Reason APIs** (enforced 2024-05-01) — declare reason codes for file timestamp, boot time, disk space, UserDefaults, etc.
- **ATT**: any IDFA read / cross-app tracking / data-broker sharing → `AppTrackingTransparency` prompt + `NSUserTrackingUsageDescription`; no opt-in → IDFA all-zeros. Answer tracking honestly in label even with zero tracking SDKs.
- **Purpose strings**: camera, photos, location, mic, contacts, tracking — specific.
- **Sign in with Apple (4.8)**: equivalent option limiting to name+email, hide-email, no ad-tracking without consent. Native `ASAuthorization` button, ≥ prominence of other social buttons.
- **Demo account (2.1)** mandatory for login-gated features; built-in demo mode allowed only with prior Apple approval.
- **Required toolchain**: Xcode 26 + iOS 26 SDK (2026-04-28). `⚠︎verify-at-submit`
- **Review time**: 90% <24h (new apps trend 1–5 days). **Expedited review** = critical live bug / tied event only; rationed.
- **TestFlight**: 100 internal (instant, no review) + 10,000 external (Beta App Review on first build of a version; ≤6 submissions/24h); **builds expire 90 days**.

---

## 8 · Google Play Gauntlet

### 8.1 Closed-test-before-production (new personal accounts)

- **≥12 testers, opted-in 14 CONTINUOUS days**, CLOSED track only, before "Apply for production." Org + pre-2023-11-13 personal exempt. Production-access review ≤7 days. Recruit 20–25 (engagement checked). No Apple equivalent — start early.

### 8.2 Other gates

| Item | Requirement |
|---|---|
| Target API | **35 (Android 15)** for new/updates since 2025-08-31; existing ≥34; Wear/Auto/TV ≥34. **Next bump: API 36 / Android 16 for new apps+updates ~Aug 31 2026 `⚠︎verify`** |
| Bundle | **.aab** only; 64-bit (arm64-v8a) if native code |
| **Data safety form** | Mandatory (even zero-data) + privacy policy. **Android ID reclassified 2025-04-10 as NO LONGER a persistent ID** — declare **Advertising ID** (+ other device IDs) under "Device or other IDs" |
| Permissions declarations | **FGS types** (App content page + matching permission); **background location** = declaration + core-purpose justification + **demo video**; sensitive (SMS/Call Log) = declaration; **over-permissioning is a top rejection** |
| App content | Privacy policy, ads declaration, app access (reviewer login), IARC content rating, target audience/Families, plus news/finance/health/gov declarations |
| **Pre-launch report** | Auto-runs on test-track upload (Firebase Test Lab, real devices), free, ~1h — flags crashes/security/accessibility |
| Staged rollout | Pick %, **raise MANUALLY** (no auto-increase), halt anytime; **updates only** (first publish = 100%) |
| Managed publishing | Holds approved changes until you manually publish |
| Per-release review | Hours to ~7 days (new accounts slower) |

- Google blocked **1.75M+ submissions** in 2025. Top causes: crashes, over-permissioning, missing/broken privacy policy, inaccurate data-safety, misleading metadata.

---

## 9 · Store Listing

| Asset | Apple | Google |
|---|---|---|
| App icon | 1024×1024 PNG (no alpha) | 512×512 PNG |
| Screenshots | Per required device sizes (6.9"/6.5" iPhone, iPad if supported); localized | Phone + 7"/10" tablet; feature graphic 1024×500 |
| Privacy | Nutrition labels + privacy-policy URL (both stores) | Data safety form + privacy policy link |
| Age rating | 4+/9+/13+/16+/18+ questionnaire | IARC (ESRB/PEGI/USK/…) |
| Metadata | Name, subtitle, keywords, description, promo text, support URL | Title, short + full description, category |
| IAP metadata | Products visible + reviewer-testable | Same |

- No misleading metadata/screenshots (rejection on both). Complete BEFORE submitting.

---

## 10 · Submission & Rollout

Submit: `eas submit` / `fastlane pilot`+`supply` / Transporter (.ipa) + Play Console (.aab). Turn ON phased/staged rollout; set billing alerts on any HARD-CAP or PAYG service before the launch spike.

- **Apple phased release**: auto **1/2/5/10/20/50/100%** over 7 days (updates only; auto-update users only; pausable ≤30 days).
- **Google staged rollout**: pick %, **raise MANUALLY**, halt anytime (updates only; first publish = 100%).
- → Full rollout mechanics comparison table in **Doc1**.

---

## 11 · Post-Launch — OTA & Monitoring

### 11.1 OTA legality

- **Apple DPLA §3.3.2** (NOT Guideline 2.5.2): interpreted code run by WebKit/JavaScriptCore is OK **only if it doesn't change the app's advertised primary purpose**. 2.5.2 = the prohibition on native-executable download.
- **Google Device & Network Abuse**: no executable code (dex/JAR/.so) download; **JS in a webview is allowed**.
- **Ship JS/HTML/CSS/asset diffs ONLY.** Native/plugin/Capacitor-version/primary-purpose changes → store build.

### 11.2 OTA tools

- **Capgo** (Capacitor, recommended): OSS self-host (plugin LGPL-3.0 / backend AGPL-3.0) or cloud from **~$12/mo** (annual-effective), 14-day trial, **no perpetual free**.
- **EAS Update** (RN/Expo): free 1k MAU; Starter $19/mo (3k); Production $199/mo (50k). MAU = downloaded ≥1 update.
- **DEAD/AVOID**: App Center/CodePush (**retired 2025-03-31**), Ionic Appflow (**closed to new customers, ends 2027-12-31**), code-push-server (**repo archived 2025-05-20, unmaintained**).
- → Full OTA-tools pricing table in **Doc1**.

### 11.3 Distribution / beta channels

TestFlight internal ≤100 (instant) / external ≤10,000 (Beta App Review, expire 90d); Play internal ≤100 / closed 2,000×50 lists (**satisfies the 12-tester/14-day gate**) / open unlimited; Firebase App Distribution (free, 500/project, iOS needs UDIDs); Play Internal App Sharing (instant link, ≤100 users, 60-day expiry).
→ Full channel-limits table in **Doc1**.

### 11.4 Monitoring

- Crash: Sentry / Crashlytics (upload dSYMs/source maps). Analytics: PostHog/Amplitude/GA4. Track signup→activation→key-action→purchase→retention.
- Emergency: Play **halt rollout** / Apple **pause phased release** + OTA channel rollback.
- Move off any auth/BaaS/media free tier that HARD-CAPS or pauses before spikes.

---

## MASTER LAUNCH CHECKLIST (one page)

**Accounts (start day 1)**
- [ ] Apple Developer Program $99/yr enrolled + 2FA
- [ ] Google Play Console $25 paid + identity verified
- [ ] Org: D-U-N-S requested (~5–7d Apple path / ≤30d D&B) + org domain email + live website
- [ ] Bundle ID / applicationId chosen (reverse-DNS, PERMANENT)

**Scaffold**
- [ ] Framework chosen (Capacitor = web-first default) + `.ipa`/`.aab` building
- [ ] Real native features added (clears Apple 4.2)
- [ ] Mac/CI + Fastlane + `.p8` ASC key + Play App Signing set up (or **Xcode Cloud**: 25 free compute-hrs/mo)
- [ ] Android target API 35 + 64-bit + `.aab` (plan **API 36 / ~Aug 2026**)

**Push**
- [ ] Firebase project; FCM HTTP v1 + service-account JSON
- [ ] APNs `.p8` (Key ID+Team ID) uploaded; Push capability + Background Modes
- [ ] iOS `requestAuthorization`; Android `POST_NOTIFICATIONS`; token sync on launch
- [ ] 4.5.4 opt-in + opt-out for marketing push

**IAP**
- [ ] Products/subs created (ASC groups + Play base plans/offers)
- [ ] StoreKit 2 (iOS 15+) + Play Billing v8/v9; RevenueCat or native plugin
- [ ] Server validation (ASC Server API + ASSN V2 / Play Developer API + RTDN; ack ≤3 days)
- [ ] Restore button; SBP + Google reduced-fee tier enrolled
- [ ] Store dunning wired: Apple Billing Grace Period + retry (ASSN V2 states); Google grace period + Account Hold (RTDN states)
- [ ] Offer/promo codes + win-back offers set (if using)
- [ ] Tested via `.storekit` + Sandbox / license testers

**Legal/compliance**
- [ ] Privacy-policy URL live; in-app account deletion + Google web deletion URL
- [ ] Age-rating questionnaires (Apple + IARC) answered
- [ ] Nutrition label + Data safety form + Privacy Manifest all agree
- [ ] Purpose strings + ATT (if IDFA) + `ITSAppUsesNonExemptEncryption=NO`
- [ ] Sign in with Apple (if social login); Android Credential Manager
- [ ] Paid Apps Agreement + tax/banking (if selling)

**Submit**
- [ ] Demo account (2FA off, backend live) + reviewer notes
- [ ] Binary under size caps (heavy media from CDN post-install)
- [ ] TestFlight internal clean; Google pre-launch report clean
- [ ] **Google: 12 testers × 14 continuous days closed test** (new personal accts) → apply for production
- [ ] Submit; Apple phased + Google staged rollout ON; billing alerts set

**Post-launch**
- [ ] OTA wired (Capgo/EAS), JS-only diffs
- [ ] Crash + analytics live; rollback plan (halt/pause) ready
- [ ] Free-tier hard-caps upgraded before spike

**⚠︎verify-at-submit before shipping:** US external-link fee (SCOTUS pending), Google new fee model, Play Billing v8 cutoff (2026-08 window), **Android API 36 / Android 16 target bump (~Aug 31 2026)**, Xcode/iOS SDK min, Google Sept-2026 dev verification, EU DMA fee stack.