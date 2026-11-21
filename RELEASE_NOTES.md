# Linux Wallpaper Engine — aarch64 (Fedora 45)

Unofficial **ARM64** builds of the wallpaper engine runtime and the desktop UI.

| Component | Upstream | Version |
| --- | --- | --- |
| Engine runtime | Almamu/linux-wallpaperengine | commit `a0ddce5` (2026-06-09) |
| Desktop UI | jagrat7/linux-wallpaper-engine | v0.4.15 |
| CEF | official `linuxarm64` minimal | 135.0.17 |
| Electron | official `linux-arm64` | 39.7.0 |
| Steamworks SDK | Valve `redistributable_bin/linuxarm64` | v1.65 |

## Install

```bash
sudo dnf install ./linux-wallpaperengine-0.20260609-1.gita0ddce5.fc45.aarch64.rpm
sudo dnf install ./linux-wallpaper-engine-0.4.15-aarch64.rpm
```

The UI finds `linux-wallpaperengine` in `PATH`; the engine package installs
`/usr/bin/linux-wallpaperengine`, which runs the runtime from
`/usr/lib64/linux-wallpaperengine`. Portable ZIPs are attached too.

You still need **Wallpaper Engine from Steam** for assets and workshop
content, same as on x86_64.

## What was rebuilt

- Engine: C++20, CEF 135.0.17 `linuxarm64` minimal, X11 + Wayland
  (wlr-layer-shell) enabled, SDL2 via Fedora's `sdl2-compat`.
- UI: Vite bundles + asar; the Steamworks native addon was rebuilt for
  aarch64 from `steamworks.js` 0.4.0 sources against Steamworks SDK v1.65,
  because upstream only ships x64/arm64-macOS binaries.
- Electron 39.7.0 `linux-arm64` from the official release (SHA-256 verified).

## Verification

- `file`/`readelf`: every shipped ELF is `ARM aarch64`.
- Engine: `linux-wallpaperengine --help` under `qemu-aarch64` with the
  Fedora 45 aarch64 userland.
- UI: packaged `linux-wallpaper-engine --version` under `qemu-aarch64` +
  `Xvfb`.
- Steamworks addon: loaded in aarch64 Node 22 under `qemu-aarch64`; exports
  `init`, `runCallbacks`, `workshop.getAllItems`, `workshop.subscribe`,
  `workshop.unsubscribe`, `workshop.download`, `workshop.downloadInfo`,
  `workshop.installInfo`, `callback.register`.

## Notes

- Built by cross-compiling on x86_64 against the Fedora 45 aarch64 sysroot;
  the RPMs target Fedora 45 (glibc 2.42).
- No AppImage: electron-updater's AppImage target is not produced for aarch64.
- The engine RPM contains CEF's unstripped runtime minus DWARF sections.
