Updated Windows build instructions based on https://bitcointalk.org/index.php?topic=149479.0


---------------------
1.1 Install msys shell:
---------------------

http://sourceforge.net/projects/mingw/files/Installer/mingw-get-setup.exe/download
From MinGW installation manager -> All packages -> MSYS
mark the following for installation:

msys-base-bin
msys-autoconf-bin
msys-automake-bin
msys-libtool-bin

then click on Installation -> Apply changes

Make sure no mingw packages are checked for installation or present from a previous install. Only the above msys packages should be installed. Also make sure that msys-gcc and msys-w32api packages are not installed.

---------------------
1.2 Install MinGW-builds project toolchain:
---------------------

Download http://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win32/Personal%20Builds/mingw-builds/4.9.1/threads-posix/dwarf/i686-4.9.1-release-posix-dwarf-rt_v3-rev1.7z/download
and unpack it to C:\

---------------------
1.3. Ensure that mingw-builds bin folder is set in your PATH environment variable. On Windows 7 your path should look something like:
---------------------

C:\mingw32\bin;%SystemRoot%\system32;%SystemRoot%;%SystemRoot%\System32\Wbem;%SYSTEMROOT%\System32\WindowsPowerShell\v1.0\

---------------------
1.4 Additional checks:
---------------------

C:\MinGW\bin should contain nothing but mingw-get.exe.

---------------------
2. Download, unpack and build required dependencies.
---------------------

Save them in C:\deps\ folder.
All commands are to be enterred into the msys (linux) shell, unless windows cmd prompt is specified.

---------------------
2.1 OpenSSL: http://www.openssl.org/source/openssl-1.0.1j.tar.gz
---------------------

cd /c/deps
tar xvfz openssl-1.0.1j.tar.gz
cd openssl-1.0.1j
./configure no-zlib no-shared no-dso no-krb5 no-camellia no-capieng no-cast no-cms no-dtls1 no-gost no-gmp no-heartbeats no-idea no-jpake no-md2 no-mdc2 no-rc5 no-rdrand no-rfc3779 no-rsax no-sctp no-seed no-sha0 no-static_engine no-whirlpool no-rc2 no-rc4 no-ssl2 no-ssl3 mingw
make depend
make

---------------------
2.2 Berkeley DB: http://download.oracle.com/berkeley-db/db-4.8.30.NC.tar.gz
---------------------

cd /c/deps
tar xvfz db-4.8.30.NC.tar.gz
cd db-4.8.30.NC/build_unix
../dist/configure --enable-mingw --enable-cxx --disable-shared --disable-replication
make

---------------------
2.3 Boost: http://sourceforge.net/projects/boost/files/boost/1.55.0/
**Download either the zip or the 7z archive, unpack boost inside your C:\deps folder, then bootstrap and compile from a Windows command prompt:
---------------------

(windows cmd prompt)
cd C:\deps\boost_1_55_0\
bootstrap.bat mingw
b2 --build-type=complete --with-chrono --with-filesystem --with-program_options --with-system --with-thread toolset=gcc variant=release link=static threading=multi runtime-link=static stage

---------------------
2.5 protoc and libprotobuf:
Download and unpack http://protobuf.googlecode.com/files/protobuf-2.5.0.zip
---------------------

cd /c/deps/protobuf-2.5.0
configure --disable-shared
make

---------------------
2.6 libpng:
Download and unpack http://prdownloads.sourceforge.net/libpng/libpng-1.6.14.tar.gz?download inside your deps folder then configure and make:
Download and unpack http://fukuchi.org/works/qrencode/qrencode-3.4.4.tar.gz inside your deps folder then configure and make:
---------------------

tar xvfz libpng-1.6.14.tar.gz
cd /c/deps/libpng-1.6.14
configure --disable-shared
make
cp .libs/libpng16.a .libs/libpng.a

---------------------
2.7 qrencode:
---------------------

tar xvfz qrencode-3.4.4.tar.gz
cd /c/deps/qrencode-3.4.4
LIBS="../libpng-1.6.14/.libs/libpng.a ../../mingw32/i686-w64-mingw32/lib/libz.a" \
png_CFLAGS="-I../libpng-1.6.14" \
png_LIBS="-L../libpng-1.6.14/.libs" \
configure --enable-static --disable-shared --without-tools
make

---------------------
2.8 Qt:
http://download.qt-project.org/official_releases/qt/5.3/5.3.2/submodules/qtbase-opensource-src-5.3.2.7z
http://download.qt-project.org/official_releases/qt/5.3/5.3.2/submodules/qttools-opensource-src-5.3.2.7z
(note that the following assumes qtbase has been unpacked to C:\Qt\5.3.2 and qttools have been unpacked to C:\Qt\qttools-opensource-src-5.3.2)
---------------------

(windows cmd prompt)
set INCLUDE=C:\deps\libpng-1.6.14;C:\deps\openssl-1.0.1j\include
set LIB=C:\deps\libpng-1.6.14\.libs;C:\deps\openssl-1.0.1j
set MAKE=C:\mingw32\bin\mingw32-make.exe
cd C:\Qt\5.3.2
configure.bat -release -opensource -confirm-license -static -make libs -no-sql-sqlite -no-opengl -system-zlib -qt-pcre -no-icu -no-gif -system-libpng -no-libjpeg -no-freetype -no-angle -no-vcproj -openssl -no-dbus -no-audio-backend -no-wmf-backend -no-qml-debug
mingw32-make

set PATH=%PATH%;C:\Qt\5.3.2\bin
cd C:\Qt\qttools-opensource-src-5.3.2
qmake qttools.pro
mingw32-make





---------------------
Configure and build Dimecoin:
---------------------

cd /c/dimecoin
MAKE=/c/mingw32/bin/mingw32-make.exe
./autogen.sh

(the following is a single command)

CPPFLAGS="-I/c/deps/boost_1_55_0 \
-I/c/deps/db-4.8.30.NC/build_unix \
-I/c/deps/openssl-1.0.1j/include \
-I/c/deps/protobuf-2.5.0/src \
-I/c/deps/libpng-1.6.14 \
-I/c/deps/qrencode-3.4.4" \
LDFLAGS="-L/c/deps/boost_1_55_0/stage/lib \
-L/c/deps/db-4.8.30.NC/build_unix \
-L/c/deps/openssl-1.0.1j \
-L/c/deps/protobuf-2.5.0/src/.libs \
-L/c/deps/libpng-1.6.14/.libs \
-L/c/deps/qrencode-3.4.4/.libs" \
./configure \
--without-miniupnpc \
--disable-tests \
--with-qt-incdir=/c/Qt/5.3.2/include \
--with-qt-libdir=/c/Qt/5.3.2/lib \
--with-qt-bindir=/c/Qt/5.3.2/bin \
--with-qt-plugindir=/c/Qt/5.3.2/plugins \
--with-boost-system=mgw49-mt-s-1_55 \
--with-boost-filesystem=mgw49-mt-s-1_55 \
--with-boost-program-options=mgw49-mt-s-1_55 \
--with-boost-thread=mgw49-mt-s-1_55 \
--with-boost-chrono=mgw49-mt-s-1_55 \
--with-protoc-bindir=/c/deps/protobuf-2.5.0/src

make

strip src/dimecoin-cli.exe
strip src/dimecoind.exe
strip src/qt/dimecoin-qt.exe

(run /c/dimecoin/src/qt/dimecoin-qt.exe to start GUI wallet) 
