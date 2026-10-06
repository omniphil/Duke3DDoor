# DUKE NUKEM 3D for TERMinator: a BBS door

3D Realms' Duke Nukem 3D (1996), shareware episode 1 "L.A. Meltdown" (v1.3D), playable from a BBS. This is the source
of the Linux door, `duke3ddoor`, and of the game module it sends. It works like the DOOM door
(https://github.com/omniphil/DOOMDoor) and the Wolfenstein 3D door (https://github.com/omniphil/Wolf3DDoor):

- **TRACE** (TERMinator 1.1.2+): the whole game runs on the caller's PC inside TERMinator's sandbox at 640x400, with
  General MIDI music and digitised sound. The BBS sends it once; after that almost nothing crosses the wire.
- **JPEG XL graphics** (terminals that speak the CTerm APC picture and sound commands): the door runs the game on the
  BBS and sends its 320x200 picture as JPEG XL, up to 30 frames a second at a quality that follows the link, with its
  music and effects played from the caller's own cache. Uses the system's libjxl.
- **ANSI 24-bit / 256 / 16** (every other terminal): the door runs the same game on the BBS and sends it as ANSI
  half-blocks, with the menus redrawn as text and a text status line. It is silent.

The start page lets each caller pick, and remembers the choice.

**This repository exists so that anyone who plays the door can have the source of the game they were sent**, which is
what the GNU GPL asks for (see `door/LICENSE.md`). It is the address the Linux door itself gives out. The same source
also builds a door for Mystic BBS for Windows, published with Windows build steps at
https://github.com/omniphil/Duke3DDoor-Win.

| Folder | What |
|---|---|
| `third_party/jfduke3d` | JFDuke3D (github.com/jonof/jfduke3d 55c5f95, with jfbuild 40a98be, jfaudiolib 68be97f, jfmact 731c0cc), unmodified. 3D Realms' code under the GPL (`GPL.TXT`) |
| `third_party/TinySoundFont` | TinySoundFont (github.com/schellingb/TinySoundFont 853a0a1, MIT), the MIDI synthesiser |
| `third_party/soundfont` | `TimGM6mb.sf2`, a General MIDI sound font by Tim Brechbill (GPL-2) |
| `module/` | `duke3d.wasm`, the game as a TRACE module (see `module/README.md`) |
| `door/` | `duke3ddoor` (see `door/INSTALL.md`) |
| `door/patches/` | the door's one change to its own copy of JFDuke3D (menu and status text for the ANSI mode) |
| `tools/get_shareware.sh` | fetches the shareware `DUKE3D.GRP` into `data/`. **Not in the repository**: it is 3D Realms', given away as shareware (its LICENSE.TXT lets free BBSs and BBSs of 250 nodes or fewer offer it) |
| `tools/mkpak.py` | packs the GRP and the sound font into `door/duke3d.pak`, the one asset the door sends |
| `tools/winlaunch.c`, `tools/test_windoor.py` | test the Windows build of the door from WSL, without a BBS |

## Build

On Linux (Debian, Ubuntu, Mint, Raspberry Pi OS; WSL too):

```
sudo apt install build-essential python3 curl ffmpeg libjxl-dev     # libjxl: the JPEG XL mode (only its .so is used)
```

and [wasi-sdk](https://github.com/WebAssembly/wasi-sdk/releases/tag/wasi-sdk-34) 34 (`wasi-sdk-34.0-x86_64-linux.tar.gz`)
unpacked into `~/tools`, which is where the module's Makefile looks (or give `WASI_SDK=<folder>`). Then:

```
sh tools/get_shareware.sh                         # the shareware data, into data/
make -C module && cp module/duke3d.wasm door/     # the TRACE module
python3 tools/mkpak.py data/DUKE3D.GRP third_party/soundfont/TimGM6mb.sf2 door/duke3d.pak
make -C door                                      # door/duke3ddoor
make -C door music sound                          # the JPEG XL mode's sound (needs ffmpeg with libvorbis)
```

`door/music/` and `door/sound/` are made from 3D Realms' data, so they aren't in the repository. Without them the
JPEG XL mode plays silently (TRACE and ANSI don't use them). Without libjxl the JPEG XL mode isn't offered.

To build on one machine and run on another (the BBS): `make -C door bundle` copies the game's sources into
`door/native/`, so the `door/` folder builds there on its own with `make` (gcc, make and patch). Installing it in the
BBS: `door/INSTALL.md`.
