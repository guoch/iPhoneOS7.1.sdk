# Compiling C/C++ project on jailbroked iOS devices (e.g. iPhone, iPad or iPod)
-----------------
Recently, I am trying to build a portable web server on my old iPhone4 (iOS v7.1.2), and need a Vim v7.4 (with Python, Ruby and Lua support) for my remote editing/coding, but the BigBoss (only v7.1) and Radare (only v7.3-1) can't help. So I have to work it out by myself, let's see how.

# 0. Assumption
I am assuming you have a jailbroked iDevice and have basis knowledge about Cydia, if not, pls follow this [TaiG jailbreak tutorial](http://www.taig.com/en/tutorial.html) and this pic
[![Add Source](/media/cydia-add-source.png "Add Source")](/media/cydia-add-source.png)


# 1. Preparation
Search and install require packages after adding sources into Cydia.

## 1.1 Add Cydia Sources

* `http://repo.insanelyi.com/` (It's the only source I can get *fake-libgcc*, kindly let me know if you have another new one.)

## 1.2 Search and/or install Packages

* `BigBoss Recommended Tools`
    - (A bunch of useful tools)

* `Darwin CC Tools` (v286-8, iOS compilation toolchain)

* `fake-libgcc` (v1.0-2, GCC compilation toolchain. **DO NOT** upgrade/remove it even Cydia always prompt you to upgrade with *libgcc*)
    - fake-libgcc
    - GNU C Compiler
    - iphone-gcc Headers

* `Lua`(v5.3.0-1): 

* `Ruby`(v1.9.2-p0-10): 

* `Python`(v2.7.6-3): 

    Since the Python in Cydia's sources is outdated, so need to install it manually.

    1. SSH to your iDevice with root ID
    2. Download Python DEB to /var/root/dl/
    
        ```nohightlight
        curl -Ok https://github.com/linusyang/python-for-ios/releases/download/v2.7.6-3/python_2.7.6-3_iphoneos-arm.deb /var/root/dl/```
        ```
        or
        ```nohightlight
        curl -Ok https://github.com/neotan/ios-tools/raw/master/python_2.7.6-3_iphoneos-arm.deb /var/root/dl/
        ```
    
    3. Install it
    
        ```nohightlight
        dpkg -i /var/root/dl/python_2.7.6-3_iphoneos-arm.deb
        ```

* `iPhoneOS7.1.sdk`

    You have 2 opthions to have it:

    1. Extracted from [Xcode 5.1.1 dmg](https://developer.apple.com/devcenter/download.action?path=/Developer_Tools/xcode_5.1.1/xcode_5.1.1.dmg): 

        ```nohightlight
        /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.1.sdk 
        ```
        to
        ```nohightlight
        /var/root/dl/iPhoneOS7.1.sdk
        ```
        and from 
        ```nohightlight
        /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator.sdk/usr/include/crt_externs.h 
        ```
        to
        ```nohightlight
        /var/root/dl/iPhoneOS7.1.sdk/usr/inlude/crt_externs.h
        ```

    2. Or simply clone from my repo (crt_externs.h is ready):

        ```nohightlight
        git clone https://github.com/neotan/iPhoneOS7.1.sdk.git /var/root/dl/
        ```
        
# 2. Compilation

Make a place for binaries and other stuffs.

```nohighlight
mkdir -p /var/root/dl/vimbuilt/share
```

Create a *config.site* with following content for Make, Make will read it while makeing Makefiles.

```nohighlight
nano /var/root/dl/local/share/config.site
```
 Content of *config.site:*
```nohighlight
CPPFLAGS='-I/private/var/root/dl/iPhoneOS71SDK/usr/include -I/private/var/root/dl/iPhoneOS71SDK/usr/include -F/private/var/root/dl/iPhoneOS71SDK/System/Library/Frameworks'
LDFLAGS='-L/private/var/root/dl/iPhoneOS71SDK/usr/lib -L/private/var/root/dl/iPhoneOS71SDK/usr/lib -L/private/var/root/dl/iPhoneOS71SDK/usr/lib/system -F/private/var/root/dl/iPhoneOS71SDK/System/Library/Frameworks'
LUA_PREFIX='/'
```

Now the folder structure:
```nohighlight
/var/root/dl/vimbuilt
└── share
    └── config.site  # Configurations to let Make know 
```

Fetch the latest source code of Vim
```nohighlight
git clone https://github.com/vim/vim.git /var/root/dl/vim74src
```
Go to the source code folder, you will see a *configure* files is over there.
```nohighlight
cd /var/root/dl/vim74src/src
```
Then run this command to build Makefiles
```nohighlight
./configure --with-features=huge --disable-darwin --enable-gui=no --without-x --enable-multibyte --prefix=/var/root/dl/vimbuilt --enable-luainterp --enable-pythoninterp --enable-rubyinterp 
```
>*--diable-darwin* was required to disable trying to use the *Carbon Framework*, which isn't available on iOS. 

You will see 
```nohighlight
configure: loading site script /var/root/dl/vimbuilt/share/config.site
configure: loading cache auto/config.cache
checking whether make sets $(MAKE)... (cached) yes
checking for gcc... (cached) gcc
checking whether the C compiler works... yes
checking for C compiler default output file name... a.out
...
checking for GCC 3 or later... yes
checking whether we need -D_FORTIFY_SOURCE=1... yes
checking linker --as-needed support... no
configure: creating auto/config.status
config.status: creating auto/config.mk
config.status: creating auto/config.h
config.status: auto/config.h is unchanged
root@iPhone:[~/dl/vim74src/src]
```
>Error info. will be logged to */var/root/dl/vim74src/src/auto/config.log* if any. 

and run this to start comilation
```nohighlight
make && make install
```

Finally, the binaries, manual and other stuffs will be place to */var/root/dl/vimbuilt*
```nohighlight
/var/root/dl/vimbuilt
├── bin                   # Vim's binaries
│   ├── ex -> vim
│   ├── rview -> vim
│   ├── rvim -> vim
│   ├── view -> vim
│   ├── vim
│   ├── vimdiff -> vim
│   ├── vimtutor
│   └── xxd
├── deb
│   ├── DEBIAN
│   │   └── control
│   └── usr
│       ├── bin            # Currently, empty
│       └── share          # Currently, empty
└── share
    ├── config.site
    ├── man                # Vim's manual
    └── vim
        └── vim74
```

In this moment, you may feel free to copy the `bin/*` to **/usr/bin/**, `share/*` to **/usr/share/**, that will work perfectly. But I will show you how to packing our work into a .deb file for further distribution. Keep going~

# 3. Building .deb package
Make a place for packing.

```nohighlight
mkdir -p /var/root/dl/vimbuilt/deb/DEBIAN
mkdir -p /var/root/dl/vimbuilt/deb/usr
```

We need to create a *control* with following content for *dpkg* command.
```nohighlight
nano /var/root/dl/vimbuilt/deb/DEBIAN/control
```
 Content of *control:*
```nohighlight
Package: io.github.neotan
Name: Vim v7.4
Version: 7.4
Section: Vim
Maintainer: Your Name <Your email or website>
Architecture: iphoneos-arm
Description: Vim v7.4 (with Python2.7, Ruby1.1 and Lua5.3 support)
Author: Your Name <Your email or website>
Sponsor: Your Name
Replaces: Vim (<= 7.4)
```

>*Replaces: Vim (<= 7.4)* means will replace current existing/installed Vim if it's version lower than our new version.

Now the folder structure:
```nohighlight
/var/root/dl/vimbuilt
├── bin              # Auto created by Make, for storaging binaries.
├── deb
│   ├── DEBIAN
│   │   └── control  # For building .deb file with dpkg command later
│   └── usr
└── share
    └── config.site  # Configurations to let Make know 
```

And the `vimbuilt/bin`  and the `vimbuilt/share` are the major things, the `vimbuilt/deb` is the place we start packing, so put them together:
```nohighlight
cd /var/root/dl/vimbuilt
cp -R bin deb/usr/
cp -R share deb/usr/
```
The structure looks like this before building packge:
```
/var/root/dl/vimbuilt/deb
├── DEBIAN
│   └── control
└── usr
    ├── bin
    │   ├── ex -> vim
    │   ├── rview -> vim
    │   ├── rvim -> vim
    │   ├── view -> vim
    │   ├── vim
    │   ├── vimdiff -> vim
    │   ├── vimtutor
    │   └── xxd
    └── share
        ├── config.site
        ├── man
        │   ├── fr
        │   ├── fr.ISO8859-1
        │   ├── fr.UTF-8
        │   ├── it
        │   ├── it.ISO8859-1
        │   ├── it.UTF-8
        │   ├── ja
        │   ├── man1
        │   ├── pl
        │   ├── pl.ISO8859-2
        │   ├── pl.UTF-8
        │   ├── ru.KOI8-R
        │   └── ru.UTF-8
        └── vim
            └── vim74
```
Build .deb package.
```nohighlight
dpkg -b deb vim74.deb
```
Ignore these warning.
```nohighlight
warning, `deb/DEBIAN/control' contains user-defined field `Name'
warning, `deb/DEBIAN/control' contains user-defined field `Author'
warning, `deb/DEBIAN/control' contains user-defined field `Sponsor'
dpkg-deb: building package `io.github.neotan' in `vim74.deb'.
dpkg-deb: ignoring 3 warnings about the control file(s)
```
Install new Vim from .deb package:
```nohighlight
dpkg -i vim74.deb
```
Check the Vim version and features
```nohighlight
vim --version
```
Now you will see `+Python, +Lua, +Ruby` tags

That's it. Let me know if you built any funny tool. 
Enjoy compiling! Cheers!

# References & Thanks
Thanks for following contributors of every threads! I can't make above blog without them!

* [ios - How to download Xcode 4 / 5 / 6 / 7 and get the DMG file? - Stack Overflow](http://stackoverflow.com/questions/10335747/how-to-download-xcode-4-5-6-7-and-get-the-dmg-file)
* [How To: Set up GCC on iOS 4 — n8.io](http://webcache.googleusercontent.com/search?q=cache:Q0hwMjnGSOQJ:https://n8.io/how-to-set-up-gcc-on-ios-4+&cd=5&hl=en&ct=clnk&gl=us)
* [bash - how to add include and lib paths to configure/make cycle - Stack Overflow](http://stackoverflow.com/questions/7561509/how-to-add-include-and-lib-paths-to-configure-make-cycle)
* [Github - linusyang/python-for-ios](https://github.com/linusyang/python-for-ios/releases)
