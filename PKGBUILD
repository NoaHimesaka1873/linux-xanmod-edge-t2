# Maintainer: Noa Himesaka <himesaka@noa.codes>
# Original Maintainer: Joan Figueras <ffigue at gmail dot com>
# Contributor: Torge Matthies <openglfreak at googlemail dot com>
# Contributor: Jan Alexander Steffens (heftig) <jan.steffens@gmail.com>

##
## The following variables can be customized at build time. Use env or export to change at your wish
##
##   Example: env _microarchitecture=98 use_numa=n use_tracers=n makepkg -sc
##
## Look inside 'choose-gcc-optimization.sh' to choose your microarchitecture
## Valid numbers between: 0 to 99
## Default is: 0 => generic
## Good option if your package is for one machine: 98 (Intel native) or 99 (AMD native)
if [ -z ${_microarchitecture+x} ]; then
  _microarchitecture=0
fi

## Disable NUMA since most users do not have multiple processors. Breaks CUDA/NvEnc.
## Archlinux and Xanmod enable it by default.
## Set variable "use_numa" to: n to disable (possibly increase performance)
##                             y to enable  (stock default)
if [ -z ${use_numa+x} ]; then
  use_numa=y
fi

## For performance you can disable FUNCTION_TRACER/GRAPH_TRACER. Limits debugging and analyzing of the kernel.
## Stock Archlinux and Xanmod have this enabled. 
## Set variable "use_tracers" to: n to disable (possibly increase performance)
##                                y to enable  (stock default)
if [ -z ${use_tracers+x} ]; then
  use_tracers=y
fi

## NOTICE: clang config is not ready yet in 5.17.x
## Choose between GCC and CLANG config (default is GCC)
if [ -z ${_compiler+x} ] || [ "$_compiler" = "clang" ]; then
  _compiler=gcc
fi

# Choose between the 3 main configs for EDGE branch. Default x86-64-v2 which use CONFIG_GENERIC_CPU2:
# Possible values: config_x86-64 / config_x86-64-v2 (default) / config_x86-64-v3
# This will be overwritten by selecting any option in microarchitecture script
# Source files: https://github.com/xanmod/linux/tree/5.17/CONFIGS/xanmod/gcc
if [ -z ${_config+x} ]; then
  _config=config_x86-64-v2
fi

# Compress modules with ZSTD (to save disk space)
if [ -z ${_compress_modules+x} ]; then
  _compress_modules=n
fi

# Compile ONLY used modules to VASTLY reduce the number of modules built
# and the build time.
#
# To keep track of which modules are needed for your specific system/hardware,
# give module_db script a try: https://aur.archlinux.org/packages/modprobed-db
# This PKGBUILD read the database kept if it exists
#
# More at this wiki page ---> https://wiki.archlinux.org/index.php/Modprobed-db
if [ -z ${_localmodcfg} ]; then
  _localmodcfg=n
fi

# Tweak kernel options prior to a build via nconfig
if [ -z ${_makenconfig} ]; then
  _makenconfig=n
fi

### IMPORTANT: Do no edit below this line unless you know what you're doing

pkgbase=linux-xanmod-edge-t2
_major=6.0
pkgver=${_major}.8
_branch=6.x
xanmod=1
pkgrel=1
pkgdesc='Linux Xanmod - Latest Mainline (EDGE) for T2 Macs'
url="http://www.xanmod.org/"
arch=(x86_64)

license=(GPL2)
makedepends=(
  bc cpio kmod libelf perl tar xz
)
if [ "${_compiler}" = "clang" ]; then
  makedepends+=(clang llvm lld python)
fi
options=('!strip')
_srcname="linux-${pkgver}-xanmod${xanmod}"

