# Maintainer: Trevor Bergeron <aur@sec.gd>

_pkgname=cwtch-ui
pkgname=cwtch
pkgver=1.13.1
pkgrel=1
pkgdesc="UI for Privacy Preserving Infrastructure for Asynchronous, Decentralized and Metadata Resistant Applications"
arch=('x86_64')
url="https://cwtch.im/"
license=('MIT')
conflicts=('cwtch-bin' 'cwtch-git')
depends=('cwtch-autobindings')
makedepends=('flutter' 'ninja')
source=("${_pkgname}-v${pkgver}.tar.gz::https://git.openprivacy.ca/api/v1/repos/cwtch.im/${_pkgname}/archive/v${pkgver}.tar.gz")
sha512sums=('88bdcfd356aa3e403e276191f1fd9426105a7552a464715bcf1aa83bb2a79290388fe6f4c2db6cb1312e3017bd642c9e0bb5f83b0e93e7b6830e599813c22522')

prepare() {
    cd "$srcdir/$_pkgname"
    # Remove deprecated isAlwaysShown for compat with newer dart SDKs
    sed -re 's/(scrollbarTheme: .*)isAlwaysShown: false(, )?/\1/' -i lib/themes/opaque.dart
    # Remove Tor binary and libCwtch.so from package script, since we don't vendor them
    sed -re 's/^cp( -r)? linux\/(libCwtch\.so|Tor) /#\0/' -i linux/package-release.sh
}

build() {
    cd "$srcdir/$_pkgname"

    # If using the AUR 'flutter'/'flutter-beta' packages, we need a group.
    if ! id -nG | grep -qw flutterusers ; then
        if [ "`which flutter`" == "/usr/bin/flutter" ] ; then
            warning "You are not in the 'flutterusers' group. The build may fail."
            warning "Run 'sudo usermod -a -G flutterusers $USER' and reboot to fix."
            warning "You may need to use the flutter-beta package (any channel)."
        fi
    fi

    flutter="flutter --suppress-analytics"
    # no way to local-enable this... let's try to clean up after ourselves
    $flutter config | grep -qE '^\s*enable-linux-desktop: true\b' || flutter_set_linux=y
    flutter_set_linux="$?"
    [ "$flutter_set_linux" == "y" ] || $flutter config --enable-linux-desktop

    # See https://git.openprivacy.ca/cwtch.im/cwtch-ui/src/branch/trunk/.drone.yml
    $flutter pub get
    $flutter build linux \
        --dart-define BUILD_VER="${pkgver}-${pkgrel}-ARCH" \
        --dart-define BUILD_DATE="`date +%G-%m-%d-%H-%M`"

    [ "$flutter_set_linux" == "y" ] || $flutter config --no-enable-linux-desktop
}

package() {
    cd "$srcdir/$_pkgname"
    linux/package-release.sh
    cd build/linux/x64/release/bundle
    INSTALL_PREFIX="$pkgdir/usr" DESKTOP_PREFIX="/usr" ./install.sh
}
