# Modifications and modules guide

Me and madmonkey spent many hours to create modules system for hakchi/hakchi2 (it's compatible) to let people create, share and download various modules for NES Classic Mini with custom skins, music, emulators, etc. It's time to write some guide to let you create your own modifications! I'll show you some examples below. So if you can't understand a word, please read until examples section.


## Tools

At first time, we did not have feedback from NES Mini over USB connection, so we used UART cable which was soldered inside NES Mini. So it was very difficult to to create some mods without this cable and soldering skills. But I discovered method which allows to access NES Mini's file system and command line using only USB connection. I called it "clovershell" and it's built in hakchi2 since version 2.14. So you don't need any extra hardware to explore NES Mini. Yes, UART cable still recommended but you really need only this software now:
* Telnet client - software to access NES Mini's command line, bundled with Windows "telnet.exe" is fine but "putty" is recommended.
* FTP client - software to transfer files, bundled with Windows will work but you can use any FTP client.


## System overview

NES Classic Mini is just a tiny computer with ARM processor and Linux. So basic Linux knowledge will be very handy if you want to create some advanced scripts. It's not so difficult, really.


### NAND flash, file system and directory structure

There are main directories of NES Classic Mini after system boot by default:

* **/bin**, **/sbin**, **/usr/bin**, **/usr/bin** - directories for executable files
* **/dev** - pseudofilesystem with devices
* **/etc** - configuration files and startup scripts
* **/etc/init.d** - startup scripts
* **/lib** - libraries and modules
* **/usr/share** - images, sounds and other resources
* **/usr/share/games/nes/kachikachi** - games are here
* **/var**, **/tmp**, **/run** - temporary files
* **/var/lib** - all writable user's data (savestates, settings)

The first thing you need to understand is mounting. In Linux devices and directories can be mounted on other directories. Internal flash memory devided into several partitions.
* ~**20MB** in size, factory pre-programmed, **read-only**, contains all system data and games, mounted on **/** (root)
* ~**384MB** in size, **writable**, contains all user settings and save-states, mounted on **/var/lib**

So you can't rewrite any original data but there are much free space for additional data. Also there are RAM disks:
* Mounted on **/tmp**
* Mounted on **/var**
* Mounted on **/run**
Data on this disks are stored in RAM only and erased every time you turn NES Mini off.

Note that the directories are nested inside each other. It means that:
* **/** - points to root of read-only, factory pre-programmed partition
* **/etc** - points to "**/etc**" folder on read-only, factory pre-programmed partition
* **/usr/share/games** - points to "**/usr/share/games**" folder on read-only, factory pre-programmed partition
* **/tmp** - points to other writable RAM disk
* **/var/lib** - points to writable partition on NAND flash memory

It can be difficult to understand for Windows users but you'll get it.


### File system when hakchi installed

So, we can't edit any internal files or add some games to **/usr/share/games** directory, it's read-only file system. But actually we don't need to do it. We can create any files/directories into "**/var/lib" and overmount them over the original files/directories. So when hakchi during installation doing the following:
* Creates directory "**/var/lib/hakchi**" to store all additional data, it remains after system shutdown
* Creates directory "**/var/lib/hakchi/rootfs**" to store all files/directories which need to replace
* Creates directory "**/var/lib/hakchi/rootfs/usr/share/games/nes/kachikachi**"
* Copies all games from "**/usr/share/games/nes/kachikachi**" to "**/var/lib/hakchi/rootfs/usr/share/games/nes/kachikachi**"
* Overmounts "**/var/lib/hakchi/rootfs/usr/share/games/nes/kachikachi**" over "**/usr/share/games/nes/kachikachi**", so it's writable now

He is doing the same with "**/etc**" and "**/bin**", after this hakchi installs additional configs and scripts there. So we can write to "**/etc**" and "**/bin**" while original files will be untouched. It's very important. You can delete *all* data you want but your NES Mini will not be bricked since original files are safe.

So, why we need to install custom kernel? Actually it has only one important modification: When NES Mini boots up it executes "**/etc/preinit**" (or "**/var/lib/hakchi/rootfs/etc/preinit**", it's *the same* file) script on the very early boot stage. This script, in turn, executes files from "**/etc/preinit.d**" directory. So we can store our overmounting scripts there.

Let's sum everything up. To edit any file/directory on read-only file system you need:
* Create copy of this file/directory into "**/var/lib/hakchi/rootfs/*your_directory***"
* Create script file into **/etc/preinit.d**" which overmounts "**/var/lib/hakchi/rootfs/*your_directory***" on "**/*your_directory***"
* After reboot edit this file/directory without any problems

It's better to understand on practice, so keep reading.


## How to create module correctly

### Built in functions of hakchi

There are some useful functions defined into ""**/etc/preinit.d/b0010_functions**" script. You can use them into your preinit scripts since "b0010_functions" executed before.
* **overmount <*file/directory*>** - overmounts file directory, example: "**overmount /usr/share/games/nes/kachikachi**" overmounts "/var/lib/hakchi/usr/share/games/nes/kachikachi" on "overmount /usr/share/games/nes/kachikachi" making it writable, it can be used with two arguments too to set custom destination mount point
* **copy <*path_a*> <*path_b*>** - copies file/directory using rsync (recommended to use instead of **cp**)
* **copy_mask <*path_a*> <*path_b*>** - same as **copy** but can be used with mask, this function is unsafe, avoid spaces in filenames!

Please read ""**/etc/preinit.d/b0010_functions**" to understand other functions. Also there are useful pre-defined variables:
* **modname**=hakchi
* **modpath**=/var/lib/hakchi (but really it's "*/newroot/var/lib/hakchi*" on pre-init stage)
* **installpath**=$**mountpoint**/var/lib/$**modname**
* **firmwarepath**=$**installpath**/firmware
* **rootfs**=$**installpath**/rootfs
* **preinit**=$**rootfs**/etc/preinit
* **preinitpath**=$**preinit**.d
* **gamepath**=/usr/share/games/nes/kachikachi
* **temppath**=/tmp


### Structure of .hmod files

Modules for hakchi/hakchi2 should be distributed as files with "**.hmod**" extension. Actually it's just renamed .tar.gz archive, so you can rename any .hmod file, unpack and inspect how it's made. There are several such files into "**mods\hmods**" directory of hakchi2 (it's a pre-installed mods: controller hack, font hack and clovershell daemon). Also hakchi2 can install unpacked hmods if it's a directory, you can find one such example into "**user_mods\music_hack.hmod**", this mod replaces menu music (with silence by default but you can change wav file with your favorite music track).

So, what .hmod archive *can* contain:
* First of all, any files which you need to install on NES Mini, it's recommended keep directory structure
* Optional "**install**" script
* Optional "**uninstall** script
* Optional "**readme.txt**" file with some description/info

What happens during module installation:
* If "**install**" file exists, it will be executed, it can contain any hakchi functions and variables from above
* If "**install**" returned non-zero value or it not exists, all files will be copied to "**/var/lib/hakchi/rootfs**" directory (excluding "install", "uninstall" and readme files)
* If "**uninstall**" file exists it will be stored into "**/var/lib/hakchi/hmod**" directory
* If "**uninstall**" file does not exists, it will be created automatically based on installed files list

What happens during module uninstallation:
* Corresponding "uninstall" file will be executed and removed


## Examples

todo