# Note about compiling in Manjaro

## Summary

YGOPro is good, but it may be hard to compile it because its configurations are not compatible for all GNU/Linux releases. This note records my solutions.

The Chinese edition of this note can be found in my blog, welcome!

[Time] 2020-12-17  
[ OS ] Manjaro, which usually despised by ArcLinux community ･ﾟ( ﾉд`ﾟ)  
PS: Building YGOPro is very easy in Ubuntu, just follow the wiki.

## Steps

1. There are some packages should be installed from pacman:  gcc make premake freetype2 libevent sqlite irrlicht git.

2. Clone the repo and its two submodules from github. Then create a folder named `build` under the repo.

3. 'irrKlang' is an unfree library which should be downloaded from its official website. It only provides some header files and 3 dynamic libraries. Put header files into `build/include/` under the repo, and `libIrrKlang.so` should be placed in `build/lib/`. What about `ikpFlac.so` and `ikpMP3.so`? They are plugins that should be placed under the runtime environment, now we can just put them into `build/assetes/lib/`.

4. Because the 'lua53' installed from pacman doesn't contain a file named `liblua5.3-c++.so`. If you link objects with `liblua5.3.so`, you will get errors because C use different symbols from C++. It seems that Manjaro does not provide a C++ edition of lua dynamic library, so we must download source codes from the official website. Edit its default `Makefile` and use g++ (with `-std=c++14`, same as YGOPro) to compile them into a static library. Then we rename it to `liblua5.3-c++.a` and put it into the folder `build/lib/` under the repo, waiting for linking.

5. In `premake4.lua`:
   
     * Let `USE_IRRKLANG = true` so you can build sound manager.
     * Add `libdirs { "build/lib" }` and `includedirs { "build/include" }` under the item `configuration "linux"`.
     
     * Set `objdir` to `objdir "build/obj"`.
     * Set two `targetdir` to `"build/bin/debug"` and `"build/bin/release"`.
     
6. Download card database, sounds, pictures, scripts and so on from YGO communities you can find on the Internet. Put them into `build/assetes/`. The wiki of this repo recognized some necessary files. Don't forget to set fonts in `system.conf`, otherwise you can find crash log in `error.log`.

7. Now input `premake5 gmake` and change your directory into the folder `build/`, just `make config=release` it. After all, we can find a binary file in `build/bin/`. Copy it into `build/assetes/`
   and enjoy yourself.

Your `build/` should be like this:

```plain
├── assets/
│   ├── bot.conf
│   ├── cards.cdb
│   ├── deck/
│   ├── error.log (important!!! check it if crashed)
│   ├── expansions/
│   ├── lflist.conf
│   ├── ikpFlac.so (must be put in this folder)
│   ├── ikpMP3.so (must be put in this folder)
│   ├── libIrrKlang.so (can also be put in /usr/lib)
│   ├── pics/
│   ├── replay/
│   ├── script/
│   ├── single/
│   ├── sound/
│   ├── strings.conf
│   ├── system.conf (fonts should be set)
│   ├── textures/
│   └── ygopro (copied binary)
├── bin/
│   └── release/
│       ├── libclzma.a
│       ├── libcspmemvfs.a
│       ├── libocgcore.a
│       └── ygopro (built binary)
├── clzma.make
├── cspmemvfs.make
├── include/
│   ├── ik_ESoundEngineOptions.h
│   ├── ...
│   └── irrKlang.h
├── lib/
│   ├── libIrrKlang.so
│   └── liblua5.3-c++.a
├── Makefile
├── obj
│   └── Release/
├── ocgcore.make
└── ygopro.make
```

## Other

If you got a message that the binary cannot find `irrKlang.so`, you can copy this dynamic library into `/usr/lib`. The command `ldd` helps a lot. A better method is to write a script to boot the binary, in which we can set an environment variable like `export LD_LIBRARY_PATH=/path/to/your/libdir`.
