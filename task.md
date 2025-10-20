# Focus Guardian Android Studio Kotlin Task List

1. **Initialize Project** — Create a new Android Studio project (Minimum SDK 26+) in Kotlin, configure buildSrc or Gradle catalog, add required dependencies (Jetpack, Room, WorkManager, Security crypto) and set up multi-module structure (`app`, `core`, `data`).
2. **Define Data Models & Storage** — Implement Room database schemas for `AppRule`, `GlobalPolicy`, `Profile`, `AuditEvent`, and `Settings`; configure encrypted storage via SQLCipher or EncryptedSharedPreferences; establish repositories with Kotlin coroutines/Flow APIs.
3. **Scheduler Engine Implementation** — Build Kotlin scheduling logic using `ZonedDateTime` to evaluate per-app and global rules, manage overrides, and expose unit-tested decision APIs meeting latency requirements.
4. **Usage & Accessibility Monitoring** — Develop services leveraging UsageStats and Accessibility to detect foreground app changes, integrate with the scheduler, and prepare overlay triggers within 150 ms.
5. **Blocking Overlay & Enforcement** — Create high-contrast overlay UI and auto-close logic using Accessibility actions and `SYSTEM_ALERT_WINDOW`; ensure Accessibility compliance and retry strategy for stubborn apps.
6. **Install Guard Flow** — Implement Accessibility handlers for Play Store/Package Installer UIs to interrupt restricted installations, showing rationale screens and logging attempts.
7. **Override & Cooldown Mechanisms** — Build PIN-protected override workflow with biometric option, duration selection, cooldown enforcement, notifications, and audit logging.
8. **Profiles & Rule Management UI** — Design Compose screens for profile selection, rule editing, schedule builder, and app picker; integrate search, categories, and activation delays.
9. **Onboarding & Permission Orchestration** — Craft onboarding wizard guiding users through Usage Access, Accessibility, overlay, Device Admin (optional), and app selection; include health status dashboard.
10. **Anti-Tamper & Watchdog Services** — Implement foreground service monitoring permission revocations, time changes, reboot events, and uninstall attempts with remediation flows.
11. **Managed Mode (Device Owner) Support** — Add optional provisioning guide, DevicePolicyManager integration for suspending apps, and policy toggles gated behind advanced settings.
12. **Localization & UX Polish** — Provide English and German strings, accessibility labels, randomized keypad option, and compliance disclosures for Accessibility usage.
13. **Telemetry & Audit Trail** — Finalize encrypted local logging, export/import functionality, and summary analytics surfaced in UI.
14. **Testing & QA Automation** — Set up unit tests (scheduler edge cases), instrumentation tests for blocking latency and overlay behavior, and accessibility UI test coverage.
15. **Release Hardening & Documentation** — Prepare production build configs, privacy notice, managed mode setup guide, and Play Store-compliant disclosures.
