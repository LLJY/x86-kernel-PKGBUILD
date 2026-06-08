# Maintainer: Lucas Lee Jing Yi <lucasleeeeeeeee@gmail.com>

pkgbase=linux-duality
pkgname=("${pkgbase}" "${pkgbase}-headers")
pkgrel=2
pkgver=7.0.11 # NOTE: Hardcoded version, pkgver() function below might override if uncommented properly
_localmodver="-Duality"
_basepkgdesc="Custom Linux kernel (Duality build with Clang/LTO)"
pkgdesc="${_basepkgdesc}"
arch=('x86_64')
url="https://github.com/LLJY/x86-kernel"
license=('GPL2')
makedepends=(
    'base-devel' 'bc' 'pahole' 'git'
    'xmlto' 'docbook-xsl' 'kmod' 'inetutils' 'cpio' 'perl'
    'clang' 'lld' 'llvm'
)
options=(!strip)

source=(
    "git+https://github.com/LLJY/x86-kernel.git#branch=7.0"
    'config.x86_64'
    "${pkgbase}.preset"
)
sha256sums=('SKIP'
            'eb2e20d190ada94ee330b546e5abdda6e74c1c4bc4226eda8ff49443173ef7e5'
            '754ddad0aadd7a58148d076788d489a0958c0e30d89460aa1946b249b2fb429d')

# Automatically determine pkgver from git repo
pkgver() {
 cd "x86-kernel"
 printf "%s" "$(make -s kernelversion)"
}

# --- PREPARE ---
prepare() {
    cd "x86-kernel"
    export LLVM=1
    export CC=clang
    export CXX=clang++
    export HOSTCC=clang
    export LLVM_IAS=1
    cp ../config.x86_64 .config
    echo "OBJECT_FILES_NON_STANDARD_amdgpu.o := y" >> drivers/gpu/drm/amd/amdgpu/Makefile
    make modules_prepare
}

# --- BUILD ---
build() {
    cd "x86-kernel"
    export LLVM=1
    export CC=clang
    export CXX=clang++
    export HOSTCC=clang
    export LLVM_IAS=1
    make LOCALVERSION=${_localmodver}
}

# --- PACKAGE ---
package_linux-duality() {
    pkgdesc="${_basepkgdesc} kernel and modules"
    depends=(
        'coreutils' 'linux-firmware' 'kmod'
    )
    optdepends=(
       'mkinitcpio: Default Arch initramfs generator (needed for system hook / preset support)'
       'dracut: Alternative initramfs generator (user must configure hooks/trigger)'
    )
    provides=("linux=${pkgver}" "linux-mainline=${pkgver}")
    conflicts=('linux-duality')
    backup=('etc/mkinitcpio.d/linux-duality.preset')
    install=${pkgbase}.install

    local _pkgbase="${pkgbase}"
    local _kernelfullver_real
    local _modules_dir

    cd "x86-kernel"
    _kernelfullver_real=$(make -s kernelrelease LOCALVERSION=${_localmodver})
    _modules_dir="${pkgdir}/usr/lib/modules/${_kernelfullver_real}"

    # Install kernel modules
    make INSTALL_MOD_PATH="${pkgdir}/usr" INSTALL_MOD_STRIP=1 modules_install

    # Headers package owns the build/source links and build tree.
    rm -f "${_modules_dir}/build" "${_modules_dir}/source"

    # --- Create and populate pkgbase file this will ensure dracut and mkinitcpio hooks run correctly---
    install -Dm644 /dev/null "${_modules_dir}/pkgbase"
    echo "${_pkgbase}" > "${_modules_dir}/pkgbase"
    # --- End pkgbase file creation ---

    # Dracut expects the kernel image in the module directory.
    install -Dm644 "arch/x86_64/boot/bzImage" "${_modules_dir}/vmlinuz"

    # Install files to /boot
    install -Dm644 "arch/x86_64/boot/bzImage" "${pkgdir}/boot/vmlinuz-${pkgname}"
    install -Dm644 "System.map" "${pkgdir}/boot/System.map-${pkgname}"
    install -Dm644 ".config" "${pkgdir}/boot/config-${pkgname}"

    # Install mkinitcpio preset (Useful for mkinitcpio users, ignored by dracut users)
    install -Dm644 "../${pkgname}.preset" "${pkgdir}/etc/mkinitcpio.d/${pkgname}.preset"
    sed -i -e "s|%PKGNAME%|${pkgname}|g" \
           -e "s|%KERNELVER%|${_kernelfullver_real}|g" \
           "${pkgdir}/etc/mkinitcpio.d/${pkgname}.preset"

    install -Dm644 COPYING "${pkgdir}/usr/share/licenses/${pkgname}/COPYING"
}

