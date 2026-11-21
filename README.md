# linux-wallpaperengine-arm64

Unofficial **aarch64 (ARM64)** builds, packaged for **Fedora 45**, of:

- [Almamu/linux-wallpaperengine](https://github.com/Almamu/linux-wallpaperengine) @ `a0ddce586e9ff12e9d31ca6b103f6036cc1b68fe` — the wallpaper engine runtime
- [jagrat7/linux-wallpaper-engine](https://github.com/jagrat7/linux-wallpaper-engine) @ `v0.4.15` — the desktop UI

Both projects are MIT / GPL-3.0 licensed; this repository only hosts the
rebuilt binaries and the build recipe.

## Download

All artifacts are attached to the
[`v0.4.15-arm64` release](https://github.com/NEURAX-FX/linux-wallpaperengine-arm64/releases/tag/v0.4.15-arm64),
with a `SHA256SUMS` file.

## Install (Fedora 45 aarch64)

```bash
sudo dnf install ./linux-wallpaperengine-0.20260609-1.gita0ddce5.fc45.aarch64.rpm \
                 ./linux-wallpaper-engine-0.4.15-aarch64.rpm
```

The UI looks up `linux-wallpaperengine` in `PATH`; the engine package installs
`/usr/bin/linux-wallpaperengine`, a wrapper that runs the runtime from
`/usr/lib64/linux-wallpaperengine`.

Portable ZIPs are attached as well (`linux-wallpaper-engine-0.4.15-arm64.zip`,
`linux-wallpaperengine-0.20260609-aarch64.zip`) if you prefer not to install
packages.

You still need **Wallpaper Engine from Steam** for the assets and workshop
content, exactly like the x86_64 builds.

## What is inside

| Component | Build |
| --- | --- |
| Engine runtime | C++20, CEF 135.0.17 `linuxarm64` minimal, X11 + Wayland (wlr-layer-shell) |
| UI | Electron 39.7.0 official `linux-arm64`, asar-packed Vite bundles |
| Steam Workshop addon | `steamworks.js` 0.4.0 sources rebuilt for aarch64 against Steamworks SDK v1.65 |

## Verification performed

- All ELF files are `ARM aarch64` (`file`/`readelf`).
- `linux-wallpaperengine --help` runs under `qemu-aarch64` against the Fedora 45 aarch64 sysroot.
- The Steamworks addon loads in aarch64 Node 22 (`qemu-aarch64`) and exposes
  `init`, `runCallbacks`, `workshop.getAllItems`, `workshop.subscribe`,
  `workshop.unsubscribe`, `workshop.download`, `workshop.downloadInfo`,
  `workshop.installInfo`, `callback.register`.
- `electron --version` runs under `qemu-aarch64`.

## Build recipe

See [BUILDING.md](BUILDING.md).

## Notes / known limits

- Built by cross-compiling on x86_64 with the Fedora 45 aarch64 sysroot, so the
  RPMs target Fedora 45 userland (glibc 2.42).
- The UI's AppImage target is not built for aarch64; use the RPM or ZIP.
- Wallpaper Engine itself is Windows-only; this is the Linux reimplementation.
