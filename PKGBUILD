# Maintainer: Thomas Bächler <thomas@archlinux.org>

pkgbase=linux-firmware
pkgname=(linux-firmware amd-ucode)
_commit=2b016afc348ba4b5fb2016ffcb2822f4a293da0c
pkgver=20191022.2b016af
pkgrel=1
pkgdesc="Firmware files for Linux (Manjaro Overlay Package)"
makedepends=('git')
arch=('any')
url="https://git.kernel.org/?p=linux/kernel/git/firmware/linux-firmware.git;a=summary"
license=('GPL2' 'GPL3' 'custom')
options=(!strip)
validpgpkeys=('4CDE8575E547BF835FE15807A31B6BD72486CFD6') # Josh Boyer <jwboyer@fedoraproject.org>
source=("git+https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git#commit=${_commit}?signed")

prepare() {
  echo ${pkgbase}

  cd "${srcdir}/${pkgname}"
}

pkgver() {
  cd "${srcdir}/${pkgname}"

  # Commit date + short rev
  echo $(TZ=UTC git show -s --pretty=%cd --date=format-local:%Y%m%d HEAD).$(git rev-parse --short HEAD)
}

build() {
  mkdir -p kernel/x86/microcode
  cat ${pkgbase}/amd-ucode/microcode_amd*.bin > kernel/x86/microcode/AuthenticAMD.bin
  # Make the .bin reproducible 
  [ ! -z $SOURCE_DATE_EPOCH ] && touch -d @$SOURCE_DATE_EPOCH kernel/x86/microcode/AuthenticAMD.bin
  echo kernel/x86/microcode/AuthenticAMD.bin | bsdcpio -o -H newc -R 0:0 > amd-ucode.img
}

package_linux-firmware() {
  groups=('base')
  conflicts=('linux-firmware-git'
             'kernel26-firmware'
             'ar9170-fw'
             'iwlwifi-1000-ucode'
             'iwlwifi-3945-ucode'
             'iwlwifi-4965-ucode'
             'iwlwifi-5000-ucode'
             'iwlwifi-5150-ucode'
             'iwlwifi-6000-ucode'
             'rt2870usb-fw'
             'rt2x00-rt61-fw'
             'rt2x00-rt71w-fw')
  replaces=('kernel26-firmware'
            'ar9170-fw'
            'iwlwifi-1000-ucode'
            'iwlwifi-3945-ucode'
            'iwlwifi-4965-ucode'
            'iwlwifi-5000-ucode'
            'iwlwifi-5150-ucode'
            'iwlwifi-6000-ucode'
            'rt2870usb-fw'
            'rt2x00-rt61-fw'
            'rt2x00-rt71w-fw')

  cd "${srcdir}/${pkgname}"

  make DESTDIR="${pkgdir}" FIRMWAREDIR=/usr/lib/firmware install

  install -d "${pkgdir}/usr/share/licenses/${pkgname}"
  install -Dm644 LICEN* WHENCE "${pkgdir}/usr/share/licenses/linux-firmware/"

  # Trigger a microcode reload for configurations not using early updates
  install -d "${pkgdir}/usr/lib/tmpfiles.d"
  echo 'w /sys/devices/system/cpu/microcode/reload - - - - 1' \
    >"${pkgdir}/usr/lib/tmpfiles.d/${pkgname}.conf"
}

package_amd-ucode() {
  pkgdesc='Microcode update files for AMD CPUs'

  install -D -m0644 amd-ucode.img "${pkgdir}"/boot/amd-ucode.img  
  install -D -m0644 "${srcdir}/${pkgbase}/LICENSE.amd-ucode" "${pkgdir}/usr/share/licenses/amd-ucode/LICENSE"
}

# vim:set ts=2 sw=2 et:
sha256sums=('SKIP')
