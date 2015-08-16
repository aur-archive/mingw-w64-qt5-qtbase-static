# Maintainer: Filip Brcic <brcha@gna.org>

pkgname=mingw-w64-qt5-qtbase-static
pkgver=5.0.2
pkgrel=1
pkgdesc='A cross-platform application and UI framework (mingw-w64)'
arch=('i686' 'x86_64')
url="http://qt-project.org/"
license=('GPL3' 'LGPL')
depends=(
  'mingw-w64-crt'
  'mingw-w64-zlib'
  'mingw-w64-libjpeg-turbo'
  'mingw-w64-win-iconv'
  'mingw-w64-libtiff'
  'mingw-w64-sqlite3'
  'mingw-w64-libpng'
  'mingw-w64-openssl'
  'mingw-w64-dbus'
  'mingw-w64-pkg-config'
  'mingw-w64-angleproject'
)
optdepends=(
  'mingw-w64-postgresql-libs: PostgreSQL support'
  'mingw-w64-libmariadbclient: MariaDB/MySQL support'
)
makedepends=('mingw-w64-gcc')
options=(!strip !buildflags !libtool)
_pkgfqn="qtbase-opensource-src-${pkgver}"
source=("http://releases.qt-project.org/qt5/${pkgver}/submodules/${_pkgfqn}.tar.xz"
        'qmake.conf.win32'
	'qmake.conf.win64'
        'qt5-fix-detection-of-openssl.patch'
        'qt5-merge-static-and-shared-library-trees.patch'
        'qt5-add-angle-support.patch'
        'qt5-use-external-angle-library.patch'
        'qt5-dont-segfault-when-platform-dll-cant-be-found.patch'
        'qt5-define-qt-needs-qmain-for-win32-g++-targets.patch'
        'qt5-workaround-qtbug-29426.patch'
        'qt5-prevent-conflict-with-interface-keyword.patch'
        'qt5-workaround-pkgconfig-install-issue.patch'
        'qt5-dwrite-compile-fix.patch'
  )
md5sums=('a4fec8ed03867c4ee4fe5a46001a11f0'
         '630986facff0865277b4f47d397ee706'
         'f57984cac562203d27e12216f8bec53c'
         'a4c8db04484569f0d2ab8f046a3430ea'
         '91d490a5a6810d9f0502baa7838aa435'
         '2205c14dbb12040fde22d13b4796ab46'
         '9cad5d3a484f32f9fe125aced6d0a710'
         '56e8232bfffe0904095392bc61148f2d'
         '6d6ba846983910486ca3b5fc170ef518'
         'a7e42d7062e5d5d08b46de2fe81d48e9'
         '56bd328bf1ec4b9f377112869dcdb7de'
         'bdced81702cb0c81812d45e11d7f98e6'
         '815a657dc5d4d364de76febfda94eddc')

platform_win32='win32-g++-cross'
platform_win64='win32-g++-cross-x64'

# Helper functions for the split builds
isRelease() {
  [ $pkgname = "mingw-w64-qt5-qtbase" ]
}
isStatic() {
  [ $pkgname = "mingw-w64-qt5-qtbase-static" ]
}
isShared() {
  ! isStatic
}

isStatic && depends=($depends "mingw-w64-qt5-qtbase")

