# Modifications and modules guide

Me and madmonkey spent many hours to create modules system for hakchi/hakchi2 (it's compatible) to let people create, share and download various modules for NES Classic Mini with custom skins, music, emulators, etc. It's time to write some guide to let you create your own modifications! I'll show you some examples below. So if you can't understand a word, please read until examples section.


## Tools

At first time, we did not have feedback from NES Mini over USB connection, so we used UART cable which was soldered inside NES Mini:

![UART pinout](http://clusterrr.com/dump/nes_mini_uart.jpg)

So it was very difficult to to create some mods without this cable and soldering skills. But I discovered method which allows to access NES Mini's file system and command line using only USB connection. I called it "clovershell" and it's built in hakchi2 since version 2.14. So you don't need any extra hardware to explore NES Mini. Yes, UART cable still recommended but everything can be done without soldering now, you need only software.

All required software is built in hakchi2 since version 2.16, check "Tools" menu:

![hakchi2 tools](http://clusterrr.com/dump/hakchi2_servers.png)

But there are some extra tools recommended:
* [clovershell client](https://github.com/ClusterM/clovershell-client/releases) - my tool which allows to execute shell commands on NES Mini and push/pull files via USB. 
* Telnet client - software to access NES Mini's command line, bundled with Windows "telnet.exe" is fine but [putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) is recommended.
* FTP client - software to transfer files, bundled with Windows will work but it's better to use something like [FileZilla Client](https://filezilla-project.org/) or [FAR](http://farmanager.com/index.php?l=en). FAR is my favorite one.
* It's good idea to install also [MSYS](http://www.mingw.org/wiki/MSYS) - a collection of GNU utilities for Windows.


## System overview

NES Classic Mini is just a tiny computer with ARM processor and Linux. So basic Linux knowledge will be very handy if you want to create some advanced scripts. It's not so difficult, really.


### NAND flash, file system and directory structure

There are directories of NES Classic Mini after system boot by default (without hakchi installed):

![Original directory structure](http://clusterrr.com/dump/nes_mini_filesystem_original.png)

The first thing you need to understand is mounting. In Linux devices and directories can be mounted on other directories. Internal flash memory devided into several partitions.
* ~**20MB** in size, factory pre-programmed, **read-only**, contains all system data and games, mounted on **/** (root)
* ~**384MB** in size, **writable**, contains all user settings and save-states, mounted on **/var/lib**

So you can't rewrite any original data but there are much free space for additional data. Also there are RAM disks:
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

Lets sum everything up. To edit any file/directory on read-only file system you need:
* Create copy of this file/directory into "**/var/lib/hakchi/rootfs/*your_directory***"
* Create script file into **/etc/preinit.d**" which overmounts "**/var/lib/hakchi/rootfs/*your_directory***" on "**/*your_directory***"
* After reboot edit this file/directory without any problems

I'll try to visualize it:

![Modified directory structure](http://clusterrr.com/dump/nes_mini_filesystem_hakchi.png)

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
* Corresponding "**uninstall**" file will be executed and removed


## Examples

### Simple file replacement - custom skin mod

Lets start with something very simple. Do you want to customize your NES Mini and create your own theme for it? So lets locate it inside NES Classic Mini. At first, enable FTP server:

![FTP](http://clusterrr.com/dump/hakchi2_ftp.png)

And login into it using "**127.0.0.1**" as address, **1021** as port "**root**" as login and "**clover**" as password.

It's easy to realize that all images stored into **/usr/share/clover-ui/resources/sprites/nes.png** file:

![Skin](http://clusterrr.com/dump/nes_mini_skin.png)

So we can download this file and edit using your favorite graphic editor without any problems! But... we can't rewrite this file, it's stored on read-only file system inside NES Mini. But we can write files inside "/var/lib" directory, so lets save our modified file as ***/var/lib/hakchi/rootfs*/usr/share/clover-ui/resources/sprites/nes.png**:

![Skin](http://clusterrr.com/dump/nes_mini_skin2.png)

It saved successfully but it will not replace original **/usr/share/clover-ui/resources/sprites/nes.png** on it's own. We need to create pre-init script to overmount this file on early boot stage. Actually it's very easy to do. All pre-init scripts are stored into "**/etc/preinit.d**" (actually it's "**/var/lib/hakchi/rootfs/etc/preinit.d**" but it's overmounted onto "**/etc/preinit.d**", so who cares) directory. We need to create new file here:

![preinit.d](http://clusterrr.com/dump/nes_mini_preinit.png)

File must be named as **p*xxxx*_*name*** where "xxxx" is a hexadecimal digits and "name" is any text. You can choose any valid name but keep in mind that all pre-init scripts will be executed in alphabetical order. It can be important in some cases.

So lets create "**/etc/preinit.d/p8032_skin**" file with only one line:

    overmount /usr/share/clover-ui/resources/sprites/nes.png


![preinit.d](http://clusterrr.com/dump/nes_mini_preinit2.png)

That's all. Just reboot your NES Classic Mini and you will see your custom theme:

![Custom theme photo](http://clusterrr.com/dump/nes_mini_skin_photo.jpg)

But what if you want to share your mod? Open "**user_mods**" directory inside hakchi2's directory and create folder named "**my_skin.hmod**" here. You can choose any name but it must be with ".hmod" on the end.
Then just copy all files that you created inside NES Mini into this "**my_skin.hmod**" folder. You must keep all directory structure under "**/var/lib/hakchi/rootfs**", so it must contain "**\usr\share\clover-ui\resources\sprites\nes.png**" and "**etc\preinit.d\p8032_skin**" files.

![hmod directory](http://clusterrr.com/dump/hakchi2_skin_hmod.png)

Also it's good idea to create some "readme.txt" file inside "**my_skin.hmod**" folder with some mod description. We can also create "**install**" and "**uninstall**" files but this is not required for such simple mod, hakchi will create install and uninstall scenarios automatically.

You should be able to install/uninstall this mod using hakchi2 now:

![mod installer](http://clusterrr.com/dump/hakchi2_mod_installer.png)

But you should distribute your mods not as directories but as single ".hmod" files which are actually tar.gz archives. So lets create this archive file using tar util from MSYS collection:

![Packing hmod](http://clusterrr.com/dump/hakchi2_hmod_packed.png)

So we have "**awesome_skin.hmod**" file now which can be easily shared with other users. All they need to do is just drag-and-drop this file on hakchi2's window. And it's still compatible with original madmonkey's hakchi. But actually I'm not sure that it's totally legal to share skin files when they are based on original NES Mini's files.

You can use the same way to edit any other NES Mini's files: music, sounds, text, etc.


### Basic scripting - lets remove thumbnails

Many people asked me to remove thumbnails at the bottom of the screen. This feature was introduced in early versions but removed later. So lets create mod to do it!

Let's check files of NES Mini... There is "**/usr/share/clover-ui/resources/prefab/sys_game_thumbnail.scn**" file with some JSON data. And there is "enabled" boolean variable!

![sys_game_thumbnail.scn](http://clusterrr.com/dump/thumbnails_scn.png)

You already know what to do from previous example:
* Download "**sys_game_thumbnail.scn**" file to PC
* Edit it, change "enabled" from "true" to "false"
* Upload it to NES Mini as "***/var/lib/hakchi/rootfs*/usr/share/clover-ui/resources/prefab/sys_game_thumbnail.scn**"
* Create pre-init script to overmount this file

But we will not do it. We will use other method. Open "**user_mods**" directory inside hakchi2's directory, create folder named "**remove_thumbnails.hmod**" here and create file "**install**" inside it. It's our installation script which will be executed during mod installation process. Lets write it:

    # Lines started with "#" are ignored and can be used as comments
    # Next line defines "scnfile" variable with path to "sys_game_thumbnail.scn" file
    scnfile=/usr/share/clover-ui/resources/prefab/sys_game_thumbnail.scn
    # This line defines "preinitfile" variable with pre-init file name
    preinitfile=p81a8_hide_thumnbnails
    # "restore" is hakchi function which copies original file to corresponding path in /var/lib/hakchi/rootfs
    restore $scnfile
    # sed is GNU util to modify file, this command replaces "enabled:true" to "enabled:false"
    # Please note that we need to edit $rootfs$scnfile (writable file), not a just $scnfile (original read-only file)
    sed -i -e 's/"enabled":true/"enabled":false/g' $rootfs$scnfile
    # Create pre-init script and echo "overmount" command to it
    echo "overmount $scnfile" > $preinitpath/$preinitfile
    # We should return 1 to prevent execution of automatic installer 
    return 1

That's all. But don't forget to create "uninstall" script too:

    # All we need is to delete created files, original file is safe and will be used again after reboot
    scnfile=/usr/share/clover-ui/resources/prefab/sys_game_thumbnail.scn
    preinitfile=p81a8_hide_thumnbnails
    rm -f $preinitfile
    rm -f $rootfs$scnfile

Lets install our mod...

![mod installer](http://clusterrr.com/dump/hakchi2_mod_installer2.png)

Oops...

![Oops](http://clusterrr.com/dump/nes_mini_thumbnails_oops.jpg)

Thumbnails are removed but cursor is still there. It's park of skin, so we can just make it's transparent, but it will conflict with previous skin mod. It's very important to prevent mods conflicts. 

There is other config file - "**/usr/share/clover-ui/resources/sprites/nes.json**", and it contains coordinates and properties for every sprite on the screen:

![nes.json](http://clusterrr.com/dump/nes_json.png)

So lets change our "**install**" script to replace "*"sourceSize":[12,8]*" on "*"sourceSize":[0,0]*":

    # Lines started with "#" are ignored and can be used as comments
    # Next line defines "scnfile" variable with path to "sys_game_thumbnail.scn" file
    scnfile=/usr/share/clover-ui/resources/prefab/sys_game_thumbnail.scn
    # Same with "nes.json" file
    nesjson=/usr/share/clover-ui/resources/sprites/nes.json
    # This line defines "preinitfile" variable with pre-init file name
    preinitfile=p81a8_hide_thumnbnails
    # "restore" is hakchi function which copies original file to corresponding path in /var/lib/hakchi/rootfs
    restore $scnfile
    restore $nesjson
    # sed is GNU util to modify file, this command replaces "enabled:true" to "enabled:false"
    # Please note that we need to edit $rootfs$scnfile (writable file), not a just $scnfile (original read-only file)
    sed -i -e 's/"enabled":true/"enabled":false/g' $rootfs$scnfile
    sed -i -e 's/"sourceSize":[12,8]/"sourceSize":[0,0]/g' $rootfs$nesjson
    # Create pre-init script and echo "overmount" command to it
    echo "overmount $scnfile" > $preinitpath/$preinitfile
    echo "overmount $nesjson" >> $preinitpath/$preinitfile
    # We should return 1 to prevent execution of automatic installer 
    return 1

Done! No more thumbnails and cursor at the bottom of the screen! I'll bundle this mod with hakchi v2.17, so you can check and edit it on your own.