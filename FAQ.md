**Q: What is it?**

A: This application can add more games to your Nintendo Classic Mini. All you need is to connect it to PC via microUSB cable. No soldering, no disassembling.


**Q: So you are hacked NES Mini?**

A: No! It was my russian сomrade madmonkey. He created original “hakchi” tool. It was not very user friendly so I decided to create tool which is simple for everyone, not only Linux users. I named it “hakchi2” because I don’t like to coming up with names. So first version was 2.0 :)


**Q: How to use it?**

A: Basically you need just to unpack it somewhere (installation not required), run it, press “Add more games”, select some ROMs and press “Synchronize”. Application will guide you.


**Q: How it’s working?**

A: You don’t need to worry about it. But if you really want to know it’s using FEL mode. FEL is a low-level subroutine contained in the BootROM on Allwinner devices. It is used for initial programming and recovery of devices using USB. So we can upload some code into RAM and execute it. In this way we can read Linux kernel (yes, NES Mini runs on Linux), write kernel or execute kernel from memory without writing it to flash. So we can dump kernel image of NES Mini, unpack it, add some games and script which will copy them to flash, repack, upload and execute. But games directory is on read only partition. So we need also to create and flash custom kernel with special script that creates sandbox folder on writable partition and mounts it over original games folder. So your original files are safe. You can’t delete or harm original files in any way. For kernel patching my application just executing other applications, that’s why there is “tools” folder.


**Q: Which games are supported?**

A: On this moment confirmed that emulator of NES Mini can run this mappers:

* 0 (NROM) - very simple games like Ice Climber, Pac-Man, etc.
* 1 (MMC1) - many good games, this is the second most popular mapper.
* 2 (UxROM - UNROM/UOROM) - games like Castlevania, Contra, Duck Tales, etc.
* 3 (CNROM) - mostly simple games but with much graphics, like Adventure Island, Friday The 13th, etc.
* 4 (MMC3) - most popular mapper, used by huge bunch of great games.
* 5 (MMC5) - very complex and most powerful mapper on NES, used only by Castlevania 3 and few japanese games. Is there at least one default game using it? I’m surprised that NES Mini can emulate it.
* 7 (AxROM - ANROM/AMROM/ANROM/etc.) - simple mapper used by games like Battletoads.
* 9 (MMC2) - used only by Punch Out!!
* 10 (MMC4) - used by few japanese games.
* Famicom Disk System images - japanese ROMs with .fds extension, like original Super Mario Bros. 2, Doki Doki Panic, japanese version of Metroid, etc.

It is possible that emulator supports some other mappers too. You can add those ROMs but application will warn you and game likely will not work. If it WILL work please report me about this game. I’ll add this mapper to list of confirmed.

Also if you will patch unsupported game with mapper hack/patch/conversion it should work. There are many MMC3 patches over the Internet. Also it's easy to port games from Codemasters/Camerica to UNROM.

But even game’s mapper is supported some games are not working good without patches. Emulator in NES Mini tested only on default 30 games and actually sucks.
Known problem games:
* Battletoads - crashed on second level. Patch available: http://www.romhacking.net/hacks/2528/
* Robocop 3 - it should work but not working at all. MMC3 port is working fine. There are many weird code in this game. I created my own [patch](http://clusterrr.com/roms/nes-patches/RoboCop%203%20(U)%20%5b!%5d%20-%20NES%20Mini%20patch.ips).
* Snow Bros. - When the CPU writes to the serial port on consecutive cycles, the MMC1 ignores all writes but the first. This happens when the 6502 executes read-modify-write (RMW) instructions, such as DEC and ROR, by writing back the old value and then writing the new value on the next cycle. It's very easy to fix. I created my own [patch](http://clusterrr.com/roms/nes-patches/Snow%20Bros.%20(U)%20%5b!%5d%20-%20NES%20Mini%20patch.ips).
* Bill & Ted's Excellent Adventure - same as Snow Bros. [My patch](http://clusterrr.com/roms/nes-patches/Bill%20&%20Ted's%20Excellent%20Video%20Game%20Adventure%20(U)%20-%20NES%20Mini%20patch.ips) .

Feel free to ask me for patched if you found some game that should work (no mapper working) but it crashes at startup.


**Q: Where can I find list of games with mappers?**

A: http://bootgod.dyndns.org:7777 and http://tuxnes.sourceforge.net/nesmapper.txt


**Q: Can I play european/PAL games?**

