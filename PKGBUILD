# Maintainer: build local — libfprint MR #547 "Implement SDCP v2 (including egismoc implementation)"
# https://gitlab.freedesktop.org/libfprint/libfprint/-/merge_requests/547
# Basé sur le PKGBUILD officiel Arch (Jan Alexander Steffens), source remplacée par le commit du MR.

pkgname=libfprint-sdcp-v2
pkgver=1.94.100.r547.g2d7c5277
pkgrel=1
pkgdesc="Library for fingerprint readers (egismoc SDCP v2, MR #547 — fix capteur Egis 1c7a:05a1)"
url="https://gitlab.freedesktop.org/libfprint/libfprint/-/merge_requests/547"
arch=(x86_64)
license=(LGPL-2.1-or-later)
depends=(
  libgcc
  glib2
  glibc
  libgudev
  libgusb
  openssl
  pixman
)
makedepends=(
  git
  glib2-devel
  gobject-introspection
  gtk-doc
  meson
  python-cairo
  python-gobject
  systemd
)
checkdepends=(
  cairo
  umockdev
)
provides=(libfprint libfprint-2.so)
conflicts=(libfprint)
source=("git+https://gitlab.freedesktop.org/libfprint/libfprint.git#commit=2d7c5277de08c6b29b3fac7447f17a516fbc4d1c")
b2sums=('SKIP')

prepare() {
  cd libfprint
}

build() {
  local meson_options=(
    # Add virtual drivers for integration tests (e.g. in fprintd)
    -D drivers=all

    -D installed-tests=false
  )

  arch-meson libfprint build "${meson_options[@]}"
  meson compile -C build
}

check() {
  meson test -C build --print-errorlogs
}

package() {
  meson install -C build --destdir "$pkgdir"
}
