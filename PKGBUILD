pkgname=(
    linux
    linux-headers
    linux-docs
)
pkgbase=linux
pkgver=6.17.2
pkgrel=2
pkgdesc="The linux kernel and modules"
arch=('x86_64')
url="https://www.kernel.org/"
license=('GPL-2.0-only')
makedepends=(
    'bc'
    'coreutils'
    'cpio'
    'dracut'
    'gettext'
    'kmod'
    'libelf'
    'pahole'
    'perl'
    'python'
    'rust'
    'rust-bindgen'
    'rust-src'
    'tar'
    'xz'
)
options=('!strip')
source=(https://cdn.kernel.org/pub/linux/kernel/v6.x/${pkgbase}-${pkgver}.tar.xz
    config)
sha256sums=(fdebcb065065f5c1b8dc68a6fb59cda50cdddbf9103d207c2196d55ea764f57f
    bf86e96ca71c8dcc8e582fc1268fc7a749004197756fe09ef67387a2e05d123e)

export KBUILD_BUILD_HOST=flarebird
export KBUILD_BUILD_USER=${pkgbase}
export KBUILD_BUILD_TIMESTAMP="$(date -Ru${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH})"

prepare() {
    cd ${pkgbase}-${pkgver}

    echo "Setting version..."
    echo "-$pkgrel" > localversion.10-pkgrel
    echo "${pkgbase#linux}" > localversion.20-pkgname

    echo "Setting config..."
    cp ${srcdir}/config .config
    make olddefconfig
    diff -u ${srcdir}/config .config || :

    make -s kernelrelease > version
    echo "Prepared ${pkgbase} version $(<version)"
}

build() {
    cd ${pkgbase}-${pkgver}

    make all
    make -C tools/bpf/bpftool vmlinux.h feature-clang-bpf-co-re=1
}

 package_linux() {
    pkgdesc="The linux kernel and modules"
    depends=('coreutils' 'dracut' 'kmod')

    cd ${pkgbase}-${pkgver}

    local modulesdir=${pkgdir}/usr/lib/modules/$(<version)

    echo "Installing boot image..."
    # systemd expects to find the kernel here to allow hibernation
    # https://github.com/systemd/systemd/commit/edda44605f06a41fb86b7ab8128dcf99161d2344
    install -Dm644 $(make -s image_name) ${modulesdir}/vmlinuz

    # Used by mkinitcpio to name the kernel
    echo ${pkgbase} | install -Dm644 /dev/stdin ${modulesdir}/pkgbase

    echo "Installing modules..."
    ZSTD_CLEVEL=19 make INSTALL_MOD_PATH=${pkgdir}/usr INSTALL_MOD_STRIP=1 DEPMOD=/doesnt/exist modules_install

    # remove build and source links
    rm ${modulesdir}/build

    # now we call depmod...
    # /usr/sbin/depmod -b ${pkgdir}/usr -F System.map $(<version)
 }

