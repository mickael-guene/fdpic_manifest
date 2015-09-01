# Arm fdpic toolset
 Arm fdpic toolset implement the so-called arm fdpic abi. This toolset allow to enable the use
 of shared libraries support on mmu-less platforms. Using shared libraries allow to reduce
 memory requirement of the system. Indeed code segments are only load once in memory and then
 share accross process in the system.

## How to build
 Following installation instructions have been tested on an ubuntu 14.04 docker image (docker pull ubuntu:14.04).

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
* build
```
~/fdpic $ ./scratch/build/scripts/build.sh
```

 At the end the process you will find a tarball in out directory that contains the compile toolset.

 If you want to build cortex-r toolset then replace fdpic-v7-m by fdpic-v7-r in the repo init phase.

 To speed up your build you can setup JOBNB environment variable to the value you want to use $JOBNB parallel jobs.

## How to install it
* install toolset

For that just untar in a directory the tarball you have generated during compilation.
```
~/fdpic $ mkdir install_test
~/fdpic $ tar -C install_test -xf out/toolset-8b4eab1e-armv7-r.tgz
```
* setup path and test your toolset is alive.
```
~/fdpic $ export PATH=~/fdpic/install_test/bin:${PATH}
~/fdpic $ arm-v7-linux-uclibceabi-gcc -v
```

## Limitations of the toolset
Main limitations of the toolset are due to the lack of mmu and are mainly :
* no fork support. Instead use vfork.
* memory limitations (mprotect, ....).
* stack has a static size define at link time.

Other know limitations not due to the lack of mmu are :
* lazy binding is not supported. All the plt entries are resolved at load time.
