## Game Controllers Support
QEMU featuring **`qemu-3dfx`** brings yet another exciting feature for making retro gaming a pleasant experience on QEMU Virtual Machines -- **support for Game Controllers in Guest VMs with QEMU USB Gamepad**. QEMU USB Gamepad implementation is based on standard USB HID device model, fully leveraging QEMU existing USB framework that supports USB keyboard, mouse and tablet devices. It is completely orthogonal to Host side implementation that manages the physical Joysticks/Game controllers. The Host side implementation simply let SDL2 deal with the real Joysticks/Game controllers that provides cross-platform compatibility among Linux, Windows and macOS. The result is the event-driven, near real-time input latency USB Gamepad that works for any Guest VMs that support the USB HID device class. Good Old Windows Games, especially in sim-racing and space/flight-sim genres, played on QEMU Virtual Machines will enjoy the premium Game Experience of analog precisions from assorted flight sticks, steering wheels, pedals & rudders.

While QEMU on Linux can always do USB host device pass-through with `libusb`, it requires one-time system modification with privilege escalation to udev rules for enabling user-mode accesses to specific or class of USB devices. Windows and macOS have never been so fortunate as `libusb/UsbDk` solutions are unreliable outside of Linux. The SDL2-based QEMU USB Gamepad is platform agnostic, never ever requires root privilege and works for any Joysticks/Game controllers that show up on SDL2 in all types of connectivity, wired, wireless or Bluetooth. Connecting Bluetooth Game controller is as simple as using the **Host OSes -- Linux, Windows & macOS --** respective **Bluetooth Settings** and QEMU & SDL2 will take care of setting up the rest for Guest VMs.

### QEMU USB Gamepad Specification
* Support up to 6-axis (X, Y, Z, Rz, Ry, Rx)
* Support up to 24-button
* POV Hat
* Remap of (Z, Rz) to inverse (Rx, Ry) for 4-axis Host Joystick/Game controller
* Remap of POV Hat as buttons

**Tested Game Controllers**
* Logitech F310/Dual Action (USB-wired)
* Logitech F510/Rumblepad (USB-wired)
* 8BitDo M30 Bluetooth gamepad (Bluetooth)
* 8BitDo SN30 Pro Bluetooth gamepad (Bluetooth)

### QEMU Command Line Options
```
-device usb-gamepad
```
Enable QEMU USB Gamepad. Mapped to SDL2 `Joystick #0`.
```
-device usb-gamepad -device usb-gamepad,index=1
```
Enable two instances of QEMU USB Gamepad. Mapped to SDL2 `Joystick #0 and #1`.
```
-device usb-gamepad,split=1 -device usb-gamepad,index=1
```
Enable two instances of QEMU USB Gamepad by splitting SDL2 `Joystick #0`. Splitting SDL2 `Joystick #0` requires Host Joystick/Game controller with at least 4-axis and POV Hat. The 2nd pair of axes, typically the right-thumb stick, and POV Hat are remapped into next QEMU Gamepad instance as standard (X, Y) 2-axis, 4-button joystick.

**On-the-Fly Host Joysticks/Game Controllers Hot-plug & Removal**

At any given time, Host Joysticks/Game Controllers can be hot-plugged or removed to change to different Game controllers to suit the gaming needs without rebooting QEMU Virtual Machine. For instance, one would prefer D-pad based Game controllers for action arcade-style games and dual analog thumb-sticks Game controllers for sports racing. Simply by disconnecting one Game controller for another, SDL2 will make the new one `Joystick #0`. No changes are necessary on Guest VMs.

### Bonus: NO Joystick
Without any Host Joystick/Game controller, QEMU USB Gamepad implementation offers a bonus feature to use **MOUSE** relative motions as Joystick and the buttons mapped accordingly. This feature is enabled with the following QEMU command line options. Whenever a real Joystick/Game controller showed up on SDL2, it will be disabled automatically, and re-enabled again with real Joystick/Game controller disconnected.
```
-display sdl,mraxes=1 -device usb-gamepad
```
Such idea was indeed out of the Game Preservation efforts that **`qemu-3dfx`** put up for perfect restoration of LucasArts STAR WARS: X-Wing 95, TIE Fighter 95 and X-Wing vs TIE Fighter 95 with mandatory joystick requirement to play the games. Check out the [YouTube video](https://www.youtube.com/watch?v=MiF8CcU34kc). For STAR WARS fans who enjoyed the games with keyboard & mouse mechanics in the DOS series, QEMU featuring **`qemu-3dfx`** is capable of replicating the same Game Experience with the much improved 3D accelerated graphics Windows versions of the games, ***ALL in pristine condition***.

## QEMU USB Gamepad In a Nutshell
QEMU USB Gamepad is simple, efficient and truly usable for enjoying any Good Old Windows games with Game controllers. It is far superior to any poll-based game port emulation for joysticks. It can even be used for DOS games that work under Windows 98/98SE/ME VMs. Windows 98/98SE/ME DOS sessions automatically get virtualized game port emulation by the OS for attached USB Game controller as standard (X, Y) 2-axis, 4-button joystick.
```
1. Simple, Direct mapping

 / Host OS                         /          / Guest OS (USB HID)   /
/---------------------------------/          /----------------------/
| SDL2 Joystick                   |          | QEMU Gamepad         |
|    [ #0 joystick ]              |          | [ index #0 Gamepad ] |
|    [ #1 joystick ]              |=========>| [ index #1 Gamepad ] |
|    [ ...                        |          | [ index ...        ] |
|    [ #3 joystick ]              |/         | [ index #3 Gamepad ] |/
/---------------------------------/          /----------------------/


2. Split, Dual-controller mapping

 / Host OS                   /               / Guest OS (USB HID)    /
/---------------------------/               /-----------------------/
| SDL2 Joystick             |               | QEMU Gamepad          |
|                           /               /                       |
| [ #0 joystick ] =================+==========>[ index #0 Gamepad ] |
|                           /      |        /                       |
|                           |      +==========>[ index #1 Gamepad ] |
|                           |/              /                       |/
/---------------------------/               /-----------------------/


3. NO Joystick

 / Host OS                   /               / Guest OS (USB HID)    /
/---------------------------/               /-----------------------/
| SDL_MOUSEMOTION           |               | QEMU Gamepad          |
| ( xrel, yrel )            |======\        /                       |
| SDL_MOUSEBUTTONDOWN       |       \=========>[ index #0 Gamepad ] |
| SDL_MOUSEBUTTONUP         |/              /                       |/
/---------------------------/               /-----------------------/

```
## Reference
https://www.reddit.com/r/VFIO/comments/878ymp/gamepad_support_in_qemu/
https://www.psdevwiki.com/ps4/DS4-USB#HID_Report_Descriptor
