
# Arch Linux Installation Guide (2026)

A clean, modern, step-by-step walkthrough for installing Arch Linux on UEFI systems with GPT, systemd, and modern defaults.

## 1. Pre-Installation Checks

### Verify Boot Mode (UEFI)

Verify that the live installation environment booted in UEFI mode:

```bash
ls /sys/firmware/efi/efivars
If the directory exists without error, the system is booted in UEFI mode. If it returns an error, verify that UEFI mode is enabled and CSM/Legacy boot is disabled in firmware settings.

Set Console Font & Keyboard Layout (Optional)
If working on a high-DPI display or using a non-US layout:

bash
setfont ter-132b
loadkeys us
Connect to the Internet
For wired connections (Ethernet), DHCP should configure automatically. Verify connection:

bash
ping -c 3 archlinux.org
For Wi-Fi connections, run iwctl:

bash
iwctl
Then:

Text
[ictl]# device list
[ictl]# station wlan0 scan
[ictl]# station wlan0 get-networks
[ictl]# station wlan0 connect "SSID_NAME"
[ictl]# exit
Synchronize System Clock
bash
timedatectl set-ntp true
2. Disk Partitioning & Formatting
Identify your target drive (for example /dev/nvme0n1 or /dev/sda):

bash
lsblk
Partitioning with fdisk or cfdisk
Launch cfdisk on the target storage disk:

bash
cfdisk /dev/nvme0n1
Select the GPT partition label and create the following layout:

Partition	Size	Type	Mount Point
/dev/nvme0n1p1	1024 MiB (1 GB)	EFI System Partition	/boot
/dev/nvme0n1p2	Remaining space	Linux Filesystem	/
Note: Modern systems generally favor zram-generator over dedicated swap partitions, but you can allocate a 4–8 GB swap partition if you require disk-based hibernation.

Formatting Partitions
Format the EFI partition to FAT32 and the root partition to Ext4:

bash
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.ext4 /dev/nvme0n1p2
Mounting the Filesystems
Mount the root filesystem first, create the boot mount directory, and mount the EFI partition:

bash
mount /dev/nvme0n1p2 /mnt
mount --mkdir /dev/nvme0n1p1 /mnt/boot
3. Base System Installation
Optimize Mirror List
Enable parallel downloads and rate mirrors using Reflector:

bash
reflector --country Worldwide --latest 10 --sort rate --save /etc/pacman.d/mirrorlist
Install Core Packages (pacstrap)
Install the Linux base system, kernel, core command utilities, microcode, and network stack:

bash
# For Intel processors:
pacstrap -K /mnt base base-devel linux linux-headers linux-firmware intel-ucode networkmanager nano git

# For AMD processors:
pacstrap -K /mnt base base-devel linux linux-headers linux-firmware amd-ucode networkmanager nano git
Generate fstab
Generate the filesystem table using UUIDs:

bash
genfstab -U /mnt >> /mnt/etc/fstab
Verify the contents of the generated table:

bash
cat /mnt/etc/fstab
4. System Configuration
Chroot into the newly installed environment:

bash
arch-chroot /mnt
Time Zone & Hardware Clock
Set your local time zone (for example, Asia/Kolkata or UTC):

bash
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc
Localization
Edit /etc/locale.gen and uncomment:

Text
en_US.UTF-8 UTF-8
Then run:

bash
sed -i 's/#en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen
locale-gen
Set the system locale:

bash
echo "LANG=en_US.UTF-8" > /etc/locale.conf
Network Configuration & Hostname
Set the machine hostname:

bash
echo "arch-machine" > /etc/hostname
Add loopback definitions to /etc/hosts:

bash
cat <<EOF > /etc/hosts
127.0.0.1   localhost
::1         localhost
127.0.1.1   arch-machine.localdomain arch-machine
EOF
Enable NetworkManager so network interfaces are managed upon reboot:

bash
systemctl enable NetworkManager
5. User Creation & Privileges
Set the root administrative password:

bash
passwd
Create a standard user with sudo permissions:

bash
useradd -m -G wheel -s /bin/bash yourusername
passwd yourusername
Grant privileges to the wheel group using visudo:

bash
EDITOR=nano visudo
Uncomment the following line:

Text
%wheel ALL=(ALL:ALL) ALL
6. Bootloader Setup (systemd-boot)
systemd-boot is built into systemd, lightweight, and requires no external packages on modern UEFI systems.

Install the bootloader into /boot:

bash
bootctl install
Configure the Bootloader
Create /boot/loader/loader.conf:

bash
cat <<EOF > /boot/loader/loader.conf
default  arch.conf
timeout  3
console-mode max
editor   no
EOF
Add the Arch Linux Entry
Determine your root partition's UUID:

bash
ROOT_UUID=$(blkid -s UUID -o value /dev/nvme0n1p2)
Create /boot/loader/entries/arch.conf:

For Intel systems:

bash
cat <<EOF > /boot/loader/entries/arch.conf
title   Arch Linux
linux   /vmlinuz-linux
initrd  /intel-ucode.img
initrd  /initramfs-linux.img
options root=UUID=$ROOT_UUID rw
EOF
For AMD systems:

bash
cat <<EOF > /boot/loader/entries/arch.conf
title   Arch Linux
linux   /vmlinuz-linux
initrd  /amd-ucode.img
initrd  /initramfs-linux.img
options root=UUID=$ROOT_UUID rw
EOF
7. Post-Installation & Exit
Exit the chroot environment, unmount partitions, and restart the system:

bash
exit
umount -R /mnt
reboot
Remove the bootable USB drive when prompted.

8. Essential Post-Install Steps
Once logged in as your non-root user:

Enable Multilib & Parallel Downloads
Edit /etc/pacman.conf and uncomment:

[multilib]
its Include
ParallelDownloads = 5
Audio Setup
bash
sudo pacman -S pipewire pipewire-pulse pipewire-alsa wireplumber
Graphics Drivers
Intel:

bash
sudo pacman -S mesa vulkan-intel
AMD:

bash
sudo pacman -S mesa vulkan-radeon
NVIDIA:

bash
sudo pacman -S nvidia nvidia-utils
Desktop Environment or Window Manager
Example: KDE Plasma

bash
sudo pacman -S plasma sddm
sudo systemctl enable sddm
Optional: Install a Desktop Environment or Window Manager
You may choose from:

KDE Plasma
GNOME
XFCE
Cinnamon
i3
AwesomeWM
bspwm
Install example:

bash
sudo pacman -S plasma-meta
Or for a lighter environment:

bash
sudo pacman -S xfce4 xfce4-goodies
