# Contributor: Danct12 <danct12@disroot.org>
# Maintainer: Danct12 <danct12@disroot.org>
# Modified by Caleb Fontenot (CCF_100) <foley2431@gmail.com>
buildarch=8

pkgbase=linux-pine64
_desc="Pine64 PinePhone"
pkgver=5.10
pkgrel=1
arch=('aarch64')
url="https://github.com/megous/linux"
license=('GPL2')
makedepends=('xmlto' 'docbook-xsl' 'kmod' 'inetutils' 'bc' 'git' 'uboot-tools' 'dtc')
options=('!strip')

# Source
_commit="3eeeb82a1599049c86fe8720e2665fe00cfbe4a1"
source=("linux-$_commit.tar.gz::https://github.com/megous/linux/archive/${_commit}.tar.gz"
        'config'
        'enable-hdmi-output-pinetab.patch'
        'enable-jack-detection-pinetab.patch'
        'pinetab-bluetooth.patch'
        'pinetab-accelerometer.patch'
        'camera-added-bggr-bayer-mode.patch'
        'media-ov5640-Implement-autofocus.patch'
        '0002-dts-add-pinetab-dev-old-display-panel.patch'
        '0013-fix-pogopin-i2c.patch'
        'dts-pinephone-drop-modem-power-node.patch'
        'linux.preset'
        '60-linux.hook'
        '90-linux.hook'
        '0001-revert-fbcon-remove-now-unusued-softback_lines-cursor-argument.patch'
        '0002-revert-fbcon-remove-no-op-fbcon_set_origin.patch'
        '0003-revert-fbcon-remove-soft-scrollback-code.patch'
        '0001-bootsplash.patch'
        '0002-bootsplash.patch'
        '0003-bootsplash.patch'
        '0004-bootsplash.patch'
        '0005-bootsplash.patch'
        '0006-bootsplash.patch'
        '0007-bootsplash.patch'
        '0008-bootsplash.patch'
        '0009-bootsplash.patch'
        '0010-bootsplash.patch'
        '0011-bootsplash.patch'
        '0012-bootsplash.patch'
        'ec25.patch'
        'cacule-5.13.patch'
        'rdb-5.13.patch')

prepare() {
  cd linux-$_commit

  # PinePhone patches
  #patch -p1 -N < ../0013-fix-pogopin-i2c.patch
  #patch -p1 -N < ../dts-pinephone-drop-modem-power-node.patch

  # camera
  patch -p1 -N < ../media-ov5640-Implement-autofocus.patch

  # PineTab patches
  #patch -p1 -N < ../pinetab-bluetooth.patch
  #patch -p1 -N < ../pinetab-accelerometer.patch
  #patch -p1 -N < ../0002-dts-add-pinetab-dev-old-display-panel.patch
  #patch -p1 -N < ../enable-jack-detection-pinetab.patch
  #patch -p1 -N < ../enable-hdmi-output-pinetab.patch

  # bootsplash stuffs (took from glorious manjaro arm)
  patch -Np1 -i "${srcdir}/0001-revert-fbcon-remove-now-unusued-softback_lines-cursor-argument.patch"
  patch -Np1 -i "${srcdir}/0002-revert-fbcon-remove-no-op-fbcon_set_origin.patch"
  patch -Np1 -i "${srcdir}/0003-revert-fbcon-remove-soft-scrollback-code.patch"
  #patch -Np1 -i "${srcdir}/0001-bootsplash.patch"
  #patch -Np1 -i "${srcdir}/0002-bootsplash.patch"
  #patch -Np1 -i "${srcdir}/0003-bootsplash.patch"
  #patch -Np1 -i "${srcdir}/0004-bootsplash.patch"
  #patch -Np1 -i "${srcdir}/0005-bootsplash.patch"
  #patch -Np1 -i "${srcdir}/0006-bootsplash.patch"
  #patch -Np1 -i "${srcdir}/0007-bootsplash.patch"
  #patch -Np1 -i "${srcdir}/0008-bootsplash.patch"
  #patch -Np1 -i "${srcdir}/0009-bootsplash.patch"
  #patch -Np1 -i "${srcdir}/0010-bootsplash.patch"
  #patch -Np1 -i "${srcdir}/0011-bootsplash.patch"
  #patch -Np1 -i "${srcdir}/0012-bootsplash.patch"
  # CacULE
  patch -Np1 -i "${srcdir}/cacule-5.13.patch"
  patch -Np1 -i "${srcdir}/rdb-5.13.patch"
   # FOSS Modem stuff
  patch -Np1 -i "${srcdir}/ec25.patch"
  cat "${srcdir}/config" > ./.config

  # add pkgrel to extraversion
  sed -ri "s|^(EXTRAVERSION =)(.*)|\1 \2-${pkgrel}|" Makefile

  # don't run depmod on 'make install'. We'll do this ourselves in packaging
  sed -i '2iexit 0' scripts/depmod.sh

  # Fix DistCC compatibility
  sed -i '/HAVE_GCC_PLUGINS/d' arch/x86/Kconfig
}

