# Maintainer: Ungoogled Software Contributors
# Maintainer: Jakob-Niklas See <git@nwex.de>

# Based on aur/chromium-vaapi, with ungoogled-chromium patches
# Based on extra/chromium

# Maintainer: Evangelos Foutras <evangelos@foutrelis.com>
# Contributor: Pierre Schmitz <pierre@archlinux.de>
# Contributor: Jan "heftig" Steffens <jan.steffens@gmail.com>
# Contributor: Daniel J Griffiths <ghost1227@archlinux.us>

pkgname=ungoogled-chromium
pkgver=96.0.4660.0
pkgrel=1
_launcher_ver=8
pkgdesc="A lightweight approach to removing Google web service dependency"
arch=('x86_64')
url="https://github.com/Eloston/ungoogled-chromium"
license=('BSD')
depends=('gtk3' 'nss' 'alsa-lib' 'xdg-utils' 'libxss' 'libcups' 'libgcrypt'
         'ttf-liberation' 'systemd' 'dbus' 'libpulse' 'pciutils' 'libva'
         'desktop-file-utils' 'hicolor-icon-theme')
makedepends=('python' 'gn' 'ninja' 'clang' 'lld' 'llvm' 'gperf' 'nodejs' 'pipewire'
             'java-runtime-headless')
optdepends=('pipewire: WebRTC desktop sharing under Wayland'
            'kdialog: support for native dialogs in Plasma'
            'org.freedesktop.secrets: password storage backend on GNOME / Xfce'
            'kwallet: support for storing passwords in KWallet on Plasma')
provides=('chromium')
conflicts=('chromium')
source=(https://commondatastorage.googleapis.com/chromium-browser-official/chromium-$pkgver.tar.xz
        $pkgname::git://github.com/usertam/ungoogled-chromium.git
        https://github.com/foutrelis/chromium-launcher/archive/v$_launcher_ver/chromium-launcher-$_launcher_ver.tar.gz
        https://github.com/stha09/chromium-patches/releases/download/chromium-96-patchset-2/chromium-96-patchset-2.tar.xz
        chromium-drirc-disable-10bpc-color-configs.conf
        sql-VirtualCursor-standard-layout.patch
        wayland-egl.patch
        use-oauth2-client-switches-as-default.patch
        replace-blacklist-with-ignorelist.patch
        chromium-93-ffmpeg-4.4.patch
        chromium-94-ffmpeg-roll.patch
        unexpire-accelerated-video-decode-flag.patch
        add-a-TODO-about-a-missing-pnacl-flag.patch
        use-ffile-compilation-dir.patch
        chromium-96-patchset-2.patch)
sha256sums=('e7816403a598354154f158ab7dc306000b1ba5da553b972e990750c7cc3ee76f'
            'SKIP'
            '213e50f48b67feb4441078d50b0fd431df34323be15be97c55302d3fdac4483a'
            'ba26b864f599bc05c6a276f3e8a865bf34115c9668d163f0d911315b7bb6e579'
            'babda4f5c1179825797496898d77334ac067149cac03d797ab27ac69671a7feb'
            '23d6b14530acb66762c5d8b895c100203a824549e0d9aa815958dfd2513e6a7a'
            '34d08ea93cb4762cb33c7cffe931358008af32265fc720f2762f0179c3973574'
            'e393174d7695d0bafed69e868c5fbfecf07aa6969f3b64596d0bae8b067e1711'
            'd3344ba39b8c6ed202334ba7f441c70d81ddf8cdb15af1aa8c16e9a3a75fbb35'
            '1a9e074f417f8ffd78bcd6874d8e2e74a239905bf662f76a7755fa40dc476b57'
            '56acb6e743d2ab1ed9f3eb01700ade02521769978d03ac43226dec94659b3ace'
            '2a97b26c3d6821b15ef4ef1369905c6fa3e9c8da4877eb9af4361452a425290b'
            'd53da216538f2e741a6e048ed103964a91a98e9a3c10c27fdfa34d4692fdc455'
            '921010cd8fab5f30be76c68b68c9b39fac9e21f4c4133bb709879592bbdf606e'
            '476e870a258a5b1f5c5d91058fcf239b8be63505bbaea6a3ad0ea99c45a08227')

# Possible replacements are listed in build/linux/unbundle/replace_gn_files.py
# Keys are the names in the above script; values are the dependencies in Arch
declare -gA _system_libs=(
  [ffmpeg]=ffmpeg
  [flac]=flac
  [fontconfig]=fontconfig
  [freetype]=freetype2
  #[harfbuzz-ng]=harfbuzz
  [icu]=icu
  [libdrm]=
  [libjpeg]=libjpeg
  [libpng]=libpng
  #[libvpx]=libvpx
  [libwebp]=libwebp
  [libxml]=libxml2
  [libxslt]=libxslt
  [opus]=opus
  [re2]=re2
  [snappy]=snappy
  [zlib]=minizip
)
_unwanted_bundled_libs=(
  $(printf "%s\n" ${!_system_libs[@]} | sed 's/^libjpeg$/&_turbo/')
)
depends+=(${_system_libs[@]})

