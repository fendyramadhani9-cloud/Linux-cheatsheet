# 06 - Linux System & Performance Monitoring

Panduan monitoring kondisi kesehatan dan performa server Linux: analisis pemanfaatan CPU dan Load Average, kapasitas RAM dan Swap, bottleneck I/O disk, koneksi socket/jaringan, serta inspeksi log kernel dan systemd.

---

## Daftar Isi
- [1. Kerangka Analisis Resource (USE Method)](#1-kerangka-analisis-resource-use-method)
- [2. Monitoring CPU & Load Average](#2-monitoring-cpu--load-average)
- [3. Monitoring Memori / RAM & Swap](#3-monitoring-memori--ram--swap)
- [4. Monitoring Penyimpanan & Disk I/O](#4-monitoring-penyimpanan--disk-io)
- [5. Monitoring Jaringan & Port Aktif](#5-monitoring-jaringan--port-aktif)
- [6. Inspeksi Log Sistem & Kernel](#6-inspeksi-log-sistem--kernel)
- [7. Checklist Investigasi Insiden 60 Detik](#7-checklist-investigasi-insiden-60-detik)
- [8. Navigasi Lanjutan Modul](#8-navigasi-lanjutan-modul)

---

## 1. Kerangka Analisis Resource (USE Method)

| Pilar Resource | Parameter yang Diukur | Alat Monitoring Utama |
| :--- | :--- | :--- |
| **CPU** | Load Average, Utilisasi User/System, Menunggu I/O (%wa) | `uptime`, `top`, `htop`, `mpstat` |
| **RAM (Memory)** | Kapasitas Tersedia (*available*), Buff/Cache, Swap Activity | `free -h`, `vmstat` |
| **Storage / Disk** | Kapasitas Partisi (GB), Ketersediaan Inode, I/O Saturation (%util) | `df -h`, `df -i`, `du`, `iostat`, `ncdu` |
| **Network** | Port Listening, State Koneksi TCP, Pemakaian Bandwidth | `ss -tulpn`, `lsof -i`, `ip -s link`, `iftop` |

---

## 2. Monitoring CPU & Load Average

### Evaluasi Load Average (`uptime`)

| Parameter Output `uptime` | Penjelasan Nilai | Interpretasi Kondisi Server |
| :--- | :--- | :--- |
| **Load 1 Menit** | Rata-rata antrean proses dalam 1 menit terakhir | Jika bernilai sama dengan jumlah CPU Core (`nproc`), utilisasi tepat 100%. |
| **Load 5 Menit** | Rata-rata antrean proses dalam 5 menit terakhir | Menunjukkan tren beban jangka pendek. |
| **Load 15 Menit** | Rata-rata antrean proses dalam 15 menit terakhir | Menunjukkan stabilitas beban jangka panjang. |

### Matriks Metrik CPU pada Perintah `top`

| Kolom Metrik | Nama Asli | Definisi Teknis | Ambang Batas Waspada |
| :---: | :--- | :--- | :--- |
| **`us`** | User CPU Time | Beban CPU yang dialokasikan untuk aplikasi pengguna biasa (Node.js, Nginx, Python, MySQL). | Tinggi saat ada proses komputasi berat aplikasi. |
| **`sy`** | System CPU Time | Beban CPU yang digunakan oleh Kernel sistem operasi. | Jika tinggi (>30%), ada terlalu banyak syscall / context switching. |
| **`id`** | Idle Time | Persentase waktu CPU yang sedang menganggur / bebas tugas. | Semakin tinggi nilainya semakin stabil sistem. |
| **`wa`** | IO Wait | Waktu CPU terhenti menunggu proses baca/tulis disk selesai. | **Waspada jika > 15-20%**, pertanda terjadi bottleneck lambatnya hard disk. |
| **`st`** | Steal Time | Persentase CPU yang diambil oleh hypervisor pada lingkungan cloud/VPS untuk VM tetangga. | Jika > 5%, hubungi penyedia cloud terkait noisy neighbor. |

### Shortcut Navigasi Interaktif dalam `top` & `htop`

| Tombol Shortcut | Aplikasi | Fungsi Operasi |
| :---: | :---: | :--- |
| `P` | `top` | Mengurutkan daftar proses berdasarkan penggunaan CPU terbesar. |
| `M` | `top` | Mengurutkan daftar proses berdasarkan penggunaan RAM terbesar. |
| `1` | `top` | Menampilkan atau menyembunyikan rincian beban masing-masing core CPU secara individual. |
| `k` | `top` | Membuka prompt untuk mengirim sinyal Kill ke PID tertentu. |
| `F3` | `htop` | Membuka filter pencarian proses berdasarkan nama. |
| `F6` | `htop` | Membuka menu pemilihan kolom pengurutan (Sort by). |
| `F9` | `htop` | Membuka menu pengiriman sinyal (SIGTERM, SIGKILL, dll) ke proses terpilih. |
| `q` / `F10` | `top` / `htop` | Keluar dari aplikasi monitoring. |

---

## 3. Monitoring Memori / RAM & Swap

### Pembacaan Output `free -h`

| Kolom Output | Definisi Teknis | Makna Praktis bagi Admin |
| :--- | :--- | :--- |
| **`total`** | Total kapasitas fisik RAM terpasang | Total memory yang terdeteksi oleh kernel. |
| **`used`** | RAM yang dikonsumsi aktif oleh aplikasi | RAM yang sedang tidak dapat dialokasikan ke proses lain. |
| **`free`** | RAM yang benar-benar tidak terisi | RAM yang menganggur total (biasanya sengaja diminimalkan oleh Linux). |
| **`buff/cache`** | RAM yang dipinjam Linux untuk file caching | Mempercepat pembacaan disk. Otomatis dibebaskan seketika jika aplikasi butuh memori. |
| **`available`** | Kapasitas RAM yang benar-benar siap digunakan | **Indikator paling penting**. Menunjukkan sisa RAM riil sebelum sistem terpaksa swap. |
| **`Swap`** | Memori virtual berbasis partisi storage disk | Digunakan saat RAM fisik penuh. Swap yang aktif berlebihan menandakan server kekurangan RAM. |

### Pembacaan Kolom Kritis `vmstat` (`vmstat 1 5`)

| Bagian | Kolom | Definisi | Titik Kritis Bottleneck |
| :--- | :---: | :--- | :--- |
| **Procs** | `r` | Jumlah proses yang aktif berjalan / mengantre giliran CPU | Nilai > jumlah CPU Core menandakan CPU tersaturasi. |
| **Procs** | `b` | Jumlah proses yang terblokir (*uninterruptible sleep*) | Nilai > 0 menunjukkan antrean I/O disk sedang macet. |
| **Swap** | `si` | Swap-In (Data dibaca dari disk masuk ke RAM per detik) | Nilai tinggi terus menerus berarti RAM fisik sangat kurang. |
| **Swap** | `so` | Swap-Out (Data dipindah dari RAM keluar ke disk per detik) | Terjadi proses swapping aktif yang membuat sistem lambat. |

---

## 4. Monitoring Penyimpanan & Disk I/O

| Perintah | Opsi Rekomendasi | Fungsi & Tujuan |
| :--- | :--- | :--- |
| `df -h` | `-h` (Human Readable) | Memeriksa persentase sisa ruang kapasitas penyimpanan seluruh partisi disk. |
| `df -i` | `-i` (Inodes) | Memeriksa ketersediaan nomor Inode (mencegah error "No space left on device" saat GB masih ada). |
| `du -sh *` | `-s` (Summary), `-h` (Human) | Menampilkan total ukuran direktori tingkat pertama di folder saat ini. |
| `du -ah . \| sort -rh \| head -n 10` | Pipeline Sort | Menemukan 10 berkas/folder berukuran terbesar di direktori aktif. |
| `ncdu /` | TUI Interaktif | Menjelajahi struktur penggunaan storage disk secara interaktif dan cepat. |
| `lsblk` | `-f` (Filesystem info) | Menampilkan daftar block device, partisi, UUID, dan titik mount point. |
| `iostat -xz 1 5` | Extended stats | Memantau statistik performa disk setiap 1 detik. Perhatikan kolom `%util` (jika 100%, disk overload). |

---

## 5. Monitoring Jaringan & Port Aktif

### Perintah Pengecekan Socket & Port (`ss`)

| Perintah | Deskripsi | Kegunaan |
| :--- | :--- | :--- |
| `sudo ss -tulpn` | Menampilkan semua port TCP & UDP yang berstatus LISTEN | Memastikan port service (web, DB, SSH) sudah terbuka beserta PID-nya. |
| `ss -s` | Menampilkan statistik agregat seluruh socket | Memantau total koneksi TCP, UDP, dan RAW yang aktif di sistem. |
| `ss -t state established` | Menampilkan seluruh koneksi TCP yang sedang aktif terhubung | Memeriksa klien yang sedang tersambung ke server. |

### Perintah Pemeriksaan Port & File Terbuka (`lsof`)

| Perintah | Fungsi | Contoh Kasus |
| :--- | :--- | :--- |
| `sudo lsof -i :80` | Mencari proses yang sedang mengikat port 80 | Mengetahui aplikasi mana yang memblokir port web server. |
| `sudo lsof -i :3306` | Mencari proses di port database MySQL | Memverifikasi apakah daemon database aktif. |
| `sudo lsof -p <PID>` | Melihat seluruh file yang sedang dibuka oleh suatu nomor PID | Mengetahui file konfigurasi, log, atau socket yang dibuka oleh aplikasi. |
| `sudo lsof -u www-data` | Melihat aktivitas file dari akun pengguna tertentu | Memeriksa aktivitas proses user web server. |

---

## 6. Inspeksi Log Sistem & Kernel

### Manajemen Log Layanan Systemd (`journalctl`)

| Perintah | Opsi / Filter | Tujuan Analisis |
| :--- | :--- | :--- |
| `sudo journalctl -u nginx -f` | Follow live log | Memantau log service Nginx secara real-time saat ada request masuk. |
| `sudo journalctl -xe` | Penjelasan error detail | Memeriksa penyebab kegagalan saat sebuah service gagal start (*failed to start*). |
| `sudo journalctl -b` | Boot aktif | Menampilkan log sejak sistem dinyalakan pada sesi boot saat ini. |
| `sudo journalctl -p err..emerg` | Filter prioritas | Hanya menampilkan log dengan level Error, Critical, Alert, atau Emergency. |
| `sudo journalctl --since "30 min ago"` | Filter waktu | Menampilkan log yang tercatat dalam kurun waktu 30 menit terakhir. |

### Diagnosa Kernel & OOM Killer (`dmesg`)

| Perintah | Filter / Opsi | Kasus Penggunaan |
| :--- | :--- | :--- |
| `sudo dmesg -T` | Timestamp manusia | Membaca ring buffer kernel dengan format tanggal dan waktu standar. |
| `sudo dmesg -T \| grep -i -E "oom\|killed process"` | Pencarian OOM Killer | Mendeteksi apakah kernel mematikan paksa aplikasi karena memori RAM habis total. |
| `sudo dmesg -T \| grep -i "error"` | Deteksi kegagalan hardware | Memeriksa kegagalan controller disk, memory fault, atau link interface jaringan. |

---

## 7. Checklist Investigasi Insiden 60 Detik

Jalankan 6 perintah berikut secara runtut saat terjadi degradasi performa pada server:

| Urutan | Perintah Eksekusi | Objek Pemeriksaan |
| :---: | :--- | :--- |
| **1** | `uptime` | Periksa Load Average dan bandingkan dengan jumlah core (`nproc`). |
| **2** | `free -h` | Periksa kolom `available` dan apakah nilai `Swap used` melonjak drastis. |
| **3** | `df -h && df -i` | Pastikan kapasitas disk (GB) dan kapasitas Inode tidak ada yang 100%. |
| **4** | `top -b -n 1 \| head -n 20` | Identifikasi 5 proses teratas yang mengonsumsi CPU atau Memory terbesar. |
| **5** | `iostat -xz 1 2` | Evaluasi nilai `%util` dan `%wa` untuk memeriksa kemacetan baca/tulis disk. |
| **6** | `sudo ss -tulpn` | Pastikan port aplikasi utama tetap dalam status LISTEN dan tidak mati mendadak. |

---

## 8. Navigasi Lanjutan Modul

| Modul Sebelumnya | Indeks Dokumentasi |
| :---: | :---: |
| [05 - Package Management](./05-package-management.md) | [README.md - Master Index](./README.md) |
