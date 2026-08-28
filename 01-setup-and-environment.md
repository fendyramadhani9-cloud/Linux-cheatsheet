# 01 - Setup & Environment Linux

Panduan fundamental sistem operasi Linux, konfigurasi environment, navigasi command line interface (CLI), variabel, alias, dan manajemen riwayat perintah.

---

## Daftar Isi
- [1. Konsep Dasar Linux & Shell](#1-konsep-dasar-linux--shell)
- [2. Shortcut Navigasi Keyboard Terminal](#2-shortcut-navigasi-keyboard-terminal)
- [3. Perintah Identifikasi Sistem Dasar](#3-perintah-identifikasi-sistem-dasar)
- [4. Environment Variables](#4-environment-variables)
- [5. Manajemen Alias](#5-manajemen-alias)
- [6. Riwayat Perintah (History Management)](#6-riwayat-perintah-history-management)
- [7. Navigasi Lanjutan Modul](#7-navigasi-lanjutan-modul)

---

## 1. Konsep Dasar Linux & Shell

| Istilah | Definisi | Keterangan |
| :--- | :--- | :--- |
| **Linux** | Kernel sistem operasi open-source | Dibuat oleh Linus Torvalds pada tahun 1991, menjadi inti dari distro seperti Ubuntu, Debian, CentOS, RHEL, dan Rocky Linux. |
| **Shell** | Command Language Interpreter | Program perantara antara pengguna dan Kernel Linux. |
| **Bash** | *Bourne Again Shell* | Shell standar bawaan di mayoritas distribusi Linux server. |
| **Zsh / Fish** | *Z Shell / Friendly Interactive Shell* | Shell alternatif modern yang sering digunakan pada workstation lokal. |

---

## 2. Shortcut Navigasi Keyboard Terminal

| Shortcut | Kategori | Fungsi Utama | Penjelasan Detail |
| :--- | :--- | :--- | :--- |
| `Ctrl + C` | Kontrol Eksekusi | Cancel / Interrupt | Mengirim sinyal SIGINT untuk membatalkan proses aktif di terminal saat itu. |
| `Ctrl + Z` | Kontrol Eksekusi | Suspend / Pause | Menghentikan sementara proses aktif dan memindahkannya ke background queue. |
| `Ctrl + L` | Tampilan | Clear Screen | Membersihkan tampilan layar terminal (ekuivalen dengan perintah `clear`). |
| `Ctrl + D` | Sesi Terminal | Exit / EOF | Mengirim sinyal End-Of-File untuk keluar dari sesi shell atau menutup koneksi SSH. |
| `Ctrl + A` | Kursor | Jump to Beginning | Memindahkan posisi kursor langsung ke karakter paling awal baris perintah. |
| `Ctrl + E` | Kursor | Jump to End | Memindahkan posisi kursor langsung ke karakter paling akhir baris perintah. |
| `Ctrl + U` | Penghapusan | Cut Line Before | Menghapus seluruh teks dari posisi kursor mundur hingga awal baris. |
| `Ctrl + K` | Penghapusan | Cut Line After | Menghapus seluruh teks dari posisi kursor maju hingga akhir baris. |
| `Ctrl + W` | Penghapusan | Delete Word | Menghapus 1 kata tepat di sebelah kiri kursor. |
| `Ctrl + R` | Pencarian | Reverse History Search | Membuka antarmuka pencarian riwayat perintah lama secara interaktif. |
| `Tab` | Produktivitas | Auto-Complete | Melengkapi nama perintah, path file, atau folder secara otomatis. |
| `Tab + Tab` | Produktivitas | List Options | Menampilkan daftar seluruh opsi pelengkap jika terdapat lebih dari satu pilihan. |

---

## 3. Perintah Identifikasi Sistem Dasar

| Perintah | Deskripsi | Contoh Output |
| :--- | :--- | :--- |
| `whoami` | Menampilkan username yang sedang aktif digunakan | `ubuntu` / `root` |
| `id` | Menampilkan UID, GID, dan grup yang diikuti user | `uid=1000(ubuntu) gid=1000(ubuntu) groups=1000(ubuntu),27(sudo)` |
| `hostname` | Menampilkan nama host mesin saat ini | `server-prod-01` |
| `hostname -I` | Menampilkan seluruh IP Address internal mesin | `192.168.1.50 172.17.0.1` |
| `uname -a` | Menampilkan informasi lengkap kernel dan arsitektur OS | `Linux srv1 5.15.0-101-generic x86_64 GNU/Linux` |
| `cat /etc/os-release` | Menampilkan rincian versi distribusi Linux | `NAME="Ubuntu", VERSION="22.04.4 LTS (Jammy Jellyfish)"` |
| `echo $SHELL` | Menampilkan path binary shell yang aktif | `/bin/bash` |
| `cat /etc/shells` | Menampilkan daftar seluruh shell yang terpasang di sistem | `/bin/sh`, `/bin/bash`, `/usr/bin/zsh` |

---

## 4. Environment Variables

### Variabel Bawaan Sistem (Standard Environment Variables)

| Variabel | Definisi | Contoh Nilai |
| :--- | :--- | :--- |
| `$USER` | Nama pengguna yang sedang login ke sesi shell | `ubuntu` |
| `$HOME` | Path absolut direktori home pengguna | `/home/ubuntu` atau `/root` |
| `$PATH` | Daftar direktori berisi executable yang dicari otomatis oleh shell | `/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin` |
| `$PWD` | Path direktori kerja saat ini (*Print Working Directory*) | `/var/www/html` |
| `$SHELL` | Lokasi program shell yang sedang dijalankan | `/bin/bash` |
| `$$` | Process ID (PID) dari proses shell aktif | `2415` |
| `$?` | Status kode keluar dari perintah terakhir (`0` = Sukses, `1-255` = Gagal) | `0` |

### Perintah Pengelolaan Variabel

| Perintah / Sintaks | Kategori | Fungsi & Contoh |
| :--- | :--- | :--- |
| `printenv` atau `env` | Inspeksi | Menampilkan seluruh environment variable global yang aktif. |
| `echo $VARIABLE_NAME` | Inspeksi | Menampilkan nilai dari variabel tertentu. Contoh: `echo $PATH`. |
| `VAR_NAME="value"` | Deklarasi Lokal | Membuat variabel yang hanya berlaku di dalam shell sesi saat ini (tidak diwariskan ke subproses). |
| `export VAR_NAME="value"` | Deklarasi Global | Mengekspor variabel menjadi environment variable sehingga dapat diakses oleh sub-shell/skrip anak. |
| `unset VAR_NAME` | Penghapusan | Menghapus variabel dari sesi shell. Contoh: `unset VAR_NAME`. |
| `export PATH=$PATH:/custom/bin` | Modifikasi PATH | Menambahkan direktori baru ke dalam pencarian perintah sistem. |

> [!NOTE]
> **Penyimpanan Permanen Variabel:**
> Untuk menyimpan variabel agar tetap aktif setelah reboot atau login ulang, tambahkan baris `export NAMA_VAR="nilai"` ke dalam file `~/.bashrc` (untuk pengguna lokal) atau `/etc/environment` (untuk seluruh pengguna sistem), kemudian jalankan `source ~/.bashrc`.

---

## 5. Manajemen Alias

Alias berfungsi sebagai singkatan untuk perintah yang panjang atau kompleks.

| Perintah / Sintaks | Fungsi | Contoh Penggunaan |
| :--- | :--- | :--- |
| `alias` | Menampilkan daftar seluruh alias yang aktif | `alias` |
| `alias name='command'` | Membuat alias sementara untuk sesi aktif | `alias ll='ls -lah'` |
| `alias ports='ss -tulpn'` | Membuat shortcut pengecekan port | `alias ports='sudo ss -tulpn'` |
| `alias update='apt update && apt upgrade'` | Shortcut update sistem | `alias update='sudo apt update && sudo apt upgrade -y'` |
| `unalias name` | Menghapus alias dari memori sesi aktif | `unalias ll` |

### Tabel Konfigurasi File Profil Shell

| File Konfigurasi | Lingkup Penerapan | Kapan Dimuat oleh Sistem? |
| :--- | :--- | :--- |
| `~/.bashrc` | Khusus pengguna bersangkutan | Dimuat saat membuka interactive non-login shell (membuka terminal baru). |
| `~/.bash_profile` / `~/.profile` | Khusus pengguna bersangkutan | Dimuat satu kali saat login pertama via console atau SSH. |
| `/etc/bash.bashrc` | Seluruh pengguna sistem | Dimuat setiap kali terminal interaktif dibuka oleh user siapa pun. |
| `/etc/profile` | Seluruh pengguna sistem | Dimuat saat login awal oleh seluruh user. |

---

## 6. Riwayat Perintah (History Management)

| Perintah | Format / Opsi | Fungsi & Penjelasan |
| :--- | :--- | :--- |
| `history` | Default | Menampilkan seluruh riwayat perintah yang tersimpan di memori sesi saat ini. |
| `history <N>` | `history 15` | Menampilkan N baris perintah terakhir. |
| `history \| grep "<kata>"` | `history \| grep "docker"` | Mencari baris perintah masa lalu yang mengandung kata kunci tertentu. |
| `!<nomor>` | `!142` | Menjalankan kembali perintah pada baris nomor tertentu di daftar history. |
| `!!` | Eksekusi Instan | Menjalankan kembali perintah yang tepat satu langkah sebelum perintah ini. |
| `sudo !!` | Eskalasi Hak | Menjalankan kembali perintah terakhir dengan menambahkan hak akses `sudo` di depannya. |
| `!$` | Argumen Terakhir | Mengambil argumen terakhir dari perintah sebelumnya. Contoh: `mkdir /var/log/app && cd !$`. |
| `history -c` | Pembersihan | Menghapus riwayat history di sesi aktif saat ini. |
| `history -w` | Sinkronisasi Disk | Menulis paksa riwayat sesi aktif langsung ke file `~/.bash_history`. |

---

## 7. Navigasi Lanjutan Modul

| Modul Sebelumnya | Modul Berikutnya |
| :---: | :---: |
| - | [02 - Filesystem Tree Layout](./02-filesystem-tree-layout.md) |