build() {
  cd linux-$_commit

  # get kernel version
  make prepare

  # load configuration
  # Configure the kernel. Replace the line below with one of your choice.
  #make menuconfig # CLI menu for configuration
  #make nconfig # new CLI menu for configuration
  #make xconfig # X-based configuration
  #make oldconfig # using old config from previous kernel version
  # ... or manually edit .config

  # Copy back our configuration (use with new kernel version)
  #cp ./.config ../${pkgbase}.config

  ####################
  # stop here
  # this is useful to configure the kernel
  #msg "Stopping build"
  #return 1
  ####################

  #yes "" | make config

  # build!
  unset LDFLAGS
  make ${MAKEFLAGS} Image Image.gz modules
  # Generate device tree blobs with symbols to support applying device tree overlays in U-Boot
  make ${MAKEFLAGS} DTC_FLAGS="-@" dtbs
}

_package() {
  pkgdesc="The Linux Kernel and modules - ${_desc}"
  depends=('coreutils' 'kmod' 'mkinitcpio>=0.7')
  optdepends=('crda: to set the correct wireless channels of your country')
  provides=('kernel26' "linux=${pkgver}")
  conflicts=('linux')
  backup=("etc/mkinitcpio.d/${pkgbase}.preset")
  install=${pkgname}.install

  cd linux-$_commit

  KARCH=arm64

  # get kernel version
  _kernver="$(make kernelrelease)"
  _basekernel=${_kernver%%-*}
  _basekernel=${_basekernel%.*}

  mkdir -p "${pkgdir}"/{boot,usr/lib/modules}
  make INSTALL_MOD_PATH="${pkgdir}/usr" modules_install
  make INSTALL_DTBS_PATH="${pkgdir}/boot/dtbs" dtbs_install
  cp arch/$KARCH/boot/Image{,.gz} "${pkgdir}/boot"

  # make room for external modules
  local _extramodules="extramodules-${_basekernel}${_kernelname}"
  ln -s "../${_extramodules}" "${pkgdir}/usr/lib/modules/${_kernver}/extramodules"

  # add real version for building modules and running depmod from hook
  echo "${_kernver}" |
    install -Dm644 /dev/stdin "${pkgdir}/usr/lib/modules/${_extramodules}/version"

  # remove build and source links
  rm "${pkgdir}"/usr/lib/modules/${_kernver}/{source,build}

  # now we call depmod...
  depmod -b "${pkgdir}/usr" -F System.map "${_kernver}"

  # sed expression for following substitutions
  local _subst="
    s|%PKGBASE%|${pkgbase}|g
    s|%KERNVER%|${_kernver}|g
    s|%EXTRAMODULES%|${_extramodules}|g
  "

  # install mkinitcpio preset file
  sed "${_subst}" ../linux.preset |
    install -Dm644 /dev/stdin "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"

  # install pacman hooks
  sed "${_subst}" ../60-linux.hook |
    install -Dm644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/60-${pkgbase}.hook"
  sed "${_subst}" ../90-linux.hook |
    install -Dm644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/90-${pkgbase}.hook"
}

