# Maintainer: Helmut Stult <helmut[at]manjaro[dot]org>

# Arch credits:
# Maintainer: Thomas Bächler <thomas@archlinux.org>

pkgbase=linux-firmware
pkgname=(linux-firmware amd-ucode)

_commit='168452ee695b5edb9deb641059bc110b9c5e8fc7'
pkgver=20210719.r1990.168452e
pkgrel=1
pkgdesc="Firmware files for Linux (Manjaro Overlay Package)"
url="https://git.kernel.org/?p=linux/kernel/git/firmware/linux-firmware.git;a=summary"
license=('GPL2' 'GPL3' 'custom')
arch=('any')
makedepends=('git')
options=(!strip)
source=("git+https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git#commit=${_commit}?signed")
sha256sums=('SKIP')
validpgpkeys=('4CDE8575E547BF835FE15807A31B6BD72486CFD6') # Josh Boyer <jwboyer@fedoraproject.org>

_backports=(
)

prepare() {
  cd ${pkgname}

  local _c
  for _c in "${_backports[@]}"; do
    git log --oneline -1 "${_c}"
    git cherry-pick -n "${_c}"
  done
}

pkgver() {
  cd ${pkgname}

  # Commit date + short rev
  echo $(TZ=UTC git show -s --pretty=%cd --date=format-local:%Y%m%d HEAD).r$(git rev-list --count HEAD).$(git rev-parse --short HEAD)
}

build() {
  mkdir -p kernel/x86/microcode
  cat ${pkgbase}/amd-ucode/microcode_amd*.bin > kernel/x86/microcode/AuthenticAMD.bin

  # Reproducibility: set the timestamp on the bin file
  if [[ -n ${SOURCE_DATE_EPOCH} ]]; then 
    touch -d @${SOURCE_DATE_EPOCH} kernel/x86/microcode/AuthenticAMD.bin
  fi

  # Reproducibility: strip the inode and device numbers from the cpio archive
  echo kernel/x86/microcode/AuthenticAMD.bin |
    bsdtar --uid 0 --gid 0 -cnf - -T - |
    bsdtar --null -cf - --format=newc @- > amd-ucode.img
}

package_linux-firmware() {
  cd ${pkgname}

  make DESTDIR="${pkgdir}" FIRMWAREDIR=/usr/lib/firmware install

  # Trigger a microcode reload for configurations not using early updates
  echo 'w /sys/devices/system/cpu/microcode/reload - - - - 1' |
    install -Dm644 /dev/stdin "${pkgdir}/usr/lib/tmpfiles.d/${pkgname}.conf"

  install -Dt "${pkgdir}/usr/share/licenses/${pkgname}" -m644 LICEN* WHENCE
}

package_amd-ucode() {
  pkgdesc="Microcode update image for AMD CPUs (Manjaro Overlay Package)"
  license=(custom)

  install -Dt "${pkgdir}/boot" -m644 amd-ucode.img

  install -Dt "${pkgdir}/usr/share/licenses/${pkgname}" -m644 ${pkgbase}/LICENSE.amd-ucode
}
