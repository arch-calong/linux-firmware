# Maintainer: Philip Müller <philm@manjaro.org>
# Maintainer: Mark Wagie <mark at manjaro dot org>
# Contributor: Thomas Bächler <thomas@archlinux.org>

pkgbase=linux-firmware
pkgname=(linux-firmware-whence linux-firmware amd-ucode
         linux-firmware-{nfp,mellanox,marvell,qcom,liquidio,qlogic,bnx2x}
)
_tag=20230625
pkgver=20230625.ee91452d
pkgrel=1
pkgdesc="Firmware files for Linux"
url="https://git.kernel.org/?p=linux/kernel/git/firmware/linux-firmware.git;a=summary"
license=('GPL2' 'GPL3' 'custom')
arch=('any')
makedepends=('git')
options=(!strip)
source=("git+https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git#tag=${_tag}?signed"
        'dcn_3_1_5_dmcub.bin::https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/plain/amdgpu/dcn_3_1_5_dmcub.bin?id=045b2136a61968e7984caeae857a326150bfe851')
sha256sums=('SKIP'
            '5fa3d19a477dcd8e7644a739a7c38c776548948f4dc6c832752f30e96a247a4b')
validpgpkeys=('4CDE8575E547BF835FE15807A31B6BD72486CFD6') # Josh Boyer <jwboyer@fedoraproject.org>

_backports=(
)

prepare() {
  cd ${pkgbase}

  local _c
  for _c in "${_backports[@]}"; do
    git log --oneline -1 "${_c}"
    git cherry-pick -n "${_c}"
  done
}

pkgver() {
  cd ${pkgbase}

  # Commit date + short rev
  echo $(TZ=UTC git show -s --pretty=%cd --date=format-local:%Y%m%d HEAD).$(git rev-parse --short HEAD)
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

_pick() {
  local p="$1" f d; shift
  for f; do
    d="$srcdir/$p/${f#$pkgdir/}"
    mkdir -p "$(dirname "$d")"
    mv "$f" "$d"
    rmdir -p --ignore-fail-on-non-empty "$(dirname "$f")"
  done
}

package_linux-firmware-whence() {
  pkgdesc+=" - contains the WHENCE license file which documents the vendor license details"

  install -Dt "${pkgdir}/usr/share/licenses/${pkgname}" -m644 ${pkgbase}/WHENCE
}

package_linux-firmware() {
  depends=('linux-firmware-whence')

  cd ${pkgbase}

  make DESTDIR="${pkgdir}" FIRMWAREDIR=/usr/lib/firmware install

  # useless without CONFIG_MICROCODE_LATE_LOADING
  # https://bugs.archlinux.org/task/46591
  #
  # # Trigger a microcode reload for configurations not using early updates
  # echo 'w /sys/devices/system/cpu/microcode/reload - - - - 1' |
  #   install -Dm644 /dev/stdin "${pkgdir}/usr/lib/tmpfiles.d/${pkgname}.conf"

  install -Dt "${pkgdir}/usr/share/licenses/${pkgname}" -m644 LICEN*

  cd "${pkgdir}"
  
  # https://gitlab.freedesktop.org/drm/amd/-/issues/2666
  install -Dm644 "${srcdir}"/dcn_3_1_5_dmcub.bin -t "${pkgdir}"/usr/lib/firmware/amdgpu

  # remove arm64 firmware https://bugs.archlinux.org/task/76583
  rm usr/lib/firmware/mrvl/prestera/mvsw_prestera_fw_arm64-v4.1.img

  # split
  _pick linux-firmware-nfp usr/lib/firmware/netronome
  _pick linux-firmware-nfp usr/share/licenses/${pkgname}/LICENCE.Netronome

  _pick linux-firmware-mellanox usr/lib/firmware/mellanox

  _pick linux-firmware-marvell usr/lib/firmware/{libertas,mwl8k,mwlwifi,mrvl}
  _pick linux-firmware-marvell usr/share/licenses/${pkgname}/LICENCE.{Marvell,NXP}

  _pick linux-firmware-qcom usr/lib/firmware/{qcom,a300_*}
  _pick linux-firmware-qcom usr/share/licenses/${pkgname}/LICENSE.qcom*

  _pick linux-firmware-liquidio usr/lib/firmware/liquidio
  _pick linux-firmware-liquidio usr/share/licenses/${pkgname}/LICENCE.cavium_liquidio

  _pick linux-firmware-qlogic usr/lib/firmware/{qlogic,qed,ql2???_*,c{b,t,t2}fw-*}
  _pick linux-firmware-qlogic usr/share/licenses/${pkgname}/LICENCE.{qla1280,qla2xxx}

  _pick linux-firmware-bnx2x usr/lib/firmware/bnx2x*
}

package_amd-ucode() {
  pkgdesc="Microcode update image for AMD CPUs"
  license=(custom)

  install -Dt "${pkgdir}/boot" -m644 amd-ucode.img

  install -Dt "${pkgdir}/usr/share/licenses/${pkgname}" -m644 ${pkgbase}/LICENSE.amd-ucode
}

package_linux-firmware-nfp() {
  pkgdesc+=" - nfp / Firmware for Netronome Flow Processors"
  depends=('linux-firmware-whence')

  mv -v linux-firmware-nfp/* "${pkgdir}"
}

package_linux-firmware-mellanox() {
  pkgdesc+=" - mellanox / Firmware for Mellanox Spectrum switches"
  depends=('linux-firmware-whence')

  mv -v linux-firmware-mellanox/* "${pkgdir}"
}

package_linux-firmware-marvell() {
  pkgdesc+=" - marvell / Firmware for Marvell devices"
  depends=('linux-firmware-whence')

  mv -v linux-firmware-marvell/* "${pkgdir}"
}

package_linux-firmware-qcom() {
  pkgdesc+=" - qcom / Firmware for Qualcomm SoCs"
  depends=('linux-firmware-whence')

  mv -v linux-firmware-qcom/* "${pkgdir}"
}

package_linux-firmware-liquidio() {
  pkgdesc+=" - liquidio / Firmware for Cavium LiquidIO server adapters"
  depends=('linux-firmware-whence')

  mv -v linux-firmware-liquidio/* "${pkgdir}"
}

package_linux-firmware-qlogic() {
  pkgdesc+=" - qlogic / Firmware for QLogic devices"
  depends=('linux-firmware-whence')

  mv -v linux-firmware-qlogic/* "${pkgdir}"
}

package_linux-firmware-bnx2x() {
  pkgdesc+=" - bnx2x / Firmware for Broadcom NetXtreme II 10Gb ethernet adapters"
  depends=('linux-firmware-whence')

  mv -v linux-firmware-bnx2x/* "${pkgdir}"
}

# vim:set sw=2 et:
