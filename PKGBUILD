# Maintainer: Philip Müller <philm[at]manjaro[dog]org>

pkgname=calamares
pkgver=3.2.17
pkgrel=3
_commit=0df4112d5eab912f5cf11c5ae7e66cfcd02b5e0d
pkgdesc='Distribution-independent installer framework'
arch=('i686' 'x86_64')
license=(GPL)
url="https://gitlab.manjaro.org/applications/calamares"
license=('LGPL')
depends=('kconfig' 'kcoreaddons' 'kiconthemes' 'ki18n' 'kio' 'solid' 'yaml-cpp' 'kpmcore3' 'mkinitcpio-openswap'
         'boost-libs' 'ckbcomp' 'hwinfo' 'qt5-svg' 'polkit-qt5' 'gtk-update-icon-cache' 'plasma-framework'
         'qt5-xmlpatterns' 'squashfs-tools') # 'pythonqt>=3.2')
makedepends=('extra-cmake-modules' 'qt5-tools' 'qt5-translations' 'git' 'boost')
backup=('usr/share/calamares/modules/bootloader.conf'
        'usr/share/calamares/modules/displaymanager.conf'
        'usr/share/calamares/modules/initcpio.conf'
        'usr/share/calamares/modules/unpackfs.conf')

source+=(#"$pkgname-$pkgver-$pkgrel.tar.gz::$url/-/archive/v$pkgver/calamares-v$pkgver.tar.gz"
         "$pkgname-$pkgver-$pkgrel.tar.gz::$url/-/archive/$_commit/$pkgname-$_commit.tar.gz"
        )
sha256sums=('acd3348b121186bfa97c8ef329ef4b37e59f057ee70a97e2821cd86cade86df7')

prepare() {
	mv ${srcdir}/calamares-${_commit} ${srcdir}/calamares-${pkgver}
	#mv ${srcdir}/calamares-v${pkgver} ${srcdir}/calamares-${pkgver}
	cd ${srcdir}/calamares-${pkgver}
	sed -i -e 's/"Install configuration files" OFF/"Install configuration files" ON/' CMakeLists.txt

	# patches here

	# change version
	_ver="$(cat CMakeLists.txt | grep -m3 -e "  VERSION" | grep -o "[[:digit:]]*" | xargs | sed s'/ /./g')"
	printf 'Version: %s-%s' "${_ver}" "${pkgrel}"
	sed -i -e "s|\${CALAMARES_VERSION_MAJOR}.\${CALAMARES_VERSION_MINOR}.\${CALAMARES_VERSION_PATCH}|${_ver}-${pkgrel}|g" CMakeLists.txt
	sed -i -e "s|CALAMARES_VERSION_RC 1|CALAMARES_VERSION_RC 0|g" CMakeLists.txt

	# change branding
	sed -i -e "s/default/manjaro/g" src/branding/CMakeLists.txt
}

build() {
	cd ${srcdir}/calamares-${pkgver}

	mkdir -p build
	cd build
        cmake .. \
              -DCMAKE_BUILD_TYPE=Release \
              -DCMAKE_INSTALL_PREFIX=/usr \
              -DCMAKE_INSTALL_LIBDIR=lib \
              -DWITH_PYTHONQT:BOOL=ON \
              -DBoost_NO_BOOST_CMAKE=ON \
              -DSKIP_MODULES="tracking webview interactiveterminal initramfs \
                              initramfscfg dracut dracutlukscfg \
                              dummyprocess dummypython dummycpp \
                              dummypythonqt services-openrc"
        make
}

package() {
	cd ${srcdir}/calamares-${pkgver}/build
	make DESTDIR="$pkgdir" install
	install -Dm644 "../data/manjaro-icon.svg" "$pkgdir/usr/share/icons/hicolor/scalable/apps/calamares.svg"
	install -Dm644 "../data/calamares.desktop" "$pkgdir/usr/share/applications/calamares.desktop"
	install -Dm755 "../data/calamares_polkit" "$pkgdir/usr/bin/calamares_polkit"
	install -Dm644 "../data/49-nopasswd-calamares.rules" "$pkgdir/etc/polkit-1/rules.d/49-nopasswd-calamares.rules"
	chmod 750      "$pkgdir"/etc/polkit-1/rules.d

	# rename services-systemd back to services
	mv "$pkgdir/usr/lib/calamares/modules/services-systemd" "$pkgdir/usr/lib/calamares/modules/services"
	mv "$pkgdir/usr/share/calamares/modules/services-systemd.conf" "$pkgdir/usr/share/calamares/modules/services.conf"
	sed -i -e 's/-systemd//' "$pkgdir/usr/lib/calamares/modules/services/module.desc"
	sed -i -e 's/-systemd//' "$pkgdir/usr/share/calamares/settings.conf"
}
