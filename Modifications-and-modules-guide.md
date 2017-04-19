# Modifications and modules guide

Me and madmonkey spent many hours to create modules system for hakchi/hakchi2 (it's compatible) to let people create, share and download various modules with custom skins, music, emulators, etc. It's time to write some guide to let you create your own modifications!


## System overview

NES Classic Mini is just tiny computer with ARM processor and Linux. So basic Linux knowledge will be very handy if you want to create some advanced scripts. It's not so difficult, really.


### NAND flash, file system and directory structure

There are main directories of NES Classic Mini after system boot by default:

* **/bin**, **/sbin**, **/usr/bin**, **/usr/bin** - directories for executable files
* **/dev** - pseudofilesystem with devices
* **/etc** - configuration files and startup scripts
* **/etc/init.d** - startup scripts
* **/lib** - libraries and modules
* **/usr/share** - images, sounds and other resources
* **/usr/share/games/kachikachi/nes** - games are here
* **/var**, **/tmp**, **/run** - temporary files
* **/var/lib** - all writable user's data (savestates, settings)

The first thing you need to understand is mounting. In Linux devices and directories can be mounted on other directories. Internal flash memory devided into several partitions.
* ~20MB in size, factory preprogrammed, read-only, contains all system data and games, mounted on **/** (root)
* ~384MB in size, writable, contains all user settings and save-states, mounted on **/var/lib**

So you can't rewrite any original data but there are much free space for additional data. Also there are RAM disks:
* Mounted on **/tmp**
* Mounted on **/var**
* Mounted on **/run**
Data on this disks are stored in RAM only and erased every time you turn NES Mini off.