# Maintainer: Muflone http://www.muflone.com/contacts/english/
# Contributor: Arne Hoch <arne@derhoch.de>

pkgname=dbeaver
pkgver=21.0.0
pkgrel=1
pkgdesc="Free universal SQL Client for developers and database administrators (community edition)"
arch=('x86_64')
url="https://dbeaver.io/"
license=("Apache")
depends=('java-runtime>=11' 'gtk3' 'gtk-update-icon-cache' 'libsecret')
makedepends=('maven' 'java-runtime<15')
optdepends=('dbeaver-plugin-office: export data in Microsoft Office Excel format'
            'dbeaver-plugin-svg-format: save diagrams in SVG format')
source=("${pkgname}-${pkgver}.tar.gz"::"https://github.com/serge-rider/dbeaver/archive/${pkgver}.tar.gz"
        "${pkgname}.desktop"
        "${pkgname}.sh"
        "${pkgname}.profile.gz"
        "${pkgname}.hook"
        "${pkgname}.install")
sha256sums=('3924947380b5da47beb7494a5b82744a1fc183a838f912314f3e6f6089df2fb6'
            '27573b6ddb62a3d4dde4841a633e9b52cb020deb338b327a6d460fd3a29c8ded'
            '406a2980806c394670e88b1ae70134900be376c2ea4a4216610591cc8b557526'
            '1863e74bdcf22b7328e6e8487cbebff7d5360e34bde85c1dd226b168b4737034'
            'f8b763ca210bfa4d9a4e407b656ba4f5d1bf2f3f54c67044f7a4dd0c3625fc22'
            'f8d65dd933049b587a5815ea75a30ef944300b812df383ca1c2dcd68280bc7ab')
install="${pkgname}.install"

prepare() {
  # Fix version number in profile file
  gzip --decompress --keep --stdout "${pkgname}.profile.gz" | 
    sed "s/DBEAVER_VERSION/${pkgver}/g" |
    gzip -9 > "${pkgname}.profile-${pkgver}.gz"
  # Avoid the use of any Java 15, actually incompatible with the build
  java_version=$(archlinux-java status | tail -n +2 | cut -d' ' -f 3 | sort -n -t '-' -k2 | egrep '^java-[11-14]{2}' | tail -n 1)
  export JAVA_HOME="/usr/lib/jvm/${java_version}"
  # Download dependencies during prepare FS#55873
  # https://bugs.archlinux.org/task/55873
  cd "${pkgname}-${pkgver}"
  export MAVEN_OPTS="-Xmx2048m"
  mvn --batch-mode validate
}

build() {
  cd "${pkgname}-${pkgver}"
  mvn --batch-mode package
}

package() {
  cd "${pkgname}-${pkgver}/product/standalone"
  # Install icons into /usr/share/icons/hicolor
  for _size in 16 32 48 64 128 256 512
  do
    install -m 644 -D "icons-sources/icon_${_size}x${_size}.png" \
      "${pkgdir}/usr/share/icons/hicolor/${_size}x${_size}/apps/dbeaver.png"
  done

  # Move into the target directory
  cd "target/products/org.jkiss.dbeaver.core.product/linux/gtk/${CARCH}"

  # Initially install everything into /usr/lib/dbeaver
  install -m 755 -d "${pkgdir}/usr/lib"
  cp -r "dbeaver" "${pkgdir}/usr/lib/${pkgname}"

  # Move shared data to /usr/share/dbeaver
  cd "${pkgdir}/usr/lib/${pkgname}"
  install -m 755 -d "${pkgdir}/usr/share/${pkgname}"
  for _file in configuration features p2 .eclipseproduct artifacts.xml dbeaver.ini readme.txt
  do
    mv "${_file}" "${pkgdir}/usr/share/${pkgname}"
    ln -s "/usr/share/${pkgname}/${_file}" .
  done

  # Install additional licenses
  install -m 755 -d "${pkgdir}/usr/share/licenses"
  mv licenses "${pkgdir}/usr/share/licenses/${pkgname}"

  # Install icons
  install -m 755 -d "${pkgdir}/usr/share/pixmaps"
  mv dbeaver.png "${pkgdir}/usr/share/pixmaps/${pkgname}.png"
  mv icon.xpm "${pkgdir}/usr/share/pixmaps/${pkgname}.xpm"

  # Install executable script into /usr/bin
  install -m 755 -d "${pkgdir}/usr/bin"
  install -m 755 "${srcdir}/dbeaver.sh" "${pkgdir}/usr/bin/${pkgname}"

  # Install application launcher into /usr/share/applications
  install -m 755 -d "${pkgdir}/usr/share/applications"
  install -m 755 -t "${pkgdir}/usr/share/applications" "${srcdir}/${pkgname}.desktop"

  # Clean up and install new profile
  rm -rf "${pkgdir}/usr/share/${pkgname}/p2/org.eclipse.equinox.p2.core"
  cd "${pkgdir}/usr/share/${pkgname}/p2/org.eclipse.equinox.p2.engine/profileRegistry/DefaultProfile.profile"
  find . -name "*.profile.gz" -delete
  install -m 644 "${srcdir}/${pkgname}.profile-${pkgver}.gz" "1502633007017.profile.gz"
  cd "${pkgdir}/usr/share/${pkgname}/p2/org.eclipse.equinox.p2.engine"
  rm -f ".settings/org.eclipse.equinox.p2.artifact.repository.prefs"
  rm ".settings/org.eclipse.equinox.p2.metadata.repository.prefs"
  rmdir ".settings"

  # Install system hook
  install -m 755 -d "${pkgdir}/usr/share/libalpm/hooks"
  install -m 644 "${srcdir}/${pkgname}.hook" "${pkgdir}/usr/share/libalpm/hooks"

  # Create configuration file (handled by the hook)
  cd "${pkgdir}/usr/share/dbeaver/configuration/org.eclipse.equinox.simpleconfigurator"
  install -m 755 -d "${pkgdir}/etc/${pkgname}/bundles.d"
  mv "bundles.info" "${pkgdir}/etc/${pkgname}/bundles.d/00-${pkgname}.info"
  ln -s "/etc/${pkgname}/bundles.info" .
}