prepare() {
  cd "${srcdir}/${_pkgfqn}"

  # Try harder to locate external OpenSSL libraries
  # https://codereview.qt-project.org/#change,47079
  patch -p0 -i ${srcdir}/qt5-fix-detection-of-openssl.patch

  # When building Qt as static library some files have a different content
  # when compared to the static library. Merge those changes manually.
  # This patch also applies some additional changes which are required to make
  # linking against the static version of Qt work without any manual fiddling
  patch -p0 -i ${srcdir}/qt5-merge-static-and-shared-library-trees.patch
  
  # Add support for Angle
  patch -p0 -i ${srcdir}/qt5-add-angle-support.patch

  # Make sure our external Angle package is used instead of the bundled one
  patch -p0 -i ${srcdir}/qt5-use-external-angle-library.patch

  # Prevent a segfault when no suitable platform plugin could be detected
  patch -p0 -i ${srcdir}/qt5-dont-segfault-when-platform-dll-cant-be-found.patch

  # Make sure QT_NEEDS_QMAIN is defined for our mkspecs profiles
  patch -p0 -i ${srcdir}/qt5-define-qt-needs-qmain-for-win32-g++-targets.patch

  # Workaround cross-compilation issue when using a non-x86 host
  # https://bugreports.qt-project.org/browse/QTBUG-29426
  patch -p0 -i ${srcdir}/qt5-workaround-qtbug-29426.patch

  # While building qdbusviewer (part of qt5-qttools) the build fails
  # because various QtDBus headers use the keyword 'interface' which is a
  # reserved keyword on Win32 environments (defined in rpc.h)
  patch -p0 -i ${srcdir}/qt5-prevent-conflict-with-interface-keyword.patch

  # Make sure the .pc files of the Qt5 modules are installed correctly
  patch -p0 -i ${srcdir}/qt5-workaround-pkgconfig-install-issue.patch

  # Fix compilation of the DirectWrite specific code
  # Commit 9a89d614f26262fcb6895d1dab93519d732ba011
  patch -p1 -i ${srcdir}/qt5-dwrite-compile-fix.patch

  # Cross-compilation qmake target.
  mkdir mkspecs/${platform_win32}
  mkdir mkspecs/${platform_win64}
  install -m644 ${srcdir}/qmake.conf.win32 mkspecs/${platform_win32}/qmake.conf
  install -m644 ${srcdir}/qmake.conf.win64 mkspecs/${platform_win64}/qmake.conf
  install -m644 mkspecs/win32-g++/qplatformdefs.h mkspecs/${platform_win32}/
  install -m644 mkspecs/win32-g++/qplatformdefs.h mkspecs/${platform_win64}/

  # Make sure the Qt5 build system uses our external ANGLE library
  rm -rf src/3rdparty/angle
}

