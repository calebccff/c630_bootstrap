# C630 Arch Linux setup

## Build image

### Setup rootfs

```bash
mkdir rootfs
fallocate -l 4G rootfs.img
mkfs.ext4 rootfs.img
mount rootfs.img rootfs
bsdtar -xpf ArchLinuxARM-aarch64-latest.tar.gz -C /rootfs
# chroot to the container, you can also use arch-chroot from arch-install-scripts
sudo systemd-nspawn -D /rootfs
```

### Setup pacman

If using systemd-resolved on your host... (and /etc/resolv.conf is a symlink)
```bash
rm /etc/resolv.conf
echo "nameserver 1.1.1.1" > /etc/resolv.conf
```


```bash
passwd

pacman-key --init
pacman-key --populate archlinuxarm
pacman-key --refresh-keys
pacman-key --lsign-key 77193F152BDBE6A6    # Arch Linux ARM Build System <builder@archlinuxarm.org>
pacman -Rdd linux-aarch64
pacman -Syyu neofetch htop vim terminus-font sudo grub linux-firmware-qcom arch-install-scripts efibootmgr rmtfs wget iwd
# We need these for wifi later... (Expect these links to break)
wget https://gitlab.com/kupfer/packages/prebuilts/-/raw/main/aarch64/main/tqftpserv-git-r12.783425b-2-aarch64.pkg.tar.xz
wget https://gitlab.com/kupfer/packages/prebuilts/-/raw/main/aarch64/main/pd-mapper-git-r13.d7fe25f-2-aarch64.pkg.tar.xz
pacman -U *.xz
rm *.xz

sed -e '0,/^#en_US/s//en_US/' -i /etc/locale.gen
locale-gen
echo -e 'LANG=en_US.UTF-8\nLC_COLLATE=C' > /etc/locale.conf

userdel alarm
useradd -g users -G wheel,storage,disk,rfkill,network,input,log -m YOURUSER # Pick your own
passwd YOURUSER
echo 'HOSTNAME' > /etc/hostname # Pick your own
# Enable sudo
sed -i 's/%wheel ALL=(ALL) ALL/%wheel ALL=(ALL) ALL/g' /etc/sudoers
```

### Add firmware

On your host:

```bash
git clone https://gitlab.com/sdm845-mainline/firmware-lenovo-yogac630
sudo rsync -rv firmware-lenovo-yogac630/lib/firmware/qcom rootfs/usr/lib/firmware/
sudo rsync -rv firmware-lenovo-yogac630/lib/firmware/postmarketos/* rootfs/usr/lib/firmware/
```


### Set up grub

Back in the container.. Set the following in `/etc/default/grub`:

```bash
# Adjust/remove earlycon and serial console. This will put kernel logs on tty2
GRUB_CMDLINE_LINUX_DEFAULT="ignore_loglevel earlycon=qcom_geni clk_ignore_unused console=ttyMSM0,115200 console=tty2"
# For grub over Serial
GRUB_TERMINAL_INPUT="console serial"
GRUB_TERMINAL_OUTPUT="gfxterm serial"
GRUB_SERIAL_COMMAND="serial --speed=115200 --unit=0 --word=8 --parity=no --stop=1"
# Devicetree requires some changes to grub-mkconfig
GRUB_DEVICETREE="sdm850-lenovo-yoga-c630"
```

Edit `/usr/bin/grub-mkconfig` and add `GRUB_DEVICETREE` to the list (find `export GRUB_DEFAULT`):

```bash
...
  GRUB_DEVICETREE \
...
```


For auto-generated grub config, the following snippet needs to be added to `/etc/grub.d/10_linux`:

```bash
# Add the following before the line "if test -n "${initrd}" ; then"
  if [ -n "${GRUB_DEVICETREE}" ]; then
    message="$(gettext_printf "Loading devicetree ...")"
    sed "s/^/$submenu_indentation/" << EOF
        echo    '$(echo "$message" | grub_quote)'
        devicetree ${rel_dirname}/dtbs/${GRUB_DEVICETREE}.dtb
EOF
  fi
```

### Set up wifi

In container:

```bash
systemctl enable rmtfs pd-mapper tqftpserv iwd

echo << EOF > /etc/systemd/network/20-wifi.network
[Match]
Name=wlan0

[Network]
DHCP=yes
EOF

```

### Install kernel