package_linux-duality-headers() {
    pkgdesc="Headers and scripts for building modules for the ${_basepkgdesc} kernel"
    depends=(
        "${pkgbase}=${pkgver}-${pkgrel}"
        'binutils' 'glibc' 'libelf' 'openssl' 'pahole' 'xxhash' 'zlib' 'zstd'
    )
    provides=("linux-headers=${pkgver}")
    conflicts=()
    backup=()
    install=

    cd "x86-kernel"
    local _kernelfullver_real
    _kernelfullver_real=$(make -s kernelrelease LOCALVERSION=${_localmodver})
    local _builddir="${pkgdir}/usr/lib/modules/${_kernelfullver_real}/build"

    echo "Installing build files..."
    install -Dt "${_builddir}" -m644 .config Makefile Module.symvers System.map vmlinux
    install -Dt "${_builddir}/kernel" -m644 kernel/Makefile
    install -Dt "${_builddir}/arch/x86" -m644 arch/x86/Makefile
    cp -t "${_builddir}" -a scripts
    ln -srt "${_builddir}" "${_builddir}/scripts/gdb/vmlinux-gdb.py"

    # Required when STACK_VALIDATION/objtool and BTF module builds are enabled.
    install -Dt "${_builddir}/tools/objtool" tools/objtool/objtool
    if [ -f tools/bpf/resolve_btfids/resolve_btfids ]; then
        install -Dt "${_builddir}/tools/bpf/resolve_btfids" tools/bpf/resolve_btfids/resolve_btfids
    fi

    echo "Installing headers..."
    cp -t "${_builddir}" -a include
    cp -t "${_builddir}/arch/x86" -a arch/x86/include
    install -Dt "${_builddir}/arch/x86/kernel" -m644 arch/x86/kernel/asm-offsets.s
    install -Dt "${_builddir}/drivers/md" -m644 drivers/md/*.h
    install -Dt "${_builddir}/net/mac80211" -m644 net/mac80211/*.h

    # Headers required by some out-of-tree media drivers.
    install -Dt "${_builddir}/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h
    install -Dt "${_builddir}/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
    install -Dt "${_builddir}/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
    install -Dt "${_builddir}/drivers/media/tuners" -m644 drivers/media/tuners/*.h
    install -Dt "${_builddir}/drivers/iio/common/hid-sensors" -m644 drivers/iio/common/hid-sensors/*.h

    echo "Installing Kconfig files..."
    find . -name 'Kconfig*' -exec install -Dm644 {} "${_builddir}/{}" \;

    echo "Installing unstripped VDSO..."
    make INSTALL_MOD_PATH="${pkgdir}/usr" vdso_install link=

    echo "Removing unneeded architectures..."
    local _arch
    for _arch in "${_builddir}"/arch/*/; do
        [[ ${_arch} = */x86/ ]] && continue
        rm -r "${_arch}"
    done

    echo "Removing documentation and broken symlinks..."
    rm -rf "${_builddir}/Documentation"
    find -L "${_builddir}" -type l -printf 'Removing %P\n' -delete

    echo "Removing loose objects..."
    find "${_builddir}" -type f -name '*.o' -printf 'Removing %P\n' -delete

    echo "Stripping build tools..."
    local _file
    while read -rd '' _file; do
        case "$(file -Sib "${_file}")" in
            application/x-sharedlib\;*)      strip -v ${STRIP_SHARED} "${_file}" ;;
            application/x-archive\;*)        strip -v ${STRIP_STATIC} "${_file}" ;;
            application/x-executable\;*)     strip -v ${STRIP_BINARIES} "${_file}" ;;
            application/x-pie-executable\;*) strip -v ${STRIP_SHARED} "${_file}" ;;
        esac
    done < <(find "${_builddir}" -type f -perm -u+x ! -name vmlinux -print0)

    echo "Stripping vmlinux..."
    strip -v ${STRIP_STATIC} "${_builddir}/vmlinux"

    echo "Adding source symlink..."
    mkdir -p "${pkgdir}/usr/src"
    ln -sr "${_builddir}" "${pkgdir}/usr/src/${pkgbase}"
    ln -sr "${_builddir}" "${pkgdir}/usr/lib/modules/${_kernelfullver_real}/source"
}