A: NES Mini can emulate only NTSC NES. There is command line argument to enable PAL emulation but it’s not working for some reason. All european NES Minis actually the same as USA versions and they are running NTSC versions of games. So you can play not all PAL games and this games will run faster. Use “(U)” and “(J)” ROMs if possible.


**Q: How many games can be uploaded to NES Mini?**

A: I don’t know. I have not tested it for maximum yet. Internal storage in NES Mini is really huge for ROMs (about ~300MB) but seems like built in shell of NES Mini can work only with ~90-100 games without glitches. Seems like it’s cannot allocate so much RAM. Investigation required.


**Q: Will it work with Famicom Mini too?**

A: Yes, It’s confirmed.


**Q: I can’t install driver!**

A: If you are using Windows Vista, 7, 8 or 10 disable driver signature verification (Google it) and try again.


**Q: It says that MD5 checksum is unknown! What I need to do?**

A: There are two possible reasons:

* You have some unique unknown revision of NES Mini. Please send me your MD5 checksum is this case. You can continue flashing custom kernel on your own risk.
* You are using hakchi2 not the first time with this NES Mini (used it on another PC e.g.) and your kernel already patched. Of course it has unknown MD5. It’s safe to continue in this case.
In any case it’s better not to lose kernel image. It is stored in the “dump” folder. Backup it somewhere. But don’t worry if you still lost it. This kernel image is near the same for all NES Minis.


**Q: Can I brick my NES Mini?**

A: It’s pretty hard to brick it. You always can flash original kernel back (via menu). Even if your flashing process was terminated for some reason you just can do it again. But flash memory can handle only 100,000 erase cycles for any sector typical.


**Q: How to update hakchi2 to new version? I don’t want to lose my games, kernel image and settings.**

A: Just copy all files of the new version into the folder of old version with replacement. Or just copy folders “dump” and “games” into the directory of new version. Also copy “config.ini” if it’s exists.


**Q: Some games are displayed with an incorrect name, some characters are missed. Why?**

A: Nes Mini doesn’t contain some characters in their font. But I created my own font. You can use it on the latest version. Don’t forget to enable it in “Settings” menu (enabled by default).


**Q: Can you modify emulator so the combination of buttons on gamepad will trigger reset?**

A: It is possible in theory but need to disassemble binary of emulator. I have not so much skill.


**Q: Can I use savestates on added games?**

A: Yes, you can. Battery backed games can use their internal saves too. Note that saves are stored on writable partition along with other savestates. When you delete a game, all savestates remains in the memory. You can delete them at once using factory reset. But they take only few kilobytes.


**Q: Why it is distributed in the SFX archive (as .exe)?**

A: Some versions of WinRAR can corrupt file attributes which are required by the “mkbootfs” util.


**Q: Your english is awful! Can we do something about it?**

A: hakchi2 is opensource: https://github.com/ClusterM/hakchi2
So you can clone it, fix, upload and make pull request. Same way you can add other languages and some cool features.
Or just send me list of fixes.


**Q: How to uninstall it?**

A: Just flash original kernel using command in the menu. But it will not delete sandbox folder. I'll made uninstall feature soon if you want.


**Q: I want to upload games to my brothers/systers/mothers/fathers NES Mini on the same computer as the one I used to flash my NES. How to do it?**

A: hakchi2 is a portable application. Most safe way is just extract the hakchi2 to a second separate folder, and run hakchi2 from there. Doing so should allow to backup kernel, and add games to his liking/flash accordingly. You don't need to install driver.
99% safe way: just flash custrom kernel and sync games in same application. This is shouldn't break anything but I can't 100% guarantee it.


**Q: How to delete a game?**

A: If you want to delete game from NES Mini just uncheck it in list. If you want to delete it from hakchi2 list do right click on game and select "Delete".


**Q: Stop... You are uploading to NES Mini .nes files in iNES format which was developed by authors of first NES emulators... So Nintendo officially using stuff developed by pirates?**

A: Yes. And probably they are using pirated copies of their own games %)


**Q: I just got black screen. My NES Mini is not working anymore! Can you help me?**

A: Don't panic. Just flash original kernel back. Everything should work again.
Try to unpack application and do everything again. Make sure that all files in place and not corrupted.


**Q: Where I can find list of all command line arguments?**

A: Internal emulator of NES Mini has many command line arguments. Seems like some of them are not working but there is full listing of "--help" output:
[https://github.com/ClusterM/hakchi2/wiki/Command-line-arguments](https://github.com/ClusterM/hakchi2/wiki/Command-line-arguments)


**Q: How can I donate you?**

A: My PayPal: clusterrr@clusterrr.com