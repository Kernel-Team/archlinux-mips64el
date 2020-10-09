archlinux mips64el bootstrap
========================

1. Introduction
---------------

This project is an attempt to bootstrap a self-contained parabola
Archlinux system for the following architectures:

 - mips64le (stage4 complete)
 
The scripts are created with the goal to be as architecture agnostic as
possible, to make future porting efforts easier.

The build process is split into four stages, the rationale of which is outlined
in section 2 below.



1.1 System Requirements
------------------------

The scripts require, among probably other things, to be running on a fairly
POSIX-conforming GNU/Linux system, and in particular need the following tools
to be present and functional:

 * decently up-to-date GNU build toolchain (gcc, glibc, binutils) most of the
 * things in base-devel pacman, makepkg

I have tried to make the script smart enough to check for required bits and
pieces where needed, and to report when anything is missing ahead of time, but
some requirements may be missing.

1.2 A note to the reader
-------------------------

The scripts assume to be running on a parabola mips64el system, and may
fail in subtle and unexpected ways otherwise. They also may fail in subtle and
unexpected ways anyway, because they are modifying upstream PKGBUILD files,
which are very volatile and change without notice. When adapting this project
to a new architecture, or a different flavour of archlinux, execrcise caution,
pay close attention to any output, and be prepared to fix and modify patches
and scripts.

Also, if you found this project useful, and want to chat about anything, you
can email me at <shipujin.t@gmail.com>

1.3 Current state of the project
---------------------------------

All four stages of the riscv64 bootstrap are complete, and efforts to add
additional architectures has begun. A pointer where to find future development
efforts for the parabola ports will be added here in due time.

2. Build Stages
---------------

The following subsections outline the reasoning behind the separate bootstrap
stages. More details about *how* things are done may be gathered from reading
the inline comments of the respective scripts.

stage0: lfs-tools-rootfs
stage1: compile stage1.list
stage2: compile stage2.list
stage3: compile stage3.list
stage4: compile stage4.list


2.1 Stage 1
------------

The first stage creates and installs a cross-compile toolchain for the target
triplet defined in $CHOST, consisting of binutils, linux-libre-api-headers, gcc
and glibc. The scripts will check for $CHOST-ar and $CHOST-gcc in $PATH to
determine whether binutils and gcc are installed, and will then proceed to look
for the following files in $CHOST-gcc's sysroot:

  $sysroot/lib/libc.so.6              # for $CHOST-glibc
  $sysroot/include/linux/kernel.h     # for $CHOST-linux-api-headers

If your system contains these files, the toolchain bootstrap process will be
skipped. Otherwise, the scripts will create a new toolchain as follows:

 - compile $CHOST-binutils
 - compile $CHOST-linux-api-headers
 - compile $CHOST-gcc-bootstrap, without glibc dependency
 - cross-compile $CHOST-glibc using $CHOST-gcc-bootstrap
 - compile $CHOST-gcc against the created $CHOST-glibc

The sysroot of the created toolchain is set as /usr/$CHOST.

The $CHOST-{binutils,linux-api-headers,gcc,glibc} packages are installed
as regular pacman packages on the build system, and are not automatically
removed by the scripts after the build is completed (or has failed).



2.2 Stage 2
------------

Stage 2 uses the toolchain created in Stage 1 to cross-compile a subset of the
packages of the base-devel group plus transitive runtime dependencies.

The script creates an empty skeleton chroot, into which the cross-compiled
packages are going to be installed, and creates a sane makepkg.conf and a
patched makepkg.sh to work in the prepared chroot root directory. To make the
sysroot of the compiler available to builds in the chroot and vice-versa, the
/usr directory of the chroot is mounted into the sysroot. At the end of Stage
2, or in case of an error, this directory is unmounted automatically.

To build the packages, the stageN.list is traversed and packages are cross-compiled
using upstream PKGBUILDs and custom patches, and the compiled packages are
installed into the chroot immediately. For this stage, the patches are
mandatory for each built package.

Note that this process is a bit fragile and dependent on arbitrary
particularities of the host system, and thus might fail for subtle reasons,
like missing, or superfluous build-time installed packages on the host.
Exercise caution and common sense.


2.3 Stage 3
------------

Stage 3 uses the cross-compiled makepkg chroot created in Stage 2 to natively
recompile the base-devel group of packages. This stage requires to build more
packages, since a reduced set of make-time dependencies need to be present in
the makepkg chroot, as well as runtime dependencies. Additionally, running the
cross-compiled native compiler instead of the cross compiler takes longer,
since user-mode emulation needs to be applied. As a result, stage 3 is expected
to takes much longer than stage 2.

However, since now the process is isolated from the host systems installed
packages, since everything is built cleanly in a chroot, the process is much
more stable and less prone to hard to diagnose problems with the host system.

The scripts create a clean librechroot from the cross-compiled packages
produced in stage 2. A modified libremakepkg script is created to perform
config fragment regeneration and to skip the check() phase, and then packages
are built in order of the stageN.list.

Note that the cross-compiled packages from stage 3 can be a bit derpy at times,
hence the stage 3 build scripts prioritize building bash and make natively, to
avoid some known issues down the line.

2.4 Stage 4
------------

Stage 4 does a final recompile of the packages of the base-devel group, similar
to stage 3, with the difference that more make-time dependencies are enabled,
and the packages of the base group are added to the DEPTREE.

Stage 4 relies entirely on the packages natively compiled in stage 3, and no
cross-compiled packages are present in the build chroot at any time. This
results in reliable builds and reproducible build failures. However, since the
number of packages to be built in stage 4 is again much larger, expect the
builds to take a long time (days / weeks, not including work required to fix
broken patches and builds).

The result of stage 4 is a repository of packages that should allow to
bootstrap and boot a virtual machine of the bootstrapped architecture, with the
packages required to build the entirety of the arch / parabola package
repository. At this point, the port becomes self-hosting and I consider the
bootstrap process done. Note that a bootloader is missing from the bootstrap
process, but this can easily be built manually after stage4 using the
bootstrapped makepkg chroot.

3. Acknowledgements
===================


I would like to thank the awesome abaumann from the archlinux32 project for
pointers on how to bootstrap a PKGBUILD based system for a new architecture,
and for his work on the bootstrap32 project, which helped a lot in getting this
project started:

  https://github.com/archlinux32/bootstrap32
  https://github.com/archlinux-riscv/archlinux-cross-bootstrap



