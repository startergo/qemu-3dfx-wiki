# Welcome to the qemu-3dfx wiki!
## Setting Up Windows 98 SE Virtual Machine on QEMU
The syntax of instructions are designed to be agnostic whether one uses Windows 10 or modern Linux distro and based on BASH shell environment. BASH shell is the standard console environment for most Linux. For Windows 10 users, it is highly recommended that one switches to similar BASH shell environment instead of staying with Windows 10 PowerShell or plain Command Prompt. MSYS2 (https://www.msys2.org) and MinGW - Minimalist GNU for Windows (https://sourceforge.net/projects/mingw) provide BASH shell on Windows 10.

**Requirements**:
+ Somewhat familiar with basic DOS commands
+ Windows 98 SE Bootable CD ISO image `w98se.iso`
+ Windows 98 SE Bootable floppy image `fd.img` with:
  - ATTRIB.EXE
  - EDIT.COM, EDIT.HLP
  - FDISK.EXE
  - FORMAT.COM
  - SYS.COM
  - XCOPY.EXE, XCOPY32.MOD, XCOPY32.EXE
+ VBEMP Win9x Universal VESA display driver (http://bearwindows.zcm.com.au/140214.zip)
+ SigmaTel C-Major Audio AC97 WDM driver
+ Realtek RTL8139 Win98 driver

It is recommended that all drivers are uncompressed and placed in the same bootable floppy image. They should fit in 1.44M floppy image.

## Windows 98 SE Installation Phases
There are 3 phases of installation, a required reboot from one phase to another and the last phase will present the "Welcome to Windows 98" theme.
+ Phase 1: Select installation type, enter serial and files copy
+ Phase 2: Hardware detection, control panels and system setup
+ Phase 3: Boot up & "Welcome to Windows 98"

Phase 1 can be accelerated with KVM on Linux `"accel=kvm"` or WHPX on Windows 10 `"accel=whpx"`, but the remaining phases will have to do without it until VBEMP display driver can be installed successfully and other modifications required for KVM/WHPX to provide full acceleration for the virtual machine. Rebooting Windows 98 SE does not quite work for QEMU, so we disable it with `"-no-reboot"` and reboot will simply shut down QEMU gracefully. We will also be installing from HDD instead of directly from the CD media to speed things up.

## Getting Started
1. Create Windows 98 SE HDD image
```
$ cd ~/myqemu/qemu-3dfx/build && mkdir ../vmimgs
$ ./qemu-img create -f qcow2 ../vmimgs/w98.qcw 1024M
```
2. Boot from floppy image, FDISK and FORMAT HDD image
```
$ ./qemu-system-i386 -L pc-bios \
    -rtc base=localtime -no-hpet -no-reboot \
    -drive file=../vmimgs/w98.qcw \
    -device VGA \
    -M pc,accel=kvm -device ac97 \
    -netdev user,id=net0 -device rtl8139,netdev=net0 \
    -drive if=floppy,file=fd.img -boot a
```
3. Boot from CD with CD-ROM support
```
$ ./qemu-system-i386 -L pc-bios \
    -rtc base=localtime -no-hpet -no-reboot \
    -drive file=../vmimgs/w98.qcw \
    -device VGA \
    -M pc,accel=kvm -device ac97 \
    -netdev user,id=net0 -device rtl8139,netdev=net0 \
    -cdrom w98se.iso -boot d
```
4. Copy CD content "`D:\WIN98`" to HDD "`C:\MYCD`" and start Windows 98 SE installation from HDD
```
C:\> mkdir MYCD
C:\> xcopy /S /E D:\WIN98 C:\MYCD
C:\> cd MYCD
C:\MYCD> setup /nm /pj
```
5. Proceed on Phase 1 installation. Reboot after finished files copy will shut down QEMU. Restart QEMU **without** `accel=kvm|whpx` for the remaining phases. It will take a long time, about 30~45 mins on reasonably fast modern system
```
$ ./qemu-system-i386 -L pc-bios \
    -rtc base=localtime -no-hpet -no-reboot \
    -drive file=../vmimgs/w98.qcw \
    -device VGA \
    -M pc -device ac97 \
    -netdev user,id=net0 -device rtl8139,netdev=net0 \
    -drive if=floppy,file=fd.img
```
6. Reboot at end of Phase 2 will shut down QEMU. Boot from floppy image to perform some modifications before proceed with Phase 3.
```
$ ./qemu-system-i386 -L pc-bios \
    -rtc base=localtime -no-hpet -no-reboot \
    -drive file=../vmimgs/w98.qcw \
    -device VGA \
    -M pc -device ac97 \
    -netdev user,id=net0 -device rtl8139,netdev=net0 \
    -drive if=floppy,file=fd.img -boot a
```
Disable Windows 98 startup logo by editing `C:\MSDOS.SYS` and placing `"Logo=0"` under `[Options]`. Disable Windows 98 shutdown logo by removing or renaming `LOGOW.SYS` in `C:\WINDOWS`.

7. Boot up QEMU as in `step #5` to proceed with Phase 3 and "Welcome to Windows 98" with 640x480 16-color desktop. Open "Device Manager" and update "Display adapters" with VBEMP display driver. Reboot after driver update finished.

8. Boot up QEMU as in `step #5` and we should be greeted with 640x480 256-color desktop. Switch to High Color (16 bit) desktop at 800x600 or 1024x768 and apply the settings **without** reboot. Proceed with the remaining drivers update for "PCI Multimedia controller" (AC97) and "PCI Ethernet controller" (Realtek RTL8139).

9. Boot up QEMU as in `step #5` with all drivers are installed and "Device Manager"  is clean. The next modifications workaround issues with VBEMP display driver of not supporting windowed "MS-DOS Prompt". Modify shortcut for "MS-DOS Prompt" to `"Full-screen"` and create `_DEFAULT.PIF` for MS-DOS programs to always launch in `"Full-screen"`. (https://jeffpar.github.io/kbarchive/kb/131/Q131877/). The final modification workaround KVM/WHPX reboot/shutdown, create a 2-line `_STARTUP.BAT` in `C:\WINDOWS`.
```
@echo off
exit
```
Create a PIF and add `_STARTUP.BAT` to `"Start->Programs->StartUp"` so that it is run automatically in `"Full-screen"` and `"Close on Exit"`.

10. Finally, we are in for **KVM/WHPX super-charged QEMU accelerated** Windows 98 SE virtual machine that brings modern CPU/GPU prowess to retro Windows games.
```
$ ./qemu-system-i386 -L pc-bios \
    -rtc base=localtime -no-hpet -no-reboot \
    -drive file=../vmimgs/w98.qcw \
    -device VGA \
    -M pc,accel=kvm -device ac97 \
    -netdev user,id=net0 -device rtl8139,netdev=net0 \
    -drive file=../vmimgs/mygames.qcw 
```
## What About Adlib/Gravis UltraSound/SoundBlaster 16 For DOS Games
For DOS games, DOSBox (https://www.dosbox.com) is a far superior emulator with superb audio hardware emulation. For QEMU, AC97 WDM drivers provide the best audio solution for Windows 98 SE from more faithfully emulated AC97 audio controller due to the openness of AC97 industry specification. For a few exceptions when DOS games may have worked out better from within Windows 98 SE due to protected-mode CD-ROM and MOUSE drivers, AC97 WDM drivers support SoundBlaster Pro compatibility for PCM playback. DOS games such as TES Redguard, Tomb Raider, PYL, Screamer 2 etc. are known to work well with WDM Audio SoundBlaster Pro emulation from within Windows 98 SE. 