package_linux-headers() {
    pkgdesc="Header files and scripts for building modules for linux kernel"
    depends=('pahole')

    cd ${pkgbase}-${pkgver}

    local builddir=${pkgdir}/usr/lib/modules/$(<version)/build

    echo "Installing build files..."
    install -Dt ${builddir} -m644 .config Makefile Module.symvers System.map \
        localversion.* version vmlinux tools/bpf/bpftool/vmlinux.h

    install -Dt ${builddir}/kernel -m644 kernel/Makefile
    install -Dt ${builddir}/arch/x86 -m644 arch/x86/Makefile
    cp -t ${builddir} -a scripts
    ln -srt ${builddir} ${builddir}/scripts/gdb/vmlinux-gdb.py

    # required when STACK_VALIDATION is enabled
    install -Dt ${builddir}/tools/objtool tools/objtool/objtool

    # required when DEBUG_INFO_BTF_MODULES is enabled
    install -Dt ${builddir}/tools/bpf/resolve_btfids tools/bpf/resolve_btfids/resolve_btfids

    echo "Installing headers..."
    cp -t ${builddir} -a include
    cp -t ${builddir}/arch/x86 -a arch/x86/include
    install -Dt ${builddir}/arch/x86/kernel -m644 arch/x86/kernel/asm-offsets.s

    install -Dt ${builddir}/drivers/md -m644 drivers/md/*.h
    install -Dt ${builddir}/net/mac80211 -m644 net/mac80211/*.h

    # https://bugs.archlinux.org/task/13146
    install -Dt ${builddir}/drivers/media/i2c -m644 drivers/media/i2c/msp3400-driver.h

    # https://bugs.archlinux.org/task/20402
    install -Dt ${builddir}/drivers/media/usb/dvb-usb -m644 drivers/media/usb/dvb-usb/*.h
    install -Dt ${builddir}/drivers/media/dvb-frontends -m644 drivers/media/dvb-frontends/*.h
    install -Dt ${builddir}/drivers/media/tuners -m644 drivers/media/tuners/*.h

    # https://bugs.archlinux.org/task/71392
    install -Dt ${builddir}/drivers/iio/common/hid-sensors -m644 drivers/iio/common/hid-sensors/*.h

    echo "Installing KConfig files..."
    find . -name 'Kconfig*' -exec install -Dm644 {} ${builddir}/{} \;

#     echo "Installing Rust files..."
#     install -Dt ${builddir}/rust -m644 rust/*.rmeta
#     install -Dt ${builddir}/rust rust/*.so

    echo "Installing unstripped VDSO..."
    make INSTALL_MOD_PATH=${pkgdir}/usr vdso_install \
        link=  # Suppress build-id symlinks

    echo "Removing unneeded architectures..."
    local arch
    for arch in ${builddir}/arch/*/; do
        [[ ${arch} = */x86/ ]] && continue
        echo "Removing $(basename ${arch})"
        rm -r ${arch}
    done

    echo "Removing documentation..."
    rm -r ${builddir}/Documentation

    echo "Removing broken symlinks..."
    find -L ${builddir} -type l -printf 'Removing %P\n' -delete

    echo "Removing loose objects..."
    find ${builddir} -type f -name '*.o' -printf 'Removing %P\n' -delete

    echo "Stripping build tools..."
    local file
    while read -rd '' file; do
        case "$(file -Sib "${file}")" in
            application/x-sharedlib\;*)      # Libraries (.so)
                strip ${STRIP_SHARED} "${file}";;
            application/x-archive\;*)        # Libraries (.a)
                strip ${STRIP_STATIC} "${file}" ;;
            application/x-executable\;*)     # Binaries
                strip ${STRIP_BINARIES} ""${file}"";;
            application/x-pie-executable\;*) # Relocatable binaries
                strip ${STRIP_SHARED} "${file}" ;;
        esac
    done < <(find ${builddir} -type f -perm -u+x ! -name vmlinux -print0)

    echo "Stripping vmlinux..."
    strip -v ${STRIP_STATIC} ${builddir}/vmlinux

    echo "Adding symlink..."
    install -vdm755 ${pkgdir}/usr/src
    ln -sr ${builddir} ${pkgdir}/usr/src/${pkgbase}

}

 package_linux-docs() {
    pkgdesc="Documentation for the linux kernel"

    cd ${pkgbase}-${pkgver}

    local builddir=${pkgdir}/usr/lib/modules/$(<version)/build

    echo "Installing documentation..."
    local src dst
    while read -rd '' src; do
        dst=${src#Documentation/}
        dst=${builddir}/Documentation/${dst#output/}
        install -Dm644 ${src} ${dst}
    done < <(find Documentation -name '.*' -prune -o ! -type d -print0)

    echo "Adding symlink..."
    install -vdm755 ${pkgdir}/usr/share/doc
    ln -sr ${builddir}/Documentation ${pkgdir}/usr/share/doc/${pkgbase}-${pkgver}
 }
