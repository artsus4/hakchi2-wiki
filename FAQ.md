**Q: What is it?**

A: This application can add more games to your Nintendo Classic Mini. All you need is to connect it to PC via microUSB cable. No soldering, no disassembling.


**Q: So you were the first to hack the NES Classic Mini?**
No! It was my Russian сomrade, madmonkey, who first published a successful hack of the the NES Classic Mini. He created the original “hakchi” tool. However, it was not very user-friendly, so I decided to create a tool which is simple to use by anyone--not only Linux users. I named it “hakchi2” because I don’t like to come up with names. So my first version was a 2.0 release :)


**Q: How do I use the tool?**
Basically you just need to unpack it somewhere on your harddrive (installation is not required), run it, press “Add more games”, select some game ROMs and press “Synchronize”. The application will guide you through this process.


**Q: How is it working?**

A: You don’t need to worry about it. But if you really want to know it’s using FEL mode. FEL is a low-level subroutine contained in the BootROM on Allwinner devices. It is used for initial programming and recovery of devices using USB. So we can upload some code into RAM and execute it. In this way we can read Linux kernel (yes, NES Mini runs on Linux), write kernel or execute kernel from memory without writing it to flash. So we can dump kernel image of NES Mini, unpack it, add some games and script which will copy them to flash, repack, upload and execute. But games directory is on read only partition. So we need also to create and flash custom kernel with special script that creates sandbox folder on writable partition and mounts it over original games folder. So your original files are safe. You can’t delete or harm original files in any way. For kernel patching my application just executing other applications, that’s why there is “tools” folder.


**Q: Which games are supported?**

A: If you just want to enable support for near all games, read the next question. But if you want to satisfy your curiosity and thirst for knowledge... First of all, it depends on game's mapper. Mapper is hardware stored inside game cartridge, so emulator must emulate not only NES/Famicom hardware but mapper (cartridge's hardware) too. But there are hundreds of different mappers, many pirate/unlicensed mappers are not discovered ant documented at all. But NES Mini's emulator can emulate most popular mappers. My hakchi2 detects unsupported mapper automatically and will warn you. It still can add unsupported ROMs but the games likely will not work without additional modding.

At this time, it is confirmed that the NES Mini's emulator can run these mappers:

* 0 (NROM) - very simple games like Ice Climber, Pac-Man, etc.
* 1 (MMC1) - many good games, this is the second most popular mapper.
* 2 (UxROM - UNROM/UOROM) - games like Castlevania, Contra, Duck Tales, etc.
* 3 (CNROM) - mostly simple games but with heavy graphics like Adventure Island, Friday The 13th, etc.
* 4 (MMC3) - most popular mapper, used by a huge bunch of great games.
* 5 (MMC5) - very complex and the most powerful mapper on NES, used only by Castlevania 3 and a few Japanese games. Is there at least one default game using it? I’m surprised that NES Mini can emulate it.
* 7 (AxROM - ANROM/AMROM/etc.) - simple mapper used by games like Battletoads.
* 9 (MMC2) - used only by Punch Out!!
* 10 (MMC4) - used by a few Japanese games.
* 86 - some Japanese games.
* 87 - some Japanese games.
* 184 - some... games
* Famicom Disk System images - Japanese ROMs with .fds extension, like original Super Mario Bros. 2, Doki Doki Panic, Japanese version of Metroid, etc.

Also, if you patch unsupported game with a mapper hack/patch/conversion, this should work. There are many MMC3 patches over the Internet.

But even when a mapper is supported, some games are not working well without patches. The emulator in the NES Mini was tested only on default 30 games and actually sucks.
Known problem games:
* Battletoads - crashed on second level. Patch available: http://www.romhacking.net/hacks/2528/
* Robocop 3 - it should work but not working at all. MMC3 port is working fine. There is quite a bit of weird code in this game. I created my own [patch](http://clusterrr.com/roms/nes-patches/RoboCop%203%20(U)%20%5b!%5d%20-%20NES%20Mini%20patch.ips).
* Snow Bros. - When the CPU writes to the serial port on consecutive cycles, the MMC1 ignores all writes but the first. This happens when the 6502 executes read-modify-write (RMW) instructions, such as DEC and ROR, by writing back the old value and then writing the new value on the next cycle. It's very easy to fix. I created my own [patch](http://clusterrr.com/roms/nes-patches/Snow%20Bros.%20(U)%20%5b!%5d%20-%20NES%20Mini%20patch.ips).
* Bill & Ted's Excellent Adventure - same as Snow Bros. [My patch](http://clusterrr.com/roms/nes-patches/Bill%20&%20Ted's%20Excellent%20Video%20Game%20Adventure%20(U)%20-%20NES%20Mini%20patch.ips) .
* Startropics II - actually, it uses the MMC6 mapper which is very similar to MMC3. The only difference is PRG protection. It's very easy to create a patch. This is mine: (http://clusterrr.com/roms/nes-patches/Startropics%20II%20-%20Zoda's%20Revenge%20(U)%20[!]_MMC3.ips)[patch].

