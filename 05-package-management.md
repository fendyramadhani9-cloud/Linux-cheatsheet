# 05 - Package Management Systems

Panduan manajemen paket perangkat lunak di Linux: perbandingan ekosistem Debian/Ubuntu (`APT` & `DPKG`) vs Red Hat/CentOS/Rocky Linux (`DNF`, `YUM`, & `RPM`), pengelolaan repositori dan kunci GPG, hingga tahapan kompilasi source code dari arsip tarball.

---

## Daftar Isi
- [1. Konsep Package Manager & Dua Tingkatan](#1-konsep-package-manager--dua-tingkatan)
- [2. Ekosistem Debian & Ubuntu (APT & DPKG)](#2-ekosistem-debian--ubuntu-apt--dpkg)
- [3. Ekosistem Red Hat, CentOS & Rocky Linux (DNF, YUM, RPM)](#3-ekosistem-red-hat-centos--rocky-linux-dnf-yum-rpm)
- [4. Tabel Padanan Perintah Lengkap (Rosetta Stone APT vs DNF)](#4-tabel-padanan-perintah-lengkap-rosetta-stone-apt-vs-dnf)
- [5. Konfigurasi Repositori & Kunci GPG](#5-konfigurasi-repositori--kunci-gpg)
- [6. Kompilasi Source Code dari Tarball (.tar.gz)](#6-kompilasi-source-code-dari-tarball-targz)
- [7. Navigasi Lanjutan Modul](#7-navigasi-lanjutan-modul)

---

## 1. Konsep Package Manager & Dua Tingkatan

| Tingkatan Package Manager | Contoh Tools | Format Berkas | Peran & Karakteristik Teknis |
| :--- | :--- | :---: | :--- |
| **High-Level** | `apt`, `dnf`, `yum` | - | Mengunduh paket dari server repositori via internet, memeriksa integritas via GPG key, dan otomatis menyelesaikan rantai dependensi yang dibutuhkan. |
| **Low-Level** | `dpkg`, `rpm` | `.deb`, `.rpm` | Menginstal, mengekstrak, dan menghapus berkas paket biner lokal pada filesystem. Tidak mampu mengunduh dari internet dan tidak otomatis menginstal dependensi. |

---

## 2. Ekosistem Debian & Ubuntu (APT & DPKG)

### Perintah High-Level: `apt`

| Perintah | Deskripsi & Kegunaan |
| :--- | :--- |
| `sudo apt update` | Menyinkronkan dan memperbarui indeks daftar paket lokal dari server repositori resmi. |
| `sudo apt upgrade -y` | Mengunduh dan memasang versi terbaru untuk seluruh paket yang terinstal di sistem. |
| `sudo apt install -y <paket>` | Mengunduh dan memasang satu atau beberapa paket baru sekaligus. Contoh: `sudo apt install -y nginx git`. |
| `sudo apt remove <paket>` | Menghapus binary aplikasi, namun tetap mempertahankan file konfigurasinya di `/etc/`. |
| `sudo apt purge <paket>` | Menghapus binary aplikasi beserta seluruh file konfigurasinya secara bersih. |
| `sudo apt autoremove -y` | Menghapus otomatis paket library dependensi lama yang sudah tidak lagi digunakan. |
| `apt search <keyword>` | Mencari ketersediaan paket di repositori berdasarkan nama atau deskripsi. |
| `apt show <paket>` | Menampilkan informasi lengkap paket (versi, ukuran, dependensi, pembuat). |
| `apt list --installed` | Menampilkan seluruh paket yang saat ini terpasang di sistem. |

### Perintah Low-Level: `dpkg`

| Perintah | Opsi / Argumen | Fungsi |
| :--- | :--- | :--- |
| `sudo dpkg -i <file.deb>` | Install Local | Memasang file paket biner `.deb` lokal ke sistem. |
| `sudo apt install -f` | Fix Dependencies | Memperbaiki dependensi yang rusak/kurang setelah eksekusi `dpkg -i`. |
| `dpkg -l` | List All | Menampilkan daftar seluruh paket `.deb` yang terdaftar di database sistem. |
| `sudo dpkg -r <nama_paket>` | Remove | Menghapus paket dari sistem. |
| `dpkg -L <nama_paket>` | List Files | Menampilkan daftar seluruh file dan lokasi instalasi yang dibawa oleh paket. |
| `dpkg -S </path/ke/file>` | Search Origin | Mencari tahu file tertentu di sistem berasal dari paket mana (*Reverse Lookup*). |

---

## 3. Ekosistem Red Hat, CentOS & Rocky Linux (DNF, YUM, RPM)

### Perintah High-Level: `dnf` / `yum`

| Perintah | Deskripsi & Kegunaan |
| :--- | :--- |
| `sudo dnf check-update` | Memeriksa ketersediaan pembaruan paket dari repositori. |
| `sudo dnf upgrade -y` | Meng-upgrade seluruh sistem dan paket yang terpasang ke versi terbaru. |
| `sudo dnf install -y <paket>` | Memasang paket baru berserta seluruh dependensinya. Contoh: `sudo dnf install -y httpd`. |
| `sudo dnf remove <paket>` | Menghapus paket dari sistem. |
| `dnf search <keyword>` | Mencari paket di repositori berdasarkan kata kunci. |
| `dnf info <paket>` | Menampilkan informasi teknis detail mengenai paket. |
| `sudo dnf clean all` | Membersihkan cache metadata dan file unduhan paket lokal. |
| `dnf history` | Menampilkan riwayat transaksi paket yang pernah dilakukan di sistem. |
| `sudo dnf history undo <ID>` | Mengembalikan sistem (*rollback*) sebelum transaksi instalasi tertentu dilakukan. |

### Perintah Low-Level: `rpm`

| Perintah | Opsi / Argumen | Fungsi |
| :--- | :--- | :--- |
| `sudo rpm -ivh <file.rpm>` | Install Verbose Hash | Memasang file paket `.rpm` lokal dengan visual progress bar. |
| `sudo rpm -Uvh <file.rpm>` | Upgrade | Meng-upgrade paket `.rpm` yang sudah ada ke versi lebih baru. |
| `rpm -qa` | Query All | Menampilkan seluruh paket `.rpm` yang terpasang di sistem. |
| `sudo rpm -e <nama_paket>` | Erase / Remove | Menghapus paket dari sistem. |
| `rpm -ql <nama_paket>` | Query List | Menampilkan seluruh file dan path yang dipasang oleh paket tersebut. |
| `rpm -qf </path/ke/file>` | Query File Origin | Mengetahui paket `.rpm` mana yang menyediakan file bersangkutan. |

---

## 4. Tabel Padanan Perintah Lengkap (Rosetta Stone APT vs DNF)

| Operasi Manajemen Paket | Debian / Ubuntu (`APT` / `DPKG`) | RHEL / Rocky / CentOS (`DNF` / `RPM`) |
| :--- | :--- | :--- |
| **Pembaruan Daftar Indeks** | `sudo apt update` | `sudo dnf check-update` |
| **Upgrade Seluruh Sistem** | `sudo apt upgrade` | `sudo dnf upgrade` |
| **Instalasi Paket Baru** | `sudo apt install <paket>` | `sudo dnf install <paket>` |
| **Hapus Paket Standar** | `sudo apt remove <paket>` | `sudo dnf remove <paket>` |
| **Hapus Paket Bersih (Purge)**| `sudo apt purge <paket>` | `sudo dnf remove <paket>` |
| **Pembersihan Dependensi** | `sudo apt autoremove` | `sudo dnf autoremove` |
| **Pencarian di Repositori** | `apt search <keyword>` | `dnf search <keyword>` |
| **Informasi Detail Paket** | `apt show <paket>` | `dnf info <paket>` |
| **Bersihkan Cache Lokal** | `sudo apt clean` | `sudo dnf clean all` |
| **Instal Berkas Paket Lokal** | `sudo dpkg -i berkas.deb` | `sudo rpm -ivh berkas.rpm` |
| **Cari Pemilik Berkas** | `dpkg -S /path/file` | `rpm -qf /path/file` |
| **Daftar File Dalam Paket** | `dpkg -L <nama_paket>` | `rpm -ql <nama_paket>` |

---

## 5. Konfigurasi Repositori & Kunci GPG

| Distro Family | Lokasi File Konfigurasi Repo | Format File |
| :--- | :--- | :--- |
| **Debian / Ubuntu** | `/etc/apt/sources.list` dan `/etc/apt/sources.list.d/*.list` | Format baris teks APT (`deb [options] URL suite components`) |
| **RHEL / Rocky / Fedora** | `/etc/yum.repos.d/*.repo` | Format INI (`[repo-id] name=... baseurl=... gpgcheck=1`) |

### Prosedur Penambahan Repositori Eksternal (Contoh Docker di Ubuntu):

| Langkah | Perintah Eksekusi | Tujuan |
| :---: | :--- | :--- |
| **1** | `curl -fsSL https://download.docker.com/linux/ubuntu/gpg \| sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg` | Mengimpor kunci GPG resmi untuk validasi tanda tangan digital paket. |
| **2** | `echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \| sudo tee /etc/apt/sources.list.d/docker.list` | Menambahkan baris konfigurasi URL repositori ke sistem. |
| **3** | `sudo apt update` | Menyinkronkan katalog paket baru dari repositori yang baru saja ditambahkan. |

---

## 6. Kompilasi Source Code dari Tarball (.tar.gz)

| Tahapan | Perintah Eksekusi | Deskripsi Operasi |
| :---: | :--- | :--- |
| **Persiapan Compiler** | `sudo apt install build-essential` *(Ubuntu)* <br> `sudo dnf groupinstall "Development Tools"` *(RHEL)* | Menginstal GCC compiler, G++, Make, dan pustaka header pengembangan. |
| **Ekstraksi Arsip** | `tar -zxvf software-1.0.tar.gz && cd software-1.0/` | Membongkar file arsip source code ke direktori kerja. |
| **Konfigurasi Lingkungan** | `./configure --prefix=/usr/local` | Memeriksa ketersediaan library sistem dan membuat file `Makefile`. |
| **Kompilasi Biner** | `make -j$(nproc)` | Mengompilasi kode sumber C/C++ menjadi binary menggunakan seluruh core CPU yang tersedia. |
| **Instalasi ke Sistem** | `sudo make install` | Menyalin binary hasil kompilasi ke direktori biner sistem (`/usr/local/bin`). |

---

## 7. Navigasi Lanjutan Modul

| Modul Sebelumnya | Modul Berikutnya |
| :---: | :---: |
| [04 - Linux Signals](./04-linux-signals.md) | [06 - System Monitoring](./06-system-monitoring.md) |