source=("https://cdn.kernel.org/pub/linux/kernel/v${_branch}/linux-${_major}.tar."{xz,sign}
        "https://github.com/xanmod/linux/releases/download/${pkgver}-xanmod${xanmod}/patch-${pkgver}-xanmod${xanmod}.xz"
        choose-gcc-optimization.sh
  # apple-bce, apple-ibridge
  apple-bce::git+https://github.com/t2linux/apple-bce-drv#commit=6988ec2f08ed7092211540ae977f4ddb56d4fd49
  apple-ibridge::git+https://github.com/Redecorating/apple-ib-drv#commit=467df9b11cb55456f0365f40dd11c9e666623bf3

  1001-Put-apple-bce-and-apple-ibridge-in-drivers-staging.patch
  1002-add-modalias-to-apple-bce.patch

  # Fix some acpi errors
  2001-fix-acpica-for-zero-arguments-acpi-calls.patch

  # Misc BCE patches
  2011-change-many-info-logs-to-debug.patch
  2013-aaudio-set-the-card-driver-name-to-AppleT2x-channel-.patch

  # Apple SMC ACPI support
  3001-applesmc-convert-static-structures-to-drvdata.patch
  3002-applesmc-make-io-port-base-addr-dynamic.patch
  3003-applesmc-switch-to-acpi_device-from-platform.patch
  3004-applesmc-key-interface-wrappers.patch
  3005-applesmc-basic-mmio-interface-implementation.patch
  3006-applesmc-fan-support-on-T2-Macs.patch
  3007-applesmc-Add-iMacPro-to-applesmc_whitelist.patch

  # T2 USB Touchpad support
  4001-Input-bcm5974-Add-support-for-the-T2-Macs.patch

  # Keyboard Layout fixes
  5001-HID-apple-fix-key-translations-where-multiple-quirks.patch
  5002-HID-apple-enable-APPLE_ISO_TILDE_QUIRK-for-the-keybo.patch

  # Hack for i915 overscan issues
  7001-drm-i915-fbdev-Discard-BIOS-framebuffers-exceeding-h.patch

  # Broadcom WIFI device support
  # https://github.com/AsahiLinux/linux/commits/bits/080-wifi
  8001-asahilinux-wifi-patchset.patch

  # Broadcom BCM4377 BT device support
  # https://github.com/AsahiLinux/linux/commits/bluetooth-wip
  8002-asahilinux-hci_bcm4377-patchset.patch

)
        #"patch-${pkgver}-xanmod${xanmod}.xz::https://sourceforge.net/projects/xanmod/files/releases/stable/${pkgver}-xanmod${xanmod}/patch-${pkgver}-xanmod${xanmod}.xz/download"
validpgpkeys=(
    'ABAF11C65A2970B130ABE3C479BE3E4300411886' # Linux Torvalds
    '647F28654894E3BD457199BE38DBBDC86092693E' # Greg Kroah-Hartman
)

# Archlinux patches
_commit="ec9e9a4219fe221dec93fa16fddbe44a34933d8d"
_patches=()
for _patch in ${_patches[@]}; do
    #source+=("${_patch}::https://git.archlinux.org/svntogit/packages.git/plain/trunk/${_patch}?h=packages/linux&id=${_commit}")
    source+=("${_patch}::https://raw.githubusercontent.com/archlinux/svntogit-packages/${_commit}/trunk/${_patch}")
done

sha256sums=('5c2443a5538de52688efb55c27ab0539c1f5eb58c0cfd16a2b9fbb08fd81788e'
            'SKIP'
            'a620248a4f972e27433efe801c5be4c3f4cae11aa268989e326c5e601b29b971'
            'cce93915896d9136b544216f8566b80a4f4a10111e654ae12f0fc9fe4d42343a'
            'SKIP'
            'SKIP'
            '4482f285a66a31561452c81f232ec9c4396dc95c40a37645c7c47d7bc8b26184'
            'a3a43feaffccbcd119f4a1b4e1299ef07ae36ef9bffc17767bf10e447fa02a2a'
            '45b911e592dd6c717e77ec4d8cbf844860bb7c29ace7a7170e7bf59c12e91bb4'
            '32d3915b4d50cfc654dda53e65e633d1e99b6c98795cbb7416f1ae8fe1ea2321'
            '515756555e7a6178f38c82bb1dbc2919aa9660ee8b9e158f3764948578dee92c'
            'cfd23a06797ac86575044428a393dd7f10f06eff7648d0b78aedad82cbe41279'
            '8d8401a99a9dfbc41aa2dc5b6a409a19860b1b918465e19de4a4ff18de075ea3'
            '08d165106fe35b68a7b48f216566951a5db0baac19098c015bcc81c5fcba678d'
            '62f6d63815d4843ca893ca76b84a9d32590a50358ca0962017ccd75a40884ba8'
            '2827dab6eeb2d2a08034938024f902846b5813e967a0ea253dc1ea88315da383'
            '398dec7d54c6122ae2263cd5a6d52353800a1a60fd85e52427c372ea9974a625'
            'd4ca5a01da5468a1d2957b8eb4a819e1b867a3bcd1cd47389d7c9ac9154b5430'
            'b1f19084e9a9843dd8c457c55a8ea8319428428657d5363d35df64fb865a4eae'
            '7d27bd83133c2e883e854271c5f9f698c61196afc2922921675353303194ef2c'
            '4db195e0bda5712e60a78266c1458037063e5debd646b08376c4700a27d4b4ef'
            '9ede98eceb69e9c93e25fdb2c567466963bdd2f81c0ecb9fb9e5107f6142ff26'
            'e27a4acdb9027a0652d558d619b5be3dc916d2472f3b4d01d10932fc6f35f8dc'
            'fc22ff1285552a85148ec5c21a9e5d93f2420a806ebdc53894636ec5f17505a8')