It's easiest to do this manually the first time to bootstrap, then just copy
artifacts from the host and package them on device, or self-host or whatever.

In your kernel source directory:

```bash
# Copy in kernel and DTB
sudo install .output/arch/arm64/boot/Image.gz /path/to/rootfs/boot/vmlinuz-c630
sudo install -D .output/arch/arm64/boot/dts/qcom/sdm850-lenovo-yoga-c630.dtb /path/to/rootfs/boot/dtbs/
# Install kernel modules (to /usr because /lib is a symlink to /usr/lib)
sudo make O=.output ARCH=arm64 INSTALL_MOD_PATH=/hdd/cartel/mainline/c630/arch/rootfs/usr modules_install
# Record the kernel release (also make a note of it)
make O=.output ARCH=arm64 kernelrelease | sudo tee /path/to/rootfs/boot/kernel-release
```

Create an mkinitcpio preset for the kernel: `/etc/mkinitcpio.d/linux-c630.preset`:

```bash
ALL_config="/etc/mkinitcpio.conf"
ALL_kver="KERNELRELEASE_FROM_EARLIER" # Set your kernel release here (e.g. 6.1.0-rc6-sdm845+)

PRESETS=('default')

default_image="/boot/initramfs-c630.img"
```

Create an install hook to add firmware and kernel modules: `/usr/lib/initcpio/install/c630`:

```bash
#!/bin/bash

build() {
        # Probably a few more modules are needed depending
        # on the kernel config you use...
        add_module ufs_qcom
        add_module ti_sn65dsi86

        add_file /usr/lib/firmware/qcom/LENOVO/81JL/qcdxkmsuc850.mbn
        add_file /usr/lib/firmware/qcom/a630_gmu.bin
        add_file /usr/lib/firmware/qcom/a630_sqe.fw
        add_file /usr/lib/firmware/ipa_fws.mdt
}

help() {
    cat <<HELPEOF
Lenovo Yoga C630 firmware and modules required in ramdisk
HELPEOF
}
```

Add the install hook to `/etc/mkinitcpio.conf`: `HOOKS=(... c630 ...)`.

Build a new initramfs and grub config:

```bash
mkinitcpio -p linux-c630
grub-mkconfig -o /boot/grub/grub.cfg
```

Format your temporary bootable USB with a labelled fat32 boot partition and ext4
root partition. Make sure you set the ESP flag on the boot partition.

Mount them and sync over the rootfs:

```bash
mount /dev/sdX2 mnt
mount /dev/sdX1 mnt/boot
sudo rsync -raxWHAX --info=progress2 rootfs mnt
```

### Set up the bootloader

Unfortunately this seems to _require_ arch-chroot otherwise grub-install fails
```
sudo arch-chroot mnt
grub-install --efi-directory=/boot --removable --no-nvram
```

Finally copy the rootfs image to the USB and unmount it. We want a copy of the
image so we can flash it to the internal storage more easily.

```bash
sudo umount rootfs
sudo cp rootfs.img mnt/srv/
sudo umount -R mnt
sync
# If this takes a while you can watch the progress with
watch -n1 "cat /proc/meminfo | grep Dirty"
```

## Install Arch Linux

Set up internal storage and copy over the rootfs

```bash
# Manually mount the bootable USB boot partition
mkfs.ext4 -L root /dev/sda5 # Replace with your rootfs partition...
mount /dev/sda5 /mnt
mount /dev/sda1 /mnt/boot
mkdir /rootmnt
mount /srv/rootfs.img /rootmnt
# Copy over the rootfs image
rsync -raxWHAX --info=progress2 /rootmnt /mnt
```

We need to redo the final steps to set up the bootloader and mountpoints

```bash
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt
grub-mkconfig -o /boot/grub/grub.cfg
# Drop the --removable to have grub run efibootmgr to add a boot entry for itself instead
grub-install --efi-directory=/boot --removable --no-nvram
```

This should be it! Install base-devel, git and set up yay. Install your DE of choice... off to the races

## Upgrading the kernel

Build your new kernel, stick modules in /usr/lib/modules/ and image in
/boot/vmlinuz-c630. Modify /etc/mkinitcpio.d/linux-c630.preset to point to the
new kernel release. Do:

```bash
mkinitcpio -p linux-c630
grub-mkconfig -o /boot/grub/grub.cfg
```
