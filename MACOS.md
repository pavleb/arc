# Installation on macOS

The installation guide assumes native compile without any use of support systems like brew or ports.
The instllation requires xcode-command-line-tools that can be installed using the AppStore.

Apart from that everything else is based on GNU software that can be built from source.
This installation guide follows step-by-step the installation of the dependencies and the software itself.

## Directory structure
Installation does not require root privileges.
In all of the scripts we assume the following directory structure:

```bash
~/SRC_DIR # Directory where all the source code is downloaded
|- opt # Directory where all the dependencies are installed
|  |- bin # Directory where all the binaries are installed
|  |- include # Directory where all the header files are installed
|  |- lib # Directory where all the libraries are installed
```

## Install dependencies
If not otherwise specified, the latest stable version of the software is used.
The script below assumes that you are in the `~/SRC_DIR` directory.
For supporting tools like `xz`, `meson`, and `ninja` the easiest solution is to install `conda` or `miniconda`.
```bash
# Install conda
curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-x86_64.sh
bash Miniconda3-latest-MacOSX-x86_64.sh
# Follow the instructions and add conda to your PATH
# Restart the terminal
# Install xz, meson, and ninja
pip install meson ninja
```

### m4
```bash
curl -O https://ftp.gnu.org/gnu/m4/m4-latest.tar.xz
tar -xvf m4-latest.tar.xz
cd m4-*
./configure --prefix=$(pwd)/../opt
make
make install
cd ..
export PATH=$(pwd)/opt/bin:$PATH
```

### autoconf
```bash
curl -O https://ftp.gnu.org/gnu/autoconf/autoconf-latest.tar.xz
tar -xvf autoconf-latest.tar.xz
cd autoconf-*
./configure --prefix=$(pwd)/../opt
make
make install
cd ..
```

### automake
```bash
curl -O https://ftp.gnu.org/gnu/automake/automake-1.17.tar.xz
tar -xvf automake-1.17.tar.xz
cd automake-1.17
./configure --prefix=$(pwd)/../opt
make
make install
cd ..
```

### libtool
```bash
curl -O https://ftp.gnu.org/gnu/libtool/libtool-2.5.4.tar.xz
tar -xvf libtool-2.5.4.tar.xz
cd libtool-2.5.4
./configure --prefix=$(pwd)/../opt
make
make install
cd ..
```
### glib
```bash
curl -O https://download.gnome.org/sources/glib/2.84/glib-2.84.1.tar.xz
tar -xvf glib-2.84.1.tar.xz
cd glib-2.84.1
meson setup _build --prefix=$(pwd)/../opt
meson compile -C _build
meson install -C _build 
cd ..
```

### pkg-config
Since using newer versions we have to patch pkg-config before building.
```bash
git clone https://gitlab.freedesktop.org/pkg-config/pkg-config.git
cd pkg-config
```

```bash
cat <<EOF > mypatch.patch
diff --git a/glib/m4macros/glib-gettext.m4 b/glib/m4macros/glib-gettext.m4
index 5217fd8..9f39b5f 100644
--- a/glib/m4macros/glib-gettext.m4
+++ b/glib/m4macros/glib-gettext.m4
@@ -36,8 +36,8 @@ dnl We go to great lengths to make sure that aclocal won't
 dnl try to pull in the installed version of these macros
 dnl when running aclocal in the glib directory.
 dnl
-m4_copy([AC_DEFUN],[glib_DEFUN])
-m4_copy([AC_REQUIRE],[glib_REQUIRE])
+m4_copy_force([AC_DEFUN],[glib_DEFUN])
+m4_copy_force([AC_REQUIRE],[glib_REQUIRE])
 dnl
 dnl At the end, if we're not within glib, we'll define the public
 dnl definitions in terms of our private definitions.
EOF
```

```bash
git apply mypatch.patch

./autogen.sh --no-configure
export GLIB_CFLAGS="-I$(pwd)/../opt/include/glib-2.0 -I$(pwd)/../opt/lib/glib-2.0/include"
export GLIB_LIBS="-L$(pwd)/../opt/lib -lglib-2.0 -lgobject-2.0 -lffi"
./configure --prefix=$(pwd)/../opt
make
make install
cd ..  
export PKG_CONFIG_PATH=$(pwd)/opt/lib/pkgconfig:$PKG_CONFIG_PATH 
unset GLIB_CFLAGS
unset GLIB_LIBS
```

