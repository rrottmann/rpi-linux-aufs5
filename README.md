# Install Linux Kernel 5.4.51-v7+ with armv71 aufs5 support on Raspios Buster Lite

While overlayfs is in mainline kernel, aufs5 has more features and this repo
contains the cross-compiled kernel and aufs-utils that are too cumbersome to
build on the low specs of a Raspberry Pi.

* Tested with Raspberry Pi 2.
* Should work with Raspberry Pi 3.

## Licenses
See the original licenses of the Linux Kernel and aufs5.
Their git repositories at compile time are included in
`rpi-linux-aufs5.tar.bz2`.

## Install

~~~bash
# From 2020-05-27-raspios-buster-lite-armhf.zip
apt-get update
apt-get install -y git 
t=`mktemp -d`
cd $t
# make sure to have ample space (>5G) here!
git clone https://github.com/rrottmann/rpi-linux-aufs5.git
cat rpi-linux-aufs5.tar.bz2.split* > rpi-linux-aufs5.tar.bz2
rm -f rpi-linux-aufs5.tar.bz2.split*
tar -xjf rpi-linux-aufs5.tar.bz2
cd linux
make modules_install
make headers install
export KERNEL=kernel7
cp /boot/$KERNEL.img /boot/$KERNEL-backup.img
cp arch/arm/boot/zImage /boot/$KERNEL.img
cp arch/arm/boot/dts/*.dtb /boot
cp arch/arm/boot/dts/overlays/*.dtb* /boot/overlays/
cp arch/arm/boot/dts/overlays/README /boot/overlays/
cd ..
cd aufs-util
make install
update-initramfs -v -c -u -k 5.4.51-v7+
/bin/sed -i '/^initramfs /d' /boot/config.txt
echo "initramfs initrd.img-5.4.51-v7+" >> /boot/config.txt
sync
reboot
~~~

## Test aufs after reboot

~~~bash
uname -a
# Linux raspberrypi 5.4.51-v7+ #1 SMP Mon Jul 27 11:45:59 UTC 2020 armv7l GNU/Linux
mkdir /tmp/dir1
mkdir /tmp/dir2
mkdir /tmp/aufs-root
mount -t aufs -o br=/tmp/dir1:/tmp/dir2 none /tmp/aufs-root/
touch /tmp/dir1/dir1.file
touch /tmp/dir2/dir2.file
ls -al /tmp/aufs-root/
~~~

