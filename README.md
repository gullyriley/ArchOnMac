# ArchOnMac
Guide to installing Arch on a Macbook

This is a personal step-by-step log of how I installed Arch Linux on my MacBook Air, and how I overcame the challenge of an unsupported wireless chip.
It’s a clean, minimal setup using **Hyprland**, **no display manager**, and some extras like `neofetch` and `brightnessctl`.  
---

## Step 1. Boot into Arch ISO

Booted from a USB with the official Arch ISO.  

---

## Step 1.5. Install WiFi Drivers (Broadcom)

My MacBook Air uses a Broadcom WiFi chipset, which isn’t supported out of the box on the Arch ISO.  
To get internet access during the install, I had to manually install the driver.

### Identify the Chip

```bash
lspci -k | grep -A 3 -i "network"
```

This showed a Broadcom chipset — usually something like BCM43xx.

### Install the Driver

```bash
pacman -Sy
pacman -S broadcom-wl
```

If that doesn’t work for your chip, you can also try:

```bash
pacman -S b43-fwcutter
```

Most MacBooks should be fine with broadcom-wl.

### Load the Driver

```bash
modprobe wl
```

Check if your wireless interface is now visible:

```bash
ip link
```

### Connect to WiFi with iwctl

```bash
iwctl
station wlan0 scan
station wlan0 get-networks
station wlan0 connect <SSID>
```

Once connected, I had internet access for the rest of the install.

**Tip:** I also installed broadcom-wl again after the main install to make sure it was included in the system permanently.

---

## Step 2. Partitioned the Disk

Used `fdisk` to wipe and set up the drive.

- `/dev/sda1` – EFI partition (~512M)
- `/dev/sda2` – root (rest of the disk)

```bash
mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
```

---

## Step 3. Mount the Partitions

```bash
mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

---

## Step 4. Install the Base System

```bash
pacstrap /mnt base base-devel linux linux-firmware networkmanager
```

---

## Step 5. Generate `fstab`

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

---

## Step 6. Chroot In

```bash
arch-chroot /mnt
```

---

## Step 7. Set Up Time + Locale

```bash
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc
```

Locale:

```bash
nano /etc/locale.gen
# Uncomment: en_US.UTF-8 UTF-8 or en_AU.UTF-8 UTF-8

locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

---

## Step 8. Hostname + Hosts File

```bash
echo "arch" > /etc/hostname

# Then edit /etc/hosts:
127.0.0.1    localhost
::1          localhost
127.0.1.1    arch.localdomain myarch
```

---

## Step 9. Bootloader (systemd-boot)

```bash
bootctl install
```

Create loader entries in `/boot/loader/loader.conf` and `/boot/loader/entries/arch.conf`.  
Set `root=` to your disk (e.g. `/dev/sda2` or `PARTUUID=`).

---

## Step 10. Enable NetworkManager

```bash
systemctl enable NetworkManager
```

---

## Step 11. Reboot into the New System

```bash
exit
umount -R /mnt
reboot
```

---

## Step 12. Post-Install Tweaks

After reboot, WiFi worked (had installed Broadcom drivers).  
Next, installed some basics:

```bash
sudo pacman -S neofetch brightnessctl git
```

---

## Step 13. Desktop Environment

### Hyprland

Went full Wayland. No X11. Installed Hyprland:

```bash
sudo pacman -S hyprland waybar kitty
```

---

## Final Words.

- **No display manager** — I log in via TTY and start Hyprland manually (`exec Hyprland` in `.zprofile`).
---