## mm-common
```bash
git clone https://github.com/GNOME/mm-common.git
cd mm-common
# ./autogen.sh
# ./configure --prefix=$(pwd)/../opt --enable-network
# make
# make install
# ACLOCAL_PATH="/Users/pavleb/SRC/mm-common/../opt/share/aclocal"
# export ACLOCAL_PATH
# cd ..


meson setup _build --prefix=$(pwd)/../opt -Duse-network=true
meson compile -C _build
meson install -C _build 
```

## libsigcplusplus
```bash
git clone https://github.com/libsigcplusplus/libsigcplusplus.git
cd libsigcplusplus
# export LDFLAGS="-L$(pwd)/../opt/lib"
./autogen.sh --prefix=$(pwd)/../opt
make
make install
cd ..

# meson setup _build --prefix=$(pwd)/../opt -Dbuild-documentation=false -Duse-network=true
# meson compile -C _build
# meson install -C _build 
```
### glibmm
```bash
git clone https://gitlab.gnome.org/GNOME/glibmm.git
cd glibmm
# ./autogen.sh --prefix=$(pwd)/../opt
# make
# make install

meson setup _build --prefix=$(pwd)/../opt -Dbuild-documentation=false
meson compile -C _build
meson install -C _build 

cd ..
```
### libxml2
```bash
git clone https://github.com/GNOME/libxml2.git
cd libxml2
./autogen.sh --prefix=$(pwd)/../opt
make
make install
cd ..
```
### openssl
```bash
git clone https://github.com/openssl/openssl.git
cd openssl
./Configure --prefix=$(pwd)/../opt
make install_sw
# make install
cd ..
```
### sqlite
```bash
curl -O https://www.sqlite.org/2025/sqlite-src-3490200.zip
unzip sqlite-src-3490200.zip
cd sqlite-src-3490200
./configure --prefix=$(pwd)/../opt
make
make install
cd ..
```

### gettext
The complete gettext installation is a bit tricky.
The compilation of the ``arc`` software requires only the ``autopoint`` tools.
The ``make`` process below will **fail** on macOS.
However the ``autopoint`` tool is the only one that is required and it will be generated in ``./gettext-tools/misc/autopoint``.
```bash
curl -O https://ftp.gnu.org/pub/gnu/gettext/gettext-0.25.tar.gz
tar -xvf gettext-0.25.tar.gz
cd gettext-0.25

./configure --prefix=$(pwd)/../opt 
make
# Cherry-pick the autopoint tool
cp ./gettext-tools/misc/autopoint $(pwd)/../opt/bin
cd ..   
```

### help2man
```bash
curl -O https://ftp.gnu.org/gnu/help2man/help2man-1.49.3.tar.xz
tar -xvf help2man-1.49.3.tar.xz
cd help2man-1.49.3
./configure --prefix=$(pwd)/../opt
make
make install
cd ..
``` 


# ARC compile
```bash
curl -O https://ftp.gnu.org/pub/gnu/gettext/gettext-0.25.tar.gz

./autogen.sh
export CXXFLAGS="-std=c++17 -pthread"
export LDFLAGS='-L$(pwd)/../opt/lib'
./configure --prefix=$(pwd)/../opt --disable-a-rex-service --disable-doc --disable-datadelivery-service
make
make install
cd ..
```

## Installing certificates
```bash
curl -O https://dist.igtf.net/distribution/util/fetch-crl3/fetch-crl-3.0.23.tar.gz
tar -xvf fetch-crl-3.0.23.tar.gz
cd fetch-crl-3.0.23
```

In the file ```fetch-crl.cnf``` change the line
```bash 
infodir = /etc/grid-security/certificates
```
to
```bash
infodir = $(pwd)/../opt/etc/grid-security/certificates
```
This will install the certificates in the directory where ARC is installed.
For the first run you can download the certificates from this location [https://repository.egi.eu/sw/production/cas/1/current/tgz/](https://repository.egi.eu/sw/production/cas/1/current/tgz/).
In my case I need just the certificate for Slovenian CA.
So in the folder ```~/SRC_DIR/opt/etc/grid-security/certificates``` I have the following files:
```bash
curl -O https://repository.egi.eu/sw/production/cas/1/current/tgz/ca_SiGNET-CA-1.135.tar.gz
tar -xvf ca_SiGNET-CA-1.135.tar.gz
```

This will now make it possible to run ```arcproxy``` and other commands that require certificates.







