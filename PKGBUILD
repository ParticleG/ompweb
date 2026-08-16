# Maintainer: ParticleG <particle_g@outlook.com>

pkgname=ompweb
pkgver=0.3.1
pkgrel=1
pkgdesc='Web UI for the oh-my-pi coding agent'
arch=('x86_64')
url='https://github.com/kahme247/ompweb'
license=('MIT')
depends=('nodejs>=22.19.0' 'oh-my-pi')
makedepends=('npm')
options=('!strip' '!debug')

source=("$pkgname-$pkgver.tgz::https://registry.npmjs.org/@kahme247/ompweb/-/ompweb-$pkgver.tgz")
sha256sums=('f67577370e99605a84b793f8b9d3d7e3a1f83c155a857e1b99928ab6a311d965')

package() {
  npm install --global \
    --prefix "$pkgdir/usr" \
    --cache "$srcdir/npm-cache" \
    --omit=dev \
    --ignore-scripts \
    "$srcdir/$pkgname-$pkgver.tgz"

  install -Dm644 \
    "$pkgdir/usr/lib/node_modules/@kahme247/ompweb/LICENSE" \
    "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
