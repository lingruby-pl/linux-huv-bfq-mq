# Maintainer: Jacek Bienias <sp7ezd@gmail.com>
# Contributor: Piotr Gorski <lucjan.lucjanov@gmail.com>
# Contributor: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: Thomas Baechler <thomas@archlinux.org>

### BUILD OPTIONS
# Set these variables to ANYTHING that is not null to enable them

### Tweak kernel options prior to a build via nconfig
_makenconfig=

### Tweak kernel options prior to a build via menuconfig
_makemenuconfig=

### Tweak kernel options prior to a build via xconfig
_makexconfig=

### Tweak kernel options prior to a build via gconfig
_makegconfig=

### Running with a 100 HZ tick rate 
_100_HZ_ticks=y

### Setting GCC Flags for CONFIG_MHASWELL
_gcc_haswell=y

### Set performance governor as default
_per_gov=y

# NUMA is optimized for multi-socket motherboards.
# A single multi-core CPU actually runs slower with NUMA enabled.
# See, https://bugs.archlinux.org/task/31187
_NUMAdisable=y

# Compile ONLY probed modules
# As of mainline 2.6.32, running with this option will only build the modules
# that you currently have probed in your system VASTLY reducing the number of
# modules built and the build time to do it.
#
# WARNING - ALL modules must be probed BEFORE you begin making the pkg!
#
# To keep track of which modules are needed for your specific system/hardware,
# give module_db script a try: https://aur.archlinux.org/packages/modprobed-db
# This PKGBUILD will call it directly to probe all the modules you have logged!
#
# More at this wiki page ---> https://wiki.archlinux.org/index.php/Modprobed_db
_localmodcfg=

# Use the current kernel's .config file
# Enabling this option will use the .config of the RUNNING kernel rather than
# the ARCH defaults. Useful when the package gets updated and you already went
# through the trouble of customizing your config options.  NOT recommended when
# a new kernel is released, but again, convenient for package bumps.
_use_current=

### Disable Deadline I/O scheduler
_deadline_disable=y

### Disable CFQ I/O scheduler
_CFQ_disable=y

### Disable Kyber I/O scheduler
_kyber_disable=y

### Do not edit below this line unless you know what you're doing

pkgbase=linux-huv-bfq-mq-git
# pkgname=('linux-huv-bfq-mq-git-kernel' 'linux-huv-bfq-mq-git-headers' 'linux-huv-bfq-mq-git-docs')
_srcname=linux-stable
pkgver=4.12.4.r0.g13df91dbc06e
pkgrel=3
arch=('x86_64')
url="https://github.com/Algodev-github/bfq-mq/"
license=('GPL2')
options=('!strip')
makedepends=('kmod' 'inetutils' 'bc' 'libelf')
_gcc_patch="enable_additional_cpu_optimizations_for_gcc_v4.9+_kernel_v3.15+.patch"

source=('git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git#branch=linux-4.12.y'

        "http://repo-ck.com/source/gcc_patch/${_gcc_patch}.gz"
         # the main kernel config files
        'config.x86_64'
         # pacman hook for initramfs regeneration
        '90-linux.hook'
         # standard config files for mkinitcpio ramdisk
        'linux.preset'

        ### BFQ-MQ ###
        "https://gitlab.com/tom81094/custom-patches/raw/master/bfq-mq/4.12-bfq-mq-20170729.patch"
        "https://gitlab.com/tom81094/custom-patches/raw/master/bfq-mq/4.13-uuid-block-merge.patch"
        "https://gitlab.com/tom81094/custom-patches/raw/master/bfq-mq/4.13-linux-block-for-linus_sir_lucjan.patch"
        '0001-Check-presence-on-tree-of-every-entity-after-every-a.patch'
        '0002-BFQ-MQ-bugfix-from-pfkernel.patch'
	### UKSM ###
        "http://kerneldedup.org/download/uksm/0.1.2.6/uksm-0.1.2.6-for-v4.12.patch"
	### VRQ ###
        "https://bitbucket.org/alfredchen/linux-gc/downloads/v4.12_vrq096e.patch"
	### MuQSS ###
        # "http://ck.kolivas.org/patches/muqss/4.0/4.11/4.11-sched-MuQSS_156.patch"
        )

