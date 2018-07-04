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

To create a patch, just use `diff` to compare between the old source and the new source.

```sh
diff -Nuar <old source dir> <new source dir> > something.diff
```

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
* Put the patch into `debian/patches`, and add it to patch list in `debian/patches/series`.
* Clean the changes in upstream source and create the patch again for source code dir. This is the final patch.

## How to apply patch
All patches in this repository should be apply at level 1 of source code.
```sh
cd <source code dir>
patch -p1 < path/to/patch/file
```

# References

<https://wiki.debian.org/sbuild>

<https://wiki.debian.org/CrossCompiling>