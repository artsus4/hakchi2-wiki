2.00:
* It's first version actually :)

2.01:
* Added type of two players selection.
* Bugfixes.

2.02:
* Many bugfixes.

2.03:
* Another bugfixes.
* Added MD5 for Famicom Mini kernel.

2.04:
* Some fixes for dinosaurs with Windows XP.
* Kernel size check.

2.05:
* FDS support.
* Settings moved to config.ini file.
* Automatic driver installer removed due to annoying false virus reports.

2.06:
* Finally 'pipe read' problem solved!
* Some fool-protection.
* Now you can't run two copies of app accidentally.

2.07
* Driver installer added again.
* Now you can hide selected default games.
* Font fix with all missed characters.

2.08:
* Fixed unhandled exception bug when game with unsupported mapper selected.
* Fixed bug when NES Mini is not detected on some computers.
* Select all/unselect all feature, just right click on games list.
* Uninstall feature just in case you want to remove all hakchi traces.
* Four-screen check (it's not supported by NES Mini).
* Mapper #71 games automatically patched to mapper #2 , so you can play Camerica/Codemasters games: Micro Machines, Bee 52, Dizzy series, etc. One exception: Fire Hawk.
* Fix for Google search (thanks to Bin4ry!). This feature should work for all now.
* Added Drag&Drop support. You can just drag&drop .nes file to list of games.
* Help menu with links to FAQ and GitHub.

2.09:
- 'Can't repack ramdisk' error on some systems should be gone now.
- Device detection bug fixed (thanks to David Winter!)
- Automatic file attributes fix (e.g. "WinRAR" bug fixed).
- Game selection presets. Now you can create presets for favorite games, party games, etc. It's useful since NES Mini don't like huge amount of games at once.
- Search by first letters of game, just press Ctrl+F.
- Much better PNG compression for covers, x4 more disk space.
- NES carts database with release dates, publishers, etc. Just add game and all fields will be filled automatically if it exists in database.
- Automatic IPS patches. Now you don't need other application to patch problem games. Patch files stored in "patch" folder. Problem game will be detected automatically by CRC32 checksum. Release already contains patches for "Battletoads", "Robocop 3", "Snow Bros.", "Bill & Ted's Excellent Video Game Adventure" and "Startropics II". You can add and your own patches too.
- Game Genie support. Just enter your codes (comma/spaces/semicolon separated) in special field and sync. Cheats will be activated automatically.
- Mapper #87 added to list of confirmed mappers.
- Epilepsy protection settings - you can disable epilepsy protection for original 30 games or for all games at once.
- Clovercon hack! Now you can exit to menu using "Down+Select" combination. All you need now is controller cable extension...
- Some interface improvements - you can delete games using del key, etc.
- Many minor bugfixes.

2.10:
* Press Shift+F5 to update all your ROMs library info using database.
* Extended font working on Famicom Mini again.
* Now you can select button combination for reset.
* Some FDS improvements: correct cover size, automatic header fix, "--fds-auto-disk-side-switch-on-keypress" argument by default.
* 8bit PNG compression is optional now since image quality is not so good.
* Now you can remove thumbnails at the bottom of screen (via settings menu).
* Bugfixes.