prepare() {
  cd linux-${_major}

  # Apply Xanmod patch
  patch -Np1 -i ../patch-${pkgver}-xanmod${xanmod}

  msg2 "Setting version..."
  scripts/setlocalversion --save-scmversion
  echo "-$pkgrel" > localversion.10-pkgrel

  for i in apple-bce apple-ibridge; do
    echo "Copying $i in to drivers/staging..."
	# no need to copy .git/
	mkdir drivers/staging/$i
    cp -r $srcdir/$i/* drivers/staging/$i/
  done

  # Archlinux patches
  local src
  for src in "${source[@]}"; do
    src="${src%%::*}"
    src="${src##*/}"
    [[ $src = *.patch ]] || continue
    msg2 "Applying patch $src..."
    patch -Np1 < "../$src"
  done

  # Applying configuration
  cp -vf CONFIGS/xanmod/${_compiler}/${_config} .config
  # enable LTO_CLANG_THIN
  if [ "${_compiler}" = "clang" ]; then
    scripts/config --disable LTO_CLANG_FULL
    scripts/config --enable LTO_CLANG_THIN
    _LLVM=1
  fi

  # CONFIG_STACK_VALIDATION gives better stack traces. Also is enabled in all official kernel packages by Archlinux team
  scripts/config --enable CONFIG_STACK_VALIDATION

  # Enable IKCONFIG following Arch's philosophy
  scripts/config --enable CONFIG_IKCONFIG \
                 --enable CONFIG_IKCONFIG_PROC

  # User set. See at the top of this file
  if [ "$use_tracers" = "n" ]; then
    msg2 "Disabling FUNCTION_TRACER/GRAPH_TRACER only if we are not compiling with clang..."
    if [ "${_compiler}" = "gcc" ]; then
      scripts/config --disable CONFIG_FUNCTION_TRACER \
                     --disable CONFIG_STACK_TRACER
    fi
  fi

  if [ "$use_numa" = "n" ]; then
    msg2 "Disabling NUMA..."
    scripts/config --disable CONFIG_NUMA
  fi

  # Compress modules by default (following Arch's kernel)
  if [ "$_compress_modules" = "y" ]; then
    scripts/config --enable CONFIG_MODULE_COMPRESS_ZSTD
  fi
  
  # Enable Bluetooth for BCM4377
  scripts/config --module BT_HCIBCM4377
  
  # Let's user choose microarchitecture optimization in GCC
  # Use default microarchitecture only if we have not choosen another microarchitecture
  if [ "$_microarchitecture" -ne "0" ]; then
    sh ../choose-gcc-optimization.sh $_microarchitecture
  fi

  # This is intended for the people that want to build this package with their own config
  # Put the file "myconfig" at the package folder (this will take preference) or "${XDG_CONFIG_HOME}/linux-xanmod/myconfig"
  # If we detect partial file with scripts/config commands, we execute as a script
  # If not, it's a full config, will be replaced
  for _myconfig in "${SRCDEST}/myconfig" "${HOME}/.config/linux-xanmod/myconfig" "${XDG_CONFIG_HOME}/linux-xanmod/myconfig" ; do
    if [ -f "${_myconfig}" ] && [ "$(wc -l <"${_myconfig}")" -gt "0" ]; then
      if grep -q 'scripts/config' "${_myconfig}"; then
        # myconfig is a partial file. Executing as a script
        msg2 "Applying myconfig..."
        bash -x "${_myconfig}"
      else
        # myconfig is a full config file. Replacing default .config
        msg2 "Using user CUSTOM config..."
        cp -f "${_myconfig}" .config
      fi
      echo
      break
    fi
  done

  ### Optionally load needed modules for the make localmodconfig
  # See https://aur.archlinux.org/packages/modprobed-db
  if [ "$_localmodcfg" = "y" ]; then
    if [ -f $HOME/.config/modprobed.db ]; then
      msg2 "Running Steven Rostedt's make localmodconfig now"
      make LLVM=$_LLVM LLVM_IAS=$_LLVM LSMOD=$HOME/.config/modprobed.db localmodconfig
    else
      msg2 "No modprobed.db data found"
      exit 1
    fi
  fi

  make LLVM=$_LLVM LLVM_IAS=$_LLVM olddefconfig

  make -s kernelrelease > version
  msg2 "Prepared %s version %s" "$pkgbase" "$(<version)"

  if [ "$_makenconfig" = "y" ]; then
    make LLVM=$_LLVM LLVM_IAS=$_LLVM nconfig
  fi

  # save configuration for later reuse
  cat .config > "${SRCDEST}/config.last"
}

