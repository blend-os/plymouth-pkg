pkgname=plymouth-blend
pkgver=22.02.122
pkgrel=10
pkgdesc="A graphical boot splash screen with kernel mode-setting support"
url="https://www.freedesktop.org/wiki/Software/Plymouth/"
arch=('i686' 'x86_64')
provides=('plymouth')
conflicts=('plymouth')
license=('GPL')

depends=('libdrm' 'pango' 'systemd')
makedepends=('docbook-xsl')
optdepends=('cantarell-fonts: For text output, e.g. during decryption'
        'xf86-video-fbdev: Support special graphic cards on early startup')
backup=('etc/plymouth/plymouthd.conf')

source=("https://gitlab.freedesktop.org/plymouth/plymouth/-/archive/${pkgver}/plymouth-${pkgver}.tar.gz"
        "https://github.com/blend-os/plymouth/tarball/master")

sha256sums=('8921cd61a9f32f5f8903ceffb9ab0defaef8326253e1549ef85587c19b7f2ab6'
            'SKIP')

prepare() {
	tar xf master
	cp blend-os-plymouth-*/* .

	cd "$srcdir"/plymouth-${pkgver}
	patch -p1 -i $srcdir/plymouth-update-initrd.patch
	patch -p1 -i $srcdir/plymouth-quit.service.in.patch
	patch -p1 -i $srcdir/plymouthd.conf.patch
	# apply upstream patches
	patch -p1 -i $srcdir/ply-utils.c.patch
	patch -p1 -i $srcdir/runstatedir.patch
}

build() {
	cd "$srcdir"/plymouth-${pkgver}

	LDFLAGS="$LDFLAGS -ludev" ./autogen.sh \
		--prefix=/usr \
		--exec-prefix=/usr \
		--sysconfdir=/etc \
		--localstatedir=/var \
		--libdir=/usr/lib \
		--libexecdir=/usr/lib \
		--sbindir=/usr/bin \
		--enable-systemd-integration \
		--enable-drm \
		--enable-tracing \
		--enable-pango \
		--enable-gtk=no \
		--with-release-file=/etc/os-release \
		--with-logo=/usr/share/plymouth/blendos-logo.png \
		--with-background-color=0x000000 \
		--with-background-start-color-stop=0x000000 \
		--with-background-end-color-stop=0x4D4D4D \
		--without-rhgb-compat-link \
		--without-system-root-install \
		--runstatedir=/run

	make
}

package() {
	cd "$srcdir"/plymouth-${pkgver}

	make DESTDIR="$pkgdir" install

	install -Dm644 "$srcdir/blendos-logo.png" "$pkgdir/usr/share/plymouth/blendos-logo.png"
	ln -s ../../blendos-logo.png "$pkgdir/usr/share/plymouth/themes/spinner/watermark.png"

	install -Dm644 "$srcdir/plymouth.encrypt_hook" "$pkgdir/usr/lib/initcpio/hooks/plymouth-encrypt"
	install -Dm644 "$srcdir/plymouth.encrypt_install" "$pkgdir/usr/lib/initcpio/install/plymouth-encrypt"
	install -Dm644 "$srcdir/plymouth.initcpio_hook" "$pkgdir/usr/lib/initcpio/hooks/plymouth"
	install -Dm644 "$srcdir/plymouth.initcpio_install" "$pkgdir/usr/lib/initcpio/install/plymouth"
	install -Dm644 "$srcdir/sd-plymouth.initcpio_install" "$pkgdir/usr/lib/initcpio/install/sd-plymouth"

	for i in {sddm,lxdm,lightdm}-plymouth.service; do
		install -Dm644 "$srcdir/$i" "$pkgdir/usr/lib/systemd/system/$i"
	done

	install -Dm644 "$srcdir/plymouth-deactivate.service" 	"$pkgdir/usr/lib/systemd/system/plymouth-deactivate.service"
	install -Dm644 "$pkgdir/usr/share/plymouth/plymouthd.defaults" "$pkgdir/etc/plymouth/plymouthd.conf"
}
