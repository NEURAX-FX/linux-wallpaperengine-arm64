# Linux Wallpaper Engine — aarch64 (Fedora 45)

Unofficial **ARM64** builds of the wallpaper engine runtime and the desktop UI.

| Component | Upstream | Version |
| --- | --- | --- |
| Engine runtime | Almamu/linux-wallpaperengine | commit `a0ddce5` (2026-06-09) |
| Desktop UI | jagrat7/linux-wallpaper-engine | v0.4.15 |
| CEF | official `linuxarm64` minimal | 135.0.17 |
| Electron | official `linux-arm64` | 39.7.0 |
| Steamworks runtime | ARM64 library bundled with `steamworks-sys` | 0.13.0 |

## Assets

The engine and desktop UI are each available as an RPM and a portable ZIP.
Download `SHA256SUMS` with the assets and check their integrity with
`sha256sum -c SHA256SUMS`.

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
  aarch64 from `steamworks.js` 0.4.0 sources against `steamworks` 0.13.1 and
  its matching `steamworks-sys` runtime, because upstream only ships
  x64/arm64-macOS binaries.
- Electron 39.7.0 `linux-arm64` from the official release (SHA-256 verified).

## Verification

- [Native ARM64 build](https://github.com/NEURAX-FX/lwe-arm64-ci/actions/runs/44913602961):
  `linux-wallpaperengine --help` exits 0 in the Fedora 45 build container;
  the packaged Steamworks addon loads in ARM64 Node 22 with `init` and
  `workshop.getAllItems` exposed.
- [Release verification](https://github.com/NEURAX-FX/linux-wallpaperengine-arm64/actions/runs/44916570654):
  both RPMs report `aarch64`, the ELF files in both ZIPs are ARM64, and all four
  files match `SHA256SUMS`.

## Notes

- Built natively on GitHub's `ubuntu-24.04-arm` runner with the engine inside a
  Fedora 45 ARM64 container. The RPMs target Fedora 45 aarch64.
- No AppImage is included; use the RPM or ZIP.
- CEF debug info was stripped; the runtime payload is ~480 MiB installed.