_kernelname=${pkgbase#linux}
        
pkgver() {
  cd "${_srcname}"

  git describe --long | sed -E 's/^v//;s/([^-]*-g)/r\1/;s/-/./g;s/\.rc/rc/'
} 

prepare() {
    cd "${srcdir}/${_srcname}"
            
    ### Patch source with BFQ-MQ
        msg "Fix naming schema in BFQ-MQ patch"
        sed -i -e "s|PATCHLEVEL = 13|PATCHLEVEL = 12|g" \
            -i -e "s|SUBLEVEL = 0|SUBLEVEL = 4|g" \
            -i -e "s|EXTRAVERSION = -rc1|EXTRAVERSION =|g" \
            -i -e "s|EXTRAVERSION = -bfq-rc1|EXTRAVERSION =|g" \
            -i -e "s|EXTRAVERSION =-bfq-mq|EXTRAVERSION =|g" "${srcdir}/4.12-bfq-mq-20170729.patch"
        msg "Patching source with BFQ-MQ patches"
        patch -Np1 -i "${srcdir}/4.12-bfq-mq-20170729.patch"
        patch -Np1 -i "${srcdir}/4.13-uuid-block-merge.patch"
        patch -Np1 -i "${srcdir}/4.13-linux-block-for-linus_sir_lucjan.patch"

    ### Patches related to BUG_ON(entity->tree && entity->tree != &st->active) in __bfq_requeue_entity();
        if [ -n "$_use_tentative_patches" ]; then
        msg "Apply tentative patches"
        for p in "${srcdir}"/0001*.patch*; do patch -Np1 -i "$p"; done
        fi
        
    ### Patches from https://github.com/pfactum/pf-kernel/
        if [ -n "$_use_pfkernel_patch" ]; then
        msg "Apply pfkernel patches"
        for p in "${srcdir}"/0002*.patch*; do patch -Np1 -i "$p"; done
        fi   
        

    ### UKSM ###
	patch -Np1 -i "${srcdir}/uksm-0.1.2.6-for-v4.12.patch"

    ### VRQ ###
	patch -Np1 -i "${srcdir}/v4.12_vrq096e.patch"

    ### MuQSS ###
	# patch -Np1 -i "${srcdir}/4.11-sched-MuQSS_156.patch"

    ### Patch source to enable more gcc CPU optimizatons via the make nconfig
        msg "Patching source with gcc patch to enable more cpus types"
	patch -Np1 -i "${srcdir}/${_gcc_patch}"

	
    ### Clean tree and copy ARCH config over
	msg "Running make mrproper to clean source tree"
	make mrproper

	cat "${srcdir}/config.${CARCH}" > ./.config
        
    ### Optionally use running kernel's config
	# code originally by nous; http://aur.archlinux.org/packages.php?ID=40191
	if [ -n "$_use_current" ]; then
		if [[ -s /proc/config.gz ]]; then
			msg "Extracting config from /proc/config.gz..."
			# modprobe configs
			zcat /proc/config.gz > ./.config
		else
			warning "You kernel was not compiled with IKCONFIG_PROC!"
			warning "You cannot read the current config!"
			warning "Aborting!"
			exit
		fi
	fi    
        
    ### Set GCC flags
        if [ -n "$_gcc_haswell" ]; then
                msg "Setting GCC flags for Intel Haswell CPU..."
                sed -i -e s'/CONFIG_GENERIC_CPU=y/# CONFIG_GENERIC_CPU is not set/' ./.config
                sed -i -e s'/# CONFIG_MHASWELL is not set/CONFIG_MHASWELL=y/' ./.config
        fi  
        
    ### Set performance governor
        if [ -n "$_per_gov" ]; then
                msg "Setting performance governor.."
                sed -i -e s'/CONFIG_CPU_FREQ_DEFAULT_GOV_SCHEDUTIL=y/# CONFIG_CPU_FREQ_DEFAULT_GOV_SCHEDUTIL is not set/' ./.config
                sed -i -e s'/# CONFIG_CPU_FREQ_DEFAULT_GOV_PERFORMANCE is not set/CONFIG_CPU_FREQ_DEFAULT_GOV_PERFORMANCE=y/' ./.config
                msg "Disabling uneeded governors..."
                sed -i -e s'/CONFIG_CPU_FREQ_GOV_ONDEMAND=m/# CONFIG_CPU_FREQ_GOV_ONDEMAND is not set/' ./.config
                sed -i -e s'/CONFIG_CPU_FREQ_GOV_CONSERVATIVE=m/# CONFIG_CPU_FREQ_GOV_CONSERVATIVE is not set/' ./.config
                sed -i -e s'/CONFIG_CPU_FREQ_GOV_USERSPACE=m/# CONFIG_CPU_FREQ_GOV_USERSPACE is not set/' ./.config
                sed -i -e s'/CONFIG_CPU_FREQ_GOV_SCHEDUTIL=y/# CONFIG_CPU_FREQ_GOV_SCHEDUTIL is not set/' ./.config
        fi 
        
    ### Optionally set tickrate to 100 
	if [ -n "$_100_HZ_ticks" ]; then
		msg "Setting tick rate to 100..."
		sed -i -e 's/^CONFIG_HZ_1000=y/# CONFIG_HZ_1000 is not set/' \
			-i -e 's/^# CONFIG_HZ_100 is not set/CONFIG_HZ_100=y/' \
			-i -e 's/^CONFIG_HZ=1000/CONFIG_HZ=100/' .config
	fi
	
	### Disable Deadline I/O scheduler
	if [ -n "$_deadline_disable" ]; then
		msg "Disabling Deadline I/O scheduler..."
		sed -i -e s'/CONFIG_IOSCHED_DEADLINE=y/# CONFIG_IOSCHED_DEADLINE is not set/' ./.config
		sed -i -e s'/CONFIG_MQ_IOSCHED_DEADLINE=y/# CONFIG_MQ_IOSCHED_DEADLINE is not set/' ./.config
	fi
	
	### Disable CFQ I/O scheduler
	if [ -n "$_CFQ_disable" ]; then
		msg "Disabling CFQ I/O scheduler..."
		sed -i -e s'/CONFIG_IOSCHED_CFQ=y/# CONFIG_IOSCHED_CFQ is not set/' ./.config
		sed -i -e s'/CONFIG_CFQ_GROUP_IOSCHED=y/# CONFIG_CFQ_GROUP_IOSCHED is not set/' ./.config	
	fi
	
	### Disable Kyber I/O scheduler
	if [ -n "$_kyber_disable" ]; then
		msg "Disabling Kyber I/O scheduler..."
		sed -i -e s'/CONFIG_MQ_IOSCHED_KYBER=y/# CONFIG_MQ_IOSCHED_KYBER is not set/' ./.config
	fi

	if [ "${_kernelname}" != "" ]; then
		sed -i "s|CONFIG_LOCALVERSION=.*|CONFIG_LOCALVERSION=\"${_kernelname}\"|g" ./.config
		sed -i "s|CONFIG_LOCALVERSION_AUTO=.*|CONFIG_LOCALVERSION_AUTO=n|" ./.config
	fi
	
	### Optionally disable NUMA since >99% of users have mono-socket systems.
	# For more, see: https://bugs.archlinux.org/task/31187
	if [ -n "$_NUMAdisable" ]; then
		if [ "${CARCH}" = "x86_64" ]; then
			msg "Disabling NUMA from kernel config..."
			sed -i -e 's/CONFIG_NUMA=y/# CONFIG_NUMA is not set/' \
				-i -e '/CONFIG_AMD_NUMA=y/d' \
				-i -e '/CONFIG_X86_64_ACPI_NUMA=y/d' \
				-i -e '/CONFIG_NODES_SPAN_OTHER_NODES=y/d' \
				-i -e '/# CONFIG_NUMA_EMU is not set/d' \
				-i -e '/CONFIG_NODES_SHIFT=6/d' \
				-i -e '/CONFIG_NEED_MULTIPLE_NODES=y/d' \
				-i -e '/# CONFIG_MOVABLE_NODE is not set/d' \
				-i -e '/CONFIG_USE_PERCPU_NUMA_NODE_ID=y/d' \
				-i -e '/CONFIG_ACPI_NUMA=y/d' ./.config
		fi
	fi

	### Set extraversion to pkgrel
        sed -ri "s|^(EXTRAVERSION =).*|\1 -${pkgrel}|" Makefile

	### Don't run depmod on 'make install'. We'll do this ourselves in packaging
	sed -i '2iexit 0' scripts/depmod.sh

	### Get kernel version
	msg "Running make prepare for you to enable patched options of your choosing"
	make prepare

	### Optionally load needed modules for the make localmodconfig
	# See https://aur.archlinux.org/packages/modprobed-db
		if [ -n "$_localmodcfg" ]; then
		msg "If you have modprobe-db installed, running it in recall mode now"
		if [ -e /usr/bin/modprobed-db ]; then
			[[ -x /usr/bin/sudo ]] || {
                        echo "Cannot call modprobe with sudo. Install sudo and configure it to work with this user."
                        exit 1; }
			sudo /usr/bin/modprobed-db recall
		fi
		msg "Running Steven Rostedt's make localmodconfig now"
		make localmodconfig
	fi

	### Running make nconfig
	
	[[ -z "$_makenconfig" ]] ||  make nconfig
	
	### Running make menuconfig
	
	[[ -z "$_makemenuconfig" ]] || make menuconfig
	
	### Running make xconfig
	
	[[ -z "$_makexconfig" ]] || make xconfig
	
	### Running make gconfig
	
	[[ -z "$_makegconfig" ]] || make gconfig
	
	### Rewrite configuration
	yes "" | make config >/dev/null

	### Save configuration for later reuse
	cat .config > "${startdir}/config.${CARCH}.last"
}

build() {
  cd "${srcdir}/${_srcname}"

  make ${MAKEFLAGS} LOCALVERSION= bzImage modules
}

_package-kernel() {
    pkgdesc='Linux Kernel and modules with the  BFQ-MQ scheduler. Fourth Gen Intel Core i3/i5/i7 optimized.'
    depends=('coreutils' 'linux-firmware' 'mkinitcpio>=0.7')
    optdepends=('crda: to set the correct wireless channels of your country' 'modprobed-db: Keeps track of EVERY kernel module that has ever been probed - useful for those of us who make localmodconfig')
    groups=('linux-huv-bfq-mq-git')
    backup=("etc/mkinitcpio.d/${pkgbase}.preset")
    install=linux.install

    cd "${srcdir}/${_srcname}"

    KARCH=x86

   # get kernel version
   _kernver="$(make LOCALVERSION= kernelrelease)"
   _basekernel=${_kernver%%-*}
   _basekernel=${_basekernel%.*}

    mkdir -p "${pkgdir}"/{lib/modules,lib/firmware,boot}
    make LOCALVERSION= INSTALL_MOD_PATH="${pkgdir}" modules_install
    cp arch/$KARCH/boot/bzImage "${pkgdir}/boot/vmlinuz-${pkgbase}"

    # set correct depmod command for install
    sed -e "s|%PKGBASE%|${pkgbase}|g;s|%KERNVER%|${_kernver}|g" \
    "${startdir}/${install}" > "${startdir}/${install}.pkg"
    true && install=${install}.pkg

    # install mkinitcpio preset file for kernel
    sed "s|%PKGBASE%|${pkgbase}|g" "${srcdir}/linux.preset" |
    install -D -m644 /dev/stdin "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"

    # install pacman hook for initramfs regeneration
    sed "s|%PKGBASE%|${pkgbase}|g" "${srcdir}/90-linux.hook" |
    install -D -m644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/90-${pkgbase}.hook"

    # remove build and source links
    rm -f "${pkgdir}"/lib/modules/${_kernver}/{source,build}
    # remove the firmware
    rm -rf "${pkgdir}/lib/firmware"
    # make room for external modules
    ln -s "../extramodules-${_basekernel}${_kernelname:--ARCH}" "${pkgdir}/lib/modules/${_kernver}/extramodules"
    # add real version for building modules and running depmod from post_install/upgrade
    mkdir -p "${pkgdir}/lib/modules/extramodules-${_basekernel}${_kernelname:--ARCH}"
    echo "${_kernver}" > "${pkgdir}/lib/modules/extramodules-${_basekernel}${_kernelname:--ARCH}/version"

    # Now we call depmod...
    depmod -b "${pkgdir}" -F System.map "${_kernver}"

    # move module tree /lib -> /usr/lib
    mkdir -p "${pkgdir}/usr"
    mv "${pkgdir}/lib" "${pkgdir}/usr/"

    # add vmlinux
    install -D -m644 vmlinux "${pkgdir}/usr/lib/modules/${_kernver}/build/vmlinux"
}

_package-headers() {
pkgdesc='Header files and scripts to build modules for linux-huv-bfq-mq-git. Fourth Gen Intel Core i3/i5/i7 optimized.'
    depends=('linux-huv-bfq-mq-git-kernel')
    groups=('linux-huv-bfq-mq-git')

    install -dm755 "${pkgdir}/usr/lib/modules/${_kernver}"

    cd "${srcdir}/${_srcname}"

    
        install -D -m644 Makefile \
		"${pkgdir}/usr/lib/modules/${_kernver}/build/Makefile"
	install -D -m644 kernel/Makefile \
		"${pkgdir}/usr/lib/modules/${_kernver}/build/kernel/Makefile"
	install -D -m644 .config \
		"${pkgdir}/usr/lib/modules/${_kernver}/build/.config"

	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/include"

	for i in acpi asm-generic config crypto drm generated keys linux math-emu \
		media net pcmcia rdma scsi soc sound trace uapi video xen; do
	cp -a include/${i} "${pkgdir}/usr/lib/modules/${_kernver}/build/include/"
	done

	# copy arch includes for external modules
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/x86"
	cp -a arch/x86/include "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/x86/"

	# copy files necessary for later builds, like nvidia and vmware
	cp Module.symvers "${pkgdir}/usr/lib/modules/${_kernver}/build"
	cp -a scripts "${pkgdir}/usr/lib/modules/${_kernver}/build"

	# fix permissions on scripts dir
	chmod og-w -R "${pkgdir}/usr/lib/modules/${_kernver}/build/scripts"
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/.tmp_versions"

	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/kernel"

	cp arch/${KARCH}/Makefile "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/"

	if [ "${CARCH}" = "i686" ]; then
		cp arch/${KARCH}/Makefile_32.cpu "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/"
	fi

	cp arch/${KARCH}/kernel/asm-offsets.s "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/kernel/"

	# add docbook makefile
	install -D -m644 Documentation/DocBook/Makefile \
	"${pkgdir}/usr/lib/modules/${_kernver}/build/Documentation/DocBook/Makefile"

	# add dm headers
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/md"
	cp drivers/md/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/md"

	# add inotify.h
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/include/linux"
	cp include/linux/inotify.h "${pkgdir}/usr/lib/modules/${_kernver}/build/include/linux/"

	# add wireless headers
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/net/mac80211/"
	cp net/mac80211/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/net/mac80211/"

	# add dvb headers for external modules
	# in reference to:
	# http://bugs.archlinux.org/task/9912
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-core"
	cp drivers/media/dvb-core/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-core/"
	# and...
	# http://bugs.archlinux.org/task/11194
	###
	### DO NOT MERGE OUT THIS IF STATEMENT
	### IT AFFECTS USERS WHO STRIP OUT THE DVB STUFF SO THE OFFICIAL ARCH CODE HAS A CP
	### LINE THAT CAUSES MAKEPKG TO END IN AN ERROR
	###
	if [ -d include/config/dvb/ ]; then
		mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/include/config/dvb/"
		cp include/config/dvb/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/include/config/dvb/"
	fi

	# add dvb headers for http://mcentral.de/hg/~mrec/em28xx-new
	# in reference to:
	# http://bugs.archlinux.org/task/13146
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends/"
	cp drivers/media/dvb-frontends/lgdt330x.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends/"
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/i2c/"
	cp drivers/media/i2c/msp3400-driver.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/i2c/"

	# add dvb headers
	# in reference to:
	# http://bugs.archlinux.org/task/20402
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/usb/dvb-usb"
	cp drivers/media/usb/dvb-usb/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/usb/dvb-usb/"
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends"
	cp drivers/media/dvb-frontends/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends/"
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/tuners"
	cp drivers/media/tuners/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/tuners/"

	# add xfs and shmem for aufs building
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/fs/xfs"
	mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/mm"
	# removed in 3.17 series
	#cp fs/xfs/xfs_sb.h "${pkgdir}/usr/lib/modules/${_kernver}/build/fs/xfs/xfs_sb.h"

	# copy in Kconfig files
	for i in $(find . -name "Kconfig*"); do
		mkdir -p "${pkgdir}"/usr/lib/modules/${_kernver}/build/`echo ${i} | sed 's|/Kconfig.*||'`
		cp ${i} "${pkgdir}/usr/lib/modules/${_kernver}/build/${i}"
	done
	
	# add objtool for external module building and enabled VALIDATION_STACK option
        if [ -f tools/objtool/objtool ];  then
        mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/tools/objtool"
        cp -a tools/objtool/objtool ${pkgdir}/usr/lib/modules/${_kernver}/build/tools/objtool/ 
        fi

	chown -R root.root "${pkgdir}/usr/lib/modules/${_kernver}/build"
	find "${pkgdir}/usr/lib/modules/${_kernver}/build" -type d -exec chmod 755 {} \;

	# strip scripts directory
	find "${pkgdir}/usr/lib/modules/${_kernver}/build/scripts" -type f -perm -u+w 2>/dev/null | while read binary ; do
	case "$(file -bi "${binary}")" in
	*application/x-sharedlib*) # Libraries (.so)
		/usr/bin/strip ${STRIP_SHARED} "${binary}";;
	*application/x-archive*) # Libraries (.a)
		/usr/bin/strip ${STRIP_STATIC} "${binary}";;
	*application/x-executable*) # Binaries
		/usr/bin/strip ${STRIP_BINARIES} "${binary}";;
	esac
	done
	
	# remove a files already in linux-bfq-haswell-git-docs package
        rm -f "${pkgdir}/usr/lib/modules/${_kernver}/build/Documentation/kbuild/Kconfig.recursion-issue-01"
        rm -f "${pkgdir}/usr/lib/modules/${_kernver}/build/Documentation/kbuild/Kconfig.recursion-issue-02"
        rm -f "${pkgdir}/usr/lib/modules/${_kernver}/build/Documentation/kbuild/Kconfig.select-break"

    # remove unneeded architectures
    rm -rf "${pkgdir}"/usr/lib/modules/${_kernver}/build/arch/{alpha,arm,arm26,arm64,avr32,blackfin,c6x,cris,frv,h8300,hexagon,ia64,m32r,m68k,m68knommu,mips,microblaze,mn10300,openrisc,parisc,powerpc,ppc,s390,score,sh,sh64,sparc,sparc64,tile,unicore32,um,v850,xtensa}
}

