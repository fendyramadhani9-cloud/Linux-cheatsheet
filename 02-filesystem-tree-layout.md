# 02 - Linux Filesystem Tree Layout & File Operations

Panduan struktur hierarki sistem file Linux (Filesystem Hierarchy Standard / FHS), manipulasi file dan direktori, sistem izin (*permissions*), kepemilikan (*ownership*), tautan file (*hard link & soft link*), serta teknik pencarian file.

---

## Daftar Isi
- [1. Standar Struktur Direktori Linux (FHS)](#1-standar-struktur-direktori-linux-fhs)
- [2. Navigasi & Simbol Path](#2-navigasi--simbol-path)
- [3. Operasi Manipulasi File & Direktori](#3-operasi-manipulasi-file--direktori)
- [4. Pembacaan & Inspeksi Konten File](#4-pembacaan--inspeksi-konten-file)
- [5. Hak Akses & Kepemilikan (Permissions & Ownership)](#5-hak-akses--kepemilikan-permissions--ownership)
- [6. Hak Akses Khusus (Special Permissions)](#6-hak-akses-khusus-special-permissions)
- [7. Perbandingan Hard Link vs Symbolic Link](#7-perbandingan-hard-link-vs-symbolic-link)
- [8. Utilitas Pencarian File (find, locate, which, whereis)](#8-utilitas-pencarian-file-find-locate-which-whereis)
- [9. Navigasi Lanjutan Modul](#9-navigasi-lanjutan-modul)

---

## 1. Standar Struktur Direktori Linux (FHS)

Linux menyatukan seluruh media penyimpanan di bawah satu akar utama yaitu Root Directory (`/`).

| Direktori | Nama Kepanjangan / Tipe | Fungsi & Deskripsi Teknis |
| :--- | :--- | :--- |
| `/` | Root | Puncak tertinggi hierarki filesystem; semua direktori dan mount point berada di bawahnya. |
| `/bin` | Essential User Binaries | Program executable dasar yang dibutuhkan seluruh user dalam mode single maupun multi-user (`ls`, `cp`, `cat`, `bash`). Modern Linux biasanya membuat symlink ke `/usr/bin`. |
| `/sbin` | System Binaries | Program executable esensial untuk administrasi sistem dan root (`iptables`, `fdisk`, `reboot`, `ip`). Modern Linux biasanya symlink ke `/usr/sbin`. |
| `/etc` | Editable Text Configurations | Berisi seluruh file konfigurasi sistem dan layanan (seperti `/etc/nginx`, `/etc/passwd`, `/etc/hosts`, `/etc/ssh/sshd_config`). Tidak boleh berisi binary file. |
| `/home` | User Home Directories | Tempat penyimpanan data pribadi dan profil pengguna biasa (misalnya `/home/budi/`). |
| `/root` | Superuser Home | Direktori home khusus untuk akun pengguna `root` (bukan di `/home/root`). |
| `/var` | Variable Data | Data dinamis yang ukurannya terus bertambah, seperti log (`/var/log`), spool mail, database files, dan web files (`/var/www`). |
| `/tmp` | Temporary Files | Tempat penyimpanan file sementara yang dapat diakses oleh seluruh proses/user. Umumnya dibersihkan otomatis saat reboot. |
| `/dev` | Device Nodes | File representasi antarmuka perangkat keras (misal disk `/dev/sda`, terminal `/dev/pts/0`, null device `/dev/null`). |
| `/proc` | Process & Kernel Info | Pseudo/virtual filesystem di memori RAM yang berisi metadata kernel dan proses yang sedang berjalan. Ukurannya di disk adalah 0 byte. |
| `/sys` | System / Hardware Info | Virtual filesystem yang mengekspos struktur objek perangkat keras dan driver dari kernel. |
| `/opt` | Optional Add-on Software | Tempat instalasi aplikasi pihak ketiga yang berdiri sendiri (*self-contained application bundles* seperti Google Chrome, Docker runtime). |
| `/usr` | User System Resources | Berisi aplikasi sekunder, pustaka (*libraries*), dan dokumentasi (`/usr/bin`, `/usr/lib`, `/usr/share/man`, `/usr/local`). |
| `/boot` | Static Bootloader Files | File-file krusial yang dibutuhkan saat proses booting (kernel `vmlinuz`, `initramfs`, konfigurasi GRUB). |
| `/mnt` | Mount Point | Lokasi standar untuk me-mount filesystem penyimpanan secara manual dan sementara oleh sysadmin. |
| `/media` | Removable Media | Titik mount otomatis untuk perangkat lepas-pasang seperti USB drive atau CD-ROM. |
| `/srv` | Service Data | Data spesifik dari layanan yang disediakan oleh server (misal FTP server data atau data spesifik web service). |

---

## 2. Navigasi & Simbol Path

| Simbol / Notasi | Arti | Contoh Penggunaan | Penjelasan |
| :--- | :--- | :--- | :--- |
| `/` | Root Directory | `cd /etc` | Mengarah langsung ke jalur mutlak (*Absolute Path*) dari pangkal root. |
| `.` | Current Directory | `./app.sh` | Menunjuk ke direktori tempat pengguna berada saat ini. |
| `..` | Parent Directory | `cd ..` atau `cd ../../` | Menunjuk ke direktori tepat satu tingkat di atas direktori aktif. |
| `~` | User Home | `cd ~` | Menunjuk ke home direktori pengguna aktif saat ini. |
| `-` | Previous Working Dir | `cd -` | Berpindah kembali ke direktori kerja yang baru saja ditinggalkan. |
| `pwd` | Print Working Dir | `pwd` | Menampilkan path absolut lengkap dari direktori saat ini. |

---

## 3. Operasi Manipulasi File & Direktori

### Daftar Opsi Perintah `ls` (List Directory Contents)

| Perintah / Opsi | Fungsi | Keterangan |
| :--- | :--- | :--- |
| `ls` | Menampilkan nama file | Tampilan ringkas standar. |
| `ls -l` | Format panjang (*long format*) | Menampilkan izin akses, jumlah link, owner, group, ukuran byte, dan waktu modifikasi. |
| `ls -a` | Menampilkan semua file (*all*) | Menyertakan hidden files (file berawalan tanda titik `.`). |
| `ls -lah` | Long format + all + human readable | Menampilkan ukuran file dalam satuan KB, MB, atau GB. |
| `ls -lt` | Urutkan berdasarkan waktu | Menampilkan file dengan modifikasi terbaru di urutan paling atas. |
| `ls -lS` | Urutkan berdasarkan ukuran | Menampilkan file dengan ukuran terbesar di urutan paling atas. |
| `ls -lR` | Tampilan rekursif | Menampilkan seluruh isi folder beserta subdirektorinya ke bawah. |

### Operasi File dan Direktori

| Perintah | Opsi Umum | Deskripsi & Contoh |
| :--- | :--- | :--- |
| `mkdir <dir>` | `-p` | Membuat direktori baru. Opsi `-p` membuat folder bertingkat sekaligus. Contoh: `mkdir -p app/src/utils`. |
| `touch <file>` | - | Membuat file kosong baru atau memperbarui timestamp modifikasi file yang sudah ada. Contoh: `touch app.js`. |
| `cp <src> <dst>` | `-r`, `-p`, `-i` | Menyalin file/folder. `-r` (rekursif untuk folder), `-p` (pertahankan ownership/permission), `-i` (konfirmasi overwrite). Contoh: `cp -rp /var/www /backup/www`. |
| `mv <src> <dst>` | `-i`, `-f` | Memindahkan file/folder atau mengganti nama (*rename*). Contoh: `mv server.js app.js`. |
| `rm <target>` | `-r`, `-f`, `-rf` | Menghapus file/folder. `-r` (rekursif folder), `-f` (paksa tanpa konfirmasi). Contoh: `rm -rf /tmp/build_cache`. |
| `rmdir <dir>` | - | Menghapus direktori yang sudah dalam kondisi benar-benar kosong. |

---

## 4. Pembacaan & Inspeksi Konten File

| Perintah | Opsi Utama | Karakteristik & Kegunaan |
| :--- | :--- | :--- |
| `cat <file>` | `-n` | Menampilkan seluruh isi file sekaligus ke layar terminal. Opsi `-n` menampilkan nomor baris. |
| `tac <file>` | - | Menampilkan seluruh isi file secara terbalik (baris terakhir dicetak lebih dulu). |
| `less <file>` | `/keyword`, `G`, `g`, `q` | Pager interaktif untuk membaca file berukuran besar. Tekan `/` untuk cari kata, `G` ke akhir, `g` ke awal, `q` untuk keluar. |
| `head <file>` | `-n <angka>` | Menampilkan baris awal file. Contoh: `head -n 20 /var/log/syslog` (20 baris pertama). |
| `tail <file>` | `-n <angka>`, `-f` | Menampilkan baris akhir file. Opsi `-f` (*follow*) memantau penambahan log secara langsung (*live streaming*). |
| `wc <file>` | `-l`, `-w`, `-c` | Menghitung jumlah baris (`-l`), kata (`-w`), atau karakter/byte (`-c`) di dalam file. |

---

## 5. Hak Akses & Kepemilikan (Permissions & Ownership)

### Format Representasi Izin (`ls -l`)

Contoh: `-rwxr-xr-- 1 budi devteam 4096 Aug 28 10:00 app.sh`

| Komponen | Karakter | Penjelasan |
| :--- | :--- | :--- |
| **Tipe File** | `-` | Tipe objek: `-` (File biasa), `d` (Direktori), `l` (Symbolic Link), `c` (Character device), `b` (Block device). |
| **User (Owner)** | `rwx` | Izin untuk pemilik file: Read (r), Write (w), Execute (x). |
| **Group** | `r-x` | Izin untuk anggota grup pemilik: Read (r), No-Write (-), Execute (x). |
| **Others** | `r--` | Izin untuk seluruh pengguna lain di sistem: Read-only (r--). |

### Tabel Nilai Numerik / Octal Permission

| Simbol | Hak Akses | Nilai Octal | Efek Terhadap File | Efek Terhadap Direktori |
| :---: | :--- | :---: | :--- | :--- |
| `r` | Read | **4** | Melihat dan membaca konten file | Melihat daftar nama file di dalam direktori (`ls`) |
| `w` | Write | **2** | Memodifikasi atau menulis konten file | Menambah, mengubah nama, atau menghapus file dalam direktori |
| `x` | Execute | **1** | Menjalankan file sebagai program/script | Masuk dan membuka direktori (`cd`) serta mengakses file di dalamnya |
| `-` | No Access | **0** | Tidak memiliki izin | Tidak memiliki izin |

### Contoh Penerapan `chmod` & `chown`

| Perintah | Kategori | Penjelasan Operasi |
| :--- | :--- | :--- |
| `chmod 755 script.sh` | Numeric Mode | Owner: `rwx` (7), Group: `r-x` (5), Others: `r-x` (5). Standar untuk executable. |
| `chmod 644 config.conf` | Numeric Mode | Owner: `rw-` (6), Group: `r--` (4), Others: `r--` (4). Standar untuk file data/konfigurasi. |
| `chmod 600 id_rsa` | Numeric Mode | Owner: `rw-` (6), Group: `---` (0), Others: `---` (0). Standar aman untuk private key SSH. |
| `chmod u+x run.sh` | Symbolic Mode | Menambahkan hak eksekusi hanya kepada User/Owner. |
| `chmod g-w file.txt` | Symbolic Mode | Mencabut hak penulisan dari Group. |
| `chmod -R 750 /opt/app` | Rekursif | Menerapkan hak akses 750 ke seluruh folder dan sub-isinya. |
| `chown user file.txt` | Ownership | Mengubah pemilik file menjadi `user`. |
| `chown user:group file.txt` | Ownership | Mengubah pemilik dan grup file secara bersamaan. |
| `chown -R www-data:www-data /var/www` | Ownership Rekursif | Mengubah kepemilikan seluruh folder web ke user dan group `www-data`. |
| `chgrp devteam project/` | Group Ownership | Mengubah group direktori menjadi `devteam`. |

---

## 6. Hak Akses Khusus (Special Permissions)

| Tipe Izin | Nilai Numerik | Representasi Simbol | Lokasi Karakter | Efek & Fungsi Teknis |
| :--- | :---: | :---: | :---: | :--- |
| **SUID** (*Set User ID*) | `4000` | `s` | Bagian User (`-rwsr-xr-x`) | Program dieksekusi dengan hak akses pemilik file (misal `root`), bukan hak akses pengguna yang menjalankannya. Contoh: `/usr/bin/passwd`. |
| **SGID** (*Set Group ID*) | `2000` | `s` | Bagian Group (`drwxrwsr-x`) | Pada direktori: Setiap file baru yang dibuat di dalamnya otomatis mewarisi Group dari folder induk tersebut (ideal untuk shared directory tim). |
| **Sticky Bit** | `1000` | `t` | Bagian Others (`drwxrwxrwt`) | Pada direktori: Semua user dapat membuat file, tetapi hanya pemilik file atau root yang berhak menghapus file tersebut. Contoh: `/tmp`. |

### Perintah Konfigurasi Special Permissions

| Perintah | Deskripsi |
| :--- | :--- |
| `chmod u+s /usr/local/bin/tool` atau `chmod 4755 /usr/local/bin/tool` | Menetapkan SUID |
| `chmod g+s /shared/team_dir` atau `chmod 2775 /shared/team_dir` | Menetapkan SGID pada shared folder |
| `chmod +t /shared/public_dir` atau `chmod 1777 /shared/public_dir` | Menetapkan Sticky Bit |

---

## 7. Perbandingan Hard Link vs Symbolic Link

| Parameter | Hard Link | Symbolic Link (Soft Link) |
| :--- | :--- | :--- |
| **Definisi** | Pointer tambahan langsung ke Inode fisik yang sama | File pointer terpisah yang menyimpan teks alamat path file target |
| **Nomor Inode** | Sama persis dengan file target | Memiliki nomor Inode baru yang berbeda |
| **Jika File Asli Dihapus** | Data tetap aman dan dapat dibaca penuh melalui hard link | Link menjadi rusak (*broken / dangling link*) |
| **Lintas Filesystem/Partisi** | Tidak didukung (harus dalam 1 partisi yang sama) | Didukung sepenuhnya |
| **Link ke Direktori/Folder** | Tidak didukung oleh sistem operasi | Didukung sepenuhnya |
| **Sintaks Perintah** | `ln file_asli.txt hardlink.txt` | `ln -s /path/file_asli /path/softlink` |

---

## 8. Utilitas Pencarian File (find, locate, which, whereis)

### Parameter Perintah `find` (Pencarian Real-time di Disk)

| Parameter / Flag | Contoh Perintah | Penjelasan |
| :--- | :--- | :--- |
| `-name` | `find /var/log -name "*.log"` | Mencari file berdasarkan pola nama (Case-Sensitive). |
| `-iname` | `find . -iname "readme.md"` | Mencari file berdasarkan nama (Case-Insensitive / huruf besar-kecil diabaikan). |
| `-type` | `find /etc -type d -name "nginx*"` | Memfilter tipe objek: `f` (file), `d` (direktori), `l` (link). |
| `-size` | `find /var -size +100M` | Mencari file berukuran lebih besar dari 100 MB (`+` lebih besar, `-` lebih kecil). |
| `-mtime` | `find /var/log -mtime -7` | Mencari file yang dimodifikasi dalam kurun waktu 7 hari terakhir. |
| `-perm` | `find /var/www -type f -perm 0777` | Mencari file dengan permission persis 777. |
| `-user` | `find /home -user ubuntu` | Mencari seluruh file yang dimiliki oleh user `ubuntu`. |
| `-exec` | `find /tmp -name "*.tmp" -exec rm -f {} \;` | Menjalankan perintah tertentu pada setiap file hasil temuan. |

### Perbandingan Utilitas Pencarian Cepat

| Perintah | Metode Pencarian | Kecepatan | Keterangan & Contoh |
| :--- | :--- | :--- | :--- |
| `locate <nama>` | Database indeks (`/var/lib/mlocate`) | Sangat Cepat | Memerlukan update berkala dengan perintah `sudo updatedb`. |
| `which <perintah>` | Menelusuri direktori di `$PATH` | Instan | Menampilkan letak file binary yang dieksekusi sistem. Contoh: `which nginx`. |
| `whereis <nama>` | Direktori standar binary, man, source | Instan | Menampilkan path binary, file manual (*man page*), dan source code program. |
| `type <nama>` | Internal Shell | Instan | Menentukan apakah perintah adalah alias, built-in shell, fungsi, atau binary eksternal. |

---

## 9. Navigasi Lanjutan Modul

| Modul Sebelumnya | Modul Berikutnya |
| :---: | :---: |
| [01 - Setup & Environment](./01-setup-and-environment.md) | [03 - Process Management](./03-process-management.md) |