build() {
  cd linux-${_major}
  make LLVM=$_LLVM LLVM_IAS=$_LLVM all
}

_package() {
  pkgdesc="The Linux kernel and modules with Xanmod patches"
  depends=(coreutils kmod initramfs)
  optdepends=('crda: to set the correct wireless channels of your country'
              'linux-firmware: firmware images needed for some devices')
  provides=(VIRTUALBOX-GUEST-MODULES
            linux-t2
            WIREGUARD-MODULE
            KSMBD-MODULE
            NTFS3-MODULE)

  cd linux-${_major}
  local kernver="$(<version)"
  local modulesdir="$pkgdir/usr/lib/modules/$kernver"

  msg2 "Installing boot image..."
  # systemd expects to find the kernel here to allow hibernation
  # https://github.com/systemd/systemd/commit/edda44605f06a41fb86b7ab8128dcf99161d2344
  install -Dm644 "$(make -s image_name)" "$modulesdir/vmlinuz"

  # Used by mkinitcpio to name the kernel
  echo "$pkgbase" | install -Dm644 /dev/stdin "$modulesdir/pkgbase"

  msg2 "Installing modules..."
  make INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 modules_install

  # remove build and source links
  rm "$modulesdir"/{source,build}
}

_package-headers() {
  pkgdesc="Headers and scripts for building modules for the $pkgdesc kernel"
  provides=(linux-t2-headers)
  depends=(pahole)

  cd linux-${_major}
  local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

  msg2 "Installing build files..."
  install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map \
    localversion.* version vmlinux
  install -Dt "$builddir/kernel" -m644 kernel/Makefile
  install -Dt "$builddir/arch/x86" -m644 arch/x86/Makefile
  cp -t "$builddir" -a scripts

  # required when STACK_VALIDATION is enabled
  install -Dt "$builddir/tools/objtool" tools/objtool/objtool

  # required when DEBUG_INFO_BTF_MODULES is enabled
  if [ -f "$builddir/tools/bpf/resolve_btfids" ]; then install -Dt "$builddir/tools/bpf/resolve_btfids" tools/bpf/resolve_btfids/resolve_btfids ; fi

  msg2 "Installing headers..."
  cp -t "$builddir" -a include
  cp -t "$builddir/arch/x86" -a arch/x86/include
  install -Dt "$builddir/arch/x86/kernel" -m644 arch/x86/kernel/asm-offsets.s

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

  msg2 "Removing unneeded architectures..."
  local arch
  for arch in "$builddir"/arch/*/; do
    [[ $arch = */x86/ ]] && continue
    echo "Removing $(basename "$arch")"
    rm -r "$arch"
  done

  msg2 "Removing documentation..."
  rm -r "$builddir/Documentation"

  msg2 "Removing broken symlinks..."
  find -L "$builddir" -type l -printf 'Removing %P\n' -delete

  msg2 "Removing loose objects..."
  find "$builddir" -type f -name '*.o' -printf 'Removing %P\n' -delete

  msg2 "Stripping build tools..."
  local file
  while read -rd '' file; do
    case "$(file -bi "$file")" in
      application/x-sharedlib\;*)      # Libraries (.so)
        strip -v $STRIP_SHARED "$file" ;;
      application/x-archive\;*)        # Libraries (.a)
        strip -v $STRIP_STATIC "$file" ;;
      application/x-executable\;*)     # Binaries
        strip -v $STRIP_BINARIES "$file" ;;
      application/x-pie-executable\;*) # Relocatable binaries
        strip -v $STRIP_SHARED "$file" ;;
    esac
  done < <(find "$builddir" -type f -perm -u+x ! -name vmlinux -print0)

  msg2 "Stripping vmlinux..."
  strip -v $STRIP_STATIC "$builddir/vmlinux"
  
  msg2 "Adding symlink..."
  mkdir -p "$pkgdir/usr/src"
  ln -sr "$builddir" "$pkgdir/usr/src/$pkgbase"
}

pkgname=("${pkgbase}" "${pkgbase}-headers")
for _p in "${pkgname[@]}"; do
  eval "package_$_p() {
    $(declare -f "_package${_p#$pkgbase}")
    _package${_p#$pkgbase}
  }"
done

# vim:set ts=8 sts=2 sw=2 et:
