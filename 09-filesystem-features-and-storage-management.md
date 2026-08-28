# 09 - Linux Filesystem Features & Storage Management

Panduan manajemen penyimpanan dan fitur filesystem Linux: pembuatan filesystem (`mkfs`), perbaikan dan pengecekan integritas (`fsck`), mounting manual & opsi lanjutan (`mount`, `umount`), persistent mounting berbasis UUID (`/etc/fstab`), Network File Sharing (NFS), inspeksi kapasitas (`df`, `du`), serta konfigurasi Swap Memory.

---

## Daftar Isi
- [1. Format & Inisialisasi Filesystem (mkfs)](#1-format--inisialisasi-filesystem-mkfs)
- [2. Pengecekan & Perbaikan Integritas Filesystem (fsck, e2fsck)](#2-pengecekan--perbaikan-integritas-filesystem-fsck-e2fsck)
- [3. Operasi Mount & Unmount (mount, umount)](#3-operasi-mount--unmount-mount-umount)
- [4. Konfigurasi Persistent Mounting (/etc/fstab)](#4-konfigurasi-persistent-mounting-etcfstab)
- [5. Network File System (NFS) Client & Mount](#5-network-file-system-nfs-client--mount)
- [6. Monitoring Kapasitas Penyimpanan (df & du)](#6-monitoring-kapasitas-penyimpanan-df--du)
- [7. Manajemen Memori Swap (Swapfile & Swappiness)](#7-manajemen-memori-swap-swapfile--swappiness)
- [8. Navigasi Lanjutan Modul](#8-navigasi-lanjutan-modul)

---

## 1. Format & Inisialisasi Filesystem (mkfs)

Perintah `mkfs` (*Make Filesystem*) adalah antarmuka pembungkus (*front-end wrapper*) untuk memformat partisi disk mentah dengan sistem file tertentu.

| Perintah Eksekusi | Filesystem Target | Opsi Kritis & Fungsi |
| :--- | :--- | :--- |
| `sudo mkfs.ext4 /dev/sdb1` | Ext4 | Format partisi standar Ext4. |
| `sudo mkfs.ext4 -L "DATA_DISK" /dev/sdb1` | Ext4 (Label) | Memberikan label penamaan volume pada partisi (`-L`). |
| `sudo mkfs.ext4 -b 4096 /dev/sdb1` | Ext4 (Block Size) | Menentukan ukuran blok data secara spesifik (default: 4096 bytes / 4KB). |
| `sudo mkfs.xfs -f /dev/sdc1` | XFS | Format partisi XFS. Opsi `-f` (*force*) menimpa filesystem lama yang sudah ada. |
| `sudo mkfs.btrfs /dev/sdd1` | Btrfs | Format partisi Btrfs modern dengan fitur Copy-on-Write. |
| `sudo mkfs.vfat -F 32 /dev/sde1` | FAT32 | Format media penyimpanan eksternal (USB flashdisk) untuk kompatibilitas multi-OS. |

---

## 2. Pengecekan & Perbaikan Integritas Filesystem (fsck, e2fsck)

Perintah `fsck` (*Filesystem Consistency Check*) digunakan untuk mendeteksi dan memperbaiki inkonsistensi struktur metadata atau blok yang rusak.

> [!CAUTION]
> **Aturan Wajib:** JANGAN PERNAH menjalankan `fsck` pada partisi yang sedang dalam kondisi ter-mount (`MOUNTED`). Lakukan unmount terlebih dahulu (`umount`) atau boot ke mode *Rescue / Single-User Mode* untuk partisi root (`/`).

| Perintah | Target Filesystem | Deskripsi & Opsi Utama |
| :--- | :--- | :--- |
| `sudo fsck -y /dev/sdb1` | Otomatis | Memeriksa filesystem dan otomatis menjawab "Yes" untuk setiap perbaikan yang disarankan (`-y`). |
| `sudo fsck -f /dev/sdb1` | Otomatis | Memaksa pemeriksaan (*Force Check*) meskipun filesystem berstatus bersih (*clean*). |
| `sudo fsck -c /dev/sdb1` | Otomatis | Memindai bad block fisik pada drive penyimpanan (`-c`). |
| `sudo e2fsck -f -y /dev/sdb1` | Ext2/Ext3/Ext4 | Utilitas perbaikan khusus keluarga Ext dengan opsi paksa dan persetujuan otomatis. |
| `sudo e2fsck -b 32768 /dev/sdb1` | Ext4 (Superblock) | Memperbaiki filesystem menggunakan Superblock cadangan (*Backup Superblock*). |
| `sudo xfs_repair /dev/sdc1` | XFS | Utilitas khusus untuk perbaikan filesystem XFS (partisi wajib unmounted). |

---

## 3. Operasi Mount & Unmount (mount, umount)

### Perintah Dasar Mounting

| Operasi | Perintah | Penjelasan |
| :--- | :--- | :--- |
| **Mount Standar** | `sudo mount /dev/sdb1 /mnt/data` | Mengaitkan partisi `/dev/sdb1` ke direktori titik kait `/mnt/data`. |
| **Mount Tipe Spesifik** | `sudo mount -t ext4 /dev/sdb1 /mnt/data` | Menentukan jenis filesystem secara eksplisit menggunakan opsi `-t`. |
| **Lihat Seluruh Mount** | `mount \| grep "^/dev"` | Menampilkan daftar seluruh partisi fisik yang saat ini aktif ter-mount. |

### Opsi Lanjutan Perintah `mount -o`

| Opsi Mount (`-o`) | Fungsi Teknis | Kasus Penggunaan Keamanan / Performa |
| :--- | :--- | :--- |
| `ro` / `rw` | Read-Only / Read-Write | Mengunci partisi agar tidak bisa diubah (`ro`) atau mengizinkan baca-tulis (`rw`). |
| `noexec` | No Execution | Melarang eksekusi biner/skrip di partisi tersebut (standar keamanan partisi `/tmp`). |
| `nosuid` | No SUID/SGID | Mengabaikan bit perizinan SUID dan SGID demi keamanan. |
| `nodev` | No Character/Block Devices | Mencegah pembuatan file node perangkat keras di partisi pengguna. |
| `remount` | Remounting Live | Mengubah opsi partisi yang sudah terpasang tanpa perlu unmount. Contoh: `sudo mount -o remount,ro /data`. |
| `bind` | Directory Mirroring | Me-mount suatu folder ke folder lain. Contoh: `sudo mount --bind /var/log /mnt/logs`. |
| `loop` | Loop Device (ISO) | Me-mount berkas citra ISO sebagai filesystem. Contoh: `sudo mount -o loop ubuntu.iso /mnt/iso`. |

### Operasi Unmount & Penanganan "Target is Busy"

| Skenario | Perintah | Solusi Masalah |
| :--- | :--- | :--- |
| **Unmount Standar** | `sudo umount /mnt/data` atau `sudo umount /dev/sdb1` | Melepas kaitan partisi dari sistem. |
| **Lazy Unmount (`-l`)** | `sudo umount -l /mnt/data` | Melepaskan titik mount seketika dan membersihkan referensi begitu tidak ada proses yang mengakses. |
| **Force Unmount (`-f`)** | `sudo umount -f /mnt/nfs` | Memaksa pelepasan kaitan (sangat efektif untuk share jaringan NFS yang hang). |
| **Lacak Proses Pengunci** | `sudo fuser -vm /mnt/data` | Menampilkan PID proses dan user yang sedang membuka file di dalam direktori mount. |
| **Tutup Paksa Pengunci** | `sudo fuser -k /mnt/data` | Mengirim sinyal kill ke seluruh proses yang menahan titik mount. |
| **Inspeksi File Terbuka** | `sudo lsof +f -- /mnt/data` | Menampilkan rincian daftar file yang sedang dibuka pada titik mount bersangkutan. |

---

## 4. Konfigurasi Persistent Mounting (/etc/fstab)

File `/etc/fstab` (*Filesystem Table*) dibaca oleh Kernel saat proses booting untuk me-mount partisi storage secara otomatis dan permanen.

### Anatomi 6 Kolom `/etc/fstab`

```text
UUID=a1b2c3d4-e5f6-7890   /data   ext4   defaults,noatime   0   2
───────────────────────   ─────   ────   ────────────────   ─   ─
           │                │       │            │          │   │
           │                │       │            │          │   └── Kolom 6: Fsck Pass (0=Lewati, 1=Root, 2=Partisi Lain)
           │                │       │            │          └────── Kolom 5: Dump Backup Flag (0=Matikan, 1=Aktif)
           │                │       │            └───────────────── Kolom 4: Mount Options (defaults, ro, noexec, nofail)
           │                │       └────────────────────────────── Kolom 3: Tipe Filesystem (ext4, xfs, btrfs, nfs)
           │                │                                       Kolom 2: Mount Point Target (/data, /var, /)
           └─────────────────────────────────────────────────────── Kolom 1: Device Identifier (UUID atau Device Path)
```

### Identifikasi UUID Partisi

Gunakan UUID (*Universally Unique Identifier*) alih-alih nama perangkat (`/dev/sdb1`) karena nama `/dev/sdX` dapat berubah saat kabel/urutan disk berganti.

| Perintah | Output yang Dihasilkan |
| :--- | :--- |
| `sudo blkid` | Menampilkan UUID, Label, dan Type seluruh partisi disk. |
| `lsblk -f` | Menampilkan struktur pohon block device beserta UUID dan mount point aktif. |

### Validasi Konfigurasi `/etc/fstab`

> [!IMPORTANT]
> **Wajib Verifikasi Sebelum Reboot!**
> Kesalahan sintaks pada `/etc/fstab` dapat menyebabkan server gagal booting (*Emergency Mode*). Selalu uji konfigurasi dengan perintah:
> ```bash
> sudo mount -a
> ```
> Jika tidak muncul output error, konfigurasi `/etc/fstab` dinyatakan valid dan aman.

| Opsi fstab Khusus | Fungsi & Kegunaan |
| :--- | :--- |
| `nofail` | Sistem tetap melanjutkan proses booting normal meskipun perangkat disk tersebut tidak terpasang/hilang. Sangat disarankan untuk external drive atau cloud EBS volume sekunder. |
| `_netdev` | Menunda proses mount hingga koneksi jaringan aktif (wajib untuk share NFS / iSCSI). |
| `noatime` | Mematikan pencatatan waktu akses file (*Access Time*) untuk mendongkrak performa I/O disk. |

---

## 5. Network File System (NFS) Client & Mount

NFS memungkinkan server Linux berbagi direktori penyimpanan melalui jaringan TCP/IP.

| Komponen / Peran | Perintah Eksekusi | Konfigurasi & Keterangan |
| :--- | :--- | :--- |
| **Instalasi Client** | `sudo apt install nfs-common` *(Ubuntu)* <br> `sudo dnf install nfs-utils` *(RHEL)* | Memasang paket pustaka pendukung protokol klien NFS. |
| **Mount NFS Manual** | `sudo mount -t nfs 192.168.1.100:/export/share /mnt/nfs` | Me-mount folder export dari IP server NFS ke direktori lokal `/mnt/nfs`. |
| **Persistent NFS Mount** | Tambahkan ke `/etc/fstab`: <br> `192.168.1.100:/export/share /mnt/nfs nfs _netdev,defaults,nofail 0 0` | Otomatis me-mount storage NFS saat booting setelah antarmuka jaringan aktif. |

---

## 6. Monitoring Kapasitas Penyimpanan (df & du)

### Perintah `df` (Disk Filesystem Space)

| Perintah | Opsi / Parameter | Keterangan |
| :--- | :--- | :--- |
| `df -h` | Human Readable | Menampilkan kapasitas total, terpakai, dan sisa dalam format GB/MB. |
| `df -hT` | Type Display | Menampilkan kolom tipe filesystem (`ext4`, `xfs`, `tmpfs`). |
| `df -i` | Inodes Capacity | Menampilkan sisa jumlah slot Inode pada setiap partisi. |
| `df -h /var` | Spesifik Path | Menampilkan kapasitas partisi tempat direktori `/var` berada. |

### Perintah `du` (Disk Usage by Files/Directories)

| Perintah | Opsi / Pipeline | Keterangan |
| :--- | :--- | :--- |
| `du -sh /var/log` | Summary Human | Menampilkan total kalkulasi ukuran folder `/var/log`. |
| `sudo du -h --max-depth=1 /var` | Kedalaman Level 1 | Menampilkan rincian ukuran setiap sub-folder langsung di dalam `/var`. |
| `sudo du -ah /var \| sort -rh \| head -n 15` | Top 15 Terbesar | Menemukan 15 file atau folder yang memakan kapasitas storage terbesar. |

---

## 7. Manajemen Memori Swap (Swapfile & Swappiness)

Swap adalah area pada media penyimpanan (disk) yang digunakan oleh kernel Linux sebagai memori virtual saat memori fisik RAM telah terisi penuh.

### Pembuatan & Aktivasi Swapfile

| Tahapan | Perintah Eksekusi | Deskripsi Operasi |
| :---: | :--- | :--- |
| **1. Alokasi Berkas** | `sudo fallocate -l 2G /swapfile` | Membuat file berukuran 2 Gigabyte secara instan. |
| **2. Penguncian Hak Akses** | `sudo chmod 600 /swapfile` | Membatasi izin akses hanya untuk root demi keamanan data memori. |
| **3. Format Struktur Swap** | `sudo mkswap /swapfile` | Menginisialisasi format area swap pada file tersebut. |
| **4. Aktivasi Swap** | `sudo swapon /swapfile` | Mengaktifkan swapfile ke dalam sistem kernel Linux. |
| **5. Verifikasi Status** | `swapon --show` atau `free -h` | Memastikan kapasitas swap bertambah dan terbaca aktif. |
| **6. Deaktivasi Swap** | `sudo swapoff /swapfile` | Menonaktifkan file swap dari sistem. |

### Persistent Swap di `/etc/fstab`

Tambahkan baris berikut ke dalam file `/etc/fstab` agar swap tetap aktif setelah reboot:

```text
/swapfile   none   swap   sw   0   0
```

### Tuning Parameter Swappiness

`vm.swappiness` menentukan seberapa agresif kernel memindahkan data dari RAM fisik ke Swap (rentang: `0` hingga `100`, default: `60`).

| Nilai Swappiness | Perilaku Kernel | Rekomendasi Penggunaan |
| :---: | :--- | :--- |
| **`0 - 10`** | Kernel sebisa mungkin menghindari swap kecuali RAM fisik benar-benar habis total. | Server Database berkinerja tinggi (MySQL, PostgreSQL, Redis). |
| **`60`** | Nilai seimbang default kernel Linux. | Desktop dan server serbaguna (*general purpose*). |
| **`100`** | Kernel sangat agresif melakukan paging data ke swap. | Sistem dengan kapasitas RAM fisik sangat minim. |

```bash
# Memeriksa nilai swappiness saat ini
cat /proc/sys/vm/swappiness

# Mengubah nilai sementara ke 10
sudo sysctl vm.swappiness=10

# Menyimpan nilai swappiness permanen (Tambahkan ke /etc/sysctl.conf)
echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

---

## 8. Navigasi Lanjutan Modul

| Modul Sebelumnya | Indeks Dokumentasi |
| :---: | :---: |
| [08 - Linux Filesystems and the VFS](./08-linux-filesystems-and-vfs.md) | [README.md - Master Index](./README.md) |
