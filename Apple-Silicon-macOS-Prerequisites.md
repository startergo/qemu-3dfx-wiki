## The Road To Apple Silicon macOS
For a long-time Windows/Linux enthusiast, Apple software ecosystem had always been remote and distant, to a certain extent "Who Cares?!"🤣 The urge for concept proving that qemu-3dfx remains portable beyond the typical Intel/AMD x86 CPUs and the emergence of Apple Silicon as the **World's Highest Performance ARM AArch64 CPU** ignited the dive into Apple Silicon macOS. However, QEMU binary package was not available to Apple Silicon macOS as the platform of choice due to hardened runtime and mandatory code signing. Anyway, my sincere appreciation remains for ALL who had supported & donated to the project, those in particular, without even exercising donor's privilege.

## QEMU Binary Package for Apple Silicon macOS
In this cordial and delightful moment, QEMU binary package for Apple Silicon is **available** to users of Apple Silicon macOS. No one will be told to do ***STUPID THINGS*** such as ***disabling GateKeeper in jeopardizing Apple macOS System Integrity Protection*** to get something working. QEMU binary package will be built and delivered for Apple Silicon macOS as the platform of choice with a small donation to support the project and Game Preservation for those cannot afford the time & determination in the **Journey of FOSS**. The strike of [⚡**FORCE Lightning**⚡] is deeply regretted.

## The Prerequisites
For QEMU binary package to work properly on Apple Silicon macOS, the system ***MUST*** meet the following prerequisites & configuration

1. Install [XQuartz](https://www.xquartz.org/index.html).
2. Install [HomeBrew](https://brew.sh/).
3. Verify & Match the Setup
```
$ which X
/opt/X11/bin/X
$ brew --prefix
/opt/homebrew
```
4. Install the bottles from brew
```
$ brew install capstone glib gettext gnutls libepoxy libgcrypt libslirp jpeg-turbo lz4 opus sdl2 zstd
```
For package bundled with **DOSBox SVN** Games essentials, install additional bottles from brew
```
$ brew install sdl_sound sdl12-compat sdl_net mt32emu
```
5. Verify Apple Command Line Tools for Xcode. Example from `macOS Monterey 12.5.1`, exact `version` may differ in future.
```
$ clang --version
Apple clang version 13.1.6 (clang-1316.0.21.2.5)
Target: arm64-apple-darwin21.5.0
Thread model: posix
InstalledDir: /Library/Developer/CommandLineTools/usr/bin
```
## Installing QEMU
Install by untar QEMU binary package, **`'sudo'`** may be required. Exact `version` and `commit id` may differ in future.
```
$ sudo tar xf qemu-7.1.0-3dfx-dae79bb-darwin-arm64.tar.zst -C /
$ cd `brew --prefix`/sign
$ bash ./qemu.sign
```
Verify the installed & signed QEMU
```
$ qemu-system-i386 --version
```
## QEMU CPU Invocation Option
The software libraries in QEMU VM Addons (since commit [`5112b64`](https://github.com/kjliew/qemu-3dfx/commit/5112b64c60d79fdcef81e9da0bee561a9a2677b8)) are optimized for modern x86 SIMD instruction sets through vectorizing code generation compilers options **`'-march=x86-64-v2 -mtune=generic'`**. There is no reason not to take advantages of such capabilities. For Intel/AMD CPUs with KVM/WHPX acceleration, the **`'x86-64-v2'`** architecture level is given for any CPUs in the last 7 years. Apple Silicon does not support x86 virtualization and QEMU TCG default CPU options is good up to SSE2. The following CPU invocation option is required for QEMU TCG to enable **`'x86-64-v2'`** architecture level for CPU emulation.
```
$ qemu-system-i386 -cpu max ...
```
## What About My OSX/macOS Native Games Collection
I felt ***SORRY*** for those with collection of OSX/macOS native Games. Apparently, Apple deprecation policy mandated that everyone stopped living in the past with Good Old Games and looked at the bright side of flourishing modern Games in iTune/AppStore. Apple Silicon is poised to eventually unite iOS and macOS as **"One-Apple Platform"** for Games. ***THAT IS THE FUTURE!!*** Fortunately with the tremendous performance offered by Apple Silicon despite lacking support for x86 virtualization, many PowerPC macOS ports of the same Windows PC Games are indeed playable on Apple Silicon through QEMU with 3D graphics acceleration. Notable examples are [Core Design Tomb Raider (I, II, III, IV, V) series](https://www.youtube.com/watch?v=-wb_IKY0Mkc) from the late 90's, Unreal Tournament GOTY, Diablo II, Shogo Mobile Armor Division, Homeworld and almost all Games based on Quake idTech1/2/3 engine and many more are playable with Windows 98 SE VM on Apple Silicon macOS. This is the results of Game Preservation from qemu-3dfx with **Peace-of-Mind VM Isolation**, **"Pristine Condition" in Retail Originality** and **Playability Beyond Windows OS and x86 PCs**.