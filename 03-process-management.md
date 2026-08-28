# 03 - Linux Process Management

Panduan manajemen proses di Linux: siklus hidup proses, identifikasi PID/PPID, pemantauan snapshot proses, job control (foreground/background), prioritas (*nice & renice*), serta inspeksi filesystem virtual `/proc`.

---

## Daftar Isi
- [1. Konsep Dasar & Identitas Proses](#1-konsep-dasar--identitas-proses)
- [2. Status Siklus Hidup Proses (Process States)](#2-status-siklus-hidup-proses-process-states)
- [3. Perintah Pemantauan Proses (ps, pstree, pgrep, pidof)](#3-perintah-pemantauan-proses-ps-pstree-pgrep-pidof)
- [4. Job Control: Foreground & Background Management](#4-job-control-foreground--background-management)
- [5. Prioritas Penjadwalan CPU (Nice & Renice)](#5-prioritas-penjadwalan-cpu-nice--renice)
- [6. Direktori Virtual Kernel /proc](#6-direktori-virtual-kernel-proc)
- [7. Navigasi Lanjutan Modul](#7-navigasi-lanjutan-modul)

---

## 1. Konsep Dasar & Identitas Proses

Proses adalah sebuah instansiasi program komputer yang sedang aktif dijalankan di memori oleh prosesor.

| Istilah / Atribut | Deskripsi | Keterangan |
| :--- | :--- | :--- |
| **PID** (*Process ID*) | Nomor pengenal unik numerik untuk setiap proses | Diberikan secara berurutan oleh kernel Linux saat proses baru diciptakan. |
| **PPID** (*Parent Process ID*) | Nomor PID dari proses induk | Menunjukkan proses mana yang memanggil (*fork*) proses saat ini. |
| **UID / Owner** | Identitas pengguna pemilik proses | Menentukan hak akses yang dimiliki oleh proses saat membaca/menulis file di sistem. |
| **PID 1 (systemd / init)** | Proses induk pertama (*Grandparent Process*) | Proses awal yang dijalankan oleh Kernel saat Linux selesai booting. Semua proses lain adalah turunan dari PID 1. |
| **Daemon** | Proses layanan background | Proses tanpa antarmuka grafis atau terminal interaktif (`TTY = ?`), berjalan terus-menerus untuk melayani request (contoh: `sshd`, `nginx`, `mysqld`). |

---

## 2. Status Siklus Hidup Proses (Process States)

Status proses dapat diamati pada kolom `STAT` (pada perintah `ps aux`):

| Kode STAT | Status Nama | Deskripsi Teknis | Penanganan |
| :---: | :--- | :--- | :--- |
| **`R`** | **Running / Runnable** | Proses sedang aktif dieksekusi CPU atau sedang menunggu giliran di antrean eksekusi (*run-queue*). | Normal / Aktif. |
| **`S`** | **Interruptible Sleep** | Proses sedang tidur menunggu *event*, input perangkat, atau sinyal masuk. | Normal; langsung merespons jika dipanggil. |
| **`D`** | **Uninterruptible Sleep** | Proses sedang menunggu operasi perangkat keras I/O (misal disk read/write). | Tidak dapat dimatikan oleh sinyal apa pun sebelum operasi hardware I/O selesai. |
| **`T`** | **Stopped / Traced** | Proses dihentikan sementara (misal ditekan `Ctrl + Z` atau menerima sinyal `SIGSTOP`). | Dapat dilanjutkan kembali dengan `SIGCONT` atau perintah `fg`/`bg`. |
| **`Z`** | **Zombie / Defunct** | Proses anak yang telah selesai dieksekusi, namun proses induk (Parent) belum membaca nilai keluarannya (*exit code*). | Menghabiskan slot nomor PID. Dapat dibersihkan dengan me-restart proses parent-nya. |

---

## 3. Perintah Pemantauan Proses (ps, pstree, pgrep, pidof)

### Tabel Opsi Perintah `ps`

| Perintah | Gaya Format | Kolom yang Ditampilkan | Kegunaan Utama |
| :--- | :--- | :--- | :--- |
| `ps aux` | BSD Style | `USER, PID, %CPU, %MEM, VSZ, RSS, TTY, STAT, START, TIME, COMMAND` | Standar DevOps untuk melihat seluruh proses, penggunaan CPU, dan pemakaian memori fisik (RSS). |
| `ps -ef` | System V Style | `UID, PID, PPID, C, STIME, TTY, TIME, CMD` | Standar sysadmin untuk mengidentifikasi hierarki relasi parent (`PPID`) dan child (`PID`). |
| `ps aux --sort=-%mem` | BSD + Sort | Diurutkan dari penggunaan RAM terbesar | Menemukan proses yang menyebabkan konsumsi RAM tinggi. |
| `ps aux --sort=-%cpu` | BSD + Sort | Diurutkan dari penggunaan CPU tertinggi | Menemukan proses yang memakan utilisasi CPU terbesar. |
| `ps -u ubuntu` | Filter User | Proses milik user `ubuntu` saja | Memeriksa seluruh aktivitas dari akun user tertentu. |

### Tabel Utilitas Pencarian & Hierarki Proses

| Perintah | Deskripsi & Contoh | Output / Fungsi |
| :--- | :--- | :--- |
| `pstree` | `pstree -p` | Menampilkan struktur pohon relasi proses induk dan anak beserta nomor PID. |
| `pgrep <nama>` | `pgrep -l nginx` | Mencari nomor PID dari proses berdasarkan pencarian nama atau regex. Opsi `-l` menampilkan nama proses. |
| `pgrep -u <user>` | `pgrep -u www-data` | Menampilkan seluruh PID yang dijalankan oleh user tertentu. |
| `pidof <binary>` | `pidof mysqld` | Menampilkan nomor PID spesifik dari nama binary yang sedang aktif. |

---

## 4. Job Control: Foreground & Background Management

| Aksi / Status | Perintah / Shortcut | Penjelasan Teknis |
| :--- | :--- | :--- |
| **Jalankan di Background** | `<command> &` | Menambahkan tanda ampersand `&` di akhir baris agar proses langsung berjalan di background tanpa mengunci terminal. Contoh: `python3 worker.py &`. |
| **Pause Proses Aktif** | `Ctrl + Z` | Mengirim sinyal `SIGTSTP` untuk menangguhkan sementara proses foreground dan memindahkannya ke antrean job status *Stopped*. |
| **Lihat Antrean Job** | `jobs -l` | Menampilkan daftar seluruh job di sesi terminal aktif beserta nomor Job ID (`%1`, `%2`) dan PID-nya. |
| **Lanjutkan di Background** | `bg %<job_id>` | Melanjutkan eksekusi proses yang di-pause agar berjalan di background. Contoh: `bg %1`. |
| **Pindah ke Foreground** | `fg %<job_id>` | Menarik proses dari background kembali ke foreground terminal. Contoh: `fg %1`. |
| **Tahan Putus Sesi SSH** | `nohup <cmd> > app.log 2>&1 &` | Menjalankan perintah yang kebal terhadap sinyal pemutusan terminal (`SIGHUP`). Seluruh log output dialihkan ke file `app.log`. |
| **Lepas dari Shell Aktif** | `disown -h %<job_id>` | Melepaskan proses background yang sudah terlanjur berjalan dari kontrol terminal sehingga tidak mati saat logout. |

---

## 5. Prioritas Penjadwalan CPU (Nice & Renice)

Kernel Linux menggunakan nilai **Niceness (`NI`)** untuk menentukan alokasi waktu CPU terhadap suatu proses.

| Nilai Niceness | Tingkat Prioritas | Hak Akses | Karakteristik Perilaku |
| :---: | :--- | :--- | :--- |
| **`-20` s.d. `-1`** | **Sangat Tinggi / Tinggi** | Membutuhkan `sudo` / `root` | Agresif dalam mengambil alokasi waktu CPU. Diberikan untuk layanan kritis. |
| **`0`** | **Normal / Default** | Semua Pengguna | Prioritas standar untuk seluruh program biasa. |
| **`1` s.d. `19`** | **Rendah / Sangat Rendah** | Semua Pengguna | "Ramah" (*Nice*), mengalah pada proses lain yang membutuhkan CPU. Cocok untuk backup atau kompresi file besar. |

### Perintah Konfigurasi Nilai Nice

| Perintah | Format | Deskripsi |
| :--- | :--- | :--- |
| `nice` | `nice -n 10 ./backup.sh` | Memulai program baru dengan prioritas rendah (Nice = 10). |
| `nice` (Tinggi) | `sudo nice -n -10 ./database_server` | Memulai program baru dengan prioritas tinggi (Nice = -10, butuh hak root). |
| `renice` | `sudo renice -n 5 -p 1450` | Mengubah nilai prioritas pada proses yang **sudah berjalan** berdasarkan nomor PID. |
| `renice` (User) | `sudo renice -n 10 -u budi` | Mengubah prioritas seluruh proses yang dimiliki oleh user `budi`. |

---

## 6. Direktori Virtual Kernel `/proc`

Direktori `/proc/<PID>/` menyediakan data telemetri langsung dari kernel terkait proses yang sedang aktif.

| File / Subfolder di `/proc/<PID>/` | Informasi yang Disimpan | Contoh Perintah Inspeksi |
| :--- | :--- | :--- |
| `/proc/<PID>/cmdline` | Perintah lengkap dan argumen saat proses dijalankan | `cat /proc/1234/cmdline \| tr '\0' ' '` |
| `/proc/<PID>/cwd` | Symbolic link ke direktori kerja proses (*Current Working Directory*) | `ls -l /proc/1234/cwd` |
| `/proc/<PID>/exe` | Symbolic link ke file executable binary asli | `ls -l /proc/1234/exe` |
| `/proc/<PID>/environ` | Seluruh environment variables yang dimuat oleh proses | `cat /proc/1234/environ \| tr '\0' '\n'` |
| `/proc/<PID>/fd/` | Folder berisi daftar File Descriptors (file, socket, pipe yang dibuka) | `ls -l /proc/1234/fd/` |
| `/proc/<PID>/status` | Ringkasan detail konsumsi memori, state, dan UID/GID | `cat /proc/1234/status` |
| `/proc/<PID>/limits` | Batasan limit sistem (*soft/hard limits*, open files, stack) | `cat /proc/1234/limits` |

---

## 7. Navigasi Lanjutan Modul

| Modul Sebelumnya | Modul Berikutnya |
| :---: | :---: |
| [02 - Filesystem Tree Layout](./02-filesystem-tree-layout.md) | [04 - Linux Signals](./04-linux-signals.md) |
