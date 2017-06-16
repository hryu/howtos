# compile qemu

    git clone git://git.qemu.org/qemu.git
    cd qemu
    ./configure
    make

# create ubuntu core rootfs images

## download ubuntu image

    wget http://cdimage.ubuntu.com/ubuntu-base/releases/16.04/release/ubuntu-base-16.04.1-base-${ARCH}.tar.gz

> ARCH : amd64, arm64, armhf, ...

## create qemu image

    qemu-img create -f qcow2 ubuntu-base-16.04.1-${ARCH}.qcow2 4G

## mount using nbd

    sudo modprobe -av nbd
    sudo qemu-nbd -c /dev/ndb0 ubuntu-base-16.04.1-${ARCH}.qcow2

## create partition

    sudo fdisk /dev/ndb0

## format the disk

    sudo mkfs.ext4 /dev/nbd0p1

## mount

    sudo mount -t ext4 /dev/nbd0p1 tmp

## copy ubuntu contents of tarball to the mount point

    sudo tar xf ubuntu-base-16.04.1-base-${ARCH}.tar.gz -C tmp

## copy qemu binary into the mount point

    sudo cp /usr/bin/qemu-${ARCH}-static tmp/usr/bin/

## chroot

    sudo chroot tmp /bin/bash

## set password for root

    passwd root

## additional works for rootfs

### x86_64 console

    ln -s /lib/systemd/system/getty@.service /etc/systemd/system/getty.target.wants/getty@ttyAMA0.service

### arm64 console

    ln -s /lib/systemd/system/getty@.service /etc/systemd/system/getty.target.wants/getty@ttyAMA0.service

## exit from chroot

    exit

## unmount

    sudo umount tmp

## remove nbd

    sudo qemu-nbd -d /dev/nbd0

# linux kernel

## clone kernel source

    git clone https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git

## fetch -next branch

    cd linux
    git remote add linux-next https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git
    git fetch linux-next
    git fetch --tags linux-next
    git remote update
    git tag -l "next-*" | tail
    git checkout -b my_local_branch next-20170203

## compile linux kernel

### x86_64

    ARCH=x86 make x86_64_defconfig O=x86_64-objs
    ARCH=x86 INSTALL_PATH=installed make -j4 all O=x86_64-objs
    ARCH=x86 INSTALL_PATH=installed make -j4 install O=x86_64-objs
    ARCH=x86 INSTALL_MOD_PATH=installed make modules_install O=x86_64-objs
    ARCH=x86 INSTALL_MOD_PATH=installed make firmware_install O=x86_64-objs
    ARCH=x86 INSTALL_HDR_PATH=installed/usr make headers_install O=x86_64-objs/installed

### arm64

    ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make defconfig O=arm64-objs
    ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_PATH=installed make all -j4 O=arm64-objs
    mkdir arm64-objs/installed
    ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_PATH=installed make -j4 install O=arm64-objs
    ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=installed make modules_install O=arm64-objs
    ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=installed make firmware_install O=arm64-objs
    ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_PATH=installed make dtbs_install O=arm64-objs
    ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_HDR_PATH=installed/usr make headers_install O=arm64-objs/

# booting linux kernel + ubuntu rootfs using qemu

## arm64 version

    ~/qemu/aarch64-softmmu/qemu-system-aarch64 -s -cpu cortex-a57 -machine type=virt -nographic -smp 2 -m 8192 -kernel ~/linux/arm64-objs/arch/arm64/boot/Image -drive file=~/ubuntu/xenial-base-arm64.img.qcow2 --append "root=/dev/vda1 rw console=ttyAMA0 --"

## x86_64 version

    ~/qemu/x86_64-softmmu/qemu-system-x86_64 -s -nographic -smp 2 -m 8192 -kernel ~/linux/x86_64-objs/arch/x86_64/boot/bzImage -drive file=~/ubuntu/xenial-base-amd64.img.qcow2 -append "root=/dev/sda1 console=ttyS0"


# attach gdb

## x86_64

    gdb-multiarch vmlinux
    set architecture  i386:x86-64
    target remote :1234

## arm64

    gdb-multiarch vmlinux
    set architecture  aarch64
    target remote :1234
