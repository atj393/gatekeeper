# Focus Guardian MVP Technical Specification

## 1. Product Summary

Focus Guardian is a self-control Android application that blocks launching selected apps and restricts installing or uninstalling those apps according to configurable schedules (for example, "weekends only" or "Monday–Friday 06:00–21:00 blocked"). The MVP operates completely on-device with strong anti-tamper features so that users cannot easily bypass their own rules.

## 2. Supported Operating Modes (MVP)

### 2.1 Standard Mode (No Device Owner)

- Works on personal phones without special enrollment.
- Uses `AccessibilityService`, overlay, auto-close (navigate Home/Back), and Usage Access for detection.
- App installation restriction is best-effort: intercepts Play Store/Package Installer UI via accessibility and blocks flows; cannot fully prevent installs at the system level.

### 2.2 Managed Mode (Device Owner) — Optional Advanced Setup

- Requires Device Owner provisioning (via ADB) to enforce system-level policies with `DevicePolicyManager`.
- Can block app installs/uninstalls for specific packages or globally with allowlists, suspend apps, or apply user restrictions.
- Intended for power users; MVP must deliver Standard Mode with Managed Mode as an optional stretch goal.

## 3. Primary Use Cases & Requirements

### 3.1 Use Cases

1. **Block Launch** — If a restricted app opens during blocked time, show Focus Guardian screen and return to Home immediately.
2. **Scheduled Access** — Define per-app schedules. Outside allowed windows, the app is blocked.
3. **Installation Guard (Best-Effort)** — Detect attempts to install target apps and interrupt the flow with a blocking screen.
4. **Uninstall Protection** — Require PIN/passphrase to disable services and make uninstall difficult; monitor and restore critical permissions.
5. **Temporary Override** — Allow time-boxed overrides with reason logging and optional cooldown.
6. **Lock Profiles** — Multiple rule profiles (e.g., Workdays, Weekend, Deep Focus) with PIN-protected, delayed activation.
7. **Audit Trail (Local)** — Keep local logs of blocked attempts, overrides, rule changes, and permission losses.

### 3.2 Functional Requirements

- FR-1: Block foreground apps immediately when outside allowed windows.
- FR-2: Per-app rules with mode, schedules, cooldowns.
- FR-3: Global block windows supersede per-app rules.
- FR-4: UI to search/select installed apps into restricted list.
- FR-5: Onboarding wizard to grant Usage Access, Accessibility, overlay, and optional Device Admin.
- FR-6: Install guard detects installer UIs via accessibility and interrupts installs, showing rationale.
- FR-7: Managed Mode uses DPM to apply restrictions and suspend packages.
- FR-8: Overrides require PIN/passphrase with configurable duration/cooldown; log reason.
- FR-9: Lock settings behind PIN.
- FR-10: Time zone & DST aware scheduling; detect backward time changes.
- FR-11: Export/Import encrypted settings and rules locally.
- FR-12: Minimal notifications for status, missing permissions, reminders.

## 4. Non-Functional Requirements (NFR)

- NFR-1: Block within ≤150 ms of app foreground change.
- NFR-2: ≤1% daily battery impact.
- NFR-3: Auto-recovers after reboot and service restarts.
- NFR-4: All data stays on-device.
- NFR-5: Secure PIN storage, encrypted config, tamper detection.
- NFR-6: Accessibility-compliant blocking UI.
- NFR-7: Comply with Google Play policies and disclose Accessibility usage.

## 5. System Architecture

### 5.1 High-Level Components

- **AppRules Repository** — CRUD for per-app/global rules and schedules.
- **Scheduler Engine** — Evaluates rules and precedence.
- **App Monitor (Standard Mode)** — Monitors foreground apps via UsageStats and Accessibility.
- **Block Enforcer** — Overlay controller and auto-closer to exit blocked apps.
- **Install Guard** — Detects installer flows and interrupts.
- **Policy Manager (Managed Mode)** — Interfaces `DevicePolicyManager`.
- **Permissions Orchestrator** — Guides and monitors critical permissions.
- **Anti-Tamper Service** — Foreground service that watches for revocations and time changes.
- **Audit Logger** — Append-only local log.
- **Crypto & Storage** — Android Keystore and encrypted storage (SQLCipher or EncryptedSharedPreferences).
- **UI Layer** — Onboarding, app selection, rules editor, profiles, overrides, logs, settings.

### 5.2 Indicative Data Models

- `AppRule`: package, mode, allowed windows, cooldown, notes.
- `GlobalPolicy`: global blocks, override policy, time-tamper behavior.
- `Profile`: identifier, name, rules, activation delay.
- `AuditEvent`: timestamp, type, package, profile, metadata.
- `Settings`: PIN hash, biometrics flag, managed mode, onboarding status.