build() {
  cd "${srcdir}/${_pkgfqn}"

  # Setup flags
  export CFLAGS="-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions --param=ssp-buffer-size=4"
  export CXXFLAGS="-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions --param=ssp-buffer-size=4"
  unset LDFLAGS

  # Now we build

  # Generic configure arguments
  # Phonon is disabled for now because we lack the directx headers
  qt_configure_args_generic="\
    -optimized-qmake \
    -verbose \
    -opensource \
    -confirm-license \
    -force-pkg-config \
    -force-debug-info \
    -audio-backend \
    -system-zlib \
    -system-libpng \
    -system-libjpeg \
    -system-sqlite \
    -no-fontconfig \
    -iconv \
    -openssl \
    -dbus-linked \
    -no-glib \
    -no-gtkstyle \
    -no-icu \
    -release \
    -nomake examples \
    -make tools \
    -make demos"

  qt_configure_args_win32="\
    -prefix /usr/i686-w64-mingw32 \
    -bindir /usr/i686-w64-mingw32/bin \
    -archdatadir /usr/i686-w64-mingw32/share/qt5 \
    -datadir /usr/i686-w64-mingw32/share/qt5 \
    -docdir /usr/i686-w64-mingw32/share/doc/qt5 \
    -examplesdir /usr/i686-w64-mingw32/share/qt5/examples \
    -headerdir /usr/i686-w64-mingw32/include/qt5 \
    -libdir /usr/i686-w64-mingw32/lib \
    -plugindir /usr/i686-w64-mingw32/lib/qt5/plugins \
    -sysconfdir /usr/i686-w64-mingw32/etc \
    -translationdir /usr/i686-w64-mingw32/share/qt5/translations \
    -xplatform ${platform_win32}"

  qt_configure_args_win64="\
    -prefix /usr/x86_64-w64-mingw32 \
    -bindir /usr/x86_64-w64-mingw32/bin \
    -archdatadir /usr/x86_64-w64-mingw32/share/qt5 \
    -datadir /usr/x86_64-w64-mingw32/share/qt5 \
    -docdir /usr/x86_64-w64-mingw32/share/doc/qt5 \
    -examplesdir /usr/x86_64-w64-mingw32/share/qt5/examples \
    -headerdir /usr/x86_64-w64-mingw32/include/qt5 \
    -libdir /usr/x86_64-w64-mingw32/lib \
    -plugindir /usr/x86_64-w64-mingw32/lib/qt5/plugins \
    -sysconfdir /usr/x86_64-w64-mingw32/etc \
    -translationdir /usr/x86_64-w64-mingw32/share/qt5/translations \
    -xplatform ${platform_win64}"

  unset PKG_CONFIG_PATH

  # MySQL lib is -lmysql not mysqlclient, so fix that in config tests
  sed -e 's|lmysqlclient_r|lmysql|g' -i ${srcdir}/${_pkgfqn}/config.tests/unix/mysql_r/mysql_r.pro
  sed -e 's|lmysqlclient|lmysql|g' -i ${srcdir}/${_pkgfqn}/config.tests/unix/mysql/mysql.pro

  # add QMAKE_CFLAGS_MYSQL to mysql cflags
  echo 'LIBS *= $$QT_LFLAGS_MYSQL' >> ${srcdir}/${_pkgfqn}/src/sql/drivers/mysql/qsql_mysql.pri
  echo 'QMAKE_CXXFLAGS *= $$QT_CFLAGS_MYSQL' >> ${srcdir}/${_pkgfqn}/src/sql/drivers/mysql/qsql_mysql.pri

  ###############################################################################
  # Win32
  #
  # We have to build Qt two times, once for the static release build and once
  # for the shared release build
  #
  # Unfortunately Qt only supports out-of-source builds which are in ../some_folder

  # Qt doesn't detect mysql correctly, so use this:
  export QT_CFLAGS_MYSQL="-I/usr/i686-w64-mingw32/include/mysql -DBIG_JOINS=1"
  export QT_LFLAGS_MYSQL="-L/usr/i686-w64-mingw32/lib -lmysql -lpthread -lz -lm -lssl -lcrypto"
  export QT_LFLAGS_MYSQL_R="-L/usr/i686-w64-mingw32/lib -lmysql -lpthread -lz -lm -lssl -lcrypto"

  # Hardcode MySQL flags into configure (really nice solution :( )
  sed -e "s|^QT_CFLAGS_MYSQL=.*$|QT_CFLAGS_MYSQL=\"${QT_CFLAGS_MYSQL}\"|g" -i ${srcdir}/${_pkgfqn}/configure
  sed -e "s|^QT_LFLAGS_MYSQL=.*$|QT_LFLAGS_MYSQL=\"${QT_LFLAGS_MYSQL}\"|g" -i ${srcdir}/${_pkgfqn}/configure
  sed -e "s|^QT_LFLAGS_MYSQL_R=.*$|QT_LFLAGS_MYSQL_R=\"${QT_LFLAGS_MYSQL_R}\"|g" -i ${srcdir}/${_pkgfqn}/configure

  qt_configure_args_mysql="-mysql_config /this/file/should/not/exist"

  if isStatic
  then
    rm -rf ../build_release_static_win32
    mkdir ../build_release_static_win32
    pushd ../build_release_static_win32
    ../${_pkgfqn}/configure \
      -static \
      $qt_configure_args_win32 $qt_configure_args_generic $qt_configure_args_mysql
    make
    popd
  fi

  if isRelease
  then
    rm -rf ../build_release_shared_win32
    mkdir ../build_release_shared_win32
    pushd ../build_release_shared_win32
    ../${_pkgfqn}/configure \
      -shared \
      $qt_configure_args_win32 $qt_configure_args_generic $qt_configure_args_mysql
    make
    popd
  fi

  ###############################################################################
  # Win64
  #
  # We have to build Qt two times, once for the static release build and once
  # for the shared release build
  #
  # Unfortunately Qt only supports out-of-source builds which are in ../some_folder

  # Qt doesn't detect mysql correctly, so use this:
  export QT_CFLAGS_MYSQL="-I/usr/x86_64-w64-mingw32/include/mysql -DBIG_JOINS=1"
  export QT_LFLAGS_MYSQL="-L/usr/x86_64-w64-mingw32/lib -lmysql -lpthread -lz -lm -lssl -lcrypto"
  export QT_LFLAGS_MYSQL_R="-L/usr/x86_64-w64-mingw32/lib -lmysql -lpthread -lz -lm -lssl -lcrypto"

  # Hardcode MySQL flags into configure (really nice solution :( )
  sed -e "s|^QT_CFLAGS_MYSQL=.*$|QT_CFLAGS_MYSQL=\"${QT_CFLAGS_MYSQL}\"|g" -i ${srcdir}/${_pkgfqn}/configure
  sed -e "s|^QT_LFLAGS_MYSQL=.*$|QT_LFLAGS_MYSQL=\"${QT_LFLAGS_MYSQL}\"|g" -i ${srcdir}/${_pkgfqn}/configure
  sed -e "s|^QT_LFLAGS_MYSQL_R=.*$|QT_LFLAGS_MYSQL_R=\"${QT_LFLAGS_MYSQL_R}\"|g" -i ${srcdir}/${_pkgfqn}/configure

  qt_configure_args_mysql="-mysql_config /this/file/should/not/exist"

  if isStatic
  then
    rm -rf ../build_release_static_win64
    mkdir ../build_release_static_win64
    pushd ../build_release_static_win64
    ../${_pkgfqn}/configure \
      -static \
      $qt_configure_args_win64 $qt_configure_args_generic $qt_configure_args_mysql
    make
    popd
  fi

  if isRelease
  then
    rm -rf ../build_release_shared_win64
    mkdir ../build_release_shared_win64
    pushd ../build_release_shared_win64
    ../${_pkgfqn}/configure \
      -shared \
      $qt_configure_args_win64 $qt_configure_args_generic $qt_configure_args_mysql
    make
    popd
  fi
}

