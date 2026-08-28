# 08 - Linux Filesystems and the VFS

Panduan mendalam arsitektur sistem file Linux dan Virtual File System (VFS): variasi filesystem, mekanisme journaling, struktur blok Ext4 (Superblock & Block Groups), XFS, sistem file khusus (*special/pseudo filesystems*), struktur Inodes, serta implementasi praktis Hard Link dan Soft Link.

---

## Daftar Isi
- [1. Konsep Dasar Filesystem & Virtual File System (VFS)](#1-konsep-dasar-filesystem--virtual-file-system-vfs)
- [2. Klasifikasi & Variasi Filesystem Linux](#2-klasifikasi--variasi-filesystem-linux)
- [3. Mekanisme Journaling Filesystem](#3-mekanisme-journaling-filesystem)
- [4. Struktur Internal Ext4: Superblock & Block Groups](#4-struktur-internal-ext4-superblock--block-groups)
- [5. Karakteristik & Utilitas XFS Filesystem](#5-karakteristik--utilitas-xfs-filesystem)
- [6. Special & Pseudo Filesystems (proc, sysfs, devtmpfs, tmpfs)](#6-special--pseudo-filesystems-proc-sysfs-devtmpfs-tmpfs)
- [7. Struktur & Karakteristik Inodes](#7-struktur--karakteristik-inodes)
- [8. Praktikum Terpandu: Hard Links vs Soft Links (Lab 8.1 - 8.3)](#8-praktikum-terpandu-hard-links-vs-soft-links-lab-81---83)
- [9. Navigasi Lanjutan Modul](#9-navigasi-lanjutan-modul)

---

## 1. Konsep Dasar Filesystem & Virtual File System (VFS)

### Apa itu Filesystem?
Filesystem adalah struktur logis yang digunakan oleh sistem operasi untuk mengorganisasi, menyimpan, mengidentifikasi, dan mengambil data pada media penyimpanan fisik (SSD, NVMe, HDD).

### Virtual File System (VFS)
VFS adalah lapisan abstraksi (*abstraction layer*) di dalam Kernel Linux yang memungkinkan aplikasi pengguna memanggil sistem operasi secara seragam (`open()`, `read()`, `write()`, `close()`) tanpa perlu memedulikan jenis filesystem fisik yang mendasarinya.

```text
[ Aplikasi Pengguna (Nginx, MySQL, Bash, Python) ]
                        │
                        ▼ (System Calls: open, read, write)
┌────────────────────────────────────────────────────────┐
│             Kernel Virtual File System (VFS)           │
└───────┬──────────────┬──────────────┬──────────┬───────┘
        ▼              ▼              ▼          ▼
     [ Ext4 ]       [ XFS ]       [ Btrfs ]   [ tmpfs ]
        │              │              │          │
        ▼              ▼              ▼          ▼
   [ NVMe/SSD ]     [ SAN/DAS ]     [ HDD ]    [ RAM ]
```

| Komponen VFS | Peran & Fungsi dalam Kernel |
| :--- | :--- |
| **Superblock Object** | Menyimpan metadata global dari filesystem yang sedang di-mount. |
| **Inode Object** | Menyimpan seluruh metadata dari sebuah file/direktori individual (kecuali nama file). |
| **Dentry Object** (*Directory Entry*) | Menghubungkan nama file (*string*) dengan nomor Inode yang bersangkutan untuk mempercepat pencarian path. |
| **File Object** | Menyimpan status interaksi file yang sedang dibuka oleh proses (misal: posisi pointer baca/tulis saat ini). |

---

## 2. Klasifikasi & Variasi Filesystem Linux

| Nama Filesystem | Kategori | Fitur Utama & Karakteristik | Distribusi Default |
| :--- | :--- | :--- | :--- |
| **Ext4** (*Fourth Extended*) | Disk Storage | Sangat stabil, kompatibilitas tinggi, mendukung journaling, ukuran partisi hingga 1 EB. | Ubuntu, Debian, Linux Mint |
| **XFS** | Disk Storage (High-Perf) | Skalabilitas I/O paralel tinggi, alokasi berbasis B+ tree, cepat untuk file berukuran gigabyte/terabyte. | RHEL, Rocky Linux, CentOS, AlmaLinux |
| **Btrfs** (*B-tree FS*) | Advanced Storage | Copy-on-Write (CoW), built-in RAID, snapshot instan, checksum data & metadata terpadu. | Fedora, openSUSE |
| **ZFS** | Enterprise Storage | Volume manager terintegrasi, integritas data tinggi, self-healing, kompresi live. | Ubuntu (Opsional), TrueNAS |
| **tmpfs** | In-Memory (RAM) | Filesystem virtual berbasis RAM dengan dukungan swapping ke disk saat RAM penuh. | Seluruh Distro Linux |
| **NFS** | Network Storage | Network File System untuk berbagi folder terdistribusi melalui jaringan LAN. | Multi-platform |

---

## 3. Mekanisme Journaling Filesystem

Journaling adalah teknik pencatatan transaksi perubahan data ke area khusus (*Journal log*) di disk sebelum data sebenarnya ditulis ke blok penyimpanan utama. Tujuannya adalah mencegah korupsi filesystem saat terjadi mati listrik mendadak (*power failure*) atau crash kernel.

| Mode Journaling (Ext4) | Cara Kerja | Keamanan Data | Performa I/O |
| :--- | :--- | :---: | :---: |
| **`data=journal`** | Seluruh data berkas dan metadata ditulis ke journal terlebih dahulu sebelum ditulis ke filesystem utama. | Paling Tinggi | Paling Lambat (Data ditulis 2x ke disk) |
| **`data=ordered`** *(Default)* | Data berkas ditulis ke disk utama terlebih dahulu, kemudian metadatanya dicatat ke journal. | Tinggi | Optimal / Seimbang |
| **`data=writeback`** | Hanya metadata yang dicatat ke journal. Tidak ada jaminan urutan penulisan data berkas. | Sedang | Paling Cepat |

---

## 4. Struktur Internal Ext4: Superblock & Block Groups

Filesystem Ext4 membagi partisi penyimpanan menjadi beberapa **Block Groups** untuk mengurangi fragmentasi dan mempercepat akses head disk.

```text
[ Ext4 Partition Layout ]
┌─────────────────┬─────────────────┬─────────────────┬─────────────────┐
│  Block Group 0  │  Block Group 1  │  Block Group 2  │  Block Group N  │
└────────┬────────┴─────────────────┴─────────────────┴─────────────────┘
         │
         ▼ Struktur di dalam Satu Block Group:
 ┌──────────────────────┬──────────────────────────────────────────────┐
 │ Superblock (Primary) │ Metadata global partisi (total blok & inode) │
 ├──────────────────────┼──────────────────────────────────────────────┤
 │ Group Descriptors    │ Lokasi tabel bitmap & inode dalam grup       │
 ├──────────────────────┼──────────────────────────────────────────────┤
 │ Block Bitmap         │ Peta bit penanda blok data yang terpakai     │
 ├──────────────────────┼──────────────────────────────────────────────┤
 │ Inode Bitmap         │ Peta bit penanda nomor Inode yang terpakai   │
 ├──────────────────────┼──────────────────────────────────────────────┤
 │ Inode Table          │ Array struktur data Inode fisik              │
 ├──────────────────────┼──────────────────────────────────────────────┤
 │ Data Blocks          │ Blok tempat isi konten data file disimpan    │
 └──────────────────────┴──────────────────────────────────────────────┘
```

### Utilitas Manajemen & Inspeksi Ext4

| Perintah | Fungsi Teknis | Contoh Penggunaan |
| :--- | :--- | :--- |
| `mkfs.ext4 <partition>` | Membuat filesystem Ext4 baru pada partisi | `sudo mkfs.ext4 /dev/sdb1` |
| `tune2fs -l <partition>` | Membaca metadata isi Superblock | `sudo tune2fs -l /dev/sdb1` |
| `dumpe2fs <partition>` | Menampilkan rincian seluruh Block Groups dan Superblock backup | `sudo dumpe2fs /dev/sdb1 \| head -n 30` |
| `fsck.ext4` / `e2fsck` | Memeriksa dan memperbaiki kerusakan struktur filesystem | `sudo e2fsck -f -y /dev/sdb1` |

> [!NOTE]
> **Superblock Backup:**
> Linux otomatis membuat salinan cadangan (*backup superblocks*) di beberapa Block Group (misal blok 32768, 98304). Jika Superblock utama rusak, kamu bisa memperbaikinya dengan:
> `sudo e2fsck -b 32768 /dev/sdb1`

---

## 5. Karakteristik & Utilitas XFS Filesystem

XFS dirancang khusus untuk throughput tinggi, konkurensi skala besar, dan pengelolaan partisi berkapasitas sangat besar.

| Fitur / Aspek | Karakteristik XFS |
| :--- | :--- |
| **Allocation Groups (AG)** | Partisi XFS dibagi menjadi Allocation Groups independen yang dapat diproses secara paralel oleh multi-core CPU. |
| **Kapasitas Skala** | Mendukung filesystem hingga 8 Exabytes (EB). |
| **Dukungan Resize** | **Bisa diperbesar (*Grow*) secara online**, namun **TIDAK BISA diperkecil (*Shrink*)**. |

### Utilitas Manajemen XFS

| Perintah | Deskripsi Operasi | Contoh Perintah |
| :--- | :--- | :--- |
| `mkfs.xfs <device>` | Format partisi dengan filesystem XFS | `sudo mkfs.xfs /dev/sdc1` |
| `xfs_info <mount_point>` | Menampilkan rincian parameter Allocation Groups dan ukuran blok | `xfs_info /data` |
| `xfs_growfs <mount_point>` | Memperbesar ukuran partisi XFS saat volume storage diperluas | `sudo xfs_growfs /data` |
| `xfs_repair <device>` | Memperbaiki inkonsistensi struktur metadata XFS (partisi harus di-unmount) | `sudo xfs_repair /dev/sdc1` |
| `xfs_admin` | Mengubah UUID atau label partisi XFS | `sudo xfs_admin -L "DATA_DISK" /dev/sdc1` |

---

## 6. Special & Pseudo Filesystems (proc, sysfs, devtmpfs, tmpfs)

Sistem file khusus tidak dialokasikan di piringan magnetik hard disk, melainkan disediakan langsung oleh kernel di memori RAM.

| Mount Point | Nama Filesystem | Fungsi & Konten yang Dikelola |
| :--- | :--- | :--- |
| `/proc` | `procfs` | Interface kernel untuk proses yang sedang berjalan (`/proc/<PID>`) dan konfigurasi kernel dinamis (`/proc/sys/`). |
| `/sys` | `sysfs` | Struktur pohon hierarki perangkat keras terpasang, bus PCI/USB, dan driver kernel. |
| `/dev` | `devtmpfs` | Node perangkat keras sistem yang diatur otomatis oleh `udev` daemon. |
| `/tmp` & `/run` | `tmpfs` | Penyimpanan berkas sementara berbasis RAM. Kecepatan baca/tulis setara kecepatan memori RAM fisik. |
| `/dev/shm` | `tmpfs` | *Shared Memory* antar proses IPC berbasis memori virtual. |

---

## 7. Struktur & Karakteristik Inodes

Setiap file di Linux diwakili oleh sebuah nomor unik bernama **Inode (*Index Node*)**.

### Metadata yang Disimpan di Dalam Inode

| Komponen Metadata | Penjelasan Teknis |
| :--- | :--- |
| **File Type** | Tipe berkas (Regular file, Directory, Symlink, Socket, FIFO pipe, Block/Char device). |
| **Permissions** | Mode hak akses Unix (`rwxrwxrwx`). |
| **Ownership** | User ID (`UID`) dan Group ID (`GID`) pemilik berkas. |
| **File Size** | Ukuran berkas aktual dalam satuan bytes. |
| **Timestamps** | `atime` (Access), `mtime` (Content modification), `ctime` (Metadata change). |
| **Link Count** | Jumlah Hard Link yang menunjuk ke Inode bersangkutan. |
| **Data Block Pointers** | Alamat fisik blok penyimpanan data di hard disk/SSD. |

> [!IMPORTANT]
> **Nama File TIDAK DISIMPAN di dalam Inode!**
> Nama file disimpan di dalam blok data milik **Direktori Induk**, yang memetakan: `[ Nama File -> Nomor Inode ]`.

### Perintah Pemeriksaan Inodes

| Perintah | Fungsi | Contoh Output |
| :--- | :--- | :--- |
| `ls -i <file>` | Melihat nomor Inode suatu file | `1441852 script.sh` |
| `ls -li <dir>` | Menampilkan daftar file lengkap beserta Inode | `1441852 -rwxr-xr-x 1 root root ...` |
| `stat <file>` | Menampilkan seluruh metadata Inode secara komprehensif | Inode, Links, Access, Modify, Change, Birth |
| `df -i` | Melihat persentase ketersediaan nomor Inode pada partisi | Mengidentifikasi kehabisan slot Inode |

---

## 8. Praktikum Terpandu: Hard Links vs Soft Links (Lab 8.1 - 8.3)

### Perbandingan Fundamental Link

| Parameter Evaluasi | Hard Link | Symbolic Link (Soft Link) |
| :--- | :--- | :--- |
| **Sifat Tautan** | Pointer langsung ke nomor Inode yang sama | File pointer baru berisi alamat path string file target |
| **Nomor Inode** | Identik / Sama persis | Berbeda (Mengalokasikan Inode baru) |
| **Lintas Partisi Storage** | Tidak didukung (Hanya dalam 1 partisi) | Didukung (Bisa lintas partisi & jaringan) |
| **Tautan ke Folder** | Tidak diizinkan | Diizinkan |
| **Jika File Asli Dihapus** | Data tetap utuh dan terbaca lewat hard link | Link menjadi rusak (*Dangling / Broken Link*) |

---

### Lab 8.1: Membuat dan Menguji Hard Link

```bash
# 1. Buat direktori kerja dan file awal
mkdir -p /tmp/link_lab && cd /tmp/link_lab
echo "Linux Filesystem Lab 8.1" > original.txt

# 2. Buat Hard Link menggunakan perintah 'ln'
ln original.txt hardlink_file.txt

# 3. Periksa nomor Inode dan Link Count (kolom ke-3 pada ls -l)
ls -li original.txt hardlink_file.txt
# Output: Kedua file memiliki nomor Inode yang SAMA dan Link Count bernilai 2

# 4. Uji perubahan isi file (Keduanya akan selalu sinkron)
echo "Penambahan baris baru" >> hardlink_file.txt
cat original.txt

# 5. Hapus file asli dan buktikan bahwa data tetap ada di hardlink
rm original.txt
cat hardlink_file.txt    # Data tetap bisa dibaca dengan sempurna!
```

---

### Lab 8.2: Membuat dan Menguji Soft Link (Symbolic Link)

```bash
# 1. Buat file target baru
echo "Testing Soft Link Lab 8.2" > source.txt

# 2. Buat Soft Link menggunakan opsi flag '-s'
ln -s source.txt softlink_file.txt

# 3. Periksa nomor Inode
ls -li source.txt softlink_file.txt
# Output: Nomor Inode BERBEDA, dan softlink_file.txt menunjuk -> source.txt

# 4. Hapus file sumber dan perhatikan efek broken link
rm source.txt
cat softlink_file.txt    # Error: No such file or directory
ls -l softlink_file.txt  # Link berubah warna merah (Broken Link)
```

---

### Lab 8.3: Membuat Link Antar Direktori & Mencari Hard Link Tersembunyi

```bash
# 1. Membuat Soft Link untuk direktori (Sangat sering digunakan di Nginx / Web server)
mkdir -p /var/www/my_app
ln -s /var/www/my_app /home/ubuntu/app_shortcut

# 2. Mencari seluruh file di filesystem yang berbagi nomor Inode yang sama (Mencari Hardlink)
find / -samefile /tmp/link_lab/hardlink_file.txt 2>/dev/null
# atau mencari berdasarkan nomor Inode spesifik:
find / -inum 1441852 2>/dev/null
```

---

## 9. Navigasi Lanjutan Modul

| Modul Sebelumnya | Indeks Dokumentasi |
| :---: | :---: |
| [07 - I/O Monitoring](./07-io-monitoring.md) | [README.md - Master Index](./README.md) |
