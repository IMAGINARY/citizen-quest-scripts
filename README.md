# Citizen-Quest scripts

These scripts are used to setup a station serving the citizen-quest exhibit and displaying the main map.

The client stations are still subject to manual setup. One possible setup is documented [below](#client-stations).

## Installation

### Kiosk OS

The target system for running the script is Ubuntu 22.04 provided that the following settings have been applied during installation:

- Choose minimal installation
- Install third-party software for graphics and WiFi hardware... : ON
- Configure secure boot: ON. (set a password; it needs to be entered on first boot)
- Account settings:
    + User name: `kiosk`
- Reboot after installation is complete
- Enroll the secure boot keys using the password you entered during secure boot configuration
- Reboot into the system

#### Bootstrapping

The script `bin/bootstrap-os` will install required packages, the scripts in this repository as
well as most of the exhibits:

```
wget https://raw.githubusercontent.com/IMAGINARY/citizen-quest-scripts/main/bin/bootstrap-os
chmod +x bootstrap-os
sudo ./bootstrap-os
```

Some changes made by the bootstrapping script might only take effect after a reboot.

### Immutable file system

The bootstrapping script will ask if it should make the file system immutable after next reboot. This can be very useful to make the whole setup immune to crashes or other modifications of the system. So even if something goes severely wrong during operation, a reboot will usually solve all problems because it resets the system to a well defined state.

The immutable file systems is achieved via `overlayroot` which writes file modifications to a `tmpfs` in RAM instead of the local hard disk. Programs that write a lot of data might cause problems since available RAM might be limited.

Immutability can be toggled using the scripts `enable-overlayroot` and `disable-overlayroot`. Both require a reboot to take effect.

In immutable mode, changes to the local file system can be performed via the `overlayroot-chroot` command but this might have unintended consequences since not all system modifications (such as modifications to the bootloader or `systemd`) work well from within a chroot. So handle with care.

Note that only the root partition is made immutable. It is still possible to write to partitions such as the configuration partition mounted to `/cfg`. 

### Updating

Important note: The scripts in this repository are designed for initial setup of the system. Using them for installing updates may or may not work reliably and some scripts will reset parts of the station configuration along the way. Running the scripts from within `overlayroot-chroot` may render the system unusable or even prevent successful booting. You have been warned.

That said: You can either re-run `bootstrap-os` or reinstall/update individual exhibits via the
`install-*` scripts. Note that in most cases, `install-kiosk-scripts` must be executed first
to update the other installation scripts to the most recent version.

### Uninstalling

It is out of the scope of these scripts to provide facilities for uninstalling.
You have to check the installation scripts to figure out how to roll back an installation manually.

Doing so might also be helpful in cases where installations or updates fail. Cleaning previous installations might fix certain issues.

## Starting exhibits
Exhibit start script names are prefixed with `exhibt-` followed by an identifier of that exhibit. See `bin/launch` for a list of known exhibits.

## Restart exhibits after a crash
The script `repeat-exhibit` will restart the executable supplied as argument indefinitely, e.g.
```
repeat-exhibit exhibit-kiosk
```
will restart `exhibit-kiosk` if it exits for whatever reason.

## Stopping exhibits
All exhibits can be stopped via
```
stop-exhibits
```
This includes the `repeat-exhibit` script and any process whose name starts with `exhibit-`.

## Autostart

By default, a plain X11 session is started that first runs an init script for the exhibit and then runs the exhibit in a loop with mouse cursor hiding enabled.

The init script is supposed to run only once and it needs to exit once the init tasks are performed. Otherwise, script execution will block and no exhibit will be started.

The system wide default for init and exhibit scripts are defined in `/etc/default/kiosk`.
User defined values can be provided in the file `$XDG_CONFIG_HOME/kiosk/default` following the same format as `/etc/default/kiosk`.
If `$XDG_CONFIG_HOME` is not set, it's default value `~/.config` will be assumed.

## Additional notes on some of the scripts

### Switching to a regular desktop session

In the plain X11 session, there is not much the user can do besides using the running exhibit (which is intended). Basically, the only way to launch (graphical) applications is via one of the virtual terminals (e.g. <kbd>CTRL</kbd>+<kbd>ALT</kbd>+<kbd>F2</kbd>) or via `ssh`.

It is also possible to leave the plain X11 session and go back to the login screen were you can select another session (e.g. regular Gnome Desktop):
```
stop-kiosk-session
```
Note that the selected session will also be used after reboot if the filessystem isn't immutable. Hence, you should switch back to the plain X11 session once your work in the regular desktop session is done.

### Hide the cursor
The script `hidecursor` hides the cursor and starts another child process supplied as an argument.
When the child exits, the cursor will not be hidden anymore.

### Fake display resolution for testing
Sometimes the display used for development does not match the resolution used in production.
Assume your test display has a default resolution of `1920x1080` (FullHD), but in production, `3840x2160` (UHD) is used.
The command
```
fakeResolution 3840x2160
```
will allocate a `3840x2160` frame buffer and will scale it down to `1920x1080`.

If aspect ratios do not match, the image will be stretched.

## Client stations

### Hardware    

 - Apple iPad 7th generation (A2197) with charging adapter and Lightning cable
 - Lightning adapter for peripherals:
    - Two adapter solution:
        - Apple Lightning to USB 3 Camera Adapter (A1619)
        - tp-link USB 3.0 3-Port Hub & Gigabit Ethernet Adapter (UE330)
    - One adapter solution:
        - XINCENG Lighting to Ethernet Adapter, 3 in 1 RJ45 Ethernet LAN Wired Network Adapter with Charging Port and USB 3.0 Port Adapter (THT-016-N)
- EG STARTS USB Gamepad/Joystick Encoder (USB-001)
- Arcade joystick and buttons, e.g.:
    - IL Eurojoystick 2 Happ Arcade Joystick
    - 2x Gold Leaf Silent Short Arcade & Pinball Button
- Optional, but recommended: [Gamepad API activator](https://github.com/IMAGINARY/gamepad-api-activator)
- RJ45 Ethernet cable

### Software

- iPadOS 16.6
- [Kiosk Pro Plus](https://apps.apple.com/app/kiosk-pro-plus/id445701154)

#### Kiosk configuration

##### iPadOS
 - Set up [guided access](https://support.apple.com/en-my/guide/ipad/ipada16d1374/ipados)
 - Disable WiFi and Bluetooth
 - Disable auto-lock
 - Disable auto-brightness
 - Disable auto-updates

##### Kiosk Pro Plus
 - Set the content URL to `http://[server-address-or-ip]/player.html?p=[player-id]`
 - [Disable the navigation bar](https://support.kioskgroup.com/article/969-show-navigation-bar)
 - [Disable zoom](https://support.kioskgroup.com/article/985-zoom-on-standard-web-pages)
 - [Allow automatic focus](https://support.kioskgroup.com/article/1169-show-keyboard-automatically-on-focus-event)
 - [Disable idle timer](https://support.kioskgroup.com/article/991-idle-time-limit)
