# Based on Arch Linux's labwc PKGBUILD, maintained by Peter Jung.
# This package adds the locally maintained DirectionalCycleWindows patch.

pkgname=labwc-directional
pkgver=0.20.2.r30.gfa6794a3
pkgrel=2
pkgdesc='Openbox-inspired Wayland compositor with directional window cycling'
arch=('x86_64')
url='https://github.com/labwc/labwc'
license=('GPL-2.0-only')

# Upstream base of the tested directional cycling patch.
# Keep this pinned; changing it requires rebasing and testing the patch.
_upstream_commit=fa6794a3d29593faf088447c30c1e9dd86cb94ed

depends=(
  cairo
  glib2
  glibc
  libinput
  libpng
  librsvg
  libsfdo
  libwlroots-0.20.so
  libxcb
  libxkbcommon
  libxml2
  pango
  pixman
  seatd
  ttf-font
  wayland
)
makedepends=(git meson scdoc wayland-protocols xorg-xwayland)
checkdepends=(python)
optdepends=(
  'bemenu: default launcher via Alt+F3'
  'xorg-xwayland: X11 application support'
)
provides=("labwc=$pkgver")
conflicts=(labwc)

source=(
  "labwc::git+https://github.com/labwc/labwc.git#commit=$_upstream_commit"
  'directional-cycle.patch'
  'directional-keybinds.xml'
)
# Git verifies the pinned commit; the two local files have content checksums.
sha256sums=(
  'SKIP'
  '868d578c3bc7af1f827e4f791662726b35d1706409808072ab72964ddd889188'
  '48ad3a658cb562d3f2c3406c3476c710311d96e77d21d4377518797cb2b28eb5'
)

prepare() {
  cd "$srcdir/labwc"
  patch --batch --forward -Np1 -i "$srcdir/directional-cycle.patch"
}

build() {
  # Keep cmocka optional; our geometry and headless tests do not require it.
  arch-meson --auto-features=auto --wrap-mode=nofallback \
    -Dman-pages=enabled -Dxwayland=enabled -Dtest=auto \
    "$srcdir/labwc" "$srcdir/build"
  meson compile -C "$srcdir/build"
}

check() {
  # Meson logs the test environment. Keep credentials out of generated logs.
  local -a test_env=(env -i "PATH=$PATH" "HOME=$HOME" 'LANG=C.UTF-8')
  "${test_env[@]}" meson test -C "$srcdir/build" --print-errorlogs
  "${test_env[@]}" python "$srcdir/labwc/t/directional-cycle-integration.py" \
    "$srcdir/build/labwc"
}

package() {
  meson install -C "$srcdir/build" --destdir "$pkgdir"
  install -Dm644 "$srcdir/directional-keybinds.xml" \
    "$pkgdir/usr/share/doc/labwc/directional-keybinds.xml"
}
