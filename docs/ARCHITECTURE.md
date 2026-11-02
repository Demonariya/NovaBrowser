# NovaBrowser Technical Architecture
**Document ID:** `DOC-ARCH-001`  
**Version:** `0.0.1`  
**Date:** 2026-11-02  

---

## 1. High-Level Architecture Overview

NovaBrowser separates the user interface and browser view rendering into distinct, secure boundaries communicating via typed IPC (Inter-Process Communication):

```text
┌─────────────────────────────────────────────────────────────┐
│                 NovaBrowser UI Shell (React)                │
│  Tab Bar  │  Omnibox  │  Sidebar  │  Profile Switcher       │
└───────────────────────────┬─────────────────────────────────┘
                            │ Typed Tauri IPC (Invoke / Event)
┌───────────────────────────▼─────────────────────────────────┐
│              NovaBrowser Rust Systems Backend               │
│                                                             │
│ ┌────────────────┐ ┌────────────────┐ ┌───────────────────┐ │
│ │  browser-core  │ │    profiles    │ │     storage       │ │
│ │  (Tab State)   │ │ (Isolation)    │ │ (SQLite / WAL)    │ │
│ └────────────────┘ └────────────────┘ └───────────────────┘ │
│ ┌────────────────┐ ┌────────────────┐ ┌───────────────────┐ │
│ │ filter-engine  │ │   downloader   │ │      proxy        │ │
│ │ (Adblock-rs)   │ │(Tokio/Reqwest) │ │ (SOCKS5/Leak Prot)│ │
│ └────────────────┘ └────────────────┘ └───────────────────┘ │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │                    native-windows                       │ │
│ │         (Win32 COM / ICoreWebView2 Controllers)         │ │
│ └─────────────────────────────────────────────────────────┘ │
└───────────────────────────┬─────────────────────────────────┘
                            │ Win32 HWND / COM Attachment
┌───────────────────────────▼─────────────────────────────────┐
│        Microsoft WebView2 Process Architecture (Evergreen)  │
│  Active Tab View 1  │  Tab View 2  │  ...  │  InPrivate View │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Crate Responsibilities & Boundaries

1. **`browser-core`**:
   - Manages tab collections, tab groups, pinning, duplication, and navigation history stacks.
   - Evaluates Omnibox input: detects whether input is an IPv4/IPv6, FQDN, local host, or search query.
   - Handles search provider URLs (Google, DuckDuckGo, Bing, Brave, Ecosia).

2. **`profiles`**:
   - Manages profile directories in `%APPDATA%/NovaBrowser/Profiles/<id>/`.
   - Isolates Google sessions, cookies, caches, and local storage per profile.
   - Coordinates profile avatar, color themes, and active identity metadata.

3. **`proxy`**:
   - Validates SOCKS4/SOCKS5/HTTP/HTTPS proxy configurations.
   - Performs active health-check probes through the proxy before routing browser traffic.
   - Enforces fail-closed behavior to prevent unencrypted direct IP leaks.

4. **`downloader`**:
   - Manages an asynchronous Tokio download queue with bounded concurrency.
   - Issues HTTP Range requests (`Range: bytes=start-end`) for paused/resumed downloads.
   - Calculates real-time instantaneous and exponential-moving-average transfer speeds.
   - Computes streaming SHA-256 hashes to verify payload integrity upon download completion.

5. **`filter-engine`**:
   - Compiles standard EasyList and EasyPrivacy rule sets into an efficient byte-code format using Rust's `adblock` engine.
   - Matches incoming network requests synchronously with zero noticeable latency.
   - Maintains real-time blocked request counters per domain and profile.

6. **`storage`**:
   - Embedded SQLite database storing non-sensitive browser metadata (bookmarks, folders, history records, proxy profiles, download queue state, and settings).
   - Utilizes Write-Ahead Logging (WAL) and automatic migration scripts.

7. **`security`**:
   - Employs AES-256-GCM / ChaCha20-Poly1305 AEAD authenticated encryption.
   - Binds master keys to Windows DPAPI (`CryptProtectData`) on Windows.

8. **`native-windows`**:
   - Interfaces directly with `WebView2Loader.dll` and COM interfaces (`ICoreWebView2_10` to `ICoreWebView2_24`).
   - Hooks into `WebResourceRequested` for ad blocking and `DownloadStarting` for custom download handling.
