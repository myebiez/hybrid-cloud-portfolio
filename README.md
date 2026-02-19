# ☁️ High-Availability Hybrid Cloud Cluster (Edge-to-Cloud Architecture)

![Architecture Badge](https://img.shields.io/badge/Architecture-Hybrid_Cloud-blue)
![Networking Badge](https://img.shields.io/badge/Networking-WireGuard_Mesh-red)
![DB Badge](https://img.shields.io/badge/Database-MariaDB_Replication-success)
![Status](https://img.shields.io/badge/Status-Fully_Operational-brightgreen)

## 📖 Ringkasan Proyek
Proyek ini merupakan implementasi nyata dari arsitektur **Hybrid Cloud** yang dirancang untuk memiliki ketersediaan tinggi (*High Availability*) dan kapabilitas *self-healing*. Sistem ini mengintegrasikan Cloud VPS publik dengan perangkat *Edge* lokal (Raspberry Pi) yang beroperasi di balik jaringan NAT (ISP rumahan). 

Fokus utama dari infrastruktur ini adalah mewujudkan **Zero-Downtime Failover**, **Replikasi Data Real-Time**, dan **Sistem Jaringan Terenkripsi** (*Secure Tunneling*). Proyek ini membuktikan bahwa arsitektur dengan reliabilitas setara skala *Enterprise* dapat dibangun menggunakan pemanfaatan perangkat keras *Edge* yang terjangkau melalui pendekatan *engineering* yang presisi.

---

## 🏗️ Topologi Arsitektur

Infrastruktur ini menggunakan topologi VPN *Hub-and-Spoke* (subnet `10.8.0.x`) untuk menghubungkan setiap *node* secara aman di dalam satu jaringan *overlay*.

1. **Cloud Node (Hub):** Ubuntu VPS (Public IP)
   - **Peran:** VPN Gateway, Nginx *Load Balancer*, dan Server Observabilitas Terpusat (Prometheus + Grafana).
2. **Edge Node 1 (Spoke A):** Raspberry Pi 3
   - **Peran:** Web Server Worker A & Database MariaDB (Master).
3. **Edge Node 2 (Spoke B):** Raspberry Pi Zero 2 W
   - **Peran:** Web Server Worker B & Database MariaDB (Slave).

---

## 🚀 Fitur Utama & Pencapaian Sistem

### 🛡️ 1. Jaringan "NAT Punching" yang Aman (WireGuard)
- **Tantangan:** Perangkat *Edge* lokal tidak dapat diakses dari internet akibat batasan NAT dan *firewall* dari penyedia layanan internet.
- **Implementasi:** Membangun *Virtual Private Network* (VPN) Mesh menggunakan WireGuard (UDP 51820) dengan konfigurasi `PersistentKeepalive` untuk menjaga koneksi NAT tetap terbuka.
- **Hasil:** Terbentuknya terowongan privat yang terenkripsi secara *end-to-end*. Hal ini memungkinkan komunikasi dua arah antara *Cloud* dan *Edge* tanpa perlu melakukan konfigurasi *port-forwarding* pada *router* fisik.
![WireGuard VPN Status](images/wireguard_status.png)

### ⚖️ 2. Zero-Downtime Load Balancing (Nginx)
- **Implementasi:** Mengonfigurasi Nginx di VPS sebagai *Reverse Proxy* untuk mendistribusikan *traffic* menggunakan algoritma *Round-Robin* ke *Edge nodes* melalui jalur VPN.
- **Tuning High Availability:** Melakukan kustomisasi parameter *health check* pasif menggunakan `proxy_connect_timeout 2s` dan `max_fails=1`. 
- **Hasil:** *Instant Failover* (< 2 detik). Jika salah satu *node* mengalami kegagalan perangkat keras atau koneksi jaringan, sistem akan secara otomatis merutekan ulang *traffic* ke *node* yang sehat. Pengguna akhir tidak akan mengalami gangguan interupsi layanan atau melihat halaman *error* `502 Bad Gateway`.
![Nginx Failover Test](images/failover_test.gif)

### 💾 3. Replikasi Data Real-Time (MariaDB)
- **Implementasi:** Mengaktifkan topologi *Master-Slave Asynchronous Replication* pada MariaDB yang merutekan *traffic* lintas server melalui VPN.
- **Hasil:** Redundansi data seketika. Setiap transaksi penulisan (*write*) di *node* Master (Raspi 3) secara otomatis direplikasi ke *node* Slave (Raspi Zero 2 W). Hal ini menjamin keamanan data (*Disaster Recovery*) apabila terjadi kerusakan perangkat keras (*hardware failure*) pada server utama.
![Database Replication](images/db_replication.gif)

### 📊 4. Ruang Pemantauan Terpusat (Observability Stack)
- **Implementasi:** Melakukan instalasi agen `Node Exporter` pada seluruh *node*. Metrik sistem kemudian ditarik (*scraped*) setiap 15 detik oleh `Prometheus` di VPS dan divisualisasikan menggunakan `Grafana`.
- **Hasil:** Terciptanya *dashboard* terpusat untuk memantau metrik kritikal (*CPU Load*, Utilisasi RAM, dan *Network Traffic*) pada seluruh ekosistem *cluster*. Menggunakan dua pendekatan tampilan:
  - **Cluster Overview:** Untuk memantau status operasional (*Resource, Uptime*) dari ke-3 mesin secara bersamaan dalam format tabular.
  - **Node Detail (Drill-Down):** Untuk keperluan analitik dan investigasi metrik secara mendalam pada mesin atau arsitektur CPU tertentu.

*(Cluster Overview)*
![Grafana Cluster Overview](images/grafana_cluster.png)

*(Detailed Node View)*
![Grafana Node Detail](images/grafana_detail.png)

---

## 💡 Keputusan Arsitektural: Adaptasi Keterbatasan Hardware

Pada tahap perancangan awal, *Monitoring Stack* (Prometheus & Grafana) direncanakan berjalan secara lokal di *Edge Node 2* (Raspberry Pi Zero 2 W). Namun, keterbatasan spesifikasi keras (arsitektur prosesor 32-bit dan memori 512MB) menyebabkan *Out-Of-Memory* (OOM) dan sistem *crash* (`signal=ILL`) saat mesin melakukan instruksi *rendering* pada Grafana UI.

**Tindakan Resolusi (*Pivoting*):** Untuk menjaga stabilitas *cluster*, saya melakukan pembersihan menyeluruh (*purge*) sistem pemantauan dari perangkat *Edge* sehingga memori dapat didedikasikan sepenuhnya untuk operasional Web Server dan *Database Slave*. 

Beban kerja pengumpulan data dan visualisasi kemudian **dimigrasikan ke Cloud VPS**. Prometheus di VPS dikonfigurasi untuk menarik data (*remote scraping*) dari *Edge nodes* secara mulus melalui *tunnel* WireGuard. Adaptasi ini terbukti mengeliminasi *bottleneck* performa dan secara akurat mencerminkan standar distribusi beban kerja di tingkat Enterprise (*Heavy processing* berada di Cloud, sementara agen *Lightweight* berada di *Edge*).

---

## 👨‍💻 Tentang Saya
Saya adalah mahasiswa ilmu Komputer (*Computer Science*) dengan minat yang mendalam pada *Cloud Infrastructure, DevOps*, dan *Systems Architecture*. Proyek ini dibangun sebagai implementasi praktis (*hands-on*) dari konsep teoretis *Distributed Systems*, *Networking*, dan *Linux Administration* ke dalam lingkungan produksi yang nyata dan fungsional.

* **Mari Terhubung melalui LinkedIn:** [linkedin.com/in/fadhil-abie-2987323a9](https://linkedin.com/in/fadhil-abie-2987323a9)
