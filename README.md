# Wine builds

Optimized [Wine](https://www.winehq.org/) (WoW64) built for modern CPUs.

## Requirements

- **Architecture:** amd64
- **CPU:** x86-64-v3 (AVX2) or x86-64-v4 (AVX-512) - pick the matching variant
- **glibc:** 2.41+

**Builds will not run** on CPUs without AVX2/AVX-512. Check support:

```bash
grep -o 'avx[0-9_]*' /proc/cpuinfo | sort -u
```

If the output is empty - **do not use these builds**.

## Package

<details>
<summary>Build details</summary>

| Component      | Description                                                                                |
|----------------|--------------------------------------------------------------------------------------------|
| Unix part      | Built with system Clang (Devuan Excalibur)                                                 |
| Windows part   | Built with Clang from [llvm-mingw](https://github.com/mstorsjo/llvm-mingw) 20260922 (UCRT) |
| Linker         | LLD                                                                                        |
| Mode           | WoW64 (`--enable-archs=i386,x86_64`)                                                       |
| Compiler flags | `-march=x86-64-v3/znver3` or `-march=x86-64-v4/znver4`, `-O3`                              |

Debug symbols are **not included** in releases - they do not affect performance and only take up space.
</details>

## Installation

### 1. Download and extract

```bash
mkdir -p ~/wine-opt && cd ~/wine-opt
gh release download --repo argonforge/wine-builds --pattern '*.tar.xz'

# Unpack the archive matching your CPU:
tar -xJf wine-*-wow64-x86-64-v4.tar.xz   # AVX-512 (Zen 4/5)
# or
tar -xJf wine-*-wow64-x86-64-v3.tar.xz   # AVX2 (Zen 1/2/3)
```

Or download the `.tar.xz` file manually from the [Releases](../../releases) page.

### 2. Add to PATH

```bash
export PATH="$HOME/wine-opt/wine-11.0-wow64-x86-64-v4/bin:$PATH"
```

Add this line to `~/.bashrc` or `~/.profile` to make it persistent.

### 3. Verify

```bash
wine --version
```

## Initial setup

Create a Wine prefix and initialize it:

```bash
export WINEPREFIX="$HOME/.local/share/wineprefixes/default"
export WINEARCH=wow64
wineboot --init
```

`WINEARCH=wow64` enables the WoW64 mode, which runs 32-bit Windows applications without 32-bit Unix libraries.

## Verify WoW64 mode

```bash
wine --version
# Should report WoW64 support and both i386 and x86_64

WINEDLLOVERRIDES="d3d11=n" winecfg
```

## Rollback

This is a manual tarball installation - no package manager integration. To roll back:

```bash
# Remove the extracted directory
rm -rf ~/wine-opt/wine-*-wow64-x86-64-*

# Download and extract the previous version from Releases
# Or restore the stock Wine from your distribution:
sudo apt install --reinstall wine
```

## Expected Performance

| Component                              | Gain | Comment                                          |
|----------------------------------------|------|--------------------------------------------------|
| CPU part of Wine (syscall translation) | 1-3% | Noticeable only in CPU-bound scenarios           |
| WoW64 mode overhead                    | ~0%  | Compared to old WoW64 with 32-bit Unix libs      |
| Overall FPS in GPU-bound games         | ~0%  | Bottleneck is GPU and memory bandwidth, not Wine |

**Honest note:** Wine is a compatibility layer, not a graphics driver. Its own CPU cost is small, and optimizing it yields a few percent in CPU-bound scenarios at best. The main FPS gains in games come from the graphics stack (Mesa, DXVK, VKD3D-Proton), not from Wine itself.

Source workflow: [`.github/workflows/build.yml`](.github/workflows/build.yml).

## Companion projects

For a complete optimized graphics stack on **AMD Zen (x86-64-v3/v4)**:

| Project                                                                    | Purpose                           |
|----------------------------------------------------------------------------|-----------------------------------|
| [`mesa-builds`](https://github.com/argonforge/mesa-builds)                 | Mesa (radeonsi, RADV)             |
| [`dxvk-builds`](https://github.com/argonforge/dxvk-builds)                 | DXVK (D3D9/10/11 -> Vulkan)       |
| [`vkd3d-proton-builds`](https://github.com/argonforge/vkd3d-proton-builds) | VKD3D-Proton (D3D12 -> Vulkan)    |
| [`gamescope-builds`](https://github.com/argonforge/gamescope-builds)       | Micro-compositor for game scaling |

## Clear shader caches after installing new drivers

After updating Mesa, DXVK, or VKD3D-Proton, clear all shader caches:

```bash
rm -rf ~/.cache/mesa_shader_cache* \
       ~/.cache/dxvk/* \
       ~/.cache/vkd3d-proton/*
```

For per-game caches, remove `vkd3d-proton.cache*` next to the game's `.exe`.

## Important

- Builds are compiled on **Devuan Excalibur** (Debian Trixie, glibc 2.41+).
  They work on any glibc-based amd64 distribution with matching libraries
  (Debian 13+, Ubuntu 25.04+, Fedora 42+, Arch/Manjaro current).
  Older distributions (Debian 12, Ubuntu 22.04, RHEL 9) will fail with
  `GLIBC_2.XX not found`.
- Builds are **not signed**. Verify integrity using SHA-256 from the release description.
- **Do not use v4 builds** if you are unsure about AVX-512 support. Use v3 if your CPU has only AVX2.
- Runtime performance of Wine is limited by the compatibility layer's CPU-side overhead. The main FPS gains come from the graphics stack.
- Wine manipulation in online multiplayer games may be considered cheating. **Use at your own risk.**
- Always keep a working stock Wine installation as fallback.
- The author is not responsible for any system issues. Always have a Live USB ready for recovery.

## License

The build scripts and GitHub Actions workflows in this repository
are licensed under the MIT License. See LICENSE file.

The Wine source code is distributed under the
[LGPL-2.1-or-later](https://www.winehq.org/license). The compiled
binaries in Releases are redistributions of Wine under its original license.