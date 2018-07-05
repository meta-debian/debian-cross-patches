# Debian Cross Build

## How to cross build

We often use `dpkg-buildpackage` and `sbuild` to cross build a Debian package.

### Cross build with dpkg-buildpackage

* Install a Debian Unstable system.

* Enable the foreign architecture and install cross toolchain.

  For example "armhf":

  ```sh
  sudo dpkg --add-architecture armhf
  sudo apt-get update
  sudo apt-get install build-essential crossbuild-essential-armhf
  ```

* Download source code, which we want to cross build, and install its dependencies.

  ```sh
  apt-get source <package>
  sudo apt-get build-dep -aarmhf <package>
  ```

* Cross build with dpkg-buildpackage.

  ```sh
  cd <package>-<version>
  CONFIG_SITE=/etc/dpkg-cross/cross-config.armhf DEB_BUILD_OPTIONS=nocheck dpkg-buildpackage -aarmhf
  ```

### Cross build with sbuild

* Set up a sbuild chroot for Debian Unstable.

  ```sh
  # Configure user
  sudo mkdir /root/.gnupg # To work around #792100; not needed on post-Jessie
  sudo sbuild-update --keygen # Not needed since sbuild 0.67.0 (postJessie, see #801798)
  sudo sbuild-adduser $LOGNAME

  # logout and re-login or use `newgrp sbuild` in your current shell

  # Create chroot
  sudo sbuild-createchroot unstable ./chroot-sbuild http://ftp.debian.org/debian
  ```

  *Note: On Debian Jessie, sbuild needs to be installed from jessie-backports (Bug [#827315](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=827315))*

* Cross build with sbuild.

  ```sh
  cd <source code dir>
  sbuild --host=armhf -d unstable
  ```

## How to create patch

### Create patch for upstream source

A Debian source package has structure like this:

```
package
├── Makefile
├── file.c
├── file.h
├── debian
│   ├── control
│   ├── rules
│   ├── patches
│   │   ├── patch1.patch
│   │   ├── patch2.patch
│   │   └── ...
│   └── ...
└── ...
```

If we change the files outside of `debian` subdir, we need create patches for those changes and put them into `debian/patches` first.

* Create patch for the changes in upstream source.

  ```sh
  diff -Nuar old_source_dir new_source_dir > patchname.patch
  ```

* Put the patch into `debian/patches`, and add it to patch list in `debian/patches/series`.
* Clean the changes in upstream source.

### Create the final patch

After successful build with `dpkg-buildpackage` or `sbuild`, files *.debian.tar.xz and *.dsc will be updated automatically with modified codes.

Example:

```sh
.
├── coreutils-8.28/
├── coreutils_8.28-1_armhf.buildinfo
├── coreutils_8.28-1_armhf.changes
├── coreutils_8.28-1_armhf.deb
├── coreutils_8.28-1.debian.tar.xz # this file contains new codes of 'debian' subdir
├── coreutils_8.28-1.dsc # dsc has been updated with new checksum
├── coreutils_8.28.orig.tar.xz
└── ...
```

We can compare the new with the old source code by `debdiff` 2 files *.dsc.

```sh
mkdir orig
cd orig
# Download the original source code from Debian
# (*.orig.tar.*, *.debian.tar.*, *.dsc, etc.)

cd ..
debdiff filename.dsc orig/filename.dsc > patchname.debdiff
```

## How to apply patch
All patches in this repository should be apply at level 1 of source code.
```sh
cd <source code dir>
patch -p1 < path/to/patch/file
```

# References

<https://wiki.debian.org/sbuild>

<https://wiki.debian.org/CrossCompiling>