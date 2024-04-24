pkgname=safe-tpm2-git
pkgdesc="Securely store secrets in a safe"
pkgver=r4.ac737ac
pkgrel=1

source=("git+https://github.com/kkernick/safe.git")
sha256sums=("SKIP")
depends=(systemd clevis openssl python-blessed python-termcolor gocryptfs sudo oath-toolkit plasma-workspace qt6-tools)
optdepends=(handle-tpm)
arch=("any")
provides=("safe-tpm2")

pkgver() {
	cd $srcdir/safe
	printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short=7 HEAD)"
}

package() {
	cd $srcdir/safe
	for binary in safe; do
		install -Dm755 "$binary" "$pkgdir/usr/bin/$binary"
	done
}
