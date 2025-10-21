# Maintainer: Lucas Lee Jing Yi <lucasleeeeeeeee@gmail.com>

pkgname=linux-duality
pkgrel=1
pkgver=6.17.4 # NOTE: Hardcoded version, pkgver() function below might override if uncommented properly
_localmodver="-Duality"
pkgdesc="Custom Linux kernel (Duality build with Clang/LTO)"
arch=('x86_64')
url="https://github.com/LLJY/x86-kernel"
license=('GPL2')
depends=(
    'coreutils' 'linux-firmware' 'kmod'
)
makedepends=(
    'base-devel' 'bc' 'pahole' 'git'
    'xmlto' 'docbook-xsl' 'kmod' 'inetutils' 'cpio' 'perl'
    'clang' 'lld' 'llvm'
)
optdepends=(
   'mkinitcpio: Default Arch initramfs generator (needed for system hook / preset support)'
   'dracut: Alternative initramfs generator (user must configure hooks/trigger)'
)
provides=("linux=${pkgver}" "linux-mainline=${pkgver}")
conflicts=('linux-duality')
backup=('etc/mkinitcpio.d/linux-duality.preset')
options=(!strip)
install=${pkgname}.install

source=(
    "git+https://github.com/LLJY/x86-kernel.git#branch=v6.17-CFS"
    'config.x86_64'
    "${pkgname}.preset"
)
sha256sums=('SKIP'
            'cf6a72b1707a8cc3c9de7d7880838c08940de8cb4445714c03b2b014933b1f50'
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
    # --- WORKAROUND for objtool/LTO/Polly conflict in AMDGPU ---
    # This disables LTO specifically for amdgpu.o, which often resolves
    # objtool validation errors with heavily optimized builds.
    echo "CFLAGS_REMOVE_amdgpu.o = -flto=auto" >> drivers/gpu/drm/amd/amdgpu/Makefile
    # --- End WORKAROUND ---
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
package() {
    local _pkgbase="${pkgname}"

    cd "x86-kernel"
    _kernelfullver_real=$(make -s kernelrelease LOCALVERSION=${_localmodver})

    # Install kernel modules
    make INSTALL_MOD_PATH="${pkgdir}/usr" INSTALL_MOD_STRIP=1 modules_install

    # --- Create and populate pkgbase file this will ensure dracut and mkinitcpio hooks run correctly---
    _modules_dir="${pkgdir}/usr/lib/modules/${_kernelfullver_real}"
    install -Dm644 /dev/null "${_modules_dir}/pkgbase"
    echo "${_pkgbase}" > "${_modules_dir}/pkgbase"
    # --- End pkgbase file creation ---

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
