# Maintainer: Chaiwat Suttipongsakul <cwt@bashell.com>

pkgbase=linux-cwt-6.6-th1520-lpi4a
_variant=cwt
_commit=d1fe4fea56f506a84fb150ac26604a747376c087
pkgver=${_commit::7}
pkgrel=3
_desc='Linux 6.6.x (-cwt) for Sipeed Lichee Pi 4A'
_reponame=th1520-linux-kernel
_srcname=${_reponame}-${_commit}
url="https://github.com/revyos/${_srcname}"
arch=(riscv64)
license=('GPL2')
makedepends=(bc libelf pahole cpio perl tar xz gcc13 inetutils)
options=('!strip')
source=("https://github.com/revyos/${_reponame}/archive/${_commit}.tar.gz"
  '1-add_TH1520_C910_CFLAGS_AFLAGS.patch'
  'config'
  'linux.preset'
  '90-linux.hook')

b2sums=('46815b3d287803d86b7ca97b40a25eeda65b80adeea6b6d29267ccafc68e8354ff66417541e5fef4d4855ff2548f86fede4928f7f9a84c2bcfcb094bbe67bea2'
        '03d734e44d7e3b7734afd56c57e7cde2925e8c5f6716ce2de53325e4c237498075755605c021428a25b74fb84a5f2e47d0e64e0e633160ae54c03f467de7d4ff'
        '94d43fd814220f7ac588efec68d4ae07bf5b88af05fdec6e99237f2f6eed36c92d820426dafd5cd81449d00749c4ef3c380348f20c9c07ded60e162e34b8b12d'
        'ee004ec1cf9161a960cb9273dd9446bd072c0ff36ccd843e9ed8fed70e3290a5450641798d060f1a20d5a6578e1b47f0b789480aacba7ae0c04ef37c6dbeff0b'
        '63df7548484b4caf0fa0c4c9f5537ad109713a325b9b9a3e310f201ac8f2c13a29cb9ff790cdbac2e461d8b668a98234e11dd2c1b62228ee3c76247136d42dfc')

