# Maintainer: Andrew Crerar <andrew (at) crerar (dot) io>
# Contributor: Jan Alexander Steffens (heftig) <jan.steffens@gmail.com>

pkgname=firefox-developer-edition
pkgver=57.0b13
pkgrel=1
pkgdesc="Developer Edition of the popular Firefox web browser"
arch=(i686 x86_64)
license=(MPL GPL LGPL)
url="https://www.mozilla.org/firefox/channel/#developer"
depends=(gtk3 gtk2 mozilla-common libxt startup-notification mime-types
         dbus-glib ffmpeg nss hunspell sqlite ttf-font libpulse)
makedepends=(unzip zip diffutils python2 yasm mesa imake gconf inetutils
             xorg-server-xvfb autoconf2.13 rust mercurial clang llvm jack)
optdepends=('networkmanager: Location detection via available WiFi networks'
            'libnotify: Notification integration'
            'pulseaudio: Audio support'
            'speech-dispatcher: Text-to-Speech')
options=(!emptydirs !makeflags !strip)
_repo=https://hg.mozilla.org/mozilla-unified
source=("hg+$_repo#tag=FIREFOX_${pkgver//./_}_RELEASE"
        $pkgname.desktop
        firefox-symbolic.svg
        wifi-disentangle.patch
        wifi-fix-interface.patch
        firefox-install-dir.patch
        no-plt.diff)
sha512sums=('SKIP'
            '36042b0034e9958b15b7f5eab70d92a4332448be40c81d308200149044022cf298d958ed9836a421ce13cde81229df777a32c074c46014cc99e165f4084088e2'
            '84e741b6a4c7675c846c16a0e0280d00e7be5477b07b693ccddac597987e8979a35d07a9ac8a3a28338b458ebdf41754ceb2119b8e41d2ec41f95b551232c64c'
            '16c2f798d371a2386c060a4652a6c82dad6aeca234d910c738cfe01b780faae8e00a37b7e38f4895f074c3b9b60c582fe77dc306771d00ab7473559b748298de'
            '4d741f9446f0529505719ccdaa69181ed7836c7469e1a5d81fa08a4555ef6a6c2acf8db7cadfb3fe365766b407720e7927b1ded337aa0700578772bf1fc5fad6'
            'ce764de6deae65ae5c888b12d163419c7828cf8b31f73d7c3bc8dc3dafbca0005ea377b5b1fcea0d1f5c613459fa393690d5bc9d8e5c3e46db940b151082fbd6'
            '4c2ef8ebedc1184c3967c123cafd63ba1abf2a274993aea8475c434f91a40b86e7c5d3c78c3b2809cd15310af4c613d5841a9114315f861338c36f498a782fd0')

# Google API keys (see http://www.chromium.org/developers/how-tos/api-keys)
# Note: These are for Arch Linux use ONLY. For your own distribution, please
# get your own set of keys. Feel free to contact foutrelis@archlinux.org for
# more information.
_google_api_key=AIzaSyDwr302FpOSkGRpLlUpPThNTDPbXcIn_FM

# Mozilla API keys (see https://location.services.mozilla.com/api)
# Note: These are for Arch Linux use ONLY. For your own distribution, please
# get your own set of keys. Feel free to contact heftig@archlinux.org for
# more information.
_mozilla_api_key=16674381-f021-49de-8622-3021c5942aff

prepare() {
  mkdir path
  ln -s /usr/bin/python2 path/python

  cd mozilla-unified
  patch -Np1 -i ../firefox-install-dir.patch

  # https://bugzilla.mozilla.org/show_bug.cgi?id=1314968
  patch -Np1 -i ../wifi-disentangle.patch
  patch -Np1 -i ../wifi-fix-interface.patch

  # https://bugzilla.mozilla.org/show_bug.cgi?id=1382942
  patch -Np1 -i ../no-plt.diff

  echo -n "$_google_api_key" > google-api-key
  echo -n "$_mozilla_api_key" > mozilla-api-key

  cat > .mozconfig << END
ac_add_options --enable-application=browser

ac_add_options --prefix=/usr
ac_add_options --enable-release
ac_add_options --enable-gold
ac_add_options --enable-pie
ac_add_options --enable-optimize="-O2"

# Branding
ac_add_options --with-branding=browser/branding/aurora
ac_add_options --enable-update-channel=aurora
ac_add_options --with-distribution-id=org.archlinux
export MOZILLA_OFFICIAL=1
export MOZ_TELEMETRY_REPORTING=1
export MOZ_ADDON_SIGNING=1
export MOZ_REQUIRE_SIGNING=0
ac_add_options "MOZ_ALLOW_LEGACY_EXTENSIONS=1"

# Keys
ac_add_options --with-google-api-keyfile=${PWD@Q}/google-api-key
ac_add_options --with-mozilla-api-keyfile=${PWD@Q}/mozilla-api-key

# System libraries
ac_add_options --with-system-zlib
ac_add_options --with-system-bz2
ac_add_options --enable-system-hunspell
ac_add_options --enable-system-sqlite
ac_add_options --enable-system-ffi

# Features
ac_add_options --enable-alsa
ac_add_options --enable-jack
ac_add_options --enable-startup-notification
ac_add_options --enable-crashreporter
ac_add_options --disable-updater

STRIP_FLAGS="--strip-debug"
END
}

