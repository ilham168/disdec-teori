# Materi Minggu 3: Sinkronisasi dan Koordinasi pada Sistem Terdistribusi

**AI yang digunakan:** Claude (Sonnet 5.5, Anthropic)

**Prompt:** "Buatkan rangkuman materi minggu 3 sesuai agenda pembahasan pada file `03--sinkronisasi-sistem-terdistribusi.pdf`, dalam format markdown untuk README.md di direktori `03` repo `disdec-teori`."

---

## 1. Apa itu Sinkronisasi?
Koordinasi proses, data, dan sumber daya di banyak node agar operasi konsisten dan andal; mencegah konflik data (*data race*) dan menjaga perilaku sistem tetap koheren.

**Contoh race condition (e-commerce):** stok produk X = 10. User A dan B sama-sama membaca stok 10, lalu masing-masing mengurangi menjadi 9. Seharusnya stok 8 setelah dua pembelian, tetapi tercatat 9.

**Mengapa penting:** integritas data, sinkronisasi state (mis. posisi pemain game, keranjang belanja lintas perangkat), koordinasi task, pengelolaan resource, mekanisme recovery, utilisasi efisien, load balancing, pencegahan deadlock, dan operasi yang scalable.

## 2. Tantangan Sinkronisasi
Network latency dan partitioning, skalabilitas, fault tolerance, concurrency control, konsistensi data, dan sinkronisasi waktu.

## 3. Sinkronisasi Waktu Fisik
Menyelaraskan real-time clock dengan standar umum (penting untuk korelasi log, penjadwalan event, dan konsistensi sistem).

- **Standar waktu:** GMT (tidak digunakan lagi); **UTC** (standar sejak 1963, berbasis **TAI**, International Atomic Time); LTC (Coordinated Lunar Time, masih dikembangkan).
- **Rantai sumber waktu:** master atomic clock → atomic clock di satelit GPS → time signal GPS → GPS dongle → server NTP/PTP → client (Linux, Windows, Mac, ponsel).
- **Sumber atomic clock:** sekitar 31 satelit GPS (U.S. Space Force, MEO), Deep Space Atomic Clock NASA (2019), misi ACES di ISS, dan master clock darat (U.S. Naval Observatory dan Schriever Space Force Base).
- **NTP (Network Time Protocol)** dan **PTP (Precision Time Protocol, IEEE 1588)** yang lebih presisi. IEEE 1588-2008 (PTP v2) tidak backward compatible dengan 2002; 1588-2019 menambah perbaikan yang kompatibel dengan 2008. IEEE 802.1AS (gPTP) untuk AVB/TSN. Syarat PTP presisi tinggi: NIC mendukung *hardware timestamping*.

## 4. Sinkronisasi Waktu Logis
Menetapkan **urutan sebab-akibat** (causal order) tanpa terkait waktu nyata. Prinsip transitivitas: jika A → B dan B → C maka A → C. Dasarnya adalah Lamport clock (lihat minggu 2).

## 5. Vector Clocks
Setiap proses menyimpan vektor berukuran jumlah proses (A index 0, B index 1, C index 2); awalnya `[0,0,0]`. Setiap event menaikkan elemen milik sendiri. Saat menerima pesan: ambil elemen-wise max, lalu naikkan elemen sendiri.

**Contoh dari slide:**
1. A internal event → `[1,0,0]`; A mengirim ke B dengan clock `[2,0,0]`.
2. B menerima: max → `[2,0,0]`, lalu B mengirim ke C dengan `[2,1,0]`.
3. C menerima → `[2,1,0]`, lalu internal event → `[2,1,1]`, event lagi → `[2,1,2]`.
4. Posisi akhir: A `[2,0,0]`, B `[2,1,0]`, C `[2,1,2]`.

**Aturan:** X terjadi sebelum Y jika semua elemen X ≤ elemen Y dan minimal satu lebih kecil. Jika tidak, keduanya **concurrent** (independen).

> **Catatan AI (koreksi):** slide menyimpulkan bahwa A `[2,0,0]` dan C `[2,1,2]` "independen". Dengan aturan di atas, `[2,0,0]` ≤ `[2,1,2]` dan ada elemen yang lebih kecil, sehingga A **terjadi sebelum** C (sebab-akibat A → B → C). Contoh concurrent yang benar misalnya `[1,0,0]` vs `[0,1,0]`. Mohon dicek ulang bersama dosen/slide.

Implementasi: lihat `vclocks.py` yang menyertai materi.

## 6. Mutual Exclusion
Memastikan hanya satu proses yang berada di **critical section (CS)** pada satu waktu untuk menjaga integritas sumber daya bersama (printer, file, database).

**Prinsip:** Mutual Exclusion (hanya 1 proses di CS), Progress (jika CS kosong, sumber daya bisa diakses), Bounded Waiting (tidak menunggu selamanya), Fairness (dilayani sesuai urutan wajar).

| Algoritma | Cara kerja | Kelebihan | Kekurangan |
|---|---|---|---|
| Centralized | Satu koordinator memberi/menarik izin akses | Sederhana, biaya rendah | Single point of failure, bottleneck |
| Ricart-Agrawala | Terdistribusi penuh; kirim request ber-timestamp, masuk CS jika semua balas OK; butuh 2×(N−1) pesan per CS | Fault tolerant | Overhead pesan tinggi |
| Token-based | Token unik beredar; hanya pemegang token boleh masuk CS | Hanya 1 pesan per entri CS | Token hilang sulit dideteksi, perlu mekanisme pemulihan |

## 7. Pemilihan Koordinator (Leader Election)
Koordinator dibutuhkan untuk mengelola sumber daya bersama, menyinkronkan operasi, dan membuat keputusan global. Jika koordinator gagal atau ada node baru bergabung, pemimpin baru dipilih otomatis.

**Algoritma:** Bully, Ring, dan berbasis konsensus (Raft, Paxos).
**Contoh industri:** Apache ZooKeeper memakai Zab (ZooKeeper Atomic Broadcast, mirip Paxos); Kubernetes memakai Raft melalui etcd.