prepare() {
  # Temporary patch, patchset is not up-to-date
  cd "$srcdir/patches"
  patch -Np1 -i ../chromium-96-patchset-2.patch

  cd "$srcdir/chromium-$pkgver"

  # Allow building against system libraries in official builds
  sed -i 's/OFFICIAL_BUILD/GOOGLE_CHROME_BUILD/' \
    tools/generate_shim_headers/generate_shim_headers.py

  # https://crbug.com/893950
  sed -i -e 's/\<xmlMalloc\>/malloc/' -e 's/\<xmlFree\>/free/' \
    third_party/blink/renderer/core/xml/*.cc \
    third_party/blink/renderer/core/xml/parser/xml_document_parser.cc \
    third_party/libxml/chromium/*.cc

  # Use the --oauth2-client-id= and --oauth2-client-secret= switches for
  # setting GOOGLE_DEFAULT_CLIENT_ID and GOOGLE_DEFAULT_CLIENT_SECRET at
  # runtime -- this allows signing into Chromium without baked-in values
  patch -Np1 -i ../use-oauth2-client-switches-as-default.patch

  # Fix build with older ffmpeg
  patch -Np1 -i ../chromium-93-ffmpeg-4.4.patch

  # Revert change to custom function av_stream_get_first_dts; will need to
  # switch to bundled ffmpeg when we're no longer using ffmpeg 4.4 in Arch
  # Upstream commit that made first_dts internal causing Chromium to add a
  # custom function: https://github.com/FFmpeg/FFmpeg/commit/591b88e6787c4
  # https://crbug.com/1251779
  patch -Rp1 -i ../chromium-94-ffmpeg-roll.patch

  # https://crbug.com/1207478
  patch -Np0 -i ../unexpire-accelerated-video-decode-flag.patch

  # Revert transition to -fsanitize-ignorelist (needs newer clang)
  patch -Rp1 -i ../replace-blacklist-with-ignorelist.patch

  # Fixes building with GCC 11  https://crbug.com/1189788
  patch -Np1 -i ../sql-VirtualCursor-standard-layout.patch

  # Fixes for building with libstdc++ instead of libc++
  patch -Np1 -i ../patches/chromium-78-protobuf-RepeatedPtrField-export.patch
  patch -Np1 -i ../patches/chromium-95-compiler.patch
  patch -Np1 -i ../patches/chromium-95-libyuv-aarch64.patch
  patch -Np1 -i ../patches/chromium-96-AppliedTextDecoration-include.patch

  # Wayland/EGL regression (crbug #1071528 #1071550)
  patch -Np1 -i ../wayland-egl.patch

  # Ungoogled Chromium changes
  _ungoogled_repo="$srcdir/$pkgname"
  _utils="${_ungoogled_repo}/utils"
  msg2 'Pruning binaries'
  python "$_utils/prune_binaries.py" ./ "$_ungoogled_repo/pruning.list"
  msg2 'Applying patches'
  python "$_utils/patches.py" apply ./ "$_ungoogled_repo/patches"
  msg2 'Applying domain substitution'
  python "$_utils/domain_substitution.py" apply -r "$_ungoogled_repo/domain_regex.list" \
    -f "$_ungoogled_repo/domain_substitution.list" -c domainsubcache.tar.gz ./

  # Link to system tools required by the build
  mkdir -p third_party/node/linux/node-linux-x64/bin
  ln -s /usr/bin/node third_party/node/linux/node-linux-x64/bin/
  ln -s /usr/bin/java third_party/jdk/current/bin/

  # Remove bundled libraries for which we will use the system copies; this
  # *should* do what the remove_bundled_libraries.py script does, with the
  # added benefit of not having to list all the remaining libraries
  local _lib
  for _lib in ${_unwanted_bundled_libs[@]}; do
    find "third_party/$_lib" -type f \
      \! -path "third_party/$_lib/chromium/*" \
      \! -path "third_party/$_lib/google/*" \
      \! -path "third_party/harfbuzz-ng/utils/hb_scoped.h" \
      \! -regex '.*\.\(gn\|gni\|isolate\)' \
      -delete
  done

  ./build/linux/unbundle/replace_gn_files.py \
    --system-libraries "${!_system_libs[@]}"
}

build() {
  make -C chromium-launcher-$_launcher_ver

  cd "$srcdir/chromium-$pkgver"

  if check_buildoption ccache y; then
    # Avoid falling back to preprocessor mode when sources contain time macros
    export CCACHE_SLOPPINESS=time_macros
  fi

  export CC=clang
  export CXX=clang++
  export AR=llvm-ar
  export NM=llvm-nm

  local _flags=(
    'custom_toolchain="//build/toolchain/linux/unbundle:default"'
    'host_toolchain="//build/toolchain/linux/unbundle:default"'
    'is_official_build=true' # implies is_cfi=true on x86_64
    'disable_fieldtrial_testing_config=true'
    'blink_enable_generated_code_formatting=false'
    'ffmpeg_branding="Chrome"'
    'proprietary_codecs=true'
    'rtc_use_pipewire=true'
    'link_pulseaudio=true'
    'use_gnome_keyring=false'
    'use_sysroot=false'
    'use_custom_libcxx=false'
    'enable_widevine=true'
    'use_vaapi=true'
  )

  if [[ -n ${_system_libs[icu]+set} ]]; then
    _flags+=('icu_use_data_file=false')
  fi

  if check_option strip y; then
    _flags+=('symbol_level=0')
  fi

  # Append ungoogled chromium flags to _flags array
  _ungoogled_repo="$srcdir/$pkgname"
  readarray -t -O ${#_flags[@]} _flags < "${_ungoogled_repo}/flags.gn"

  # See https://github.com/ungoogled-software/ungoogled-chromium-archlinux/issues/123
  CFLAGS="-march=x86-64 -mtune=generic -O2 -pipe -fno-plt"
  CXXFLAGS="$CFLAGS"

  # Facilitate deterministic builds (taken from build/config/compiler/BUILD.gn)
  CFLAGS+='   -Wno-builtin-macro-redefined'
  CXXFLAGS+=' -Wno-builtin-macro-redefined'
  CPPFLAGS+=' -D__DATE__=  -D__TIME__=  -D__TIMESTAMP__='

  # Do not warn about unknown warning options
  CFLAGS+='   -Wno-unknown-warning-option'
  CXXFLAGS+=' -Wno-unknown-warning-option'

  msg2 'Configuring Chromium'
  gn gen out/Release --args="${_flags[*]}"
  msg2 'Building Chromium'
  ninja -C out/Release chrome chrome_sandbox chromedriver
}

package() {
  cd chromium-launcher-$_launcher_ver
  make PREFIX=/usr DESTDIR="$pkgdir" install
  install -Dm644 LICENSE \
    "$pkgdir/usr/share/licenses/chromium/LICENSE.launcher"

  cd "$srcdir/chromium-$pkgver"

  install -D out/Release/chrome "$pkgdir/usr/lib/chromium/chromium"
  install -Dm4755 out/Release/chrome_sandbox "$pkgdir/usr/lib/chromium/chrome-sandbox"
  ln -s /usr/lib/chromium/chromedriver "$pkgdir/usr/bin/chromedriver"

  install -Dm644 ../chromium-drirc-disable-10bpc-color-configs.conf \
    "$pkgdir/usr/share/drirc.d/10-$pkgname.conf"

  install -Dm644 chrome/installer/linux/common/desktop.template \
    "$pkgdir/usr/share/applications/chromium.desktop"
  install -Dm644 chrome/app/resources/manpage.1.in \
    "$pkgdir/usr/share/man/man1/chromium.1"
  sed -i \
    -e 's/@@MENUNAME@@/Chromium/g' \
    -e 's/@@PACKAGE@@/chromium/g' \
    -e 's/@@USR_BIN_SYMLINK_NAME@@/chromium/g' \
    "$pkgdir/usr/share/applications/chromium.desktop" \
    "$pkgdir/usr/share/man/man1/chromium.1"

  install -Dm644 chrome/installer/linux/common/chromium-browser/chromium-browser.appdata.xml \
    "$pkgdir/usr/share/metainfo/chromium.appdata.xml"
  sed -ni \
    -e 's/chromium-browser\.desktop/chromium.desktop/' \
    -e '/<update_contact>/d' \
    -e '/<p>/N;/<p>\n.*\(We invite\|Chromium supports Vorbis\)/,/<\/p>/d' \
    -e '/^<?xml/,$p' \
    "$pkgdir/usr/share/metainfo/chromium.appdata.xml"

  local toplevel_files=(
    chrome_100_percent.pak
    chrome_200_percent.pak
    resources.pak
    v8_context_snapshot.bin

    # ANGLE
    libEGL.so
    libGLESv2.so

    chromedriver
    chrome_crashpad_handler
  )

  if [[ -z ${_system_libs[icu]+set} ]]; then
    toplevel_files+=(icudtl.dat)
  fi

  cp "${toplevel_files[@]/#/out/Release/}" "$pkgdir/usr/lib/chromium/"
  install -Dm644 -t "$pkgdir/usr/lib/chromium/locales" out/Release/locales/*.pak
  install -Dm755 -t "$pkgdir/usr/lib/chromium/swiftshader" out/Release/swiftshader/*.so

  for size in 24 48 64 128 256; do
    install -Dm644 "chrome/app/theme/chromium/product_logo_$size.png" \
      "$pkgdir/usr/share/icons/hicolor/${size}x${size}/apps/chromium.png"
  done

  for size in 16 32; do
    install -Dm644 "chrome/app/theme/default_100_percent/chromium/product_logo_$size.png" \
      "$pkgdir/usr/share/icons/hicolor/${size}x${size}/apps/chromium.png"
  done

  install -Dm644 LICENSE "$pkgdir/usr/share/licenses/chromium/LICENSE"
}

# vim:set ts=2 sw=2 et ft=sh:
