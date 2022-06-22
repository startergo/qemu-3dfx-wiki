# Welcome to the qemu-3dfx wiki!
## Setting Up Windows 98 SE Virtual Machine on QEMU -- Revised 6/20/2022
The syntax of instructions are designed to be agnostic whether one uses Windows 10 or modern Linux distro and based on BASH shell environment. BASH shell is the standard console environment for most Linux. For Windows 10 users, it is highly recommended that one switches to similar BASH shell environment instead of staying with Windows 10 PowerShell or plain Command Prompt. [MSYS2](https://www.msys2.org) and [MinGW - Minimalist GNU for Windows](https://sourceforge.net/projects/mingw) provide BASH shell on Windows 10.

**Requirements**:
+ Somewhat familiar with basic DOS commands
+ Windows 98 SE CD ISO image `W98SE.ISO`
+ Windows 98 SE Bootable floppy image `FD.IMA` with:
  - ATTRIB.EXE
  - EDIT.COM, EDIT.HLP
  - FDISK.EXE
  - FORMAT.COM
  - SYS.COM
  - XCOPY.EXE, XCOPY32.MOD, XCOPY32.EXE
  - OAKCDROM.SYS, MSCDEX.EXE or alternatives -- (**CD-ROM support from DOS**)
  - CWSDPMI.EXE, PATCH9X.EXE -- (**Fix MMU TLB Invalidation**)
+ BOXV9x fully Open-Source mini-port Display Driver for Windows 9x
+ SigmaTel C-Major Audio AC97 WDM driver
+ LSI 53C895A SCSI mini-port Storage Driver for Windows 9x

**Optional**:
+ Microsoft BATCH 98 for unattended installation
+ INFINST for **Additional Drivers** during Windows 98 SETUP

## Windows 98 SE Installation Phases
There are 3 phases of installation, a required reboot from one phase to another and the last phase will present the "Welcome to Windows 98" theme.
+ Phase 1: Select installation type, enter serial and files copy
+ Phase 2: Hardware detection, control panels and system setup
+ Phase 3: Boot up & "Welcome to Windows 98"

**All phases can now be accelerated with KVM/WHPX when 32-bit protected-mode storage driver is available during Phase 2 of Windows 98 SETUP.** QEMU emulated LSI 53C895A SCSI however is **NOT included as inbox drivers** from Windows 98 SE CD. **Reboot/Shutdown works properly** from within QEMU virtual machine.

## Getting Started
1. Create Windows 98 SE HDD image
```
$ cd ~/myqemu/qemu-3dfx/build && mkdir ../vmimgs
$ ./qemu-img create -f qcow2 ../vmimgs/w98.qcw 1024M
```
2. Boot from FD.IMA, FDISK/FORMAT HDD image
```
$ ./qemu-system-i386 -nodefaults -rtc base=localtime display sdl \
    -M pc,accel=kvm,hpet=off,usb=off -cpu host \
    -device VGA -device lsi -device ac97 \
    -netdev user,id=net0 -device pcnet,rombar=0,netdev=net0 \
    -drive if=floppy,format=raw=file=fd.ima \
    -drive id=win98,if=none,file=../vmimgs/w98.qcw -device scsi-hd,drive=win98 \
    -cdrom w98se.iso -boot a
```
3. Copy CD content "`D:\WIN98`" to HDD "`C:\MYCD`" and apply PATCH9X
```
A:\> xcopy32 /s /e D:\WIN98 C:\MYCD
A:\> patch9x C:\MYCD
```
4. Boot from HDD and run "`SETUP /nm /pj`"
```
$ ./qemu-system-i386 -nodefaults -rtc base=localtime display sdl \
    -M pc,accel=kvm,hpet=off,usb=off -cpu host \
    -device VGA -device lsi -device ac97 \
    -netdev user,id=net0 -device pcnet,rombar=0,netdev=net0 \
    -drive if=floppy,format=raw=file=fd.ima \
    -drive id=win98,if=none,file=../vmimgs/w98.qcw -device scsi-hd,drive=win98 \
```
Thanks to KVM/WHPX acceleration, from running SETUP to the Welcome to Windows 98 First Bell takes less than 10 mins on reasonably fast systems with SSD storage. With the optional additional drivers available during Phase #2 of Windows 98 Setup, the full 640x480 256-color desktop is readily available and all drivers installed. Otherwise, proceed to install all the drivers ***AFTER*** the modification to workaround KVM/WHPX reboot/shutdown. Create a 2-line `_STARTUP.BAT` in `C:\WINDOWS`.
```
@echo off
exit
```
Create a PIF for `_STARTUP.BAT` and add to `"Start->Programs->StartUp"` so that it is run automatically in `"Full-screen"` and `"Close on Exit"`.

For those who share the same OS image between Linux KVM and Windows WHPX, it is still recommended to perform the steps to disable Windows 98 Startup/Shutdown logos. **Windows WHPX seems to crash prone** during the display of logos between reboot/shutdown. Otherwise, **this step is optional for Linux KVM.** To disable Windows 98 Startup/Shutdown logos, edit `C:\MSDOS.SYS` and place `Logo=0` under `[Options]`. Remove or rename `LOGO*.SYS` in `C:\WINDOWS`.

Finally, we are in for **KVM/WHPX super-charged QEMU accelerated** Windows 98 SE virtual machine that brings modern CPU/GPU prowess to retro Windows games.
```
$ ./qemu-system-i386 -nodefaults -rtc base=localtime display sdl \
    -M pc,accel=kvm,hpet=off,usb=off -cpu host \
    -device VGA -device lsi -device ac97 \
    -netdev user,id=net0 -device pcnet,rombar=0,netdev=net0 \
    -drive if=floppy,format=raw=file=fd.ima \
    -drive id=win98,if=none,file=../vmimgs/w98.qcw -device scsi-hd,drive=win98 \
    -drive id=games,if=none,file=../vmimgs/mygames.qcw -device scsi-hd,drive=games
```
A [reference video](https://www.youtube.com/watch?v=4J9Br9ojkhg) is available at YouTube channel providing all the details in setting up Windows 98 SE QEMU virtual machine from scratch to 3D Acceleration in 15 mins.
## What About AdLib/Gravis UltraSound/SoundBlaster 16 For DOS Games
For DOS games, [DOSBox](https://www.dosbox.com) is a far superior emulator with superb audio hardware emulation. For QEMU, AC97 WDM drivers provide the best audio solution for Windows 98 SE from more faithfully emulated AC97 audio controller due to the openness of AC97 industry specification. For a few exceptions when DOS games may have worked out better from within Windows 98 SE due to protected-mode CD-ROM and MOUSE drivers, AC97 WDM drivers support SoundBlaster Pro compatibility for PCM playback. DOS games such as TES Redguard, Tomb Raider, PYL, Screamer 2 etc. are known to work well with WDM Audio SoundBlaster Pro emulation from within Windows 98 SE. 