# Changelog

All notable changes to the NovaBrowser project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.0.1] - 2026-11-02

### Added
- **Phase 0 — Technical Investigation & Environment Audit**:
  - Full environment diagnostic (Debian GNU/Linux 12 container environment vs. target Windows 10/11 x64 platform).
  - Rust stable toolchain and development tooling verification.
  - Architectural comparative study for Windows desktop browser engines (`docs/ENGINE_DECISION.md`):
    - Tauri 2 + Microsoft WebView2 (`ICoreWebView2_10`+)
    - Chromium Embedded Framework (CEF) / `cef-rs`
    - Ultralight / Qt WebEngine
  - 15-point browser engine capability evaluation matrix (Multi-tab, Isolated Profiles, Google Account Authentication, Proxy routing, Cookie management, Private browsing, Request interception, Ad/Tracker blocking, Download management, Extensions, DevTools, PiP, Certificate validation, Session restoration, Cache management).
  - Modular Rust architecture design and crate boundaries.
  - CI/CD workflow configuration for Windows x64 testing and build verification via GitHub Actions (`.github/workflows/windows-ci.yml`).
