# Reliable Download Manager — Development Plan

Developed by: Prof. Dattaraj S. Vidyasagar & Prof. Yash D. Vidyasagar, for Vidyasagar Robotics Academy.
Website: https://vsa.edu.in/software/ · Email: softinquiry@vsa.edu.in
Licence: GNU GPL v3.0 (permanently free)

## 1. Goal

A reliable, fast and resumable download manager with a small, IDM-like window.
It must be lightweight, easy to install, start instantly (no splash screen) and be easy to use.
Targets: Microsoft Store (Windows), Flathub and Snapcraft (Linux).

## 2. Technology

| Part | Choice | Why |
|---|---|---|
| App shell | Tauri v2 | Small installer (a few MB), native webview, no bundled browser |
| Download engine | Rust (tokio + reqwest with rustls) | Fast, safe, async, many parallel connections |
| User interface | Vanilla TypeScript + HTML + CSS (Vite) | No heavy framework, quick start-up |
| Package manager | pnpm only | Standing instruction |
| Builds | `build.bat` → output in `Release` folder | Standing instruction |

## 3. Download engine design (the core)

1. **Probe** the URL (HEAD, falling back to a ranged GET) to learn file size, file name, `Accept-Ranges`, `ETag` and `Last-Modified`.
2. **Segmented download**: split the file into parts (default 8 connections, user-adjustable 1–16) and download them in parallel.
3. **Dynamic segment splitting**: when a connection finishes early, it takes over half of the largest remaining part, so all connections stay busy until the end.
4. **Resume**: data is written to `<name>.rdm-part` and progress is saved to a small state file (`<name>.rdm-state`) every second. After a crash, power cut, app restart or network loss, the download continues from the saved byte offsets.
5. **Safety checks on resume**: `ETag` / `Last-Modified` / size must match; if the file changed on the server, the user is told and the download restarts cleanly.
6. **Auto-retry** with exponential back-off on network errors and timeouts; auto-resume when the network returns.
7. **Servers without range support**: single-connection download, with a clear "resume not supported" label.
8. **Finish**: verify size, rename the part file to the final name atomically, handle name clashes (`file (1).zip`).
9. **Optional**: speed limiter, SHA-256 checksum verification.

## 4. User interface (IDM-like, original design)

- **Main window** (small, about 760 × 440, resizable): toolbar (Add URL, Resume, Pause, Stop, Delete, Settings), category list (All, Compressed, Documents, Music, Programs, Video, Unfinished, Finished), and the download list (File name, Size, Status/%, Speed, Time left, Date).
- **Add URL dialog**: URL box (auto-filled from clipboard), save folder, file name, "Start now / Add to queue".
- **Progress window** per download: progress bar, per-connection segment bar, speed, time left, Pause/Cancel.
- **Settings**: download folder (default = system Downloads folder on Windows and Linux), number of connections, speed limit, clipboard monitoring, start with system, file-type categories.
- **About / Help**: Vidyasagar Robotics Academy logo, developer credits, website, email, licence.
- Light and dark theme following the system.

No IDM code, icons, names or artwork will be copied; only the general ease-of-use idea is followed.

## 5. Later features

- Browser integration (Chrome / Edge / Firefox extension sending links to the app).
- Download queue with scheduler (start/stop at a time, shut down when done).
- System tray icon and download-complete notifications.
- Clipboard link catcher.

## 6. Packaging and publishing

| Store | Format | Notes |
|---|---|---|
| Microsoft Store | MSI / MSIX from Tauri bundler | Uses Windows Downloads folder by default |
| Flathub | Flatpak manifest (`in.edu.vsa.ReliableDownloadManager`) with offline Cargo and pnpm sources | Uses XDG Download folder via portal |
| Snapcraft | `snapcraft.yaml`, strict confinement, `network` + `home` + `removable-media` plugs | Uses `~/Downloads` |

App ID (proposed): `in.edu.vsa.ReliableDownloadManager`.

## 7. Milestones

| Version | Milestone |
|---|---|
| 0.0.1 | Plan, development log, Git repository set up |
| 0.0.2 | Tauri v2 project skeleton, `build.bat`, `Release` folder, small main window |
| 0.0.3 | Single-connection download with pause/resume |
| 0.0.4 | Multi-connection segmented download with dynamic splitting |
| 0.0.5 | Crash-safe resume (state files), retries, auto-resume |
| 0.0.6 | Download list, categories, Add URL dialog, progress window |
| 0.0.7 | Settings (download folder, connections, speed limit) |
| 0.0.8 | App icon (after Professor's approval), About / Help windows |
| 0.0.9 | Windows installer and Microsoft Store package |
| 0.0.10 | Flatpak and Snap packages |
| 0.1.0 | First public release on all three stores |

Each milestone may need more than one version; every change increases the version number.