## 6. Key Flows

### 6.1 Onboarding

- Welcome screen explaining value and permissions.
- Sequentially request Usage Access, Accessibility, Draw Over Apps, optional Device Admin.
- App picker to select restricted apps.
- Default "Weekdays Focus" profile.
- Start background services and show health status tile.

### 6.2 App Launch Block (Standard Mode)

- Detect foreground package via Accessibility/UsageStats.
- Evaluate scheduler; if blocked, show full-screen overlay with message.
- Trigger global action Home/Back, retry with jitter if necessary.
- Log `BLOCKED_LAUNCH` events.

### 6.3 Install Guard

- Accessibility detects installer UI for target apps.
- Navigate back/Home and present blocking overlay with rationale.
- Log `ATTEMPT_INSTALL`.

### 6.4 Override

- User requests override from app, enters PIN/biometric.
- Specify duration and reason; scheduler marks app allowed for duration.
- Show countdown notification, enforce cooldown afterward, log events.

### 6.5 Managed Mode Provisioning

- Opt-in flow with risks/steps and Device Owner setup guidance.
- Upon success apply DPM restrictions and show Managed Mode badge.
- Use package suspension and install/uninstall blocks as configured.

## 7. Permissions & Capabilities

- **Required:** Usage Access, AccessibilityService, SYSTEM_ALERT_WINDOW, RECEIVE_BOOT_COMPLETED.
- **Optional:** Ignore battery optimizations, Device Admin.
- **Managed Mode:** Device Owner privileges via DPM. Clearly disclose Accessibility usage and purpose.

## 8. Anti-Bypass & Hardening

- PIN-locked settings with optional randomized keypad.
- Activation delays when switching profiles or disabling rules.
- Time tamper handling; treat backward adjustments as blocks or require override.
- Permission watchdog that forces resolution if critical permissions are missing.
- Uninstall friction through in-app disable flow and accessibility interception of uninstall UI.
- Reboot resilience via auto-start and grant verification.

## 9. UX Requirements (MVP)

- High-contrast blocking overlay showing reason, allowed time, and override option.
- Rules editor with search, categories, schedule builder, and profile management.
- Logs timeline with filters and encrypted export.
- Onboarding coachmarks and permission status cards.
- Localization for English and German.

## 10. Testing Strategy

- Unit tests for scheduler edge cases (cross-midnight, DST, overlapping ranges).
- Instrumentation tests for detection latency, overlay behavior, OEM variations.
- Accessibility tree variants for installer detection.
- Soak/battery tests with typical usage.
- Anti-tamper tests (time changes, permission toggles, reboot, uninstall attempts).
- Managed Mode tests for DPM policy application where available.

## 11. Telemetry & Storage

- Local-only analytics: counts of blocks, overrides per period.
- Encrypted Room database for rules, profiles, logs.
- No network in MVP; future toggle for cloud backup.

## 12. Performance Targets

- Blocking overlay visible within 150 ms of foreground change.
- Scheduler evaluation averages ≤2 ms.
- Background memory footprint ≤40 MB RSS.
- Event-driven design; avoid polling.

## 13. Edge Cases & Behaviors

- Support split-screen/PiP by treating restricted focus gains as foreground.
- Handle cross-midnight schedules and DST using `ZonedDateTime`.
- Newly installed matching apps default to blocked until reviewed.
- MVP supports primary user profile only.
- Launcher independence and OEM Accessibility resilience.

## 14. Compliance & Policy

- Explicit user consent and disclosure of Accessibility purpose.
- No data collection beyond local logs; optional encrypted export.
- Managed Mode features hidden unless sideloaded and user-initiated.

## 15. Roadmap (Post-MVP)

- App category templates, session budgets, regex/publisher blocklists.
- Optional website blocking via local VPN.
- Cloud backup with end-to-end encryption and cross-device sync.
- Companion wearables/app tile for override display.
- Behavioral nudges and family/guardian modes (out of scope for MVP).

## 16. Definition of Done (MVP)

- Standard Mode fully functional with schedules, blocking, install guard, overrides, and logging.
- Onboarding and permission health checks available.
- Reboot resilience, anti-tamper UX, encrypted storage, English/German localization.
- Tested on Android 12–15 with major OEM verification.
- Documentation covering user guide, managed mode setup, privacy notice.

## 17. Risks & Mitigations

- OEMs killing services → Foreground service, user education, battery optimization guidance.
- Play policy rejections → Transparent disclosures and optional Device Owner features limited to sideload builds.
- Install blocking limitations → Communicate best-effort approach; offer Managed Mode for robust control.
- Accessibility UI changes → Flexible heuristics and fallback blocking strategies.

