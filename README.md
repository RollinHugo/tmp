#!/bin/bash
set -e
loadkeys fr
timedatectl
echo "n
p
1

+512M
n
p
2


t
1
ef
w
" | fdisk /dev/sda
mkfs.fat -F 32 /dev/sda1
mkfs.ext4 /dev/sda2
mount /dev/sda2 /mnt
mount --mkdir /dev/sda1 /mnt/boot
pacstrap -K /mnt base linux base-devel vim htop git man-db man-pages texinfo networkmanager nano
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt /bin/bash <<"EOT"
set -e
ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime
hwclock --systohc
printf "\nen_US.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
echo "KEYMAP=fr" > /etc/vconsole.conf
echo "thisvm" > /etc/hostname
systemctl enable NetworkManager
pacman -Syu grub efibootmgr --noconfirm
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
echo '%wheel ALL=(ALL:ALL) ALL' | sudo EDITOR='tee -a' visudo
useradd -m -G wheel myuser
EOT
arch-chroot /mnt
