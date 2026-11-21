# Building the aarch64 packages

Host: Fedora 45 x86_64. Target: Fedora 45 aarch64 (glibc 2.42).

## 1. Target sysroot

Signed Fedora 45 aarch64 RPMs are fetched from the Huawei Cloud mirror and
verified with the Fedora 45 key, then extracted into `.work/f45-cross`:

```bash
dnf5 --forcearch=aarch64 --releasever=45 \
  --repofrompath=wap-fedora,https://mirrors.huaweicloud.com/fedora/releases/45/Everything/aarch64/os/ \
  --repofrompath=wap-updates,https://mirrors.huaweicloud.com/fedora/updates/45/Everything/aarch64/ \
  --repo=wap-fedora,wap-updates download --resolve --alldeps \
  --destdir=.work/fedora45-aarch64-rpms \
  glibc-devel libstdc++-devel gcc libatomic gmp-devel mpfr-devel \
  dbus-devel freeglut-devel freetype-devel fftw-devel glew-devel glfw-devel \
  glm-devel lz4-devel libXcursor-devel libXi-devel libXinerama-devel \
  libXrandr-devel libXxf86vm-devel mesa-libGL-devel libglvnd-devel mpv-devel \
  ffmpeg-free-devel pulseaudio-libs-devel zlib-ng-compat-devel wayland-devel \
  wayland-protocols-devel sdl2-compat-devel

rpmkeys --dbpath .work/f45-rpmkeys --import /usr/share/pki/rpm-gpg/RPM-GPG-KEY-45-fedora
rpmkeys --dbpath .work/f45-rpmkeys --checksig .work/fedora45-aarch64-rpms/*.rpm
```

The RPMs are extracted with `rpm2cpio | cpio` into `.work/f45-cross`; the
cross toolchain is Fedora's `aarch64-linux-gnu-gcc` 16.2.1. Because that
toolchain's own `limits.h` shadows glibc's, the build defines
`MB_LEN_MAX=16` (glibc's documented value).

Toolchain file: `.work/fedora45-aarch64-toolchain.cmake`.

## 2. Engine

```bash
cmake -S .work/engine-src -B .work/engine-build -G Ninja \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_INSTALL_PREFIX=/usr/lib64/linux-wallpaperengine \
  -DCMAKE_TOOLCHAIN_FILE=.work/fedora45-aarch64-toolchain.cmake \
  -DCMAKE_EXE_LINKER_FLAGS="-L.../usr/lib/gcc/aarch64-redhat-linux/16 \
    -Wl,-rpath-link,.../usr/lib64 -lgcc_s" \
  -DSPIRV_CROSS_SKIP_INSTALL=ON \
  -DBUILD_TESTING=OFF -DKISSFFT_STATIC=ON -DLZ4_GOOD_VERSION_EXITCODE=0
ninja -C .work/engine-build linux-wallpaperengine qjs_exe qjsc
DESTDIR=.work/engine-install cmake --install .work/engine-build
```

`-lgcc_s` is required because the cross toolchain does not add it for the
thin `linux-wallpaperengine` executable; `qjs_exe`/`qjsc` are built only
because the vendored quickjs install rules reference them.

CEF 135.0.17 `linuxarm64` minimal is the only dependency without a domestic
mirror; it is fetched from `cef-builds.spotifycdn.com` with 32 parallel HTTP
range requests and verified against the published SHA-1
`3bcb0e3994b4f2db168c314468eb7cc2f3ea0d64`.

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