Those games will be automatically patched since version 2.09. Please feel free to send me your patches.

Also it's possible to install 3rd-party emulator... since version 2.12. This can run not only all NES games but also SNES, Genesis, GBA, N64, etc.. You can download the RetroArch mod here: https://github.com/ClusterM/retroarch-clover/releases


**Q: Hey! I know that NES Mini can run those unsupported games with other mappers. Also I know that it can run games for other consoles like SNES, GBA and ever N64!**

A: It's true. Since version 2.09 it's possible to install user mods and other emulators. There is out project with RetroArch emulator: https://github.com/ClusterM/retroarch-clover/releases
Just download "retroarch_with_cores.zip", drag and drop it on hakchi2's window and follow instructions. You can find more info on RetroArch's repo: https://github.com/ClusterM/retroarch-clover/


**Q: Where can I find a list of games with mappers?**

A: http://bootgod.dyndns.org:7777 and http://tuxnes.sourceforge.net/nesmapper.txt


**Q: Can I play European/PAL games?**

A: NES Mini's default emulator can emulate only NTSC NES. There is a command line argument to enable PAL emulation, but it’s not working for some reason. All European NES Minis actually the same as USA versions, so they are running NTSC versions of games. So you can not play all PAL games, and NTSC games will run faster. Use “(U)” and “(J)” ROMs if possible. But of course 3rd-party emulators can emulate PAL.


**Q: How many games can be uploaded to NES Mini?**

A: Writable partition of NES Mini is 384 megabytes in size. It contains save-states, settings and all data installed using hakchi. So max free space varies and depends on many factors. Version 2.15 introduces memory calculator. It can show real available memory space using this formula: *[current free space] **+** [size of installed games] **-** [reserved 10MB]*
Don't be very greedy - leave some free space for save-states and mods, that's why hakchi2 requires to leave at least 10MB free.

But the shell is designed to show only 30 games and there are some issues with other numbers of games:
* The thumbnail bar at the bottom of screen can show only ~45 covers. Other covers will be drawn beyond bounds of the screen.
* The shell can show up to ~90 games but it can't allocate enough RAM to show game covers and save-state screenshots. So more games on screen means less save-states.
* Having too few games causes problems, too. There are should be at least 12 games in the menu to show them without glitches.
Version 2.11 introduces a folder feature to avoid those problems.


**Q: How does the folder feature work?**

A: Since version 2.12 you can create any custom folder structure using Folder Manager. It's possible to edit folder names, images, etc. But it's recommended to limit games to 30 per page/folder if you want to keep save-state feature fully functional.


**Q: Why my NES Mini lost some space available for games? I used to install X megabytes of games early but I can't know!

A: Writable partition can be filled not only with games. First of all, check save-states using Save-States manager. Remove unwanted ones. Then check for installed mods. If you still can't find flash memory leak, use FTP Server (available since version 2.16) and check /var/lib folder - it contains all writable data. Maybe there are some unused large file. But think twice before deleting something. Remove them on your own risk. You can't brick your NES Mini that way but you can brick some settings or save-states.


**Q: Will it work with the Famicom Mini too?**

A: Yes. You can select the console type in the menu. It will be autodetected since version 2.14.


**Q: I can’t install the driver!**

A: If you are using Windows Vista, 7, 8 or 10, disable driver signature verification (Google it) and try again. Also try to use Zadig driver installer: http://zadig.akeo.ie/


**Q: What about non-Windows users?**

A: Mac users can use Parallels for virtualization. Linux users can use Wine.


**Q: It says that the MD5 checksum is unknown! What I need to do?**

A: There are two possible reasons:

* You have some unique unknown revision of NES Mini. Please send me your MD5 checksum is this case. You can continue flashing custom kernel on your own risk.
* You are not using hakchi2 for the first time with this NES Mini (for example, you used it on another PC) and your kernel is already patched. Therefore it will have an unknown MD5. It’s safe to continue in this case.
In any case it’s best not to lose the kernel image. It is stored in the “dump” folder. Back it up somewhere. But don’t worry if you still lost it. This kernel image is near the same for all NES Minis.


**Q: Can I brick my NES Mini?**

A: It’s pretty hard to brick it. You can always flash the original kernel back (via the menu). Even if your flashing process was terminated for some reason you just can do it again. But flash memory can typically handle only 100,000 erase cycles for any sector.


**Q: How can I update hakchi2 to a new version? I don’t want to lose my games, kernel image and settings.**

