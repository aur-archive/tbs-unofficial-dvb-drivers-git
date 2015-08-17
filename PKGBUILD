# Submitter:   Wessel Dirksen "p-we" <wdirksen at gmail dot com>
# Contributor: Tycho LÃ1⁄4rsen "bas-t" (responsible for hosting and development media_build and media_tree lib's)
# Contributor: Luis Alves (responsible for original development of linux_media TBS OS lib's)
# Contributor: "Sunday" (tweak for speeding up gzip compression)

# !! This package installs the TBS drivers for you. They should not be pre-installed.

# !! Important:
# 1) If you use more than 4 DVB tuners, you must specify the number of actual DVB tuners you have below:
# 2) If you use TBS 6281 or 6285, option below to define to use DVB-C mode only

_dvbtuners=4
_DVB_C_only=no ## yes or no

pkgname=tbs-unofficial-dvb-drivers-git
pkgver=v150429
pkgrel=1
pkgdesc="Unofficial TBS DVB drivers from L.J. Alves (forked to bas-t repo) + firmware from official TBS source"
url="https://github.com/ljalves/linux_media/wiki"
arch=('i686' 'x86_64')
license=('GPL')
makedepends=('git' 'patchutils' 'perl-proc-processtable' 'linux-headers')
optdepends=('linuxtv-dvb-apps: handy DVB tools' 'v4l-utils: hardware support for some cards')
conflicts=('ffdecsawrapper' 'tbs-linux-drivers' 'tbs-dvb-drivers')
provides=('tbs-dvb-drivers')
install='tbs-dvb-drivers.install'

_tbsver=v150429

source=('git://github.com/bas-t/media_tree.git#branch=arch' \
	'git://github.com/bas-t/media_build.git#branch=arch' \
	"http://www.tbsdtv.com/download/document/common/tbs-linux-drivers_$_tbsver.zip" \
	'tbs-dvb-drivers.install')

sha256sums=('SKIP'
            'SKIP'
            'fdc905866a01231595e23c53b7b7b5e81428c10844215c1be1231c4a1297f743'
            'cc8be5c5704970d913c1ab1a825281827d93f49ac0e91042bd818edd434a187c')

pkgver() {

	_kernel=`uname -r | sed -r 's/-/_/g'`

	cd $srcdir/media_tree
 	_gitmedia_tree=`git describe --always | sed 's|-|.|g'`
	cd $srcdir/media_build
 	_gitmedia_build=`git describe --always | sed 's|-|.|g'`

	echo "$_gitmedia_tree"_"$_gitmedia_build"_"$_tbsver"_"$_kernel"
}

prepare() {

	    if [ "$_DVB_C_only" == "yes" ]; then
		   sed -i 's/SYS_DVBT, SYS_DVBT2, SYS_DVBC_ANNEX_A/SYS_DVBC_ANNEX_A/' $srcdir/media_tree/drivers/media/dvb-frontends/si2168.c
		   echo "compiling TBS-6285 and 6281 to DVB-C mode only..."
		   sleep 4
	    fi

	cd $srcdir
	rm -rf /linux-tbs-drivers
	tar xjvf linux-tbs-drivers.tar.bz2
	chmod 777 -R $srcdir/linux-tbs-drivers
}

build() {

	msg "Compiling with number of DVB tuners set to $(($_dvbtuners * 2))..."
	_v4lconfig="s/CONFIG_DVB_MAX_ADAPTERS=8/CONFIG_DVB_MAX_ADAPTERS=$(($_dvbtuners * 2))/"
	sleep 2
	
	cd $srcdir/media_build
	make distclean
	make dir DIR=../media_tree
	make allyesconfig
	sed -i $_v4lconfig v4l/.config
	make -j3
}

package() {

	mkdir -p $pkgdir/usr/lib/modules/`uname -r`/updates/v4l-tbs
	mkdir -p $pkgdir/usr/lib/firmware

	install -m0644 $srcdir/*dvb*.fw  $pkgdir/usr/lib/firmware
	install -m0644 $srcdir/media_tree/firmware_extra/*dvb*.fw  $pkgdir/usr/lib/firmware
	find "$srcdir/media_build" -name '*.ko' -exec cp {} $pkgdir/usr/lib/modules/`uname -r`/updates/v4l-tbs \;
	
	msg  "Compressing modules, this will take awhile..."
	find "$pkgdir" -name '*.ko' -print0 | xargs -0 -P`nproc` -n10 gzip -9
	
	chmod 755 -R $pkgdir/usr/lib/modules/`uname -r`/updates
}
