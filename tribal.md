# Kernel

1. [Bumping the kernel versions](#bumping-the-kernel-version)
1. [Compiling the mainline kernel](#compiling-the-mainline-kernel)
1. [Bisect the kernel](#bisect-the-kernel)

## Bumping the kernel version

Sometimes we need to experiment with later versions of the kernel than are available on the stemcell.
Here's how to bump:

1. Download the kernel version that you're after, using 4.13 as an example:

```
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.13/linux-headers-4.13.0-041300_4.13.0-041300.201709031731_all.deb

wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.13/linux-headers-4.13.0-041300-generic_4.13.0-041300.201709031731_amd64.deb

wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.13/linux-image-4.13.0-041300-generic_4.13.0-041300.201709031731_amd64.deb
```

2. Install the .debs

```
sudo dpkg -i *.deb
```

3. Edit grub conf files to update version:

```
sed -i 's/3.19.0-37-generic/4.13.0-041300-generic/g' /boot/grub/grub.conf
sed -i 's/3.19.0-37-generic/4.13.0-041300-generic/g' /boot/grub/menu.lst
```

4. Reboot

If you are bumping the kernel in a BOSH deployed VM, it is useful to
`monit stop all` before you reboot; monit finds it challenging to bring jobs
back up if you reboot without proper ceremony.

```
reboot
```

## Compiling the mainline Kernel

### About

These are instructions for compiling the mainline kernel. (For compiling the Ubuntu kernel the steps can be derived from [Bisect the Kernel](#bisect-the-kernel))

### Steps

```
cd ~/workspace/concourse-lite
vagrant init concourse/lite
vagrant up
vagrant ssh
sudo su -
mkdir -p workspace
cd workspace/

apt-get update
apt-get install git bc ncurses-dev libssl-dev

git clone https://github.com/torvalds/linux.git
cd linux
```

If you are repeating the steps from here onwards, firstly clean the git repo and
compilation artifacts:

```
git stash -u && git stash drop
make mrproper
```

Then let's get to compiling:

```
git checkout remotes/origin/aufs4.1 (or whatever version you want)

make localmodconfig # answer with default values
make -j 2
make -j 2 bzImage
make -j 2 modules_install install
INSTALL_HDR_PATH=/usr/include/linux/ make headers_install
shutdown -h now
```

The final steps to boot into the new kernel are a bit weird due to a VirtualBox
bug and the repeated steps are intentional.

```
Boot into the maxhine through VirtualBox and select the kernel version you just compiled from the GRUB bootloader
F12 -> B
power off
start
F12 -> B -> Advanced Ubuntu Options
```

## Bisect the kernel

This guide assumes you have observed some sort of regression on a BOSH deployed vm and that you suspect something in the kernel is at the root of it. Therefore these steps are intended to be run in a BOSH deployed environment which **you do not care about**.

First ensure that your BOSH vm is deployed with a (big) persistent disk:
```yaml
- name: diego-cell
  persistent_disk: 548576
```
This will end up mounted at `/var/vcap/store` which will be your workdir. This will give you more space and mean that, if you thoroughly F this up, you can recreate without losing anything :).

**Let's get started:**

If you have yet to narrow down to a particular version, you don't need to start compiling just now as you can still work with pre-compiled releases from the [ubuntu mainline](http://kernel.ubuntu.com/~kernel-ppa/mainline).

```sh
mkdir -p /var/vcap/store/kern/pre-comp && cd /var/vcap/store/kern/pre-comp

# go to http://kernel.ubuntu.com/~kernel-ppa/mainline and choose a major-minor to start from
# for example, if you know v4.4 was fine but saw a problem with v4.15, download the debs from v4.10 (sort of in between)

mkdir v4.10 && cd v4.10
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.10/linux-headers-4.10.0-041000_4.10.0-041000.201702191831_all.deb
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.10/linux-headers-4.10.0-041000-generic_4.10.0-041000.201702191831_amd64.deb
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.10/linux-image-4.10.0-041000-generic_4.10.0-041000.201702191831_amd64.deb

# unpack those
sudo dpkg -i *.deb

# edit grub files
sed -i 's/4.4.0-137-generic/4.10.0-041000-generic/g' /boot/grub/grub.conf
sed -i 's/4.4.0-137-generic/4.10.0-041000-generic/g' /boot/grub/menu.lst

# stop and reboot. monit gets very weird if you don't shut everything down cleanly first
monit stop all
reboot
```
After a minute or more (sometimes up to 15) you can ssh back on and `monit start all`. `uname -r` will show you your current kernel.

From here you can keeping using major-minor and then major-minor-patch precompiled releases until you can't get anything finer grained.

Once you have found the official version which contains the problem, you can see a list of the commits which went into it on the [ubuntu mainline](http://kernel.ubuntu.com/~kernel-ppa/mainline) version page, eg: [v4.14.60 changes](http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.10/CHANGES). You can also find further details of the commits at http://cdn.kernel.org/pub/linux/kernel/.

**Now it's proper bisection time:**

```sh
# set yourself up
mkdir -p /var/vcap/store/kern/source && cd /var/vcap/store/kern/source

# takes forever
git clone git://git.launchpad.net/~ubuntu-kernel-test/ubuntu/+source/linux/+git/mainline-crack

cd mainline-crack
git log --oneline v<last-known-good>..v<current-bad-version> # eg v4.14.59..v4.14.60
# choose some sensible sha in the middle, or one that just looks suspicious if you know what you are after
git checkout <sha>

# wget any patches from http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.14.60/, and then...
patch -p1 < 0001-base-packaging.patch
etc

# copy your existing kernel config so that you don't accidentally miss something
cp /boot/config-$(uname -r) .config
make oldconfig

# start building some custom debs, DON'T forget to name the custom kernel after the sha you are building from, it will be helpful
make clean
make -j `getconf _NPROCESSORS_ONLN` deb-pkg LOCALVERSION=-custom-<sha>

# bind mount a modules dir over the real one so that you don't flood `/`
mkdir /var/vcap/store/kern/modules/v4.14.59-custom-<sha>
mkdir /lib/modules/v4.14.59-custom-<sha>
mount --bind /var/vcap/store/kern/modules/v4.14.59-custom-<sha> /lib/modules/v4.14.59-custom-<sha>

# make modules
make modules_install -j `getconf _NPROCESSORS_ONLN`

# move your custom debs to a sensible location
mkdir /var/vcap/store/kern/custom-comp/v4.14.59-custom-<sha>
mv *.deb ../../custom-comp/v4.14.59-custom-<sha>
```

To use your newly compiled kernel just perform the same steps as above (unpack, edit, stop, reboot).

Good luck

# Miscellaneous
1. [Compiling nsenter](#nsenter)

## nsenter

```bash
sudo apt-get install build-essential libncurses5-dev libslang2-dev gettext zlib1g-dev \
libselinux1-dev debhelper lsb-release pkg-config po-debconf autoconf \
automake autopoint libtool python2.7-dev

# 2.32 was the latest version when I wrote this, you should quickly check if there is something newer
# https://mirrors.edge.kernel.org/pub/linux/utils/util-linux/
cd /tmp
wget https://mirrors.edge.kernel.org/pub/linux/utils/util-linux/v2.32/util-linux-2.32.tar.gz
tar -xvf util-linux-2.32.tar.gz

cd util-linux-2.32
./configure
make nsenter # or just `make` if you want the lot
sudo cp nsenter /usr/local/bin
nsenter --version
```
