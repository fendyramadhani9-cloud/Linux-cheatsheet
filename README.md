<div align="center">

# Linux Administration & Operations Cheatsheet
### *Comprehensive, Modular & Production-Grade Linux Reference for Engineers*

[![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)](https://www.kernel.org/)
[![Bash](https://img.shields.io/badge/GNU_Bash-4EAA25?style=for-the-badge&logo=gnu-bash&logoColor=white)](https://www.gnu.org/software/bash/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)](https://ubuntu.com/)
[![Debian](https://img.shields.io/badge/Debian-A81D33?style=for-the-badge&logo=debian&logoColor=white)](https://www.debian.org/)
[![Red Hat](https://img.shields.io/badge/Red_Hat-EE0000?style=for-the-badge&logo=redhat&logoColor=white)](https://www.redhat.com/)
[![Rocky Linux](https://img.shields.io/badge/Rocky_Linux-10B981?style=for-the-badge&logo=rockylinux&logoColor=white)](https://rockylinux.org/)

<br>

```text
  _      _                      _____ _                _       _                  _   
 | |    (_)                    / ____| |              | |     | |                | |  
 | |     _ _ __  _   ___  __  | |    | |__   ___  __ _| |_ ___| |__   ___  ___  | |_ 
 | |    | | '_ \| | | \ \/ /  | |    | '_ \ / _ \/ _` | __/ __| '_ \ / _ \/ _ \ | __|
 | |____| | | | | |_| |>  <   | |____| | | |  __/ (_| | |_\__ \ | | |  __/  __/ | |_ 
 |______|_|_| |_|\__,_/_/\_\   \_____|_| |_|\___|\__,_|\__|___/_| |_|\___|\___|  \__|
```

<p align="center">
  <img src="https://raw.githubusercontent.com/fendyramadhani9-cloud/Linux-cheatsheet/main/assets/terminal-animation.svg" alt="Linux System Telemetry" width="800" onerror="this.style.display='none'"/>
</p>

</div>

---

## Ringkasan Proyek

Koleksi dokumentasi teknis, rangkuman, dan contekan (*cheatsheet*) komprehensif administrasi sistem operasi Linux. Disusun secara terstruktur, berbasis tabel mendalam, modular, dan diformat khusus untuk kebutuhan operasional harian SysAdmin, DevOps, dan Platform Engineer.

| Parameter | Keterangan |
| :--- | :--- |
| **Penulis** | Fendy Ramadhani |
| **Fokus Keahlian** | Platform Engineering, DevOps, Cloud Infrastructure, Linux Systems |
| **Referensi Kurikulum** | ADINUSA Linux Administration (Dasar hingga System Monitoring) |
| **Bahasa Dokumentasi** | Bahasa Indonesia (Standar Terminologi Teknis Global) |
| **Format** | Modular, Table-Centric, GitHub Flavored Markdown (GFM) |

---

## Daftar Modul Dokumentasi

Seluruh modul dipisah secara sistematis berdasarkan domain fungsional sistem operasi:

| No | Modul Dokumentasi | Berkas Panduan | Cakupan Materi Utama |
| :---: | :--- | :--- | :--- |
| 1 | **Setup & Environment** | [01-setup-and-environment.md](./01-setup-and-environment.md) | Shell basics, shortcut navigasi terminal, environment variables, manajemen alias, dan riwayat perintah (`history`). |
| 2 | **Filesystem Tree Layout** | [02-filesystem-tree-layout.md](./02-filesystem-tree-layout.md) | Standar hierarki direktori (FHS), manipulasi berkas, permissions (chmod, chown, SUID, SGID, Sticky bit), symlink vs hard link, dan utilitas pencarian (`find`, `locate`). |
| 3 | **Process Management** | [03-process-management.md](./03-process-management.md) | Siklus hidup proses, identitas PID/PPID, `ps aux`, `pstree`, background/foreground job control (`jobs`, `bg`, `fg`, `nohup`), prioritas `nice`/`renice`, dan struktur `/proc`. |
| 4 | **Linux Signals** | [04-linux-signals.md](./04-linux-signals.md) | Konsep sinyal IPC kernel (SIGHUP, SIGINT, SIGKILL, SIGTERM), utilitas `kill`/`killall`/`pkill`, perbandingan terminasi aman vs paksa, serta signal trap pada Bash. |
| 5 | **Package Management** | [05-package-management.md](./05-package-management.md) | Ekosistem Debian/Ubuntu (`apt`, `dpkg`), RHEL/CentOS/Rocky (`dnf`, `rpm`), tabel padanan perintah (*Rosetta Stone*), konfigurasi repositori, kunci GPG, dan kompilasi tarball. |
| 6 | **System Monitoring** | [06-system-monitoring.md](./06-system-monitoring.md) | Metrik USE method, monitoring CPU/Load Average (`uptime`, `top`, `htop`), RAM/Swap (`free`, `vmstat`), Disk I/O (`df`, `du`, `iostat`), Jaringan/Port (`ss`, `lsof`), Log (`journalctl`, `dmesg`), dan checklist investigasi 60 detik. |

---

## Ringkasan Perintah Esensial Harian (Quick Reference)

Tabel contekan kilat perintah yang paling sering digunakan dalam operasional server:

| Kategori | Perintah Eksekusi | Deskripsi Kegunaan |
| :--- | :--- | :--- |
| **Navigasi** | `ls -lah` | Melihat seluruh file/folder dalam format detail dan ukuran human-readable. |
| **Navigasi** | `pwd` | Menampilkan path absolut dari direktori kerja saat ini. |
| **File Management** | `cp -rp <src> <dst>` | Menyalin folder secara rekursif dengan mempertahankan timestamp dan permission asli. |
| **File Management** | `mv <src> <dst>` | Memindahkan file atau mengganti nama berkas/folder. |
| **File Management** | `rm -rf <target>` | Menghapus file/folder secara paksa dan rekursif. |
| **Hak Akses** | `chmod 755 <file>` | Memberikan izin rwx untuk owner, serta r-x untuk group dan others. |
| **Hak Akses** | `chmod 600 <key_file>` | Memberikan izin rw- khusus owner (standar private key SSH). |
| **Kepemilikan** | `sudo chown -R user:group <dir>` | Mengubah kepemilikan user dan group secara rekursif pada seluruh direktori. |
| **Pencarian** | `find /var/log -name "*.log"` | Mencari file berekstensi `.log` secara real-time langsung di storage. |
| **Pencarian** | `which <command>` | Mengetahui path executable dari suatu binary sistem. |
| **Proses** | `ps aux \| grep <nama>` | Mencari snapshot proses aktif berdasarkan filter nama program. |
| **Proses** | `kill -15 <PID>` | Menghentikan proses secara aman (*graceful termination*). |
| **Proses** | `kill -9 <PID>` | Menghentikan proses secara paksa di tingkat kernel Linux. |
| **Background Job** | `nohup <cmd> > app.log 2>&1 &` | Menjalankan perintah di background yang kebal terhadap putus koneksi SSH. |
| **Paket (Ubuntu)** | `sudo apt update && sudo apt upgrade -y` | Memperbarui katalog repositori dan meng-upgrade seluruh paket sistem. |
| **Paket (RHEL/Rocky)**| `sudo dnf upgrade -y` | Meng-upgrade seluruh paket sistem pada turunan Red Hat. |
| **Monitoring** | `uptime` | Memeriksa beban kerja rata-rata (*Load Average*) dan durasi server menyala. |
| **Monitoring** | `free -h` | Memeriksa ketersediaan memori fisik RAM dan alokasi Swap. |
| **Monitoring** | `df -h` | Memeriksa sisa kapasitas penyimpanan di semua partisi storage. |
| **Monitoring** | `sudo ss -tulpn` | Menampilkan seluruh port TCP/UDP yang sedang berstatus listening beserta nomor PID-nya. |
| **Monitoring** | `sudo lsof -i :<port>` | Mencari proses spesifik yang sedang menggunakan port tertentu. |
| **Monitoring** | `sudo journalctl -u <service> -f` | Memantau log service systemd secara langsung (*real-time follow*). |

---

## Petunjuk Penggunaan

1. **Clone Repositori:**
   ```bash
   git clone https://github.com/fendyramadhani9-cloud/Linux-cheatsheet.git
   cd Linux-cheatsheet
   ```
2. **Navigasi Modul:**
   Buka berkas modul yang diinginkan langsung melalui web GitHub atau text editor lokal. Setiap berkas memiliki tautan navigasi internal ke modul sebelum dan sesudahnya.

---

<div align="center">

### **Fendy Ramadhani**
*Platform Engineer | Cloud & AI Systems Infrastructure*

[![GitHub](https://img.shields.io/badge/GitHub-fendyramadhani9--cloud-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/fendyramadhani9-cloud)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Fendy_Ramadhani-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/fendy-ramadhani9/)
[![Medium](https://img.shields.io/badge/Medium-@FendyRamadhani-000000?style=for-the-badge&logo=medium&logoColor=white)](https://medium.com/@FendyRamadhani)
[![Email](https://img.shields.io/badge/Email-fendyramadhani9@gmail.com-EA4335?style=for-the-badge&logo=gmail&logoColor=white)](mailto:fendyramadhani9@gmail.com)

</div>

> *"Discipline equals freedom. Automate the routine, optimize recovery, and build with resilient infrastructure."*

---

<div align="center">
  <sub>Built with high discipline for high performers. © 2026 by <b>Fendy Ramadhani</b>.</sub>
</div>
