# Materi Minggu 5: Fault Tolerance

**AI yang digunakan:** Claude (Sonnet 5.5, Anthropic)

**Prompt:** "Buatkan rangkuman materi minggu 5 sesuai agenda pembahasan pada file `05--fault-tolerant-sys.pdf`, dalam format markdown untuk README.md di direktori `05` repo `disdec-teori`."

---

## 1. Pengertian Fault Tolerance
Kemampuan sistem mempertahankan operasi yang benar meskipun satu atau lebih komponennya mengalami kegagalan. Penting untuk sistem *high availability*, *mission critical*, dan *life-critical*.

## 2. Arti Penting pada Sistem Terdistribusi-Terdesentralisasi
Sistem punya banyak komponen dan koneksi antar komponen bisa bermasalah. Semakin kritis aplikasi, semakin tinggi tuntutan high availability sehingga kegagalan harus diantisipasi.

## 3. Jenis Fault
| Jenis | Penjelasan |
|---|---|
| Transient | Sementara, mis. karena petir |
| Intermittent | Muncul dan hilang berulang, mis. hardware overheat |
| Permanent | Bersifat tetap |
| Byzantine | Komponen memberi hasil acak/tidak konsisten |

## 4. Klasifikasi Fault Berdasarkan Sumber
- **Hardware:** disk crash, power failure.
- **Software:** bug, memory leak, race condition.
- **Communication:** packet loss, network partition.

## 5. Design Pattern untuk Fault Tolerance
1. **Redundansi dan Replikasi:** resource direplikasi ke banyak node agar tidak ada single point of failure (lihat minggu 4).
2. **Load Balancing:** menjalankan aplikasi lebih dari satu instance dan membagi beban lewat load balancer (HAProxy, nginx).
3. **Checkpointing dan Rollback Recovery:** status sistem disimpan berkala (checkpoint); setelah gagal, sistem dikembalikan ke checkpoint terakhir sehingga kehilangan data dan waktu proses minimal.
4. **Heartbeat dan Failure Detection:** komponen mengirim sinyal periodik; jika monitor tidak menerima heartbeat dalam jangka waktu tertentu, komponen dianggap gagal. Rujukan: script `check-server.py`.
5. **Retry:** library `tenacity` (Python) untuk mengulang koneksi ke komponen eksternal yang bisa gagal. Rujukan: `check-retry.py`.
6. **Circuit Breaker:** mencegah kegagalan berantai di microservices dengan state machine tiga status:
   - **Closed:** permintaan diizinkan dan kegagalan dipantau.
   - **Open:** ambang kegagalan tercapai; permintaan langsung ditolak agar layanan sempat pulih.
   - **Half-open:** setelah batas waktu, sebagian permintaan uji dilewatkan. Jika berhasil kembali ke *closed*, jika gagal kembali ke *open*.
7. **Consensus dan Coordination:**
   - Consensus: beberapa node menyepakati satu nilai/status/urutan operasi walau ada kegagalan dan keterlambatan jaringan.
   - Coordination: pengelolaan interaksi antar node secara lebih luas, sering memakai consensus.
   - Contoh: leader election database dengan Raft; update file di Google Drive di-commit ke mayoritas server; validasi transaksi blockchain.

## 6. Actor Model untuk Supervisi Proses
Aplikasi dibangun dari *actor* independen yang berkomunikasi lewat pesan, dengan pola **Supervisor-Worker**: supervisor memantau worker dan memulai ulang bila gagal.

## 7. Implementasi pada Berbagai Bahasa/Peranti
| Bahasa | Dukungan |
|---|---|
| Erlang/Elixir | Actor model dan Supervisor-Worker bawaan |
| Pony | Bahasa khusus actor model |
| Python | Xoscar, tenacity, circuit breaker library, dll. |
| Java | Apache Pekko (actor model, Supervisor-Worker), Resilience4j |
| Lainnya | Cari library di npm (JavaScript), crates.io (Rust), dll. |
