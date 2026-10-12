# arch_btrfs_install

>1. connect ethernet or Wi-Fi using iwctl

>2. sync packages and refresh package database
pacman -Syy

>3. install reflector
pacman -S reflector

>4. enable network time protocol
timedatectl set-ntp true

>5. list block devices and determine drive name for install
lsblk

>6. use fdisk to create partitions "vda" for this example yours may be different
fdisk /dev/vda

>7. create new partition table and clear current
g

>8. create new partition
n

>9. add 200M to last sector
+200M

>10. create filesystem type
t

>11. set type to EFI
1

>12. create new partition
n

>13. add 2G to last sector (adjust as needed if you plan to use hibernate, this will be swap)
+2G

>14. create file system type
t

>15. set type to swap
19

>16. create new partition (accept defaults for remaining disk)
n

>17. write changes
w

>18. list block devices to verify changes
lsblk

>19. make file system for vda1
mkfs.fat -F32 /dev/vda1

>20. make file system for vda2
mkswap /dev/vda2 
Seph — 8/17/25, 11:07 AM
>21. enable swap
swapon /dev/vda2

>22. make file system for vda3
mkfs.btrfs /dev/vda3

>23. list block devices and verify
lsblk

>24. mount vda3
mount /dev/vda3 /mnt

>25. create subvolume for root (more subvolumes can be created but we'll do one for this guide)
btrfs su cr /mnt/@

>26. unmount partition
umount /mnt

>27. mount root subvolume with lzo compression
mount -O compress=lzo,subvol=@ /dev/vda3 /mnt

>28. make boot directory
mkdir -p /mnt/boot/efi

>29. mount vda1 on boot directory
mount /dev/vda1 /mnt/boot/efi

>30. list block devices and verify
lsblk

>31. install kernel and extras to root mount point (you can substitute nano for vim if you want a more beginner friendly text editor)
pacstrap /mnt base linux linux-firmware vim

>32. generate fstab file
genfstab -U /mnt >> /mnt/etc/fstab

>33. change apparent root directory to mount point
arch-chroot /mnt

>34. create softlink for zone information (use your zone)
ln -sf /usr/share/zoneinfo/US/Eastern /etc/localtime

>35. sync system and hardware clock
hwclock --systohc

>36. uncomment line to enable locale
vim /etc/locale.gen
en_US.UTF-8 UTF-8

>37. generate locale
locale-gen

>38. set locale in configuration file
vim /etc/locale.conf
LANG=en_US.UTF-8

>39. set hostname (can be anything)
vim /etc/hostname
arch

>40. configure hosts file
vim /etc/hosts
127.0.0.1    localhost
::1                localhost
127.0.1.1    arch.localdomain    arch
>41. set root password
passwd

>42. install grub and additional packages
pacman -S grub efibootmgr networkmanager network-manager-applet wireless_tools wpa_supplicant dialog mtools dosfstools base-devel linux-headers reflector

>43. install grub
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB

>44. generate grub configuration file
grub-mkconfig -o /boot/grub/grub.cfg

>45. enable Network Manager
systemctl enable NetworkManager

>46. create user and add to wheel group
useradd -mG wheel username

>47. create password
passwd username

>48. enable wheel group by uncommenting line
EDITOR=vim visudo
%wheel ALL=(ALL:ALL) ALL

>49. exit to installer
exit

>50. unmount all partitions
umount -a

>51. restart the system
reboot

>52. install gpu driver (if using NVIDIA)
sudo pacman -S nvidia-open nvidia-utils nvidia-settings

>53. install desktop environment or window manager (kde plasma example with sddm for login screen)
sudo pacman -S plasma-meta kde-applications-meta sddm

>54. restart
reboot