A: Simply copy all files of the new version into the folder of the old version with replacement. Or just copy folders “dump”, “games” and "config" (or "config.ini" for old versions) from the old version to the new version folder.


**Q: I missed my original kernel image :( Where I can find it?**

A: You can easily find it on torrent trackers.


**Q: Can I turn my NES Mini into Famicom Mini? And vice-versa?

A: Yes, but you need firmware of this console. You can find it on torrent trackers too.


**Q: Some games are displayed with an incorrect name; some characters are missing. Why?**

A: The NES Mini doesn’t contain some characters in its font, but I created my own font. Don’t forget to enable it in the “Settings” menu (enabled by default).


**Q: Can you modify emulator so the combination of buttons on gamepad will trigger reset?**

A: I̶t̶ ̶i̶s̶ ̶p̶o̶s̶s̶i̶b̶l̶e̶ ̶i̶n̶ ̶t̶h̶e̶o̶r̶y̶ ̶b̶u̶t̶ ̶n̶e̶e̶d̶ ̶t̶o̶ ̶d̶i̶s̶a̶s̶s̶e̶m̶b̶l̶e̶ ̶b̶i̶n̶a̶r̶y̶ ̶o̶f̶ ̶e̶m̶u̶l̶a̶t̶o̶r̶.̶ ̶I̶ ̶h̶a̶v̶e̶ ̶n̶o̶t̶ ̶s̶o̶ ̶m̶u̶c̶h̶ ̶s̶k̶i̶l̶l̶.̶
I made it. Just enable this hack in the menu.


**Q: Can I use save-states on added games?**

A: Yes, you can. Battery backed games can use their internal saves too. Note that saves are stored on a writable partition along with other save-states. When you delete a game, all save-states remains in the memory. You can delete them using Save-State Manager since version 2.16.


**Q: I want to upload games to my brother's/sister's/mother's/father's NES Mini on the same computer as the one I used to flash my NES. How can I do this?**

A: hakchi2 is a portable application. The safest way is to extract hakchi2 to a second separate folder, and run hakchi2 from there. Doing so should allow you to backup kernel, and add games to a second NES Mini without needing to reinstall the driver.
99% safe way: just flash the custom kernel and sync games in same application. This shouldn't break anything, but I can't 100% guarantee it.


**Q: How do I delete a game?**

A: If you want to delete game from the NES Mini, uncheck it in the list of games. If you want to delete it from the hakchi2 list, right click on the game and select "Delete".


**Q: Stop... You are uploading to NES Mini .nes files in iNES format which was developed by authors of the first NES emulators... So Nintendo is officially using stuff developed by pirates?**

A: Yes. And probably they are using pirated copies of their own games %)


**Q: I just got a black screen. My NES Mini is not working anymore! Can you help me?**

A: Don't panic. Just flash the original kernel back. Everything should work again.
Try to unpack the application and do everything again. Make sure that all files in place and not corrupted.


**Q: Where I can find a list of all command line arguments?**

A: The internal emulator of the NES Mini has many command line arguments. Seems like some of them are not working, but there is a full listing of "--help" output:
[https://github.com/ClusterM/hakchi2/wiki/Command-line-arguments](https://github.com/ClusterM/hakchi2/wiki/Command-line-arguments)


**Q: How do I disable this weird epilepsy protection?**

A: I think that it looks cool! But you can disable it. Just remove "--enable-armet" command line argument. Or you can disable it via the menu since version 2.09.


**Q: How to create my own "hmod" modules?**

A: Use the source Luke! I'll write some documentation soon.


**Q: Can I access NES Mini's internal file system and command line? It will help me create some cool mods!**

A: Yes, I created clovershell-client for it: https://github.com/ClusterM/clovershell-client/releases
It requires clovershell-daemon module to be installed but it's already installed if you are using hakchi2 v2.14 or newer.
Since version 2.16 all this functions are built in hakchi2, just check "Tools" menu.


**Q: What is "clover"? You are using this word everywhere!**

A: It's internal code name for NES Classic Mini. That's why NES Mini's model no. is "CLV-001". And controller is actually called "clovercon".


**Q: How do I uninstall it?**

A: There is an uninstall feature since version 2.08.


**Q: Your English is awful! Can we do something about it?**

A: hakchi2 is opensource: https://github.com/ClusterM/hakchi2
So you can clone it, fix, upload and make pull requests. Or just send me list of fixes.
Also you can help to translate hakchi2 to your language. We are using Google Sheets for it: https://docs.google.com/spreadsheets/d/1yrXH-6g61klKmuXTVf3f0ah8DCQObu-9YEJRoWTeuik/
Contact me and I'll give you access.


**Q: How can I donate to you?**

A: My PayPal: clusterrr@clusterrr.com
