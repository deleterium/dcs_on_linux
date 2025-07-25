# DCS on Linux

DCS World can run on linux through Wine and Proton, though it does take some
work to get running. The game has two distribution methods: standalone and
Steam. Both have worked successfully, though often one will be broken and the
other work; if one fails, it can be a good idea to try the other.

If you plan to use standalone version, jump to https://github.com/budderpard/DCS_Standalone_on_linux repository.
Although many problems are common in both type of installation, this repo will focus on Steam version.

The game also has other versions "closed alpha" or "closed beta" access.
This will not be explained here.

Thanks to everyone who has helped getting the game running and debugging issues
in the [proton issue
tracker](https://github.com/ValveSoftware/Proton/issues/1722). Unfortunately,
workarounds easily get buried there, so I decided to create this document with
known, up-to-date methods for getting things to work.

Outside the proton thread, additional credit goes to @akp for the initial revision
of the Opentrack instructions, and @bradley-r for the Linuxtrack, Scratchpad and V4L2 info.

To chat about DCS World on Linux there is a Matrix chat available:
* https://matrix.to/#/#dcs-on-linux:matrix.org

## Contents

   * [Installation](#installation)
   * [Bugs and Fixes](#known-issues-and-fixes)
   * [Outdated fixes](#outdated-fixes)
   * [Additional Software](#other-software)

## Installation

* Install the game.
* Set steam compatibility options to 'Proton 9' or above.
* Proton tricks should be applied. Install `protontricks` from your linux repository or via flatpak.
* If you install from distro repo, the `<tricks_command>` will be `protontricks`
* If installed with flatpak, the `<tricks_command>` will be `flatpak run com.github.Matoking.protontricks`.
* Start the game once first to create the prefix. It will not start: probably hang or show a black window, or entire black screen. Finalize the game in steam.
* Install additional protontricks packages via command line: `<tricks_command> 223750 vcrun2019 corefonts xact d3dcompiler_47` or open protontricks app and install those packages in the GUI.
* In steam command line, set the launch options: `WINEDLLOVERRIDES='wbemprox=n' %command% --no-launcher`
* Start the game again. If there are problemsn, check #known-issues-and-fixes .

## Known issues and fixes

### Game files (important to read!)

Before advancing to more fixes, locate the game files. They will be needed later on.

- `$INSTALL_DIR`: the location in program files where the game is installed.
  `/home/youruser/.local/share/Steam/steamapps/common/DCSWorld`
- `$CONFIG_DIR`: the place where user config stuff is stored
  `/home/youruser/.local/share/Steam/steamapps/compatdata/223750/pfx/drive_c/users/steamuser/Saved Games/DCS/`.
- `$WINDOWS_DIR`: the place where wine is installed
  `/home/youruser/.local/share/Steam/steamapps/compatdata/223750/pfx/drive_c/windows`.
- `$LOG_DIR`: the game log file `$CONFIG_DIR/Logs/`.
- If steam is installed via flatpak, these folders will be different. Instead of `/home/youruser/`, use `/home/youruser/.var/app/com.valvesoftware.Steam/` (for user installation) or `/var/lib/flatpak/app/com.valvesoftware.Steam/` (for system installation)

### Log file

If things go wrong, the primary thing to look for is the game log - 
`$LOG_DIR/dcs.log`.
After crashes, the crash reporter will spam a bit about various DLLs being used
recently, and just before that, the cause of the crash should be visible.

Sometimes crashes happen before the game gets far enough to create a log file.
Then your best bet is to read the Proton output. In both Lutris and Steam, you can easily get
this by starting them from a terminal.

If you can't find an issue, or have found a solution for one, please discuss it in
the [proton issue](https://github.com/ValveSoftware/Proton/issues/1722).


### Fixing Steam version permanent crashing

If your game crashes in the Steam version, it will permanently fail to start
after that. To fix that: remove `$WINDOWS_DIR/system32/lsteamclient.dll`
which was created in the crash, and the game should start back up fine.

### Avoiding launcher window - black screen at startup
Starting from DCS 2.9.6.57650 a launcher window was added. Use steam command line option `--no-launcher` (already in this tutorial). But if even so, if the game is stuck in a wrong resolution of game options, try to reset confiuration:
1) Copy the simple template file `options.lua` in this repository to your folder
`$CONFIG_DIR/Config/options.lua`
2) Launch the game. It shall open in a small window. Go the the options (gear icon
in top) and select manually your screen resolution and maybe try other configurations.
3) If the resolution (or maybe some config) is not supported and the game does not launch,
use the simple template file `options.lua` again.

### DLL Overrides

Probably you need
to add a few "dll overrides" for the game to work. `wbemprox` was already added in instalation tutorial. Sometimes `msdmo` also need to be overridden.

To add both overrides, update your command line launcher options with `WINEDLLOVERRIDES='wbemprox=n;msdmo=n'`

### Black Screen On Game Startup

You should be able to log in, but once the game starts you will see a black screen. To fix this, create a symlink from
`$INSTALL_DIR/bin/webrtc_plugin.dll` to `$INSTALL_dir/webrtc_plugin.dll`.

The game should now start.

You may also see a crash when loading a mission. This might be caused by a
Arial missing font which can not be distributed with Wine.

### fx_5_0 error shaders not compiling
This is usually caused by d3dcompiler being missing or obsolete.
Make sure you have d3dcompiler_47 installed.
If it was installed much time ago, maybe you need to update it.
To force update, use protontricks command `protontricks 223750 -f d3dcompiler_47` or if you have protontricks as flatpak `flatpak run com.github.Matoking.protontricks 223750 -f d3dcompiler_47`.

### White smoke and some other particles renders weirdly

This is caused by DXVK. Remember that there is a translation between DX11 (used by the game) and Linux Vulkan API. Check "Using WineD3D (OpenGL) instead of Vulkan".

### Game is slow after looking at F10 map

F10 map causes a lot of textures to be loaded into memory, sometimes overflowing VRAM and even computer RAM. Sometimes it returns to normal by itself, other times you switch between aplications (Alt+Tab), or maybe if you look up at the sky. Try to lower graphics options to be less affected, of avoid DXVK entirely. 

### Using WineD3D (OpenGL) instead of Vulkan

It is possible that the game run faster in OpenGL than in Vulkan.
Wine has internal transalation from DX to OpenGL, called WineD3D.
While the default option is to use DXVL instead of WineD3D, Wine D3D option may be tested for old graphic cards or if you experience problems like stutters or low performance in multiplayer games.
Add `PROTON_USE_WINED3D=1` at the beginning of launcher options.
This solves the puffy contrails, but some textures can be weird, like some "burnt grass" or "golden grass" in Caucasus map.
Otherwise, it seems to be very nice in Syria and Persian Gulf maps, where the 'F10 slowness' are frequent using default DXVK!

### F16 RWR shows a opaque square on the RWR over the priority contact

This issue occurs because some textures fail to load for an unknown reason. The
fix is simple: open the file
`${INSTALL_DIR}/Mods/aircraft/F-16C/Cockpit/IndicationResources/RWR/indication_RWR.tga`
with an image editor (GIMP or Krita have been used successfully), then just
re-export the file. The RWR should now render correctly.

### Missing multiplayer server list

For a few 2.5.6 versions, the server browser did not work, and neither did
directly connecting to servers using connect by IP. However, there is a
workaround.

Edit `$INSTALL_DIR/MissionEditor/modules/mul_password.lua`. Find the function `onChange_btnOk` and add the
line `onlyConnect = true` to the start of the function like so.

```lua
function onChange_btnOk()  
    onlyConnect = true -- This line was added
	if onlyConnect == true then
	-- ...
end
```

Now you should be able to use the connect by IP button to join servers, but the
server list is still broken. Luckily, a server list is available if you log in
on https://www.digitalcombatsimulator.com/, and from there you can get the IP
of servers.

### Crash on F10

For many DCS versions and/or Wine versions, if you press F10 (the default
binding to bring up the map) the game will crash ("permanently" on steam, see
fixing steam permanent crashing (above) for a fix).  Luckily, the problem is with the
F10 key itself, not the map, so rebind it to something else you see fit. The
same applies for the communication menu.

### Module disabled by user

You probably won't run into this, but if you do, there is a fix.

One of your modules is missing, it is not shown in the list at the bottom of
the main menu, and you can't use it. On standalone, check if it is enabled in
the module manager. On steam however, things are a bit more tricky. If you
copied your configs between standalone and steam, module manager disabled mods
will be disabled in steam too. This information is stored in
`$CONFIG_DIR/enabled.lua` or something similar. Remove it to fix the issues.

### Control issues

Due to the various differences between distributions, issues with (HOTAS) controls can be hard to nail down,
especially when Wine is involved - adding another layer of potentional problems. Users experiencing issues with
controllers are advised to read through [the information here](https://github.com/bradley-r/Linux-Controller-Fixes/).

### SteamVR

The game seems to work fine with steamVR. This is only possible in the steam
version, and seems to currently only work in proton 6.3.8 (possibly in future
proton versions, but not GE or TKG)
Currently the game only runs in proton 9+, so if you have updated info about VR, please contribute.

## Outdated fixes

Fixes from previous versions will be stored here. Maybe they can help someone but probably not needed anymore.

## Other software

While not included in DCS, here are some resources for getting external
software often used with the game.

### SRS

[SRS](http://dcssimpleradio.com/) is used by a lot of multiplayer servers. It
too works with some tweaks.

Install the game plugin by following the instructions in the SRS readme.

*Note* As of SRS 19.0.1, this method no longer works. As a replacement, I have
a custom SRS client that *kind of* works here: https://gitlab.com/TheZoq2/srsrs.

It's easiest to run SRS in its own prefix. Create one, and then run `winetricks
dotnet452 win10` in that prefix. Now you can start `SR-ClientRadio.exe` from
the downloaded files.

Credit: https://github.com/ciribob/DCS-SimpleRadioStandalone/issues/409.

### DCS Scratchpad

For those who want to make use of the excellent [DCS-Scratchpad utility](https://github.com/rkusa/dcs-scratchpad), 
[follow the installation instructions](https://github.com/rkusa/dcs-scratchpad) as normal. 
The scratchpad should appear in game, but when typing with it's '*window*' focused, nothing will appear. 
This is a font issue - by default, DCS-Scratchpad uses `CONSOLA.TTF`, a font not installed with Wine. 
Either install it with Winetricks, or edit line 172 in `Scripts/Scratchpad/ScratchpadWindow.dlg` to 
an installed font of your choosing, such as `CALIBRI.TTF`. Text should now appear in the scratchpad window. 

### Headtracking via Opentrack

**Users of custom/Lutris/non-system Wine versions, take note: 
Due to [issues with libwine](https://github.com/opentrack/opentrack/issues/1236), Opentrack does not support prefixes using versions of Wine
different to that of the system (and thus the one Opentrack recognises), making usage of
custom/performance-enhanced Wine versions impossible alongside it. Either run DCS with your
system Wine, or try Linuxtrack instead. Proton is unaffected.**

[Opentrack](https://github.com/opentrack/opentrack) can emulate a gamepad which is read and can be mapped to the
corresponding controls in the game. This should work out of the box, simply
select `lubudev joystick receiver` as the output in opentrack.

Opentrack can work out of the box with `libevdev joystick output`, however this requires you to bind headtracking
for every aircraft (and doesn't play well with Il-2 BoX or Falcon BMS.)

A better option, then, is to enable Wine (Freetrack and NPclient) output instead of joystick axis output. This allows
the use of headtracking across all aircraft (DCS interprets the input as an actual headtracker rather than joystick), and
should play well with other titles such as IL-2 BoX and Falcon BMS.

If you are building Opentrack from the [AUR](https://aur.archlinux.org/packages/opentrack/), you can modify the PKGBUILD.
Replace line 34: `-DSDK_WINE_PREFIX=/ \` with `-DSDK_WINE=ON/ \`. *This package now seems to include Wine output by default.*

Otherwise, you can clone the source code and follow [these
instructions](https://github.com/opentrack/opentrack/wiki/Building-on-Linux).
**After you cd into the directory, run `ccmake .`, press c to configure, turn
ON SDK_WINE, c to configure and g to generate.**

DCS still requires `HeadTracker.dll` in the `bin` directory for opentrack to
function.  Download Eagle Dynamics API interface DLL (64-bit) from
http://facetracknoir.sourceforge.net/information_links/download.htm.

You must open opentrack and start tracking before you launch DCS. Be sure to
point the output to the correct wine/proton prefix. In addition, you'll need to
launch DCS with WINEESYNC=1 or WINEFSYNC=1 if you enable those in the wine
output settings.

![Opentrack Wine Implementation](images/opentrackwine.png)

Context: https://github.com/ValveSoftware/Proton/issues/1722#issuecomment-749061952

### Headtracking via Linuxtrack

In the case the Opentrack fails to work (as outlined above, it cannot support custom Wine versions
such as those offered by Lutris) or you wish to try an alternative, [Linuxtrack](https://github.com/uglyDwarf/linuxtrack/) 
offers similar functionality. 

Begin by installing the [universal Linux package](https://github.com/uglyDwarf/linuxtrack/wiki/universal-Linuxtrack-package).
Once complete, run `ltr-gui` and under the 'Misc' tab, select (re)install TrackIR firmware.) Linuxtrack
will attempt to complete this task for you, but, at time of writing, the TrackIR download links have changed, so
you may need to do this manually. Download the latest TrackIR firmware, install it to your default (or
temporary) prefix, then select 'Extract from unpacked'.

![Linuxtrack Firmware Extractor](images/linuxtrackextractor.png)

Navigate to the prefix you used, and select the TrackIR 5 folder under `/drive_c/Program Files (x86)/NaturalPoint/`. 
Once done, you will be prompted to install the Wine-side components; select the prefix DCS is installed under
(only standalone has been tested.) `ltr-gui` can now be closed, and provided Linuxtrack is running
(and has been configured), use the `FreeTrackTester.exe` present in the second prefix `/drive_c/Program Files 
(x86)/Linuxtrack/`. You should see the values changing, and thus controlling the view in-game.

![Linuxtrack/Freetrack Test Dialogue](images/linuxtrackfreetracktest.png)

Note that `HeadTracker.dll` need not be present as Linuxtrack replicates TrackIR directly (in the case of DCS, at least.)

### A note on headtracking

This only applies if an IR-modified camera is used as input to your headtracking program of choice, but can be very useful if 
so. Video4Linux(2) "*is a collection of device drivers and an API for supporting realtime video capture on Linux systems*" 
and thus is the utility used by Opentrack and Linuxtrack to address IR cameras - often the venerable PS3Eye. V4L2 handles the 
configuration of attached cameras, and so is the utility to use to change any settings.

![QV4L2 Test Dialogue](images/V4L2test.png)

For IR-modded cameras, the settings of most significance are gain, auto-exposure and (automatic) white balance. The PS3Eye, 
not having any physical controls aside from an FOV setting, can be configured using the V4L2 test utility ([`v4l-utils`](https://pkgs.org/download/v4l-utils)), 
however changes made here do not persist across reboots. Opentrack seems to have this 
utility built-in, but for Linuxtrack users or those needing to change camera settings system-wide, there is a solution:

   * Ensure `v4l-utils` is installed.
      * Video4Linux, providing core functionality for attached video devices, is available on all mainline distributions. 	
      * Find `v4l-utils` for your distribution [here](https://pkgs.org/download/v4l-utils).
   * Open the V4L2 test utility, and select the correct camera if there are multiple connected.
      * Run `qv4l2` at the command line to launch the utility.
      * If multiple cameras are connected, look in `/dev/` for `videoX` devices.
   * Configure the settings to a suitable point. Of interest here are any automatic features that may interfere with tracking.
      * Settings for an IR-modded PS3Eye are included below.
      * As a general rule, automatic gain, white balance and (possibly) exposure should be disabled.
   * Once done, return to a command line and execute `v4l2-ctl --all`. This lists all the configurable values of the camera.
      * Framerate and pixel/capture format will be listed, but these cannot be changed via this method for the PS3Eye.
   * Using this information, make a `.sh` file with a relevant name (such as `IRcamfix.sh`) with contents in the format:
      > #!/bin/bash
      > --set-ctrl=brightness=0 \
      > --set-ctrl=contrast=32 \
      > --set-ctrl=saturation=0 \
      > --set-ctrl=gain_automatic=0 \
      > --set-ctrl=gain=0 \
      > --set-ctrl=power_line_frequency=0 \
      > --set-ctrl=sharpness=0 \
      > --set-ctrl=white_balance_automatic=0
      * This accomplishes the same thing as changing these values through the GUI but allows it to be done automatically.
      * These settings have been found to work well with a PS3Eye camera, but may need adjusting depending on use conditions.
   * Save this file, mark it as executable and add it to an autorun utility such as Plasma's autostart or Lutris' pre-launch script.
      * This will apply these changes when the start condition is triggered by their respective programs.

With this done, the camera will have these changes applied automatically, allowing immediate use of headtracking without the need to preemptively tinker with a GUI before every flight.