prepare() {
  cd $_srcname

  local src
  for src in $(ls ../*.patch); do
    echo "Applying patch $src..."
    patch -Np1 <"../$src"
  done

  echo "Setting version..."
  echo "-${_variant}" >localversion.10-variant
  echo "-${pkgver}" >localversion.20-pkgver
  echo "-$pkgrel" >localversion.30-pkgrel

  echo "Setting config..."
  cp ../config .config
  unset CFLAGS
  CCACHE=$(which ccache 2>/dev/null)
  if [ "$CCACHE" != "" ]; then
    gcc="${CCACHE} gcc-13"
  else
    gcc="gcc-13"
  fi
  make HOSTCC="${gcc}" CC="${gcc}" -j $(nproc) olddefconfig
  cp .config ../../config.new

  make HOSTCC="${gcc}" CC="${gcc}" -j $(nproc) -s kernelrelease >version
  echo "Prepared $pkgbase version $(<version)"
}

build() {
  unset CFLAGS
  CCACHE=$(which ccache 2>/dev/null)
  if [ "$CCACHE" != "" ]; then
    gcc="${CCACHE} gcc-13"
  else
    gcc="gcc-13"
  fi
  cd $_srcname
  make HOSTCC="${gcc}" CC="${gcc}" -j $(nproc) all
}

_package() {
  pkgdesc="The $_desc kernel and modules"
  depends=(coreutils kmod mkinitcpio)
  optdepends=('wireless-regdb: to set the correct wireless channels of your country'
    'linux-firmware: firmware images needed for some devices')
  provides=("linux=${pkgver}" "WIREGUARD-MODULE")
  conflicts=('linux')

  cd $_srcname
  local kernver="$(<version)"
  local modulesdir="$pkgdir/usr/lib/modules/$kernver"

  echo "Installing boot image..."
  install -Dm644 "arch/riscv/boot/Image" "$modulesdir/vmlinux"
  install -Dm644 "arch/riscv/boot/Image" "$pkgdir/boot/vmlinux"

  echo "Installing modules..."
  unset CFLAGS
  CCACHE=$(which ccache 2>/dev/null)
  if [ "$CCACHE" != "" ]; then
    gcc="${CCACHE} gcc-13"
  else
    gcc="gcc-13"
  fi
  make HOSTCC="${gcc}" CC="${gcc}" -j $(nproc) INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 modules_install

  echo "Installing dtbs..."
  make HOSTCC="${gcc}" CC="${gcc}" -j $(nproc) INSTALL_DTBS_PATH="$pkgdir/usr/share/dtbs/$kernver" dtbs_install
  make HOSTCC="${gcc}" CC="${gcc}" -j $(nproc) INSTALL_DTBS_PATH="$pkgdir/boot/dtbs/arch-cwt" dtbs_install

  # remove build links
  rm "$modulesdir"/build

  install -Dm644 ../linux.preset "${pkgdir}/etc/mkinitcpio.d/linux.preset"
  install -Dm644 ../90-linux.hook "${pkgdir}/usr/share/libalpm/hooks/90-linux.hook"
  
  # create link for backward compatibility
  cd "$pkgdir/boot/dtbs/arch-cwt/thead"
  ln -s th1520-lichee-pi-4a-16g.dtb light-lpi4a-16gb.dtb
  ln -s th1520-lichee-pi-4a.dtb light-lpi4a.dtb
}

_package-headers() {
  pkgdesc="Headers and scripts for building modules for the $_desc kernel"
  depends=(pahole)
  provides=("linux-headers=${pkgver}")
  conflicts=('linux-headers')

  cd $_srcname
  local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

  echo "Installing build files..."
  install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map version
  install -Dt "$builddir/kernel" -m644 kernel/Makefile
  install -Dt "$builddir/arch/riscv" -m644 arch/riscv/Makefile
  cp -t "$builddir" -a scripts

  # required when DEBUG_INFO_BTF_MODULES is enabled
  cp --parents -r -t "$builddir/" tools/bpf/resolve_btfids

  echo "Installing VDSO files..."
  cp -a --parents -r -t "$builddir" arch/riscv/kernel/vdso/*
  cp -a --parents -r -t "$builddir" lib/vdso/*
  chmod -R g+w "$builddir/arch/riscv/kernel/vdso"

  echo "Installing certificate files..."
  install -Dt "$builddir/certs" -m640 certs/*.pem
  install -Dt "$builddir/certs" -m640 certs/*.x509

  echo "Installing headers..."
  cp -t "$builddir" -a include
  chmod -R g+w "$builddir/include/generated"
  cp -t "$builddir/arch/riscv" -a arch/riscv/include
  install -Dt "$builddir/arch/riscv/kernel" -m644 arch/riscv/kernel/asm-offsets.s

  install -Dt "$builddir/drivers/md" -m644 drivers/md/*.h
  install -Dt "$builddir/net/mac80211" -m644 net/mac80211/*.h

  # https://bugs.archlinux.org/task/13146
  install -Dt "$builddir/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h

  # https://bugs.archlinux.org/task/20402
  install -Dt "$builddir/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
  install -Dt "$builddir/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
  install -Dt "$builddir/drivers/media/tuners" -m644 drivers/media/tuners/*.h

  # https://bugs.archlinux.org/task/71392
  install -Dt "$builddir/drivers/iio/common/hid-sensors" -m644 drivers/iio/common/hid-sensors/*.h

  echo "Installing KConfig files..."
  find . -name 'Kconfig*' -exec install -Dm644 {} "$builddir/{}" \;

  echo "Removing unneeded architectures..."
  local arch
  for arch in "$builddir"/arch/*/; do
    [[ $arch = */riscv/ ]] && continue
    echo "Removing $(basename "$arch")"
    rm -r "$arch"
  done

  echo "Installing RAS from x86..."
  install -Dt "$builddir/arch/x86/ras"  -m644 arch/x86/ras/Kconfig

  echo "Removing documentation..."
  rm -r "$builddir/Documentation"

  echo "Removing broken symlinks..."
  find -L "$builddir" -type l -printf 'Removing %P\n' -delete

  echo "Removing loose objects..."
  find "$builddir" -type f -name '*.o' -printf 'Removing %P\n' -delete

  echo "Stripping build tools..."
  local file
  while read -rd '' file; do
    case "$(file -bi "$file")" in
    application/x-sharedlib\;*) # Libraries (.so)
      llvm-strip -v $STRIP_SHARED "$file" ;;
    application/x-archive\;*) # Libraries (.a)
      llvm-strip -v $STRIP_STATIC "$file" ;;
    application/x-executable\;*) # Binaries
      llvm-strip -v $STRIP_BINARIES "$file" ;;
    application/x-pie-executable\;*) # Relocatable binaries
      llvm-strip -v $STRIP_SHARED "$file" ;;
    esac
  done < <(find "$builddir" -type f -perm -u+x ! -name vmlinux -print0)

  echo "Adding symlink..."
  mkdir -p "$pkgdir/usr/src"
  ln -sr "$builddir" "$pkgdir/usr/src/$pkgbase"
}

pkgname=("$pkgbase" "$pkgbase-headers")
for _p in "${pkgname[@]}"; do
  eval "package_$_p() {
    $(declare -f "_package${_p#$pkgbase}")
    _package${_p#$pkgbase}
  }"
done

# vim:set ts=8 sts=2 sw=2 et:
