## Description
Release **dbus-1.12.16**.
Patched (capicxx-dbus-runtime v3.1.12.7) for CommonAPI from https://github.com/freedesktop/dbus. Contains API functions used to communicate with the DBus message bus.

## Step 1. Install DBus
Should update before:
```bash
sudo apt-get install autoconf-archive
```
Clone repository and run build-configuration before compilation. We use default prefix=**/usr/local** that will be the same prefix for compilation capicxx-dbus-runtime also:
```bash
./configure --prefix=/usr/local
```
Build **libdbus** and install the package:
```bash
sudo make -C dbus 
sudo make -C dbus install
sudo make install-pkgconfigDATA
```
Now, it is ready to use.

#### (Optional) build:
This package does come with a testsuite, but it is not possible to run it because only part of the package was built.
```bash
make -C dbus lib_LTLIBRARIES=libdbus-1.la \
     install-libLTLIBRARIES \
     install-dbusincludeHEADERS \
     install-nodist_dbusarchincludeHEADERS
make install-pkgconfigDATA
```

The shared library needs to be moved to /lib, and as a result the .so file in /usr/lib will need to be recreated:
```bash
mv -v /usr/lib/libdbus-1.so.* /lib
ln -sfv ../../lib/$(readlink /usr/lib/libdbus-1.so) /usr/lib/libdbus-1.so
```

More deployment information from: http://www.linuxfromscratch.org/lfs/view/7.5-systemd/chapter06/libdbus.html

#### (Optional) patching from capicxx-dbus-runtime:

CommonAPI-DBus needs some api functions of libdbus which are not available in actual libdbus versions. For these additional api functions it is necessary to patch the required libdbus version with all the patches in the directory src/dbus-patches. Use autotools to build libdbus.
VERSION=1.12.16 is default version for Ubuntu-20.04 (as example)

```bash
wget http://dbus.freedesktop.org/releases/dbus/dbus-<VERSION>.tar.gz
tar -xzf dbus-<VERSION>.tar.gz
cd dbus-<VERSION>
patch -p1 < </path/to/capicxx-dbus-runtime/src/dbus-patches/patch-names>.patch 
./configure --prefix=</path to your preferred installation folder for patched libdbus>
make -C dbus 
sudo make -C dbus install
sudo make install-pkgconfigDATA
```

You can change the installation directory by the prefix option or you can let it uninstalled (skip the _make install_ commands).
WARNING: Installing the patched libdbus to **/usr/local** can prevent your system from booting correctly at the next reboot.

## Step 2. Install CommonAPI-core-runtime

Clone repository (sync to **v3.1.12.6** or newest):
```bash
git clone https://github.com/GENIVI/capicxx-core-runtime.git
```
Build:
```bash
cd <source-root>
mkdir build && cd build
cmake ..
make
sudo make install
```

## Step 3. Install CommonAPI-dbus-runtime

Clone repository (sync to **v3.1.12.7** or newest):
```bash
git clone https://github.com/GENIVI/capicxx-dbus-runtime.git
```

In order to build the CommonAPI-DBus-Runtime library the pkgconfig files of the patched libdbus library must be added to the _PKG_CONFIG_PATH_.

For example, if the patched _libdbus_ library is available in **/usr/local**, set the _PKG_CONFIG_PATH_ variable as follows:

```bash
export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH" 
```

Now use CMake to build the CommonAPI-DBus-runtime library. We assume that your source directory is _common-api-dbus-runtime_:

```bash
cd <source-root>
sudo rm -r build
mkdir build && cd build
cmake -D USE_INSTALLED_COMMONAPI=ON -D CMAKE_INSTALL_PREFIX=/usr/local ..
make
sudo make install
```

You can change the installation directory by the CMake variable _CMAKE_INSTALL_PREFIX_ or you can let it uninstalled (skip the _make install_ command). If you want to use the uninstalled version of CommonAPI set the CMake variable _USE_INSTALLED_COMMONAPI_ to _OFF_.

For further build instructions (build for windows, build documentation, tests etc.) please refer to the CommonAPI-DBus tutorial.

Full information list of CommonAPI-tools: http://genivi.github.io/capicxx-core-tools/