package() {
  cd $srcdir/${_pkgfqn}
  if isRelease
  then
    make install -C ../build_release_shared_win32 INSTALL_ROOT=${pkgdir}
    make install -C ../build_release_shared_win64 INSTALL_ROOT=${pkgdir}
  fi

  if isStatic
  then
    # Install the static libraries in a temporary prefix so we can merge everything together properly
    mkdir ${pkgdir}/static
    make install -C ../build_release_static_win32 INSTALL_ROOT=${pkgdir}/static
    make install -C ../build_release_static_win64 INSTALL_ROOT=${pkgdir}/static
  
    # The pkg-config files for Qt5Bootstrap aren't interesting as this particular
    # library only contains native code and not cross-compiled code
    rm -f ${pkgdir}/usr/{i686,x86_64}-w64-mingw32/lib/pkgconfig/Qt5Bootstrap.pc

    # Drop the qtmain and Qt5Bootstrap static libraries from the static tree as
    # they are already part of the main tree
    rm -f ${pkgdir}/static/usr/{i686,x86_64}-w64-mingw32/lib/libqtmain*
    rm -f ${pkgdir}/static/usr/{i686,x86_64}-w64-mingw32/lib/libQt5Bootstrap*

  else # not static => shared release

    # Rename the .a files to .dll.a as they're actually import libraries and not static libraries
    for FN in ${pkgdir}/usr/i686-w64-mingw32/lib/*.a ${pkgdir}/usr/x86_64-w64-mingw32/lib/*.a ; do
      # Ignore libqtmain*.a
      echo $FN | grep -q qtmain && continue
      echo $FN | grep -q libQt5Bootstrap && continue

      # Rename the file
      FN_NEW=$(echo $FN | sed s/'.a$'/'.dll.a'/)
      mv $FN $FN_NEW
    done
  fi

  if isStatic
  then
    # Move the static libraries from the static tree to the main tree
    mv ${pkgdir}/static/usr/i686-w64-mingw32/lib/*.a ${pkgdir}/usr/i686-w64-mingw32/lib/
    mv ${pkgdir}/static/usr/x86_64-w64-mingw32/lib/*.a ${pkgdir}/usr/x86_64-w64-mingw32/lib/

    # Clean up the static trees as we've now merged all interesting pieces
    rm -rf ${pkgdir}/static
  fi

  if isShared
  then
    # Rename qtmain.a to a non-conflicting file name
    # The updated filename is already set correctly in the bundled mkspecs profiles
    mv ${pkgdir}/usr/i686-w64-mingw32/lib/libqtmain.a ${pkgdir}/usr/i686-w64-mingw32/lib/libqt5main.a
    mv ${pkgdir}/usr/x86_64-w64-mingw32/lib/libqtmain.a ${pkgdir}/usr/x86_64-w64-mingw32/lib/libqt5main.a

    # The .dll's are installed in both bindir and libdir
    # One copy of the .dll's is sufficient
    rm -f ${pkgdir}/usr/i686-w64-mingw32/lib/*.dll
    rm -f ${pkgdir}/usr/x86_64-w64-mingw32/lib/*.dll

    # Drop all the files which we don't need
    rm -f ${pkgdir}/usr/i686-w64-mingw32/lib/*.prl
    rm -f ${pkgdir}/usr/x86_64-w64-mingw32/lib/*.prl

    rm -f ${pkgdir}/usr/i686-w64-mingw32/lib/libQt5Bootstrap.la
    rm -f ${pkgdir}/usr/x86_64-w64-mingw32/lib/libQt5Bootstrap.la

    # Manually install qmake and other native tools so we don't depend anymore on
    # the version of the native system Qt and also fix issues as illustrated at
    # http://stackoverflow.com/questions/6592931/building-for-windows-under-linux-using-qt-creator
    #
    # Also make sure the tools can be found by CMake
    mkdir -p ${pkgdir}/usr/bin
    mkdir -p ${pkgdir}/usr/i686-w64-mingw32/bin
    mkdir -p ${pkgdir}/usr/x86_64-w64-mingw32/bin

    for tool in qmake moc rcc uic qdbuscpp2xml qdbusxml2cpp qdoc syncqt
    do
      mv ${pkgdir}/usr/i686-w64-mingw32/bin/${tool} ${pkgdir}/usr/i686-w64-mingw32/bin/${tool}-qt5
      ln -s ../i686-w64-mingw32/bin/${tool}-qt5 ${pkgdir}/usr/bin/i686-w64-mingw32-${tool}-qt5
    done
    ln -s i686-w64-mingw32-qmake-qt5 ${pkgdir}/usr/bin/mingw32-qmake-qt5

    for tool in qmake moc rcc uic qdbuscpp2xml qdbusxml2cpp qdoc syncqt
    do
      mv ${pkgdir}/usr/x86_64-w64-mingw32/bin/${tool} ${pkgdir}/usr/x86_64-w64-mingw32/bin/${tool}-qt5
      ln -s ../x86_64-w64-mingw32/bin/${tool}-qt5 ${pkgdir}/usr/bin/x86_64-w64-mingw32-${tool}-qt5
    done
    ln -s x86_64-w64-mingw32-qmake-qt5 ${pkgdir}/usr/bin/mingw64-qmake-qt5

    # Make sure that all Qt projects use the tools which are provided by this package
    sed -i s@'#QT_TOOL'@'QT_TOOL'@ ${pkgdir}/usr/i686-w64-mingw32/share/qt5/mkspecs/${platform_win32}/qmake.conf
    sed -i s@'#QT_TOOL'@'QT_TOOL'@ ${pkgdir}/usr/i686-w64-mingw32/share/qt5/mkspecs/${platform_win64}/qmake.conf
    sed -i s@'#QT_TOOL'@'QT_TOOL'@ ${pkgdir}/usr/x86_64-w64-mingw32/share/qt5/mkspecs/${platform_win32}/qmake.conf
    sed -i s@'#QT_TOOL'@'QT_TOOL'@ ${pkgdir}/usr/x86_64-w64-mingw32/share/qt5/mkspecs/${platform_win64}/qmake.conf
  fi

  # And finaly, strip the binaries
  if isShared
  then
    i686-w64-mingw32-strip --strip-unneeded ${pkgdir}/usr/i686-w64-mingw32/bin/*.dll
    x86_64-w64-mingw32-strip --strip-unneeded ${pkgdir}/usr/x86_64-w64-mingw32/bin/*.dll
  fi

  if isStatic
  then
    i686-w64-mingw32-strip --strip-debug ${pkgdir}/usr/i686-w64-mingw32/lib/*.a # static libs
    x86_64-w64-mingw32-strip --strip-debug ${pkgdir}/usr/x86_64-w64-mingw32/lib/*.a # static libs
  else
    i686-w64-mingw32-strip --strip-unneeded ${pkgdir}/usr/i686-w64-mingw32/lib/libqt5main*.a
    i686-w64-mingw32-strip --strip-unneeded ${pkgdir}/usr/i686-w64-mingw32/lib/*.dll.a # dynamic lib interfaces
    i686-w64-mingw32-strip --strip-unneeded ${pkgdir}/usr/i686-w64-mingw32/lib/qt5/plugins/*/*.dll
    x86_64-w64-mingw32-strip --strip-unneeded ${pkgdir}/usr/x86_64-w64-mingw32/lib/libqt5main*.a
    x86_64-w64-mingw32-strip --strip-unneeded ${pkgdir}/usr/x86_64-w64-mingw32/lib/*.dll.a # dynamic lib interfaces
    x86_64-w64-mingw32-strip --strip-unneeded ${pkgdir}/usr/x86_64-w64-mingw32/lib/qt5/plugins/*/*.dll
    strip --strip-debug ${pkgdir}/usr/{i686,x86_64}-w64-mingw32/lib/libQt5Bootstrap.a # ELF library, needed for qt5-qttools
  fi
}