_package-docs() {
    pkgdesc='Kernel hackers manual - HTML documentation that comes with the linux-huv-bfq-mq-git kernel. Fourth Gen Intel Core i3/i5/i7 optimized.'
    depends=('linux-huv-bfq-mq-git-kernel')
    groups=('linux-huv-bfq-mq-git')
  
    cd "${srcdir}/${_srcname}"

    mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build"
    cp -al Documentation "${pkgdir}/usr/lib/modules/${_kernver}/build"
    find "${pkgdir}" -type f -exec chmod 444 {} \;
    find "${pkgdir}" -type d -exec chmod 755 {} \;

    # remove a file already in linux-huv-bfq-mq-git package
    rm -f "${pkgdir}/usr/lib/modules/${_kernver}/build/Documentation/DocBook/Makefile"
}

pkgname=("${pkgbase}-kernel" "${pkgbase}-headers" "${pkgbase}-docs")
for _p in ${pkgname[@]}; do
  eval "package_${_p}() {
    $(declare -f "_package${_p#${pkgbase}}")
    _package${_p#${pkgbase}}
  }"
done

sha512sums=('SKIP'
            '77d80d50d8c4323ed36fd2097ba9f6b49bb8d7cae59d32ffa76b309758a7e9f972d26fedd77046d88ce2691bb01a07909f8bdc34ba214414be3bc030ee31994d'
            '686cbb3ab781212a5f7e0800bd4fdf5916ac6c1b0bc432bcae96bdd6ff726a1738705c740e8f643ade88c82ded39b56a185af468f824e345d25ab5e403d83121'
            'd6faa67f3ef40052152254ae43fee031365d0b1524aa0718b659eb75afc21a3f79ea8d62d66ea311a800109bed545bc8f79e8752319cd378eef2cbd3a09aba22'
            '2dc6b0ba8f7dbf19d2446c5c5f1823587de89f4e28e9595937dd51a87755099656f2acec50e3e2546ea633ad1bfd1c722e0c2b91eef1d609103d8abdc0a7cbaf'
            '38792eed552a4c81679279ee468a60800f17533e3bfcef2362aba0deb787ffd9f2a7106249edc420ba795eb9abd8ec4c2496fdde11549ccf4f36ab1339a81321'
            '25741e37a225cebb6cd4f82a9eb63a86836d060c7f855b8169cf4d9753fe9d7c1c0eb8fea56b20e484f81a93c071e4ac7626586c1b306b8cf9aa35f5840d44c2'
            '8368b6847d6f32b6e0c88c097823fbdbe89e63e65a915a3198ae1a4330fe0c4a4cbfeac92b3bbd08d3828ccc8ac961e5122be56f9a41ee855f5c1615f8ca5694'
            '0f96fa9ad784709973b32eea82075ceb3e9dc2482df6441a4607612806f069254e63508b1b562279622394e4a1fbebef1b87af8401c0b1210d5d0de9954245c8'
            '4de18441481703b40b0ef3647e0b29ac579a108833d1c18a2a4a4ac43864fadffd2dcbc820ba6104013bc15e5b8164adcca1b98ffe2e4fe4bdb8dd03a37d60dc'
            '1a2de50b0d82de6fd4c6763767c1bd14be10c2d268e24f8ef7662439729e899d49620ad4f2ede3b59ffbfd7ff819766d75f0b3a4f0f5c36460be7284d901072f'
            '94d8bf17dfc03e1b81f215a61eeee2eb0b36c9471a337d11a2ab0657beea4c96a30a6a7fc0450a3b8aef588661857a4927810294a804d2fe3cd66d6f4359beca')
