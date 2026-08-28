# 04 - Linux Signals (Sinyal Kernel & IPC)

Panduan mengenai mekanisme sinyal di Linux: konsep komunikasi antar-proses (IPC), tabel nomor dan jenis sinyal, instruksi penghentian proses (`kill`, `killall`, `pkill`), perbandingan `SIGTERM` vs `SIGKILL`, serta penanganan sinyal (*signal trapping*) pada skrip Bash.

---

## Daftar Isi
- [1. Konsep Sinyal Linux](#1-konsep-sinyal-linux)
- [2. Tabel Standar Sinyal Linux Lengkap](#2-tabel-standar-sinyal-linux-lengkap)
- [3. Perbandingan Mendalam: SIGTERM (15) vs SIGKILL (9)](#3-perbandingan-mendalam-sigterm-15-vs-sigkill-9)
- [4. Perintah Pengiriman Sinyal (kill, killall, pkill)](#4-perintah-pengiriman-sinyal-kill-killall-pkill)
- [5. Alur Troubleshooting Proses Macet](#5-alur-troubleshooting-proses-macet)
- [6. Penanganan Sinyal pada Skrip Bash (trap)](#6-penanganan-sinyal-pada-skrip-bash-trap)
- [7. Navigasi Lanjutan Modul](#7-navigasi-lanjutan-modul)

---

## 1. Konsep Sinyal Linux

Sinyal di Linux adalah mekanisme asinkronus tingkat rendah yang digunakan oleh kernel, pengguna, atau suatu proses untuk memberitahu proses lain bahwa telah terjadi suatu kejadian tertentu.

| Jenis Respon Proses | Penjelasan Teknis |
| :--- | :--- |
| **Default Action** | Proses menjalankan tindakan bawaan yang ditentukan oleh kernel (misal: terminasi, dump core, atau pause). |
| **Catch / Handle** | Proses menangkap sinyal dan mengeksekusi fungsi kustom (misal: menyimpan data ke disk sebelum keluar). |
| **Ignore** | Proses mengabaikan sinyal tanpa mengambil tindakan apa pun (tidak berlaku untuk sinyal non-ignorable). |

---

## 2. Tabel Standar Sinyal Linux Lengkap

| Nomor | Nama Sinyal | Shortcut Terminal | Aksi Bawaan | Sifat | Deskripsi & Kasus Penggunaan |
| :---: | :--- | :---: | :---: | :---: | :--- |
| **`1`** | **`SIGHUP`** *(Hangup)* | - | Terminate | Catchable | Digunakan oleh daemon server (seperti Nginx atau Apache) untuk memuat ulang konfigurasi (*reload*) tanpa restart service. |
| **`2`** | **`SIGINT`** *(Interrupt)* | `Ctrl + C` | Terminate | Catchable | Menghentikan eksekusi program di terminal secara normal. |
| **`3`** | **`SIGQUIT`** *(Quit)* | `Ctrl + \` | Core Dump | Catchable | Menghentikan proses dan menghasilkan file memory core dump untuk debugging. |
| **`9`** | **`SIGKILL`** *(Kill)* | - | Terminate Paksa | **Uncatchable** | Mematikan proses seketika di tingkat Kernel. Proses tidak dapat menangkap atau menolak sinyal ini. |
| **`15`** | **`SIGTERM`** *(Terminate)* | - | Terminate | Catchable | Sinyal default dari perintah `kill`. Meminta proses keluar secara bersih (*graceful shutdown*). |
| **`18`** | **`SIGCONT`** *(Continue)* | - | Resume | Catchable | Melanjutkan kembali proses yang sedang di-pause (`SIGSTOP` atau `SIGTSTP`). |
| **`19`** | **`SIGSTOP`** *(Stop)* | - | Pause Paksa | **Uncatchable** | Menghentikan sementara proses secara paksa di level kernel. |
| **`20`** | **`SIGTSTP`** *(Terminal Stop)* | `Ctrl + Z` | Pause Terminal | Catchable | Menghentikan sementara proses dari terminal dan memindahkannya ke antrean background. |
| **`10`** | **`SIGUSR1`** *(User 1)* | - | Terminate | Catchable | Sinyal kustom bebas aplikasi (misal untuk rotasi file log Nginx). |
| **`12`** | **`SIGUSR2`** *(User 2)* | - | Terminate | Catchable | Sinyal kustom bebas aplikasi (misal untuk zero-downtime binary upgrade). |

---

## 3. Perbandingan Mendalam: SIGTERM (15) vs SIGKILL (9)

| Parameter Evaluasi | SIGTERM (Sinyal 15) | SIGKILL (Sinyal 9) |
| :--- | :--- | :--- |
| **Penerima Langsung Sinyal** | Proses target yang dituju | Linux Kernel |
| **Dapat Ditangkap (*Catchable*)?** | Ya, aplikasi dapat mengeksekusi fungsi cleanup | Tidak, aplikasi langsung dihapus dari tabel proses |
| **Dapat Diabaikan (*Ignorable*)?** | Ya, aplikasi dapat menolak jika sedang sibuk | Tidak bisa ditolak sama sekali |
| **Integritas Data & File** | Aman (file disimpan tuntas, koneksi database ditutup) | Rawan korupsi data jika sedang menulis ke storage |
| **Tingkat Prioritas Penggunaan** | **Pilihan Pertama** untuk mematikan aplikasi normal | **Pilihan Terakhir** hanya jika proses hang/freeze total |

---

## 4. Perintah Pengiriman Sinyal (kill, killall, pkill)

### Perintah `kill` (Berdasarkan nomor Process ID / PID)

| Format Perintah | Sinyal yang Dikirim | Contoh Perintah |
| :--- | :--- | :--- |
| `kill <PID>` | SIGTERM (15 - Default) | `kill 4521` |
| `kill -15 <PID>` | SIGTERM (15) | `kill -15 4521` |
| `kill -9 <PID>` | SIGKILL (9 - Paksa) | `kill -9 4521` |
| `kill -1 <PID>` | SIGHUP (1 - Reload Config) | `kill -1 4521` |
| `kill -STOP <PID>` | SIGSTOP (19 - Pause) | `kill -STOP 4521` |
| `kill -CONT <PID>` | SIGCONT (18 - Resume) | `kill -CONT 4521` |
| `kill -l` | List Signals | Menampilkan daftar seluruh nama dan nomor sinyal di sistem |

### Perintah `killall` (Berdasarkan Nama Lengkap Program)

| Format Perintah | Penjelasan | Contoh Perintah |
| :--- | :--- | :--- |
| `killall <nama_program>` | Mengirim SIGTERM ke seluruh proses dengan nama tersebut | `sudo killall nginx` |
| `killall -9 <nama_program>` | Mengirim SIGKILL ke seluruh instance program | `killall -9 firefox` |
| `killall -HUP <nama_program>` | Mengirim SIGHUP untuk reload konfigurasi seluruh instance | `sudo killall -HUP httpd` |

### Perintah `pkill` (Pencarian Pola & Filter Lanjutan)

| Parameter / Flag | Fungsi Filter | Contoh Perintah |
| :--- | :--- | :--- |
| `-f` | Cocokkan dengan seluruh baris argumen perintah | `pkill -f "python3 app.py"` |
| `-u` | Mengirim sinyal ke proses milik user tertentu | `sudo pkill -u budi` |
| `-t` | Mengirim sinyal ke proses di terminal tertentu | `sudo pkill -t pts/1` |
| `-9` | Kombinasi opsi filter dengan sinyal SIGKILL | `pkill -9 -f "worker.js"` |

---

## 5. Alur Troubleshooting Proses Macet

| Tahap | Aksi | Perintah Eksekusi | Keterangan |
| :---: | :--- | :--- | :--- |
| **1** | Identifikasi PID | `pgrep -fl "node server.js"` | Dapatkan nomor PID dari aplikasi yang bermasalah. |
| **2** | Kirim Terminasi Bersih | `kill <PID>` | Mengirim sinyal SIGTERM (15) dan tunggu 5-10 detik agar cleanup berjalan. |
| **3** | Verifikasi Status | `pgrep -fl "node server.js"` | Pastikan apakah proses sudah berhenti atau masih ada di memori. |
| **4** | Terminasi Paksa | `kill -9 <PID>` | Jalankan hanya jika tahap 2 tidak berhasil mematikan proses yang macet. |

---

## 6. Penanganan Sinyal pada Skrip Bash (`trap`)

Perintah `trap` digunakan di dalam skrip shell untuk mencegat sinyal yang masuk dan menjalankan fungsi pembersihan sebelum skrip berakhir.

| Komponen Skrip | Kode Contoh | Fungsi Teknis |
| :--- | :--- | :--- |
| **Definisi Fungsi Cleanup** | `cleanup() { rm -f /tmp/lockfile.tmp; exit 0; }` | Menghapus file sementara atau menutup koneksi. |
| **Pemasangan Trap** | `trap cleanup SIGINT SIGTERM` | Mengarahkan sinyal interupsi (Ctrl+C) dan terminasi ke fungsi `cleanup`. |

### Contoh Struktur Skrip Bash:

```bash
#!/bin/bash
TEMP_FILE="/tmp/worker_$$.tmp"
touch "$TEMP_FILE"

cleanup() {
    echo "Menerima sinyal terminasi. Menghapus $TEMP_FILE..."
    rm -f "$TEMP_FILE"
    exit 0
}

trap cleanup SIGINT SIGTERM

while true; do
    sleep 1
done
```

---

## 7. Navigasi Lanjutan Modul

| Modul Sebelumnya | Modul Berikutnya |
| :---: | :---: |
| [03 - Process Management](./03-process-management.md) | [05 - Package Management](./05-package-management.md) |