build() {
  cd mozilla-unified

  # _FORTIFY_SOURCE causes configure failures
  CPPFLAGS+=" -O2"

  export PATH="$srcdir/path:$PATH"
  export MOZ_SOURCE_REPO="$_repo"

  # Do PGO
  #xvfb-run -a -n 93 -s "-extension GLX -screen 0 1280x1024x24" \
  #  MOZ_PGO=1 ./mach build
  ./mach build
  ./mach buildsymbols
}

package() {
  cd mozilla-unified
  DESTDIR="$pkgdir" ./mach install
  find . -name '*crashreporter-symbols-full.zip' -exec cp -fvt "$startdir" {} +

  _vendorjs="$pkgdir/usr/lib/$pkgname/browser/defaults/preferences/vendor.js"
  install -Dm644 /dev/stdin "$_vendorjs" << END
// Use LANG environment variable to choose locale
pref("intl.locale.matchOS", true);

// Disable default browser checking.
pref("browser.shell.checkDefaultBrowser", false);

// Don't disable our bundled extensions in the application directory
pref("extensions.autoDisableScopes", 11);
pref("extensions.shownSelectionUI", true);

// Opt all of us into e10s, instead of just 50%
pref("browser.tabs.remote.autostart", true);
END

  _distini="$pkgdir/usr/lib/$pkgname/distribution/distribution.ini"
  install -Dm644 /dev/stdin "$_distini" << END
[Global]
id=archlinux
version=1.0
about=Mozilla Firefox Developer Edition for Arch Linux

[Preferences]
app.distributor=archlinux
app.distributor.channel=$pkgname
app.partner.archlinux=archlinux
END

  for i in 16 32 48; do
    install -Dm644 browser/branding/aurora/default$i.png \
      "$pkgdir/usr/share/icons/hicolor/${i}x${i}/apps/$pkgname.png"
  done
  install -Dm644 browser/branding/aurora/content/icon64.png \
    "$pkgdir/usr/share/icons/hicolor/64x64/apps/$pkgname.png"
  install -Dm644 browser/branding/aurora/mozicon128.png \
    "$pkgdir/usr/share/icons/hicolor/128x128/apps/$pkgname.png"
  install -Dm644 browser/branding/aurora/content/about-logo.png \
    "$pkgdir/usr/share/icons/hicolor/192x192/apps/$pkgname.png"
  install -Dm644 browser/branding/aurora/content/about-logo@2x.png \
    "$pkgdir/usr/share/icons/hicolor/384x384/apps/$pkgname.png"
  install -Dm644 ../firefox-symbolic.svg \
    "$pkgdir/usr/share/icons/hicolor/symbolic/apps/$pkgname-symbolic.svg"

  install -Dm644 ../$pkgname.desktop \
    "$pkgdir/usr/share/applications/$pkgname.desktop"

  # Use system-provided dictionaries
  rm -r "$pkgdir/usr/lib/$pkgname/dictionaries"
  ln -Ts /usr/share/hunspell "$pkgdir/usr/lib/$pkgname/dictionaries"
  ln -Ts /usr/share/hyphen "$pkgdir/usr/lib/$pkgname/hyphenation"

  # Install a wrapper to avoid confusion about binary path
  install -Dm755 /dev/stdin "$pkgdir/usr/bin/$pkgname" << END
#!/bin/sh
exec /usr/lib/$pkgname/firefox "\$@"
END

  # Replace duplicate binary with wrapper
  # https://bugzilla.mozilla.org/show_bug.cgi?id=658850
  ln -srf "$pkgdir/usr/bin/$pkgname" \
    "$pkgdir/usr/lib/$pkgname/firefox-bin"

  # Use system certificates
  ln -srf "$pkgdir/usr/lib/libnssckbi.so" \
    "$pkgdir/usr/lib/$pkgname/libnssckbi.so"
}
