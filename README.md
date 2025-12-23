# KashCal

**A privacy-focused, offline-first calendar app for Android with iCloud sync**

[![Android](https://img.shields.io/badge/Platform-Android%208.0%2B-green.svg)](https://developer.android.com)
[![Version](https://img.shields.io/badge/Version-13.0.0-blue.svg)](https://github.com/KashCal/KashCal.github.io/tree/main/releases)

---

## Overview

KashCal is a native Android calendar application designed for users who want seamless iCloud calendar synchronization without compromising on privacy or user experience. Built with modern Android architecture and a security-first mindset, KashCal offers a reliable, fast, and beautiful calendar experience.

### Why KashCal?

- **True iCloud Integration**: Full CalDAV sync with Apple's iCloud Calendar
- **Offline-First**: Create, edit, and delete events without internet - changes sync automatically when connected
- **Privacy Focused**: No tracking, no analytics, no ads - your calendar data stays yours
- **Security Conscious**: Industry-standard encryption and comprehensive security logging

---

## Features

### Core Calendar Features

| Feature | Description |
|---------|-------------|
| **Month/Agenda Views** | Clean month grid with scrollable agenda for upcoming events |
| **Quick Event Creation** | Bottom sheet interface for fast event entry |
| **Full Event Editor** | Comprehensive editing with all calendar fields |
| **Recurring Events** | Full RFC 5545 recurrence support (daily, weekly, monthly, yearly, custom) |
| **Multiple Calendars** | Support for all your iCloud calendars with color coding |
| **Calendar Filtering** | Show/hide calendars with persistent preferences |
| **Event Reminders** | Customizable alerts with notification support |
| **All-Day Events** | Proper handling of all-day and multi-day events |
| **ICS Subscriptions** | Subscribe to external calendars (holidays, sports schedules, etc.) |

### iCloud Sync

| Feature | Description |
|---------|-------------|
| **CalDAV Protocol** | Native CalDAV implementation for reliable iCloud sync |
| **Incremental Sync** | RFC 6578 sync-collection for instant updates |
| **Conflict Resolution** | Intelligent handling of simultaneous edits |
| **Background Sync** | Automatic sync via Android WorkManager |
| **Pull-to-Refresh** | Manual sync with visual feedback |
| **Sync Status** | Real-time sync progress indicators |

---

## Offline-First Architecture

KashCal is built with a true offline-first design philosophy. This isn't just "caching" - it's a fundamental architectural decision that prioritizes your experience.

### How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│                        USER ACTION                               │
│                    (Create/Edit/Delete)                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      LOCAL DATABASE                              │
│              Immediate save with sync status tracking            │
│                    UI updates instantly                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    BACKGROUND SYNC                               │
│         Changes queued and synced when online                    │
│              Automatic retry with exponential backoff            │
└─────────────────────────────────────────────────────────────────┘
```

### What This Means For You

| Scenario | KashCal Behavior |
|----------|------------------|
| **Create event on airplane** | Event appears immediately, syncs when you land |
| **Edit event in subway** | Changes saved instantly, synced when signal returns |
| **Delete event offline** | Disappears from view immediately, removed from server later |
| **Phone loses connection mid-sync** | No data loss - operation resumes automatically |
| **Server temporarily unavailable** | Full functionality continues, sync retries in background |

### Technical Implementation

- **Optimistic UI Updates**: Changes reflect immediately without waiting for server confirmation
- **Pending Operation Queue**: Failed sync operations are queued and retried automatically
- **Soft Delete Pattern**: Deleted events are marked locally before server confirmation, preventing "ghost events"
- **Network-Aware Sync**: Background sync respects network conditions via WorkManager constraints
- **Conflict Resolution**: Last-write-wins with conflict detection and user notification

---

## Security & Privacy

### Security Architecture

KashCal implements defense-in-depth security to protect your calendar data:

#### Credential Storage: AES-256-GCM Encryption

```
┌─────────────────────────────────────────────────────────────────┐
│                 ENCRYPTED SHARED PREFERENCES                     │
│                                                                  │
│   Encryption: AES-256-GCM (authenticated encryption)            │
│   Key Storage: Android Keystore (hardware-backed when available) │
│   Key Type: AES256_GCM MasterKey                                 │
│   Access: App-private sandbox only                               │
└─────────────────────────────────────────────────────────────────┘
```

Your iCloud credentials are protected using Android's **EncryptedSharedPreferences** with:

| Security Layer | Implementation |
|----------------|----------------|
| **Encryption Algorithm** | AES-256-GCM (NIST approved, authenticated encryption) |
| **Key Management** | Android Keystore with hardware backing on supported devices |
| **Key Derivation** | Per-app MasterKey, non-exportable |
| **Storage Isolation** | Android app sandbox - inaccessible to other apps |
| **At-Rest Protection** | Credentials never stored in plaintext |

#### Network Security

| Protection | Implementation |
|------------|----------------|
| **Transport Encryption** | TLS 1.2+ for all CalDAV communication |
| **Certificate Validation** | System trust store validation of Apple certificates |
| **No HTTP Fallback** | Strict HTTPS enforcement - no cleartext traffic |
| **App-Specific Passwords** | Support for Apple's two-factor authentication |

#### Security Event Logging

Inspired by Proton Mail's security transparency, KashCal maintains a local security event log:

```
┌─────────────────────────────────────────────────────────────────┐
│                    SECURITY EVENT LOG                            │
│                                                                  │
│   Events Tracked:                                                │
│   • Account authentication attempts (success/failure)            │
│   • Sync operations with timestamps                              │
│   • Credential modifications                                     │
│   • Calendar additions and removals                              │
│                                                                  │
│   Storage: Local device only, never transmitted                  │
│   Retention: Rolling buffer of recent events                     │
│   Access: Settings → Security Events                             │
└─────────────────────────────────────────────────────────────────┘
```

This allows you to audit your account activity and detect any suspicious behavior.

### Privacy Principles

| Principle | How We Implement It |
|-----------|---------------------|
| **No Analytics** | Zero tracking code, no Firebase, no Mixpanel, no telemetry |
| **No Ads** | Ad-free experience - your attention isn't our product |
| **No Third-Party SDKs** | Direct iCloud communication only - no intermediary services |
| **Local-First Storage** | All preferences and data stored on your device |
| **Minimal Permissions** | Only essential Android permissions requested |
| **Transparent Data Flow** | Your data goes: Device ↔ Apple iCloud. That's it. |

### Required Permissions Explained

| Permission | Why We Need It |
|------------|----------------|
| `INTERNET` | Sync your events with iCloud |
| `POST_NOTIFICATIONS` | Show event reminders (Android 13+) |
| `RECEIVE_BOOT_COMPLETED` | Restore scheduled reminders after device restart |
| `SCHEDULE_EXACT_ALARM` | Ensure reminders fire at the exact time you set |
| `WAKE_LOCK` | Keep device awake briefly during background sync |

**What we DON'T request:**
- No contacts access
- No location access
- No camera or microphone
- No phone state or call logs
- No SMS or storage access (beyond app sandbox)

---

## User Experience

### Thoughtful Design Decisions

| Feature | Why It Matters |
|---------|----------------|
| **Contextual Notification Permission** | Permission requested only when you add your first reminder, not on app install |
| **Smart Default Reminders** | Different event types get appropriate default alerts (meetings vs all-day events) |
| **Past Event Dimming** | Visual distinction helps you focus on upcoming events |
| **Instant Calendar Switching** | Switch event to a different calendar easily with calendar picker option |
| **Year Overlay Navigation** | Jump to any month with a 12-month grid picker |
| **Pull-to-Refresh with Feedback** | Clear visual indication of sync progress and results |

### Accessibility

- Full Material 3 design system compliance
- Proper content descriptions for screen readers
- Sufficient color contrast ratios
- Touch targets meeting accessibility guidelines

---

## Technical Architecture

### Modern Android Stack

KashCal is built with current Android development best practices:

```
┌─────────────────────────────────────────────────────────────────┐
│                         UI LAYER                                 │
│                                                                  │
│   Jetpack Compose + Material 3                                   │
│   Unidirectional Data Flow (UDF)                                │
│   Immutable UI State for predictable rendering                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      VIEWMODEL LAYER                             │
│                                                                  │
│   StateFlow for reactive UI state                                │
│   Channel for one-shot events (navigation, snackbar)             │
│   Structured concurrency with viewModelScope                     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    REPOSITORY LAYER                              │
│                                                                  │
│   EventRepository: Event CRUD + reminder scheduling              │
│   SyncRepository: Reactive sync state management                 │
│   PreferencesRepository: Type-safe preferences access            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       DATA LAYER                                 │
│                                                                  │
│   Room Database: Events, calendars, sync metadata                │
│   DataStore: User preferences (reactive, coroutine-based)        │
│   EncryptedSharedPreferences: Credentials only                   │
└─────────────────────────────────────────────────────────────────┘
```

### Technology Stack

| Component | Technology | Why This Choice |
|-----------|------------|-----------------|
| **UI** | Jetpack Compose | Declarative UI, less boilerplate, better state management |
| **Design** | Material 3 | Modern, accessible, follows platform conventions |
| **Database** | Room | Type-safe queries, migration support, coroutine integration |
| **Preferences** | DataStore | Reactive updates, no main thread blocking |
| **DI** | Hilt | Compile-time validation, Android lifecycle awareness |
| **HTTP** | OkHttp | Efficient, well-tested, interceptor support |
| **Background** | WorkManager | Battery-efficient, constraint-aware scheduling |
| **Date/Time** | java.time (JSR-310) | Modern API, timezone handling, no external dependencies |
| **Recurrence** | lib-recur | RFC 5545 compliant, battle-tested |


### Standards Compliance

| Standard | Coverage |
|----------|----------|
| **RFC 5545** | iCalendar: VEVENT, RRULE, RDATE, EXDATE, VTIMEZONE |
| **RFC 6578** | WebDAV sync-collection for incremental synchronization |
| **RFC 4791** | CalDAV calendar access protocol |
| **RFC 5545 §3.8.5** | Complete RecurrenceSet expansion: `(DTSTART ∪ RRULE ∪ RDATE) - EXDATE` |

---

## Installation

### Requirements

- Android 8.0 (Oreo) or higher
- ~15 MB storage space
- Internet connection for initial setup and sync

### Download

Get the latest APK from [Releases](https://github.com/KashCal/KashCal.github.io/tree/main/releases).

### Setup Guide

1. **Download** the APK from releases
2. **Install** (enable "Install from unknown sources" if prompted)
3. **Open** KashCal
4. **Connect** your iCloud account:
   - Tap **Settings** (gear icon)
   - Tap **iCloud Account**
   - Enter your **Apple ID email**
   - Enter an **App-Specific Password** (see below)
   - Tap **Connect**
   - Select which calendars to sync

### Creating an App-Specific Password 

For security, Apple recommends app-specific passwords for third-party apps:

1. Visit [appleid.apple.com](https://appleid.apple.com)
2. Sign in with your Apple ID
3. Go to **Sign-In and Security** → **App-Specific Passwords**
4. Click **Generate an app-specific password**
5. Name it "KashCal"
6. Copy the generated password (format: `xxxx-xxxx-xxxx-xxxx`)
7. Paste this password in KashCal's setup

**Benefits of app-specific passwords:**
- Your main Apple ID password is never stored in third-party apps
- Can be revoked independently without changing your main password
- Works with two-factor authentication enabled

---

### Sync Efficiency

KashCal uses **RFC 6578 sync-collection** for incremental synchronization:

- Server tells us only what changed since last sync
- No need to re-download your entire calendar
- Typical sync completes in under 1 second
- Falls back to full sync only when necessary (first connect, sync token expired)

---

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| **Sync not working** | Check internet connection; verify credentials in Settings → iCloud Account |
| **Events not appearing** | Pull down to refresh; check calendar filters (funnel icon) |
| **Reminders not firing** | Ensure notifications are enabled: Settings → App Info → Notifications |
| **Authentication failed** | Generate a new app-specific password and re-enter in Settings |
| **Recurring events missing instances** | Force full sync: Settings → Sync → Full Sync |


## FAQ

**Q: Is my data sent anywhere besides Apple's servers?**
A: No. KashCal communicates directly with iCloud CalDAV servers. There are no intermediary services, analytics, or telemetry.

**Q: Can I use this without iCloud?**
A: Yes! KashCal works as a standalone local calendar. iCloud sync is optional.

**Q: Why do I need an app-specific password?**
A: Apple requires app-specific passwords for third-party apps when you have two-factor authentication enabled. This is a security feature that protects your main Apple ID password.

**Q: Does KashCal support Google Calendar?**
A: Currently, KashCal focuses on iCloud sync. Google Calendar support may be added in future versions.

**Q: How do I import calendars from other apps?**
A: Use ICS subscriptions (Settings → ICS Subscriptions) for read-only calendar imports, or export/import via ICS files.

**Q: Is my data encrypted?**
A: Yes. Credentials are encrypted using AES-256-GCM with Android Keystore. Calendar data is stored in Android's app sandbox which is encrypted on devices with storage encryption enabled.

---

## Support

- **Bug Reports**: [GitHub Issues](https://github.com/KashCal/KashCal.github.io/issues)
- **Feature Requests**: [GitHub Discussions](https://github.com/KashCal/KashCal.github.io/discussions)

---


<p align="center">
  <b>KashCal</b><br>
  Your calendar. Your privacy. Your control.
</p>

  <p align="center">
    <img src="https://kashcal.github.io/images/KashCal-UI.png" height="500">&nbsp;&nbsp;
    <img src="https://kashcal.github.io/images/Apple-Calendar-Integration.png" height="500">&nbsp;&nbsp;
    <img src="https://kashcal.github.io/images/ICS-Subscription.png" height="500">
  </p>
