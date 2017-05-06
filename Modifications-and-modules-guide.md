![For dummies](http://clusterrr.com/dump/hakchi_for_dummies3.jpg)

# Modifications and modules guide

Me and madmonkey have spent many hours creating the modules system for hakchi/hakchi2 (they are compatible) to let people create, share and download various modules for NES Classic Mini with custom skins, music, emulators, etc. It's time to write a guide to let you create your own modifications! I'll show you some examples below. So if you can't understand a word, please read to the examples section.


## Tools

At first, we didn't have response from NES Mini over USB connection, so we used a UART cable soldered inside the console:

![UART pinout](http://clusterrr.com/dump/nes_mini_uart.jpg)

It was very difficult to create mods without this cable and soldering skills. But I've discovered a method which allows accessing NES Mini's file system and command line using only USB connection. I called it "clovershell", and it's built into hakchi2 since version 2.14. So you don't need any extra hardware to explore NES Mini. Yes, a UART cable is still recommended. However, everything can be done without soldering now, you need only software.

All required software is built into hakchi2 since version 2.16, check "Tools" menu:

![hakchi2 tools](http://clusterrr.com/dump/hakchi2_servers.png)

Though, there are some extra tools recommended:
* [clovershell client](https://github.com/ClusterM/clovershell-client/releases) -- my tool which allows to execute shell commands on NES Mini and push/pull files via USB. 
* Telnet client -- software to access NES Mini's command line: Windows-bundled "telnet.exe" is fine, but [putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) is recommended.
* FTP client -- software to transfer files: the one which is bundled with Windows will work, but it's better to use something like [FileZilla Client](https://filezilla-project.org/) or [FAR](http://farmanager.com/index.php?l=en). FAR is my favorite one.
* It's a good idea to install also [MSYS](http://www.mingw.org/wiki/MSYS) -- a collection of GNU utilities for Windows.


## System overview

NES Classic Mini is just a tiny computer with an ARM processor and Linux. This is why basic Linux knowledge will be very handy if you want to create some advanced scripts. It's not so difficult, really.


### NAND flash, file system and directory structure

These are default directories of NES Classic Mini (after system boot, without hakchi installed):

![Original directory structure](http://clusterrr.com/dump/nes_mini_filesystem_original.png)

The first thing you need to understand is mounting. In Linux, devices and directories can be mounted on other directories. Internal flash memory is divided into several partitions.
* ~**20MB** in size, factory pre-programmed, **read-only**, contains all system data and games, mounted on **/** (root)
* ~**384MB** in size, **writable**, contains all user settings and save-states, mounted on **/var/lib**

So you can't rewrite any original data, but there is much free space for additional data.

Also, there are RAM disks mounted on **/var**,  **/run** and **/tmp**. The data on these disks is stored in RAM only and erased every time you turn NES Mini off.

Note that the directories are nested inside each other. It means that:
* **/** -- points to the root of the read-only, factory pre-programmed partition
* **/etc** -- points to "**/etc**" folder on the read-only, factory pre-programmed partition
* **/usr/share/games** -- points to "**/usr/share/games**" folder on the read-only, factory pre-programmed partition
* **/tmp** -- points to one of the aforementioned writable RAM disk
* **/var/lib** -- points to the writable partition on the NAND flash memory

It can be difficult to understand for Windows users, but you'll get it.


### File system when hakchi installed

So, we can't edit any internal files or add any games to **/usr/share/games** directory -- it's read-only file system. But actually we don't need to do it. We can create some files/directories in "**/var/lib**" and overmount the original files/directories with them. On installation hakchi does the following:
* Creates "**/var/lib/hakchi**" directory to store all additional data, it remains after system shutdown
* Creates "**/var/lib/hakchi/rootfs**" directory to store all the files/directories for replacing the original ones
* Creates "**/var/lib/hakchi/rootfs/usr/share/games/nes/kachikachi**" directory 
* Copies all games from "**/usr/share/games/nes/kachikachi**" to "**/var/lib/hakchi/rootfs/usr/share/games/nes/kachikachi**"
* Mounts "**/var/lib/hakchi/rootfs/usr/share/games/nes/kachikachi**" over "**/usr/share/games/nes/kachikachi**", so it's writable now

It performs the latter overmounting trick with "**/etc**" and "**/bin**" too, after which hakchi installs additional configs and scripts there. So we can write to "**/etc**" and "**/bin**" while the original files are untouched. It's very important. You can delete *any* data you want, but your NES Mini will not be bricked, since the original files are safe.

 Hakchi2 also installs a custom kernel. What for? Actually it has only one important modification: when NES Mini boots up, our kernel executes "**/etc/preinit**" (or "**/var/lib/hakchi/rootfs/etc/preinit**", it's *the same* file) script on the very early boot stage. This script, in turn, executes the files from "**/etc/preinit.d**" directory. So we can store our overmounting scripts there.

Let's sum everything up. To edit any file/directory on the read-only file system you need:
* Create a copy of this file/directory in "**/var/lib/hakchi/rootfs/*your_directory***"
* Create a script file in **/etc/preinit.d**", which overmounts "**/var/lib/hakchi/rootfs/*your_directory***" on "**/*your_directory***"
* After a reboot, edit this file/directory without any problems

I'll try to visualize it:

![Modified directory structure](http://clusterrr.com/dump/nes_mini_filesystem_hakchi3.png)

It's easier to understand in practice, so keep reading.


## How to create modules correctly

### Built-in functions of hakchi

There are some useful functions defined in "**/etc/preinit.d/b0010_functions**" script. You can use them in your preinit scripts, because "b0010_functions" script is executed earlier:
* **overmount <*file/directory*>** -- overmounts a file/directory, e.g. "**overmount /usr/share/games/nes/kachikachi**" overmounts "/var/lib/hakchi/usr/share/games/nes/kachikachi" on "/usr/share/games/nes/kachikachi" making it writable; use it with two arguments to set a custom destination mount point
* **copy <*path_a*> <*path_b*>** -- copies a file/directory using rsync (recommended to use instead of **cp**)
* **copy_mask <*path_a*> <*path_b*>** -- same as **copy**, but can be used with a mask. This function is unsafe, avoid spaces in filenames!

Please read "**/etc/preinit.d/b0010_functions**" to understand the other functions. Also there are useful pre-defined variables:
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

Modules for hakchi/hakchi2 should be distributed as files with an "**.hmod**" extension. In fact, it's just a renamed .tar.gz archive, so you can rename any .hmod file, unpack and inspect how it's made. There are several such files in "**mods\hmods**" directory of hakchi2 (these are pre-installed mods: the controller hack, font hack, and clovershell daemon). Also hakchi2 can install unpacked hmods if it's a directory. You can find such an example in "**user_mods\music_hack.hmod**", this mod replaces menu music (with silence by default, but you can change the wav file to your favorite music track).

An .hmod archive *can* contain:
* First of all, any files you need to install on a NES Mini (it's recommended to respect the directory structure)
* Optional "**install**" script
* Optional "**uninstall** script
* Optional "**readme.txt**" file with some description/info

What happens during module installation:
* If "**install**" file exists, it will be executed, it can contain any hakchi functions and variables showed above
* If "**install**" has returned a non-zero value or doesn't exist, all files will be copied to "**/var/lib/hakchi/rootfs**" directory (excluding "install", "uninstall" and readme files)
* If "**uninstall**" file exists, it will be stored into "**/var/lib/hakchi/hmod**" directory
* If "**uninstall**" file does not exist, it will be created automatically based on the installed files list

What happens during module uninstallation:
* The corresponding "**uninstall**" file will be executed and removed


## Examples

### Simple file replacement -- custom skin mod

I suggest starting with something very simple. Do you want to customize your NES Mini and create your own theme for it? Let's find the location of the theme file on NES Classic Mini. At first, enable the FTP server:

![FTP](http://clusterrr.com/dump/hakchi2_ftp.png)

And login to it using "**127.0.0.1**" as an address, **1021** as a port, "**root**" as a login, and "**clover**" as a password.

It's easy to realize that all images are stored in **/usr/share/clover-ui/resources/sprites/nes.png** file:

![Skin](http://clusterrr.com/dump/nes_mini_skin.png)

So we can download this file and edit using your favorite graphic editor without any problems! But... we can't rewrite this file, it's stored on the read-only file system inside NES Mini. Although, we can place files inside "/var/lib" directory, so save your modified file as ***/var/lib/hakchi/rootfs*/usr/share/clover-ui/resources/sprites/nes.png**:

![Skin](http://clusterrr.com/dump/nes_mini_skin2.png)

It can be saved successfully, but it won't replace the original **/usr/share/clover-ui/resources/sprites/nes.png** on it's own. We need to create a pre-init script to overmount this file on the early boot stage. Actually it's very easy to do. All pre-init scripts are stored in "**/etc/preinit.d**" (in fact, it's "**/var/lib/hakchi/rootfs/etc/preinit.d**", but it's overmounted onto "**/etc/preinit.d**", so who cares) directory. We need to create a new file here:

![preinit.d](http://clusterrr.com/dump/nes_mini_preinit.png)

The file must be named as **p*xxxx*_*name***, where "xxxx" are hexadecimal digits, and "name" is any text. You can choose any valid name, but keep in mind that all pre-init scripts will be executed in alphabetical order. It can be important in some cases.

Thus, you need to create "**/etc/preinit.d/p8032_skin**" file with only one line:

    overmount /usr/share/clover-ui/resources/sprites/nes.png


![preinit.d](http://clusterrr.com/dump/nes_mini_preinit2.png)

That's all. Just reboot your NES Classic Mini and you will see your custom theme:

![Custom theme photo](http://clusterrr.com/dump/nes_mini_skin_photo.jpg)

But what if you want to share your mod? Open "**user_mods**" directory inside hakchi2's directory and create a folder named "**my_skin.hmod**" there. You can choose any name, but it must be with ".hmod" on the end.
Then just copy all files you created inside NES Mini into this "**my_skin.hmod**" folder. You must keep all directory structure under "**/var/lib/hakchi/rootfs**", so it must contain "**usr\share\clover-ui\resources\sprites\nes.png**" and "**etc\preinit.d\p8032_skin**" files.

![hmod directory](http://clusterrr.com/dump/hakchi2_skin_hmod.png)

Also, it's a good idea to create "readme.txt" file inside "**my_skin.hmod**" folder with some mod description. We can create "**install**" and "**uninstall**" files too, but this is not required for such a simple mod, hakchi will create install and uninstall scenarios automatically.

You should be able to install/uninstall this mod using hakchi2 now:

![mod installer](http://clusterrr.com/dump/hakchi2_mod_installer.png)

However, you should distribute your mods not as directories, but as single ".hmod" files which are really tar.gz archives. So let's create such an archive file using tar util from MSYS collection:

![Packing hmod](http://clusterrr.com/dump/hakchi2_hmod_packed.png)

After this we have "**awesome_skin.hmod**" file, which can be easily shared with other users. All they need to do is just drag-and-drop this file on hakchi2's window. And it's still compatible with madmonkey's original hakchi. Actually, I'm not sure it's totally legal to share skin files based on original NES Mini's files.

In the same way you can edit any other NES Mini's files: music, sounds, text, etc.


### Basic scripting -- thumbnails removal

Many people asked me to remove the thumbnails at the bottom of the screen. This feature was introduced in early versions, but was removed later. Get ready to create a mod to do it!

Let's check the files of NES Mini... There is "**/usr/share/clover-ui/resources/prefab/sys_game_thumbnail.scn**" file with some JSON data. And there is "enabled" boolean variable!

![sys_game_thumbnail.scn](http://clusterrr.com/dump/thumbnails_scn.png)

You already know what to do from the previous example:
* Download "**sys_game_thumbnail.scn**" file to your PC
* Edit it, change "enabled" from "true" to "false"
* Upload it to your NES Mini as "***/var/lib/hakchi/rootfs*/usr/share/clover-ui/resources/prefab/sys_game_thumbnail.scn**"
* Create a pre-init script to overmount this file

But we will not do it. We will use an other method. Open "**user_mods**" directory inside hakchi2's directory, create a folder named "**remove_thumbnails.hmod**" there, and create a file named "**install**" inside it. Now we have an installation script, which will be executed during the mod installation process. We can make the needed changes during this process by placing the following lines right into the file:

    # Lines started with "#" are ignored and can be used as comments
    # Next line defines "scnfile" variable with path to "sys_game_thumbnail.scn" file
    scnfile=/usr/share/clover-ui/resources/prefab/sys_game_thumbnail.scn
    # This line defines "preinitfile" variable with the pre-init file name
    preinitfile=p81a8_hide_thumnbnails
    # "restore" is a hakchi function which copies an original file to the corresponding path in /var/lib/hakchi/rootfs
    restore $scnfile
    # sed is a GNU util for modifying files, this command replaces "enabled:true" with "enabled:false"
    # Please note that we need to edit $rootfs$scnfile (writable file), not just $scnfile (read-only original) file
    sed -i -e 's/"enabled":true/"enabled":false/g' $rootfs$scnfile
    # Create the pre-init script and echo "overmount" command to it
    echo "overmount $scnfile" > $preinitpath/$preinitfile
    # We should return 1 to prevent execution of automatic installer 
    return 1

That's all. But don't forget to create "uninstall" script too:

    # All we need is to delete the installed files, original file is safe and will be used again after a reboot
    scnfile=/usr/share/clover-ui/resources/prefab/sys_game_thumbnail.scn
    preinitfile=p81a8_hide_thumnbnails
    rm -f $preinitfile
    rm -f $rootfs$scnfile

Let's install our mod...

![mod installer](http://clusterrr.com/dump/hakchi2_mod_installer2.png)

Oops...

![Oops](http://clusterrr.com/dump/nes_mini_thumbnails_oops.jpg)

The thumbnails have been removed, but the cursor is still there. It's a part of the skin, so we can just make it's transparent, but it will conflict with the previous skin mod. It's very important to prevent mod conflicts. 

There is another config file -- "**/usr/share/clover-ui/resources/sprites/nes.json**", and it contains coordinates and properties for every sprite on the screen:

![nes.json](http://clusterrr.com/dump/nes_json.png)

We need to change our "**install**" script to replace the sprite properties:

    # Lines started with "#" are ignored and can be used as comments
    # Next line defines "scnfile" variable with path to "sys_game_thumbnail.scn" file
    scnfile=/usr/share/clover-ui/resources/prefab/sys_game_thumbnail.scn
    # Same with "nes.json" file
    nesjson=/usr/share/clover-ui/resources/sprites/nes.json
    # This line defines "preinitfile" variable with the pre-init file name
    preinitfile=p81a8_hide_thumnbnails
    # "restore" is a hakchi function which copies an original file to the corresponding path in /var/lib/hakchi/rootfs
    restore $scnfile
    restore $nesjson
    # sed is a GNU util for modifying files, this command replaces "enabled:true" with "enabled:false"
    # Please note that we need to edit $rootfs$scnfile (writable file), not just $scnfile (read-only original) file
    sed -i -e 's/"enabled":true/"enabled":false/g' $rootfs$scnfile
    # Same with nes.json file, the most simple way is to replace sprite coords with zeros
    sed -i -e 's/\[93,861,12,8\]/\[0,0,0,0\]/g' $rootfs$nesjson
    sed -i -e 's/\[93,871,12,8\]/\[0,0,0,0\]/g' $rootfs$nesjson
    sed -i -e 's/\[107,861,12,8\]/\[0,0,0,0\]/g' $rootfs$nesjson
    # Create the pre-init script and echo "overmount" command to it
    echo "overmount $scnfile" > $preinitpath/$preinitfile
    echo "overmount $nesjson" >> $preinitpath/$preinitfile
    # We should return 1 to prevent execution of automatic installer 
    return 1

Done! No more thumbnails and cursors at the bottom of the screen!

![No thumbnails](http://clusterrr.com/dump/nes_mini_thumbnails_ok.jpg)

I'll bundle this mod with hakchi v2.17, so you can check and edit it on your own.


### Hardcore scripting -- password protection

Are you ready to rock and create something really complicated? My child is playing too much, and I want to limit his access to the NES Classic Mini somehow. Therefore, I desired to create a password protection mod.

One of the coolest things in Linux is an ability to do near everything using only shell scripts. Another cool thing is the "everything is a file" concept, so we can access devices as usual files. First of all, we need to enable the Telnet server and connect to NES Mini's shell using a Telnet client.

![Telnet](http://clusterrr.com/dump/hakchi2_telnet.png)

Here is where the controller pseudo-file may be located:

![Controller pseudo-file](http://clusterrr.com/dump/hakchi2_controller_file.png)

The "**/dev/input/by-path/platform-twi.1-event-joystick**" file is just what we need. It outputs data on every event from the controller. This is binary data, so we need "hexdump" util to convert this data into human readable hexademical format.

![Controller pseudo-file](http://clusterrr.com/dump/nes_mini_hexdump_raw.png)

This is what I get after pressing "Up", "Up", "Down", "Down". There is too much data just for four button presses! It outputs 32 bytes on every button press and 32 bytes on every button release. Despite this, you don't need to be a character from the Matrix movie to find the pattern:

![Controller pseudo-file](http://clusterrr.com/dump/nes_mini_hexdump_raw_pattern.png)

The button code is in the 11th and 12th bytes of a packet. Let's trim them out... Enter this command:

    while [ true ]; do buttons=$(cat /dev/input/by-path/platform-twi.1-event-joystick | hexdump -v -e '1/1 "%02x"' -n 32); echo -n "${buttons:20:4} "; done

And press some buttons...

![Buttons codes](http://clusterrr.com/dump/nes_mini_buttons_code.png)

Wow, these are our button codes! But we also need to ask a user for a password somehow. I drew three simple images in "1280x720" resolution (that is 720p, it's the screen resolution of NES Mini):

![Password images](http://clusterrr.com/dump/nes_mini_password_images.png)

We need to convert them into RAW format, since there is no any software to show images on NES Mini. Of course, we could write this software, compile it for ARM processor, and install it on NES Mini, but this would be too much just for static images. There are many tools to convert images into RAW format, NES Mini uses "RGBA" byte order. RAW files are huge, so it's better to use gzip to compress them. I stored this files as "**password.raw.gz**", "**password_fail.raw.gz**" and "**password_ok.raw.gz**" into "**/etc/**" directory. It's possible to show them on the screen using a simple command:

    gunzip -c /etc/password.raw.gz > /dev/fb0

This command loads "**/etc/password.raw.gz**" file and writes unzipped data to "**/dev/fb0**" pseudo-file, which is the frame-buffer representation. 

Finally, we are ready to write the password protection script! I'll store it as "**/etc/init.d/S810password**", which is executed after "**S79clovercon**" (controller driver), but before "**S81clover-mcp**" (main GUI). Please note that line endings **MUST** be UNIX-style (\n instead of \r\n). Also please note that you can't brick your NES Mini, but any error can cause a boot loop or any other fail during startup. Don't panic, you can remove a broken script using "uninstall" process. So, we're all set!

    #!/bin/sh

    # Some constants
    # Controller pseudo-file
    clovercon=/dev/input/by-path/platform-twi.1-event-joystick
    # Password pattern - it's Konami Code :)
    password="c202 c202 c302 c302 c002 c102 c002 c102 3101 3001 3b01"
    # Max tries
    max_tries=30

    # Init scripts are executed with "start" argument when system boots and with "stop" argument during shutdown
    # So we need to check that it's boot state, not shutdown
    [ -z "$1" ] || [ "$1" == "start" ] || exit 0

    # Drawing "ENTER PASSWORD" image
    gunzip -c /etc/password.raw.gz > /dev/fb0

    # Endless loop
    while [ true ]; do
      # Wait while pseudo-file doesn't exists (controller is not connected or is not initialized yet)
      while [ ! -e $clovercon ]; do usleep 100; done
      # Reading buttons
      buttons=$(cat $clovercon | hexdump -v -e '1/1 "%02x"' -n 32)     
      buttons=${buttons:20:4}
      # Appending pressed button to "str" variable
      str="$str $buttons"
      # If "str" variable is longer than password string, cut it
      # So we get only last presses
      [ "${#str}" -gt "${#password}" ] && str="${str:$((${#str}-${#password})):${#password}}"

      # Is password correct?    
      if [ "$str" == "$password" ]; then
        # Display "OK" image
        gunzip -c /etc/password_ok.raw.gz > /dev/fb0
        # Exit and continue system loading
        exit 0
      fi
    
      # Password is not correct yet, decrease tries counter
      max_tries=$(($max_tries-1))
      # Is it zero?
      if [ "$max_tries" == "0" ]; then
        # Display "ACCESS DENIED" image
        gunzip -c /etc/password_fail.raw.gz > /dev/fb0
        # Wait for three seconds
        sleep 3
        # Shutdown system
        poweroff
      fi
    done

That's it! NES Mini asks for a password during boot now. I'll bundle this mod with hakchi v2.17 too.


### Bonus -- translate Famicom into English language

Just in case you don't know -- it's easy to transform your NES Classic Mini into a Famicom Mini and vice-versa. All you need is to upload a firmware file into "**/var/lib/hakchi/firmware**" folder (create it). Of course, I can't share the firmware files due to copyright restrictions, but you can easily find them on torrents:
* **dp-hvc-release-v1.0.5-0-g2f04d11.hsqs** -- to transform a NES Mini into a Famicom Mini
* **dp-nes-release-v1.0.2-0-g99e37e1.hsqs** -- to transform a Famicom Mini into a NES Mini

Just upload one of those files using FTP and reboot your console. Don't forget also to change "Console type" in hakchi2, select foreign "Original 30 games", and sync if you want to get foreign games too, not only foreign GUI.

But what if you want to use a Famicom Mini in NES Mini's interface language? Actually, it's very simple to do. You need to:

* Overmount "**/usr/share/clover-ui/resources/scripts/system.lua**" file using the same file from a NES Mini, it will enable the language selection dialog and icon
* Overmount the whole "**/usr/share/clover-ui/resources/strings**" folder, which contains strings for all languages (BTW, you can edit them too if you want)
* Overmount "**/usr/share/clover-ui/resources/fonts/hvc**" folder with "**/usr/share/clover-ui/resources/fonts/nes**" folder extracted from a NES Classic Mini.

Sorry, I can't share those files. But you can extract them on your own. You can take it as your homework :)