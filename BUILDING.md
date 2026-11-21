# Building the aarch64 packages

Host: GitHub-hosted `ubuntu-24.04-arm` (native aarch64, 4 CPUs, 16 GB RAM).
The engine is built in a native Fedora 45 ARM64 container; the Steamworks addon
and UI build directly on the ARM64 runner. No cross compiler, extracted sysroot,
or QEMU is required.

Workflow: `NEURAX-FX/lwe-arm64-ci/.github/workflows/build.yml`.

## 1. Native ARM64 runner

The workflow installs host packaging tools (`fpm`, `rpm`, `zip`) and verifies
the host with `uname -m`. Fedora dependencies are installed in `fedora:45`
from the Fedora 45 aarch64 Everything repository on Huawei Cloud with RPM
signature verification enabled. The CEF archive remains pinned to
135.0.17 `linuxarm64` minimal and is checked against its published SHA-1.

## 2. Engine

Inside the Fedora 45 container, configure and install natively:

```bash
cmake -S . -B build -G Ninja \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_INSTALL_PREFIX=/usr/lib64/linux-wallpaperengine \
  -DSPIRV_CROSS_SKIP_INSTALL=ON \
  -DBUILD_TESTING=OFF -DKISSFFT_STATIC=ON \
  -DKISSFFT_TEST=OFF -DKISSFFT_TOOLS=OFF
cmake --build build --parallel 4 --target linux-wallpaperengine qjs_exe qjsc
DESTDIR=.work/engine-install cmake --install build
```

The explicit KissFFT test/tool switches keep CMake install from referencing
unused command-line targets. CI runs `linux-wallpaperengine --help` from the
installed payload before packaging.

## 3. UI

```bash
cd .work/gui-src
bun install --frozen-lockfile
node scripts/app.mjs build
./node_modules/.bin/electron-builder --linux --arm64 \
  -c electron-builder.arm64.yml --publish never
```

`electron-builder.arm64.yml` pins the verified official
`electron-v39.7.0-linux-arm64.zip` (SHA-256
`0154dfe256dbef46abd34257be10cab37cb4b117a144dbbf73d22159fa28abfb`).

## 4. Steamworks addon

`steamworks.js` 0.4.0 ships only x64/x86/arm64-macOS binaries. Its Rust
sources are rebuilt for aarch64 against `steamworks` 0.13.1 /
`steamworks-sys` 0.13.0 (Steamworks SDK v1.65), with small API updates for the
newer crate and an aarch64 branch in `index.js`. Crates come from the rsproxy
mirror.

```bash
cargo build --release --target aarch64-unknown-linux-gnu   # in .work/addon
```

The resulting `libsteamworksjs.so` is installed as
`node_modules/steamworks.js/dist/linux64/steamworksjs.linux-arm64-gnu.node`,
next to the SDK's aarch64 `libsteam_api.so` (RPATH `$ORIGIN`).

## 5. Packages

`.work/package-engine-final.sh` strips DWARF from every ELF in the payload
(CEF's `libcef.so` shrinks from 2.0 GiB to 389 MiB), drops headers/static
libraries/cmake files/quickjs CLIs, keeps `chrome-sandbox` setuid, builds the
portable ZIP, and cross-packages the RPM with `fpm` (host `rpmbuild` refuses
`BuildArch: aarch64` on x86_64):

```bash
fpm -s dir -t rpm -n linux-wallpaperengine -v 0.20260609 \
  --iteration 1.gita0ddce5 --rpm-dist fc45 --architecture aarch64 ...
```

The UI RPM/ZIP come from `electron-builder` (`release-arm64/`).
