# Panduan untuk Menginstall Archlinux, Setup, dan Rekomendasi Software

<!-- TODO: Tambahkan daftar isi -->

Repo ini merupakan panduan yang biasa saya gunakan untuk menginstall Archlinux, setup HyDE sebagai window manager utama, dan beberapa software yang biasa saya pakai.

> Apabila hendak duall boot dengan Windows, silakan install Windows terlebih dahulu baru kemudian install Archlinux, dikarenakan supaya menghindari EFI system Linux di-replace oleh Windows.

---

## Download ISO dan Buat Bootable

ISO bisa di-download di [website Archlinux](https://archlinux.org/download/). Ada beberapa metode tapi direkomendasikan download langsung ISO dengan disesuaikan negara.

> Current Release ISO di-update setiap tanggal 1. Tidak perlu download terus setiap bulan.

Untuk membuat bootable ISO bisa menggunakan Ventoy, balenaEtcher, ataupun Rufus.

## Install Archlinux

### Setup Partisi

Set partisi yang akan mau di-install. Bisa di SSD atau HDD.

> Sebelum install, pastikan backup terlebih dahulu data-data penting, dikhawatirkan terjadi sesuatu selama proses install, misalnya partisi terhapus ataupun _human error_ lainnya.

```bash
cfdisk /dev/lokasi-dride # contoh "/dev/sda" atau "/dev/sdb"
```

Buat dua partisi baru, yaitu EFI dan root. Kalau mau buat home terpisah, silakan bisa buat juga partisi untuk home. Untuk EFI cukup pakai 1 GiB saja.

> Biar tidak repot, saya rekomendasikan untuk Root dan Home disatukan saja.

---

<!-- TODO: Tambahin versi WSL -->

#### SSD Version

Ini proses install yang dikhususkan bagi SSD karena root akan di-format pakai BTRFS. Apabila pakainya HDD, bisa cek bagian "HDD Version".

Format setiap partisi. EFI pakai Fat32, Root pakai Btrfs, dan Home pakai Btrfs.

- EFI

```bash
mkfs.fat -F 32 /dev/lokasi-efi # Contoh /dev/sdb1
```

- Root

```bash
mkfs.btrfs /dev/lokasi-root # Contoh /dev/sdb2
```

- Home

```bash
mkfs.btrfs /dev/lokasi-home # Contoh /dev/sdb3
```

Sekarang mount, setup Btrfs, dan compress

```bash
mount /dev/lokasi-home /mnt
```

```bash
btrfs subvolume create /mnt/@
```

```bash
btrfs subvolume create /mnt/@home
```

```bash
umount /mnt
```

```bash
mount -o compress=zstd,subvol=@ /dev/lokasi-root /mnt
```

```bash
mkdir -p /mnt/home
```

```bash
mount -o compress=zstd,subvol=@home /dev/lokasi-home /mnt/home # Kalau Home mau dijadikan satu dengan Root, maka cukup ganti "lokasi-home" jadi "lokasi-root"
```

```bash
mkdir -p /mnt/efi
```

```bash
mount /dev/lokasi-efi /mnt/efi
```

Sekarang sinkronisasi package database dan sinkronisasi mirror supaya mengambil dari tercepat.

```bash
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup # Tujuannya buat backup kalau misalnya mirror yang disinkronisasi ternyata lebih jelek kecepatanya.
```

```bash
pacman -Sy # Sinkronisasi package database
```

```bash
pacman -S archlinux-keyring reflector rsync curl python # Download kebutuhan package
```

> Untuk sinkronisasi mirror cukup download reflector, rsync, dan curl saja. Package archlinux-keyring dan python bertujuan untuk memastikan bahwa ISO yang sekarang dipakai ini benar-benar terbaru dan mencegah error karena keyring yang belum di-update maupun python yang belum ter-download.

```bash
reflector --save /etc/pacman.d/mirrorlist --latest 10 --protocol https --sort rate # Sinkronisasi mirrorlist
```

> Ini _khusus bagi yang mau balik ke mirrorlist sebelumnya_. Kalau misalnya gak ada masalah di bagian kecepatan internetnya, gak perlu ketik ini.

```bash
cp /etc/pacman.d/mirrorlist.backup /etc/pacman.d/mirrorlist
```

Sekarang pacstrap atau pasang Archlinux.

```bash
pacstrap -K /mnt base base-devel linux-zen linux-zen-headers linux-firmware sof-firmware amd-ucode vim git networkmanager dhcpcd network-manager-applet grub efibootmgr dosfstools mtools os-prober mesa gufw ufw redshift man-db man-pages lvm2 fastfetch ntfs-3g vnstat btrfs-progs grub-btrfs inotify-tools timeshift reflector openssh man sudo
```

<!-- TODO: Tambahkan penjelasan setiap package ini berfungsi buat apa. Urutkan dari essential sampai opsional. -->

Sekarang arch-chroot untuk setup terakhir

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

```bash
arch-chroot /mnt
```

```bash
ln -sf /usr/share/zoneinfo/benua-kamu-tinggal/ibukota-kamu-tinggal /etc/localtime # Contoh /Asia/Jakarta
```

```bash
hwclock --systohc
```

```bash
vim /etc/locale.gen
```

> Uncomment bahasa atau locale yang kamu pakai. Rekomendasinya en_US.UTF-8

```bash
locale-gen
```

```bash
echo LANG=bahasa-kamu > /etc/locale.conf # Contoh LANG=en_US.UTF-8
```

```bash
export LANG=bahasa-kamu # Contoh LANG=en_US.UTF-8
```

```bash
passwd # Pasang password root
```

```bash
useradd -m user-kamu # Contoh ervin
```

```bash
usermod -aG wheel,storage,power user-kamu # Contoh ervin
```

```bash
passwd user-kamu
```

```bash
visudo # Kalau misalnya pakai text editor lain, tambahkan "EDITOR=editor-kamu visudo" misalnya EDITOR=nano
```

Uncomment "%wheel ALL=(ALL:ALL) ALL" dan tambahkan di line bawahnya "Defaults timestamp_timeout=10"

```bash
echo hostname-kamu > /etc/hostname # Saya biasanya archlinux
```

```bash
vim /etc/hosts
```

Tambahkan di paling akhir "127.0.1.1 archlinux.localdomain localhost"

```bash
vim /etc/default/grub
```

Uncomment "GRUB_DISABLE_OS_PROBER=false"

```bash
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=Archlinux
```

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

```bash
systemctl enable NetworkManager
```

```bash
systemctl enable vnstat
```

```bash
exit
```

```bash
umount -lR /mnt
```

```bash
reboot
```

<!-- TODO: Tambahkan tutorial HDD dan WSL -->

## Pasang HyDE
