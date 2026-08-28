# 07 - Linux I/O Monitoring & Performance Profiling

Panduan komprehensif monitoring dan profiling Input/Output (I/O) di Linux: analisis Disk I/O throughput, IOPS, latensi (*await*), saturasi disk (%util), per-process I/O tracking (`iotop`, `pidstat -d`), Network I/O bandwidth, serta diagnosa status *IO Wait* (`%wa`).

---

## Daftar Isi
- [1. Konsep Fundamental Linux I/O](#1-konsep-fundamental-linux-io)
- [2. Monitoring Disk I/O Sistem (iostat, sar, vmstat)](#2-monitoring-disk-io-sistem-iostat-sar-vmstat)
- [3. Pelacakan I/O Berbasis Proses (iotop, pidstat)](#3-pelacakan-io-berbasis-proses-iotop-pidstat)
- [4. Monitoring Network I/O & Bandwidth](#4-monitoring-network-io--bandwidth)
- [5. Uji Performa I/O Storage (fio, dd)](#5-uji-performa-io-storage-fio-dd)
- [6. Playbook Diagnosa Masalah Disk I/O Bottleneck](#6-playbook-diagnosa-masalah-disk-io-bottleneck)
- [7. Navigasi Lanjutan Modul](#7-navigasi-lanjutan-modul)

---

## 1. Konsep Fundamental Linux I/O

I/O (*Input/Output*) merepresentasikan proses perpindahan data antara CPU/RAM dan perangkat penyimpanan fisik (SSD, NVMe, HDD) atau antarmuka jaringan (NIC).

| Istilah / Metrik | Satuan | Definisi Teknis | Ambang Batas Evaluasi |
| :--- | :---: | :--- | :--- |
| **IOPS** | Ops/detik | *Input/Output Operations Per Second* - Jumlah transaksi baca/tulis kecil per detik. | Kritis untuk database transaksi tinggi (MySQL/PostgreSQL). SSD berkisar 10k-100k IOPS. |
| **Throughput** | MB/s | Kecepatan volume data riil yang ditransfer per detik (*Sequential Read/Write*). | Kritis untuk backup data, streaming video, dan copy file besar. |
| **Latency / Await** | Milidetik (ms) | Total waktu yang dibutuhkan mulai dari request I/O dikirim hingga perangkat disk selesai merespons. | SSD optimal < 5ms. Jika `await` > 20-50ms, disk mengalami antrean berat. |
| **Queue Size (`aqu-sz`)**| Jumlah Request | Jumlah rata-rata request I/O yang sedang mengantre (*queue length*) untuk dilayani disk. | Idealnya bernilai rendah (< 2-3 pada storage standar). |
| **Utilization (`%util`)**| Persentase (%) | Persentase waktu disk aktif memproses request I/O dalam durasi sampling. | **Jika >= 90-100%**, disk telah mencapai kapasitas saturasi maksimal (*Bottleneck*). |

---

## 2. Monitoring Disk I/O Sistem (iostat, sar, vmstat)

### Analisis Mendalam Perintah `iostat`

Utilitas standar dari paket `sysstat` untuk menganalisis throughput dan saturasi disk storage.

```bash
# Menjalankan sampling statistik detail per 1 detik sebanyak 5 kali
iostat -xz 1 5
```

| Kolom Output `iostat -xz` | Kategori | Arti Teknis |
| :--- | :--- | :--- |
| `Device` | Perangkat | Nama blok partisi/disk (misal: `sda`, `nvme0n1`, `vda`). |
| `r/s` & `w/s` | IOPS | Jumlah request Read dan Write yang diselesaikan per detik. |
| `rMB/s` & `wMB/s` | Throughput | Kecepatan Read dan Write dalam satuan Megabyte per detik. |
| `r_await` & `w_await` | Latensi | Rata-rata waktu tunggu (ms) untuk operasi Read dan Write. |
| `aqu-sz` | Queue | Rata-rata panjang antrean request I/O ke perangkat. |
| `rareq-sz` & `wareq-sz` | Ukuran Request | Ukuran rata-rata (KB) dari request Read dan Write yang dikirim. |
| **`%util`** | **Saturasi** | **Tingkat kesibukan disk**. Jika 100%, disk bekerja tanpa jeda dan request baru terpaksa mengantre. |

---

### Utilitas Monitoring Tambahan (`sar` & `vmstat`)

| Perintah | Opsi / Contoh | Fungsi & Parameter yang Dilihat |
| :--- | :--- | :--- |
| `sar -d` | `sar -d 1 5` | Merekam dan menampilkan riwayat statistik aktivitas disk secara berkala. |
| `vmstat -d` | `vmstat -d` | Menampilkan ringkasan statistik disk (total reads, merged reads, sectors written). |
| `vmstat -D` | `vmstat -D` | Menampilkan tabel agregat disk per-event secara keseluruhan. |
| `dstat -cdngy` | `dstat -cdngy 1` | Menampilkan dashboard gabungan CPU, Disk I/O, Network, dan System load secara real-time. |

---

## 3. Pelacakan I/O Berbasis Proses (iotop, pidstat)

Untuk mengetahui **proses/aplikasi mana secara spesifik** yang menyebabkan disk overload:

### 1. Perintah `iotop` (Interactive I/O Top)

```bash
# Instalasi iotop
sudo apt install iotop -y    # Debian/Ubuntu
sudo dnf install iotop -y    # RHEL/CentOS/Rocky
```

| Perintah `iotop` | Mode Eksekusi | Deskripsi Kegunaan |
| :--- | :--- | :--- |
| `sudo iotop` | TUI Interaktif | Menampilkan seluruh proses beserta bandwidth DISK READ dan DISK WRITE aktual. |
| `sudo iotop -o` | Hanya I/O Aktif | Menyembunyikan proses yang sedang idle dan hanya menampilkan proses yang sedang melakukan baca/tulis disk. |
| `sudo iotop -o -a` | Mode Akumulasi | Menghitung akumulasi total data yang telah ditulis/dibaca sejak perintah dijalankan. |
| `sudo iotop -b -n 3` | Mode Batch | Menghasilkan snapshot output teks tanpa TUI (cocok untuk dialihkan ke file log). |

---

### 2. Perintah `pidstat -d` (Detail I/O Per-Proses dari Sysstat)

```bash
# Menampilkan konsumsi I/O setiap proses setiap 2 detik sebanyak 3 kali
pidstat -d 2 3
```

| Kolom Output `pidstat -d` | Definisi | Interpretasi |
| :--- | :--- | :--- |
| `PID` | Process ID | Nomor identifikasi proses aplikasi. |
| `kB_rd/s` | Read Speed | Kecepatan baca disk oleh proses dalam Kilobyte per detik. |
| `kB_wr/s` | Write Speed | Kecepatan tulis disk oleh proses dalam Kilobyte per detik. |
| `kB_ccwr/s` | Cancelled Write | Volume penulisan yang dibatalkan oleh proses (misal file dihapus sebelum di-flush). |
| `iodelay` | I/O Delay | Waktu delay proses (dalam clock ticks) akibat terhambat antrean I/O. |
| `Command` | Nama Program | Binary atau skrip yang bertanggung jawab terhadap aktivitas I/O. |

---

## 4. Monitoring Network I/O & Bandwidth

Selain disk, I/O jaringan menentukan performa komunikasi data server dengan dunia luar.

| Perintah | Kategori | Deskripsi & Contoh |
| :--- | :--- | :--- |
| `sar -n DEV 1 3` | Statistik Interface | Memantau paket masuk (`rxpck/s`), paket keluar (`txpck/s`), dan throughput KB/s per kartu jaringan (NIC). |
| `ip -s link show eth0` | Drop & Error Packets | Memeriksa apakah terjadi packet drop, overruns, atau CRC framing error pada interface hardware. |
| `nload` | Visual Real-Time | Menampilkan grafik ASCII visual bandwidth masuk (*incoming*) dan keluar (*outgoing*) per interface. |
| `iftop -P` | Socket Traffic | Menampilkan bandwidth jaringan per-koneksi host dan nomor port secara real-time. |
| `bmon` | Bandwidth Monitor | Antarmuka monitoring multi-interface terperinci dengan analisis frame rate. |

---

## 5. Uji Performa I/O Storage (fio, dd)

Untuk menguji batas kemampuan baca/tulis penyimpanan (*Benchmarking*):

| Alat Uji | Perintah Contoh | Parameter & Tujuan |
| :--- | :--- | :--- |
| `dd` (Sequential Write) | `dd if=/dev/zero of=/tmp/test.img bs=1G count=1 oflag=direct status=progress` | Menguji kecepatan tulis sekuensial murni langsung ke disk tanpa melewati cache RAM (`oflag=direct`). |
| `fio` (Random Read/Write) | `fio --name=randwrite --ioengine=libaio --iodepth=16 --rw=randwrite --bs=4k --direct=1 --size=512M --numjobs=2 --runtime=10 --group_reporting` | Menguji IOPS dan latensi acak (*Random 4K*) yang mencerminkan beban kerja database nyata. |

---

## 6. Playbook Diagnosa Masalah Disk I/O Bottleneck

Ikuti tabel urutan berikut saat nilai `%wa` (*IO Wait*) pada server terdeteksi tinggi:

| Tahap | Perintah Eksekusi | Analisis Tindakan |
| :---: | :--- | :--- |
| **1. Verifikasi Gejala** | `top` (Periksa kolom `%wa`) | Jika `%wa` > 15-20%, CPU sedang tersendat menunggu respon dari storage. |
| **2. Identifikasi Disk Bermasalah** | `iostat -xz 1 3` | Cari baris partisi dengan `%util` mendekati 100% dan nilai `await` yang tinggi. |
| **3. Temukan Proses Penyebab** | `sudo iotop -o -P` atau `pidstat -d 1 5` | Identifikasi nomor PID dan nama aplikasi yang menghasilkan `DISK WRITE/READ` terbesar. |
| **4. Analisis File yang Dibuka** | `sudo lsof -p <PID>` | Cari tahu direktori file data atau log yang sedang diakses secara agresif oleh proses tersebut. |
| **5. Solusi Mitigasi** | `renice` / `ionice` / Optimize | Turunkan prioritas I/O proses dengan `ionice -c 3 -p <PID>` (Idle I/O priority) atau optimasi konfigurasi caching aplikasi. |

---

## 7. Navigasi Lanjutan Modul

| Modul Sebelumnya | Modul Berikutnya |
| :---: | :---: |
| [06 - System Monitoring](./06-system-monitoring.md) | [08 - Linux Filesystems and the VFS](./08-linux-filesystems-and-vfs.md) |
