# ARM FDPIC toolset
 ARM FDPIC toolset implements the so-called [ARM FDPIC ABI](https://github.com/mickael-guene/fdpic_doc/blob/master/abi.txt).
 This toolset allows to use shared libraries on MMU-less platforms. Using shared libraries allows to reduce
 memory requirement on the system. Indeed code segments are only loaded once in memory and then
 shared across processes in the system.
 
 Prebuilt [packages](https://github.com/mickael-guene/fdpic_manifest/releases) can be found for
 Ubuntu 14.04. One package contains the FDPIC toolset and will allow you to build ARM FDPIC binaries.
 The other one is useful to run ARM FDPIC binaries on your host machine, it contains PRoot,
 qemu-arm binaries and a basic FDPIC rootfs.

## How to build
 Following installation instructions have been tested on an Ubuntu 14.04 docker image (docker pull ubuntu:14.04).

* install prerequisites

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

 At the end of the process you will find two tarballs in the newly created 'out' directory. One that contains
 the toolset and the other one that contains PRoot, qemu-arm and a basic rootfs.

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

## Kernel FDPIC support
To be able to load and execute FDPIC binaries you need following kernel patches :
- [Add support to new fdpic abi binaries loading](https://github.com/mickael-guene/kernel/commit/267158143576542c59bbef6e97aa10fb04b72294)
- [Add support for fdpic binaries loading](https://github.com/mickael-guene/kernel/commit/1b8c98252261980dc79cf0090341d74d8753b891)
- [add support to return to thumb2 code from signal for fdpic binaries](https://github.com/mickael-guene/kernel/commit/10e6b818854e3e85934751f6580a0588c58d3bcd)
- [Add get_tls syscall](https://github.com/mickael-guene/kernel/commit/e76d4547c033764677d83aef44df23b5ae1e3d03)
- [Add tls support for cortex-m cpu family](https://github.com/mickael-guene/kernel/commit/7ed2bdd8544081ea41367e797547818aef409dfa)
- [Workaround to fix futex bug on mmu less](https://github.com/mickael-guene/kernel/commit/1619691a3d25436e7a9c78d66d4f17249aa3dc7c)

