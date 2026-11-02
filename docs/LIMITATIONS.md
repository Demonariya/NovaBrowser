# NovaBrowser Limitations & Environment Constraints
**Document ID:** `DOC-LIM-001`  
**Version:** `0.0.1`  
**Date:** 2026-11-02  

---

## 1. Engine Limitations (Microsoft WebView2)

### 1.1 Chrome Web Store Extensions
- **Limitation:** Microsoft WebView2 does not provide a public runtime API for installing or executing general Chrome Web Store extensions (`manifest.json` v3 service workers, `chrome.tabs`, `chrome.webRequest`, extension popups).
- **Engineering Principle:** NovaBrowser will **never** render a simulated or fake extensions store. The Settings Center and menus will explicitly mark extensions as unavailable on this engine rather than displaying broken controls.
- **Alternative:** Custom user scripts (Tampermonkey-like DOM injection) can be supported via `ICoreWebView2::AddScriptToExecuteOnDocumentCreated` if requested in future revisions.

---

## 2. Development Environment & Testing Limitations

### 2.1 Current Development Environment
- **Operating System:** Debian GNU/Linux 12 (bookworm) x86_64 running under a containerized Linux runtime.
- **Target Platform:** Windows 10 and Windows 11 x64.

### 2.2 Features That Cannot Be Tested Locally on Linux
The following Windows-specific subsystems cannot be executed inside the Linux development container:
1. **Microsoft WebView2 Runtime (`WebView2Loader.dll` & COM runtime):**
   - Win32 COM interfaces (`ICoreWebView2`, `ICoreWebView2Controller`, `ICoreWebView2Environment`).
   - Windows native window message loops (`HWND`, `WM_SIZE`, `WM_PAINT`).
2. **Windows Data Protection API (DPAPI):**
   - `CryptProtectData` and `CryptUnprotectData` for hardware/OS-bound master key encryption.
3. **Windows Credential Manager:**
   - Native Windows vault APIs.
4. **Windows Native Notifications:**
   - Action Center / WinRT toast notification integration.

### 2.3 Verification Strategy
- **Local Linux Verification:** Code formatting (`cargo fmt`), linting (`cargo clippy` with target-agnostic / mock platform gates where relevant), TypeScript typechecking (`tsc --noEmit`), Vite production frontend bundling (`vite build`), and unit tests for cross-platform crates (downloader, filter engine, SQLite schemas, URL parsing).
- **Windows Verification Pipeline:** Automatic CI execution on `windows-latest` via GitHub Actions (`.github/workflows/windows-ci.yml`) for MSVC compilation, Win32 COM linking, and real Windows installer creation (NSIS/MSI).
- **Integrity Rule:** We will strictly never claim a Windows feature has been tested or verified unless confirmed by a real Windows runner.