_package-headers() {
  pkgdesc="Header files and scripts for building modules for linux kernel - ${_desc}"
  provides=("linux-headers=${pkgver}")
  conflicts=('linux-headers')

  cd linux-$_commit
  local _builddir="${pkgdir}/usr/lib/modules/${_kernver}/build"

  echo "Installing build files..."
  install -Dt "${_builddir}" -m644 Makefile .config Module.symvers System.map \
    vmlinux
  install -Dt "${_builddir}/kernel" -m644 kernel/Makefile

  mkdir "${_builddir}/.tmp_versions"

  echo "Installing headers..."
  cp -t "${_builddir}" -a include scripts

  install -Dt "${_builddir}/arch/${KARCH}" -m644 arch/${KARCH}/Makefile
  install -Dt "${_builddir}/arch/${KARCH}/kernel" -m644 arch/${KARCH}/kernel/asm-offsets.s

  cp -t "${_builddir}/arch/${KARCH}" -a arch/${KARCH}/include
  mkdir -p "${_builddir}/arch/arm"
  cp -t "${_builddir}/arch/arm" -a arch/arm/include

  install -Dt "${_builddir}/drivers/md" -m644 drivers/md/*.h
  install -Dt "${_builddir}/net/mac80211" -m644 net/mac80211/*.h

  # http://bugs.archlinux.org/task/13146
  install -Dt "${_builddir}/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h

  # http://bugs.archlinux.org/task/20402
  install -Dt "${_builddir}/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
  install -Dt "${_builddir}/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
  install -Dt "${_builddir}/drivers/media/tuners" -m644 drivers/media/tuners/*.h

  # add xfs and shmem for aufs building
  mkdir -p "${_builddir}"/{fs/xfs,mm}

  echo "Installing KConfig files..."
  find . -name Kconfig\* -exec install -Dm644 {} "${_builddir}/{}" \;

  echo "Removing unneeded architectures..."
  local _arch
  for _arch in "${_builddir}"/arch/*/; do
    [[ ${_arch} == */${KARCH}/ || ${_arch} == */arm/ || ${_arch} == */x86/ ]] && continue
    rm -r "${_arch}"
  done

  echo "Removing documentation..."
  rm -r "${_builddir}/Documentation"

  echo "Removing broken symlinks..."
  find -L "${_builddir}" -type l -printf 'Removing %P\n' -delete

  echo "Removing loose objects..."
  find "${_builddir}" -type f -name '*.o' -printf 'Removing %P\n' -delete

  # Fix permissions
  chmod -R u=rwX,go=rX "${_builddir}"

  #echo "Stripping build tools..."
  #local _binary _strip
  #while read -rd '' _binary; do
    #case "$(file -bi "${_binary}")" in
      #*application/x-sharedlib*)  _strip="${STRIP_SHARED}"   ;; # Libraries (.so)
      #*application/x-archive*)    _strip="${STRIP_STATIC}"   ;; # Libraries (.a)
      #*application/x-executable*) _strip="${STRIP_BINARIES}" ;; # Binaries
      #*) continue ;;
    #esac
    #strip ${_strip} "${_binary}"
  #done < <(find "${_builddir}/scripts" -type f -perm -u+w -print0 2>/dev/null)

  #echo "Stripping vmlinux..."
  #${CROSS_COMPILE}strip -v ${STRIP_STATIC} "${_builddir}/vmlinux"
}

pkgname=("$pkgbase" "$pkgbase-headers")
for _p in "${pkgname[@]}"; do
  eval "package_$_p() {
    $(declare -f "_package${_p#$pkgbase}")
    _package${_p#$pkgbase}
  }"
done

md5sums=('SKIP'
         'SKIP'
         '979a787cf84bef9c60da78e72ec96550'
         'f79300740a7350d2d24ab5e120831b52'
         '85d8339bc3d5daf45aa579d578ef0255'
         'd0fd6bd627223d4c9fc001ffff9df401'
         'a020d566b1b55988476b32840b204834'
         'a25d069b2b008ae22aa7fe55716239da'
         '3182f25beb0b76e8abd2e9de8213351d'
         'c2b0ee1805e10cba7072cbcae67830b2'
         '10d200e0a964f24cdb744678a758e22b'
         '86d4a35722b5410e3b29fc92dae15d4b'
         'ce6c81ad1ad1f8b333fd6077d47abdaf'
         '3dc88030a8f2f5a5f97266d99b149f77'
         'a31a435ab6cd8e7a47601159d665ce50'
         'fed6ae4ac4c3f56178fa4aca6c934d6f'
         '594d4f69b956eaab3336b4e01f42eda8'
         'be5a873f638ff5c31947f8d28a824d3a'
         'b4acd66a564af83b5409738c40b4a566'
         'a6407dceae1838f5aa27450401a91be6'
         'cb78b1c11b917a4d31c4b1567183b76f'
         '3efea575da7f02ba94789d3b6b81e11f'
         '2529ad13791b259d80c9d5d702187a65'
         'cb296d7788098cf478c46c0d21376719'
         '50255aac36e002afa477e4527a0550af'
         '6b6def41b404422dc04b39e2f1adffc8'
         '1922e3a7727d2bf51641b98d6d354738'
         'd6b7e4e43e42128cf950251e0d0aee23'
         'ecfd8a30c480149005fcf349e4d06f4b'
	 'SKIP'
	 'SKIP'
	 'SKIP')
