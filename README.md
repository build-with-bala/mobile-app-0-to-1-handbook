# 0→1 Mobile App Handbook — iOS + Android

Two dense, no-fluff handbooks for taking a **web app to a shipped native iOS + Android app** — including push notifications and in-app purchases — the whole path from zero to live in both stores.

Every price, commission rate, tester/day requirement, SDK floor, and store deadline was checked against official Apple, Google, and vendor docs and adversarially re-verified. **Facts current as of July 2026.** Anything volatile is tagged `⚠︎verify-at-submit` in the text — store policies and pricing move fast, so re-check those before you ship.

## Contents

### [01 · Build & Ship Options](./01-BUILD-OPTIONS.md)
Choosing how to build and ship.
- Framework decision matrix — Capacitor / React Native / Flutter / Kotlin Multiplatform / .NET MAUI / PWA / TWA
- The "you need a Mac for iOS" problem + build toolchain
- CI/CD services with current pricing & free tiers (GitHub Actions, Xcode Cloud, EAS, Codemagic, Bitrise, CircleCI, Fastlane)
- Code signing (iOS `.p8` / Android Play App Signing)
- OTA updates — what's legal, and which tools are alive vs dead
- Test & distribution channels (TestFlight, Play tracks, Firebase App Distribution)
- Build-commands cheat-sheet + a recommended web-first stack

### [02 · Complete 0→1 App Build (Notifications + IAP)](./02-APP-0-TO-1.md)
The full ordered, checkable journey.
- Accounts & prerequisites (fees, D-U-N-S, identity verification, legal/compliance)
- The Google closed-testing gate (12 testers × 14 continuous days) for new personal accounts
- Project scaffold & native capabilities
- **Push notifications** — FCM + APNs setup + delivery-tool matrix
- **In-app purchases** — StoreKit 2 / Play Billing, commissions, the US external-link + EU DMA shifts, server validation, dunning
- Supporting-services matrices (realtime/video, media/CDN + storage, auth, analytics, crash)
- Apple review gauntlet + Google Play gauntlet (top rejection reasons)
- Store listing, submission & phased rollout, post-launch
- A one-page **master launch checklist**

Every tool category lists **both open-source / self-hostable and paid / managed options** with current pricing.

## Who it's for

Engineers and founders shipping their first (or next) cross-platform app who want the real requirements and current numbers in one place — not a blog post from three policy cycles ago.

## License

Released under the [MIT License](./LICENSE). Use it, fork it, adapt it.
