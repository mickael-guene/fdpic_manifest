# ARM FDPIC toolset
 ARm FDPIC toolset implements the so-called ARM FDPIC ABI. This toolset allows to use
 shared libraries on MMU-less platforms. Using shared libraries allows to reduce
 memory requirement on the system. Indeed code segments are only loaded once in memory and then
 shared across processes in the system.
 
 Prebuilt [packages](https://github.com/mickael-guene/fdpic_manifest/releases) can be found for
 Ubuntu 14.04. One package contains the FDPIC toolset and will allow you to build ARM FDPIC binaries.
 The other one is usefull to run ARM FDPIC binaries on your host machine, it contains proot,
 qemu-arm binaries and a basic FDPIC rootfs.

## How to build
 Following installation instructions have been tested on an Ubuntu 14.04 docker image (docker pull ubuntu:14.04).

* install prerequites

First install following packages
```
~/fdpic $ apt-get install build-essential texinfo flex bison libncurses5-dev curl git python wget automake autoconf libtool pkg-config zlib1g-dev libglib2.0-dev
```
Then install repo if you don't have it in your path
```
~/fdpic $ mkdir ~/bin
~/fdpic $ PATH=~/bin:$PATH
~/fdpic $ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
~/fdpic $ chmod a+x ~/bin/repo
```
* fetch the sources
```
~/fdpic $ repo init -u https://github.com/mickael-guene/fdpic_manifest -b fdpic-v7-m
~/fdpic $ repo sync
```
* build toolchain package
```
~/fdpic $ ./scratch/build/scripts/build.sh
```

* build runtime package
```
~/fdpic $ ./scratch/build/scripts/build_runtime.sh
```

 At the end of the process you will find two tarballs in the out directory. One that contains toolset and the
 other one that contains proot, qemu-arm and a basic rootfs.

 If you want to build cortex-r toolset then replace fdpic-v7-m by fdpic-v7-r in the repo init phase.

 To speed up your build you can set JOBNB environment variable to the value you want to use $JOBNB parallel jobs.

## How to install it
* install toolset

For that just untar in a directory the tarball you have generated during compilation.
```
~/fdpic $ mkdir install_test
~/fdpic $ tar -C install_test -xf out/toolset-20150901-161939-d0b51e9f-armv7-m.tgz
```
* setup path and test your toolset is alive.
```
~/fdpic $ export PATH=~/fdpic/install_test/bin:${PATH}
~/fdpic $ arm-v7-linux-uclibceabi-gcc -v
```

* install runtime if you need it and check you can run an FDPIC binary
```
~/fdpic $ tar -C install_test -xf out/runtime-20150901-161939-d0b51e9f-armv7-m.tgz
~/fdpic $ ./install_test/proot -R install_test/rootfs -q install_test/qemu-arm ./install_test/rootfs/bin/gdbserver
```

## Limitations of the toolset
Main limitations of the toolset are due to the lack of MMU and are mainly :
* no fork support. Instead use vfork.
* memory limitations (mprotect, ....).
* stack has a static size defined at link time.

Other known limitations not due to the lack of MMU are :
* lazy binding is not supported. All the PLT entries are resolved at load time.
