# 10 - Logical Volume Management (LVM)

Panduan teknis Logical Volume Management (LVM) pada Linux: arsitektur hirarki (PV, VG, LV, PE/LE), utilitas lengkap (`pv*`, `vg*`, `lv*`), prosedur inisialisasi dan konfigurasi storage pool, inspeksi status volume, alokasi snapshot, serta teknik penambahan dan pengurangan kapasitas storage (*resizing & extending*) secara dinamis.

---

## Daftar Isi
- [1. Konsep Dasar & Arsitektur LVM](#1-konsep-dasar--arsitektur-lvm)
- [2. Struktur Volumes & Volume Groups](#2-struktur-volumes--volume-groups)
- [3. Rangkaian Utilitas Perintah LVM (LVM Utilities Suite)](#3-rangkaian-utilitas-perintah-lvm-lvm-utilities-suite)
- [4. Prosedur Pembuatan Logical Volume (Creating Logical Volumes)](#4-prosedur-pembuatan-logical-volume-creating-logical-volumes)
- [5. Inspeksi & Monitoring Status LVM (Displaying Logical Volumes)](#5-inspeksi--monitoring-status-lvm-displaying-logical-volumes)
- [6. Manajemen Perubahan Kapasitas (Resizing & Extending LVM)](#6-manajemen-perubahan-kapasitas-resizing--extending-lvm)
- [7. Manajemen LVM Snapshot & Restore](#7-manajemen-lvm-snapshot--restore)
- [8. Navigasi Lanjutan Modul](#8-navigasi-lanjutan-modul)

---

## 1. Konsep Dasar & Arsitektur LVM

Logical Volume Management (LVM) adalah sistem manajemen penyimpanan tingkat lanjut yang menyediakan lapisan abstraksi antara disk fisik mentah dengan sistem berkas (*filesystem*). Berbeda dengan partisi statis tradisional, LVM memungkinkan penggabungan beberapa media penyimpanan fisik ke dalam satu kumpulan ruang (*storage pool*) elastis yang dapat dialokasikan dan diubah ukurannya secara dinamis tanpa downtime.

### Arsitektur Alur Abstraksi LVM

```text
┌────────────────────────────────────────────────────────┐
│               Titik Mount (/data, /var, /)             │
└───────────────────────────┬────────────────────────────┘
                            │ (Ext4 / XFS Filesystem)
┌───────────────────────────┴────────────────────────────┐
│              Logical Volumes (LV: lv_data)             │
└───────────────────────────┬────────────────────────────┘
                            │ (Logical Extents - LE)
┌───────────────────────────┴────────────────────────────┐
│               Volume Group (VG: vg_storage)            │
│                 [ Total Pool Kapasitas ]               │
└─────────────┬────────────────────────────┬─────────────┘
              │ (Physical Extents - PE)    │
┌─────────────┴─────────────┐┌─────────────┴─────────────┐
│ Physical Volume (/dev/sdb)││ Physical Volume (/dev/sdc)│
└─────────────┬─────────────┘└─────────────┬─────────────┘
              │                            │
┌─────────────┴─────────────┐┌─────────────┴─────────────┐
│    Disk Fisik / Partisi 1 ││    Disk Fisik / Partisi 2 │
└───────────────────────────┘└───────────────────────────┘
```

### Perbandingan: Partisi Konvensional vs LVM

| Parameter Evaluasi | Partisi Tradisional (MBR/GPT) | Logical Volume Management (LVM) |
| :--- | :--- | :--- |
| **Batasan Kapasitas** | Terikat langsung pada batas fisik 1 disk. | Dapat menggabungkan banyak disk fisik (*multi-disk spanning*). |
| **Perubahan Ukuran (*Resize*)** | Sangat berisiko, membutuhkan partisi bersebelahan yang kosong, sering butuh unmount. | Sangat fleksibel, perluasan kapasitas dapat dilakukan secara live/online. |
| **Skalabilitas Disk** | Tidak dapat menambah kapasitas disk baru ke partisi yang sudah ada. | Tinggal menambahkan disk baru sebagai PV ke VG yang sedang berjalan. |
| **Snapshot Backup** | Memerlukan level aplikasi atau filesystem khusus (misal: Btrfs/ZFS). | Didukung secara bawaan (*native LVM snapshot*) tanpa interupsi service. |
| **Abstraksi Penamaan** | Menggunakan path perangkat hardware (`/dev/sda1`, `/dev/nvme0n1p1`). | Menggunakan label logis (`/dev/mapper/vg_system-lv_data`). |

### Komponen Inti LVM

| Komponen | Akronim | Penjelasan Teknis & Peran |
| :--- | :---: | :--- |
| **Physical Volume** | **PV** | Perangkat blok mentah (seperti `/dev/sdb`, `/dev/nvme0n1`, atau partisi `/dev/sdb1`) yang diinisialisasi oleh LVM dengan header metadata khusus. |
| **Volume Group** | **VG** | Kumpulan gabungan dari satu atau lebih PV yang membentuk *storage pool* bersama. Kapasitas VG adalah total akumulasi dari semua PV di dalamnya. |
| **Logical Volume** | **LV** | Partisi virtual yang dialokasikan dari ruang kosong di dalam VG. Di atas LV inilah filesystem (Ext4, XFS) dibuat dan di-mount. |
| **Physical Extent** | **PE** | Blok alokasi terkecil di dalam PV/VG (default: 4 MB). Seluruh alokasi ruang LVM dihitung berdasarkan kelipatan unit PE ini. |
| **Logical Extent** | **LE** | Blok alokasi pada level LV yang dipetakan secara 1-ke-1 ke Physical Extent (PE) di dalam Volume Group. |

---

## 2. Struktur Volumes & Volume Groups

### 1. Physical Volume (PV)
Physical Volume dapat dibuat di atas partisi disk bertipe Linux LVM (`8e` pada MBR, `8e00` pada GPT) atau langsung pada seluruh disk mentah (*raw disk*) tanpa tabel partisi.

### 2. Volume Group (VG) & Ukuran Physical Extent (PE)
Kapasitas VG dibagi ke dalam blok-blok seragam bernama Physical Extents. Ukuran PE menentukan batas maksimum kapasitas Logical Volume:
- **Default PE Size:** 4 MB (Cukup untuk sebagian besar kebutuhan server modern).
- **Kustomisasi PE:** Ukuran PE dapat ditentukan saat inisialisasi VG menggunakan opsi `-s` (misalnya: `vgcreate -s 16M vg_data /dev/sdb`).

### 3. Tipe-Tipe Logical Volume (LV Layouts)

| Tipe Logical Volume | Karakteristik Teknis | Keuntungan | Kasus Penggunaan Ideal |
| :--- | :--- | :--- | :--- |
| **Linear Volume** | Alokasi berurutan dari satu PV ke PV berikutnya dalam VG. | Sederhana, mudah diperluas secara bertahap. | Konfigurasi standar OS, file sharing, web server. |
| **Striped Volume** | Data disebar merata di antara beberapa PV secara paralel (serupa RAID 0). | Meningkatkan throughput baca/tulis (*I/O bandwidth*). | Database berkecepatan tinggi, caching layer, video processing. |
| **Mirrored / RAID Volume**| Data diduplikasi secara identik ke dua atau lebih PV berbeda (RAID 1/5/6/10). | Redundansi tinggi dan perlindungan dari kerusakan disk fisik. | Penyimpanan data misi kritis (*mission-critical storage*). |
| **Thin-Provisioned Volume**| Alokasi ruang disk berbasis kebutuhan nyata saat data ditulis (*over-provisioning*). | Efisiensi penyimpanan maksimal, tidak memboroskan ruang kosong. | Virtualisasi (KVM, Proxmox), Container storage, Cloud multi-tenant. |
| **Snapshot Volume** | Salinan titik waktu (*point-in-time copy*) instan menggunakan mekanisme Copy-on-Write (CoW). | Backup konsisten tanpa downtime database atau file service. | Backup harian, rollback update aplikasi, testing staging. |

---

## 3. Rangkaian Utilitas Perintah LVM (LVM Utilities Suite)

Perintah manajemen LVM terstruktur rapi berdasarkan 3 tingkatan hierarki objek: awalan `pv*` untuk Physical Volume, `vg*` untuk Volume Group, dan `lv*` untuk Logical Volume.

| Operasi / Tindakan | Physical Volume (PV) | Volume Group (VG) | Logical Volume (LV) |
| :--- | :--- | :--- | :--- |
| **Inisialisasi / Buat** | `pvcreate` | `vgcreate` | `lvcreate` |
| **Informasi Ringkas** | `pvs` | `vgs` | `lvs` |
| **Informasi Detail** | `pvdisplay` | `vgdisplay` | `lvdisplay` |
| **Pindai Perangkat** | `pvscan` | `vgscan` | `lvscan` |
| **Perluas Kapasitas** | `pvresize` | `vgextend` | `lvextend` |
| **Kurangi Kapasitas** | - | `vgreduce` | `lvreduce` / `lvresize` |
| **Hapus Objek** | `pvremove` | `vgremove` | `lvremove` |
| **Ubah Nama (*Rename*)**| - | `vgrename` | `lvrename` |
| **Pindah Blok Data** | `pvmove` | - | - |
| **Pemeriksaan / Perbaikan**| `pvck` | `vgck` | - |

### Standar Penamaan Node Perangkat LVM
Setelah LV dibuat, sistem Linux menyediakan 2 tautan path simetris di direktori `/dev`:
1. **Device Mapper Path:** `/dev/mapper/<nama_vg>-<nama_lv>` (Contoh: `/dev/mapper/vg_data-lv_app`)
2. **VG Subdirectory Path:** `/dev/<nama_vg>/<nama_lv>` (Contoh: `/dev/vg_data/lv_app`)

Kedua path ini menunjuk ke block device virtual yang sama melalui subsistem Device Mapper kernel (`/dev/dm-X`).

---

## 4. Prosedur Pembuatan Logical Volume (Creating Logical Volumes)

Alur lengkap inisialisasi media penyimpanan dari disk mentah hingga siap digunakan:

```text
[ Disk: /dev/sdb ] ──(pvcreate)──> [ PV: /dev/sdb ] ──(vgcreate)──> [ VG: vg_data ] ──(lvcreate)──> [ LV: lv_app ] ──(mkfs)──> [ Mount: /app ]
```

### Tabel Langkah Pembuatan Lengkap

| Tahap | Perintah Eksekusi | Deskripsi Teknis |
| :---: | :--- | :--- |
| **1** | `lsblk` | Mengidentifikasi nama perangkat disk baru yang terpasang (misal: `/dev/sdb`). |
| **2** | `sudo pvcreate /dev/sdb` | Menginisialisasi disk `/dev/sdb` menjadi Physical Volume LVM. |
| **3** | `sudo vgcreate vg_data /dev/sdb` | Membuat Volume Group baru bernama `vg_data` dari PV `/dev/sdb`. |
| **4** | `sudo lvcreate -L 20G -n lv_app vg_data` | Membuat Logical Volume bernama `lv_app` berukuran 20 GB dari `vg_data`. |
| **5** | `sudo mkfs.ext4 /dev/vg_data/lv_app` | Memformat Logical Volume dengan filesystem Ext4. |
| **6** | `sudo mkdir -p /app && sudo mount /dev/vg_data/lv_app /app` | Membuat folder mount point dan mengaitkan volume. |
| **7** | `sudo blkid /dev/vg_data/lv_app` | Mengambil UUID volume untuk didaftarkan ke `/etc/fstab` agar persistent. |

### Variasi Opsi Perintah `lvcreate`

| Skenario Alokasi | Perintah Eksekusi | Keterangan Opsi |
| :--- | :--- | :--- |
| **Ukuran Absolut (GB/MB)** | `sudo lvcreate -L 50G -n lv_data vg_storage` | Opsi `-L` menentukan ukuran tepat dalam unit G, M, T. |
| **Seluruh Ruang Kosong (100%)** | `sudo lvcreate -l 100%FREE -n lv_backup vg_storage` | Opsi `-l` menggunakan persentase sisa ruang kosong di VG. |
| **Persentase Total VG** | `sudo lvcreate -l 50%VG -n lv_db vg_storage` | Mengalokasikan 50% dari total seluruh kapasitas VG. |
| **Jumlah Physical Extents** | `sudo lvcreate -l 250 -n lv_small vg_storage` | Mengalokasikan tepat 250 unit PE (250 x 4MB = 1000 MB). |
| **Striped Volume (2 Disk)** | `sudo lvcreate -i 2 -I 64 -L 100G -n lv_stripe vg_storage` | Opsi `-i 2` menyebar data ke 2 PV, `-I 64` ukuran stripe 64 KB. |

---

## 5. Inspeksi & Monitoring Status LVM (Displaying Logical Volumes)

### 1. Perintah Ringkas (*Scan & Summary Report*)

Perintah format ringkas (`pvs`, `vgs`, `lvs`) memberikan output tabular cepat yang ideal untuk monitoring operasional:

| Perintah | Contoh Output Utama | Informasi yang Ditampilkan |
| :--- | :--- | :--- |
| `sudo pvs` | `PV /dev/sdb VG vg_data Fmt lvm2 PSize 100.00g PFree 80.00g` | Status PV, VG induk, format LVM, total ukuran, dan sisa kapasitas bebas. |
| `sudo vgs` | `VG vg_data #PV 1 #LV 1 #SN 0 VSize 100.00g VFree 80.00g` | Jumlah PV, jumlah LV aktif, jumlah snapshot, total ukuran, dan sisa ruang VG. |
| `sudo lvs` | `LV lv_app VG vg_data Attr -wi-a----- LSize 20.00g` | Nama LV, VG asal, atribut status alokasi, dan ukuran volume. |

### 2. Memahami Atribut Status pada `lvs` (`Attr`)
Kolom `Attr` pada perintah `lvs` memiliki 10 karakter kode diagnostik penting:
- **Karakter 1 (Volume Type):** `-` (Linear/Standard), `s` (Snapshot), `t` (Thin Pool), `V` (Thin Volume), `m` (Mirrored), `r` (RAID).
- **Karakter 2 (Permissions):** `w` (Read and Write), `r` (Read Only).
- **Karakter 3 (Allocation Policy):** `i` (Inherited), `c` (Contiguous), `a` (Anywhere).
- **Karakter 5 (State / Active):** `a` (Active), `s` (Suspended), `d` (Device present without tables).

### 3. Perintah Inspeksi Detail (*Verbose Mode*)

| Perintah | Deskripsi Output | Kasus Penggunaan |
| :--- | :--- | :--- |
| `sudo pvdisplay` | Rincian metadata PV: UUID, ukuran PE, jumlah total PE, dan daftar alokasi. | Mengetahui fragmentasi PE pada disk fisik. |
| `sudo vgdisplay` | Rincian metadata VG: UUID, batas maksimum LV/PV, ukuran blok PE (`PE Size`). | Mengetahui detail parameter alokasi VG. |
| `sudo lvdisplay` | Rincian metadata LV: Path `/dev/...`, status akses, jumlah LE, status block device. | Memeriksa integritas pemetaan blok virtual. |
| `sudo lvs -o +devices` | Menampilkan kolom tambahan disk fisik (PV) mana yang menampung LV tersebut. | Mengetahui letak fisik data saat menggunakan multi-disk. |

---

## 6. Manajemen Perubahan Kapasitas (Resizing & Extending LVM)

Kekuatan utama LVM adalah kemampuannya memperluas (*extend*) kapasitas penyimpanan secara langsung (*online*) tanpa mematikan layanan (*zero downtime*).

```text
[ Tambah Disk Baru ] ──(vgextend)──> [ VG Kapasitas Membesar ] ──(lvextend -r)──> [ LV & Filesystem Otomatis Meluas ]
```

### 1. Menambah Kapasitas Volume Group (`vgextend`)
Jika kapasitas VG habis, tambahkan disk fisik baru ke dalam VG yang sudah ada:

| Tahapan | Perintah Eksekusi | Deskripsi Operasi |
| :---: | :--- | :--- |
| **1** | `sudo pvcreate /dev/sdc` | Inisialisasi disk kedua (`/dev/sdc`) sebagai Physical Volume baru. |
| **2** | `sudo vgextend vg_data /dev/sdc` | Memasukkan disk `/dev/sdc` ke dalam Volume Group `vg_data`. |
| **3** | `sudo vgs vg_data` | Memverifikasi bahwa kapasitas total dan ruang bebas VG telah bertambah. |

### 2. Memperluas Logical Volume & Filesystem (Extending)

> [!TIP]
> **Praktik Terbaik:** Gunakan flag `-r` atau `--resizefs` pada `lvextend`. Opsi ini otomatis memperbesar filesystem (Ext4 maupun XFS) secara online tanpa perlu menjalankan perintah resize filesystem secara terpisah.

| Skenario Perluasan | Perintah Satu Langkah (Direkomendasikan) | Perintah Manual 2 Langkah |
| :--- | :--- | :--- |
| **Tambah Ukuran Spesifik (+10G)** | `sudo lvextend -r -L +10G /dev/vg_data/lv_app` | 1. `sudo lvextend -L +10G /dev/vg_data/lv_app`<br>2. *Ext4:* `sudo resize2fs /dev/vg_data/lv_app`<br>2. *XFS:* `sudo xfs_growfs /app` |
| **Gunakan Seluruh Sisa VG (100%)** | `sudo lvextend -r -l +100%FREE /dev/vg_data/lv_app` | 1. `sudo lvextend -l +100%FREE /dev/vg_data/lv_app`<br>2. *Ext4:* `sudo resize2fs /dev/vg_data/lv_app`<br>2. *XFS:* `sudo xfs_growfs /app` |
| **Set Total Target Ukuran (50G)** | `sudo lvextend -r -L 50G /dev/vg_data/lv_app` | 1. `sudo lvextend -L 50G /dev/vg_data/lv_app`<br>2. *Ext4:* `sudo resize2fs /dev/vg_data/lv_app`<br>2. *XFS:* `sudo xfs_growfs /app` |

### 3. Mengurangi Kapasitas Logical Volume (Shrinking / Reducing)

> [!CAUTION]
> **Peringatan Kritis Data:**
> 1. **XFS TIDAK MENDUKUNG PENGURANGAN UKURAN:** Filesystem XFS secara arsitektural tidak dapat diperkecil (*cannot be shrunk*). Pengecilan hanya bisa dilakukan pada filesystem Ext2/Ext3/Ext4.
> 2. **Wajib Unmount & Backup:** Pengecilan filesystem TIDAK BISA dilakukan secara online. Partisi wajib di-unmount terlebih dahulu untuk mencegah kerusakan struktur data.

Urutan langkah aman pengecilan LV Ext4:

| Urutan | Perintah Eksekusi | Deskripsi Tindakan Wajib |
| :---: | :--- | :--- |
| **1** | `sudo umount /mnt/data` | Melepaskan kaitan filesystem dari sistem (*unmount*). |
| **2** | `sudo e2fsck -f /dev/vg_data/lv_data` | Memeriksa dan membersihkan integritas filesystem secara paksa (*wajib lulus*). |
| **3** | `sudo resize2fs /dev/vg_data/lv_data 10G` | Memperkecil filesystem Ext4 terlebih dahulu ke batas target ukuran (misal: 10 GB). |
| **4** | `sudo lvreduce -L 10G /dev/vg_data/lv_data` | Memperkecil batas Logical Volume tepat sama atau lebih besar dari filesystem. |
| **5** | `sudo mount /dev/vg_data/lv_data /mnt/data` | Mengaitkan kembali filesystem yang telah diperkecil. |

---

## 7. Manajemen LVM Snapshot & Restore

Snapshot LVM mencatat status volume pada suatu titik waktu menggunakan mekanisme Copy-on-Write (CoW). Snapshot hanya menggunakan ruang disk saat data pada volume asli mengalami perubahan.

| Operasi Snapshot | Perintah Eksekusi | Penjelasan & Skenario |
| :--- | :--- | :--- |
| **Buat Snapshot** | `sudo lvcreate -L 5G -s -n lv_app_snap /dev/vg_data/lv_app` | Membuat snapshot berkapasitas 5 GB bernama `lv_app_snap` untuk volume sumber `lv_app`. |
| **Mount Snapshot (Read-Only)** | `sudo mount -o ro /dev/vg_data/lv_app_snap /mnt/snapshot` | Membaca dan mengambil file cadangan dari kondisi snapshot tanpa mengubah isinya. |
| **Mount Snapshot XFS** | `sudo mount -o nouuid,ro /dev/vg_data/lv_app_snap /mnt/snapshot` | *Khusus XFS:* Wajib opsi `nouuid` karena XFS melarang mount dua volume dengan UUID identik. |
| **Pantau Konsumsi Ruang** | `sudo lvs /dev/vg_data/lv_app_snap` | Memeriksa kolom `Data%`. Jika mencapai 100%, snapshot akan rusak (*dropped*). |
| **Kembalikan Volume (*Restore*)** | `sudo lvconvert --merge /dev/vg_data/lv_app_snap` | Mengembalikan status volume `lv_app` ke titik snapshot (volume wajib unmounted / reboot). |
| **Hapus Snapshot** | `sudo lvremove -y /dev/vg_data/lv_app_snap` | Menghapus snapshot setelah proses backup atau pemeliharaan selesai. |

---

## 8. Navigasi Lanjutan Modul

| Modul Sebelumnya | Indeks Dokumentasi |
| :---: | :---: |
| [09 - Linux Filesystem Features & Storage Management](./09-filesystem-features-and-storage-management.md) | [README.md - Master Index](./README.md) |
