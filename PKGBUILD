# $Id: PKGBUILD 110942 2014-05-10 09:33:56Z lcarlier $
# Maintainer: Fantix King <fantix.king@gmail.com>
# Upstream Maintainer: Jan Alexander Steffens (heftig) <jan.steffens@gmail.com>
# Contributor: Allan McRae <allan@archlinux.org>

# toolchain build order: linux-api-headers->glibc->binutils->gcc->binutils->glibc
# NOTE: libtool requires rebuilt with each new gcc version

pkgname='gcc-x32-seed'
pkgver=4.9.0_2
_pkgver=4.9
pkgrel=1
_snapshot=4.9-20140507
pkgdesc="An incomplete GCC build for building Glibc with x32 ABI support"
arch=('x86_64')
license=('GPL' 'LGPL' 'FDL' 'custom')
url="http://gcc.gnu.org"
makedepends=('binutils>=2.24' 'libmpc' 'cloog' 'doxygen'
             'lib32-glibc>=2.19')
checkdepends=('dejagnu' 'inetutils')
depends=('binutils>=2.24' 'libmpc' 'cloog' 'glibc-x32-seed')
provides=("gcc-multilib-x32=${pkgver}-${pkgrel}")
options=('!emptydirs')
source=(#ftp://gcc.gnu.org/pub/gcc/releases/gcc-${pkgver%_*}/gcc-${pkgver%_*}.tar.bz2
        ftp://gcc.gnu.org/pub/gcc/snapshots/${_snapshot}/gcc-${_snapshot}.tar.bz2
        gcc-4.8-filename-output.patch
       gcc-4.9-tree-ssa-threadedge.patch
	https://sites.google.com/site/x32abi/documents/gcc-x32-seed.tar.bz2)
md5sums=('47dc2b91d2876daff53c20c30164c38f'
         '40cb437805e2f7a006aa0d0c3098ab0f'
         '311ece7f5446d550e84e28692d2fb823'
         '1672d3283e6827b8b8ff634f85ddf523')


if [ -n "${_snapshot}" ]; then
  _basedir=gcc-${_snapshot}
else
  _basedir=gcc-${pkgver%_*}
fi

_libdir="opt/gcc-x32-seed/lib/gcc/$CHOST/${pkgver%_*}"

prepare() {
  cd ${srcdir}/${_basedir}

  # Do not run fixincludes
  sed -i 's@\./fixinc\.sh@-c true@' gcc/Makefile.in

  # Arch Linux installs x86_64 libraries /lib
  [[ $CARCH == "x86_64" ]] && sed -i '/m64=/s/lib64/lib/' gcc/config/i386/t-linux64

  echo ${pkgver%_*} > gcc/BASE-VER

  # hack! - some configure tests for header files using "$CPP $CPPFLAGS"
  sed -i "/ac_cpp=/s/\$CPPFLAGS/\$CPPFLAGS -O2/" {libiberty,gcc}/configure

  # http://gcc.gnu.org/bugzilla/show_bug.cgi?id=57653
  patch -p0 -i ${srcdir}/gcc-4.8-filename-output.patch
  
  # http://gcc.gnu.org/bugzilla/show_bug.cgi?id=60902
  patch -p1 -i ${srcdir}/gcc-4.9-tree-ssa-threadedge.patch

  mkdir ${srcdir}/gcc-build
}

build() {
  cd ${srcdir}/gcc-build

  # using -pipe causes spurious test-suite failures
  # http://gcc.gnu.org/bugzilla/show_bug.cgi?id=48565
  CFLAGS=${CFLAGS/-pipe/}
  CXXFLAGS=${CXXFLAGS/-pipe/}

  ${srcdir}/${_basedir}/configure --prefix=/opt/gcc-x32-seed \
      --libdir=/opt/gcc-x32-seed/lib --libexecdir=/opt/gcc-x32-seed/lib \
      --mandir=/opt/gcc-x32-seed/share/man --infodir=/opt/gcc-x32-seed/share/info \
      --with-bugurl=https://bugs.archlinux.org/ \
      --enable-languages=c,c++ \
      --enable-shared --enable-threads=posix \
      --with-system-zlib --enable-__cxa_atexit \
      --disable-libunwind-exceptions --enable-clocale=gnu \
      --disable-libstdcxx-pch --disable-libssp \
      --enable-gnu-unique-object --enable-linker-build-id \
      --enable-cloog-backend=isl --disable-cloog-version-check \
      --enable-lto --enable-plugin --enable-install-libiberty \
      --with-linker-hash-style=gnu \
      --enable-multilib --disable-werror \
      --with-multilib-list=m32,m64,mx32 \
      --enable-checking=release
  make || true

  rmdir fixincludes || true
  ln -s build-*/fixincludes || true
}

package()
{
  options=('staticlibs')

  cd ${srcdir}/gcc-build

  make -C gcc DESTDIR=${pkgdir} install-driver install-cpp install-gcc-ar \
    c++.install-common install-headers install-plugin install-lto-wrapper

  install -m755 gcc/gcov $pkgdir/opt/gcc-x32-seed/bin/
  install -m755 -t $pkgdir/${_libdir}/ gcc/{cc1,cc1plus,collect2,lto1}

  make -C $CHOST/libgcc DESTDIR=${pkgdir} install || true
  make -C $CHOST/32/libgcc DESTDIR=${pkgdir} install
  make -C $CHOST/x32/libgcc DESTDIR=${pkgdir} install

  make DESTDIR=${pkgdir} install-fixincludes
  make -C gcc DESTDIR=${pkgdir} install-mkheaders
  make -C lto-plugin DESTDIR=${pkgdir} install

  make -C libiberty DESTDIR=${pkgdir} install
  # install PIC version of libiberty
  install -m644 ${srcdir}/gcc-build/libiberty/pic/libiberty.a ${pkgdir}/opt/gcc-x32-seed/lib


  make -C libcpp DESTDIR=${pkgdir} install

  # many packages expect this symlinks
  ln -s gcc ${pkgdir}/opt/gcc-x32-seed/bin/cc

  # POSIX conformance launcher scripts for c89 and c99
  cat > $pkgdir/opt/gcc-x32-seed/bin/c89 <<"EOF"
#!/bin/sh
fl="-std=c89"
for opt; do
  case "$opt" in
    -ansi|-std=c89|-std=iso9899:1990) fl="";;
    -std=*) echo "`basename $0` called with non ANSI/ISO C option $opt" >&2
	    exit 1;;
  esac
done
exec gcc $fl ${1+"$@"}
EOF

  cat > $pkgdir/opt/gcc-x32-seed/bin/c99 <<"EOF"
#!/bin/sh
fl="-std=c99"
for opt; do
  case "$opt" in
    -std=c99|-std=iso9899:1999) fl="";;
    -std=*) echo "`basename $0` called with non ISO C99 option $opt" >&2
	    exit 1;;
  esac
done
exec gcc $fl ${1+"$@"}
EOF

  chmod 755 $pkgdir/opt/gcc-x32-seed/bin/c{8,9}9

  # Fix a wrong symbol link
  rm ${pkgdir}/opt/gcc-x32-seed/libx32/libgcc_s.so
  ln -s ../lib/gcc/x86_64-unknown-linux-gnu/4.8.2/x32/libgcc_s.so.1 \
      ${pkgdir}/opt/gcc-x32-seed/libx32/libgcc_s.so

  # Install x32 seed
  install -dm755 ${pkgdir}/opt/gcc-x32-seed/lib/gcc/$CHOST/${pkgver%_*}/x32
  cp ${srcdir}/x32/* ${pkgdir}/opt/gcc-x32-seed/lib/gcc/$CHOST/${pkgver%_*}/x32
}

