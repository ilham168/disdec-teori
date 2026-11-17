# Materi Minggu 2: Proses dan Komunikasi Antar Proses pada Sistem Terdistribusi

**AI yang digunakan:** Claude (Sonnet 5.5, Anthropic)

**Prompt:** "Buatkan rangkuman materi minggu 2 sesuai agenda pembahasan pada file `02--proses-dan-komunikasi-antar-proses.pdf`, dalam format markdown untuk README.md di direktori `02` repo `disdec-teori`."
---

## 1. Proses dan Manajemen Proses
Proses adalah program yang sedang dieksekusi. Manajemen proses mencakup **process creation**, **process migration**, dan **process monitoring**.

## 2. Sistem Terdistribusi dan Komunikasi
- Sistem terdistribusi = kumpulan node independen yang bekerja sebagai satu kesatuan logis; setiap node menjalankan proses.
- Agar bekerja sebagai satu sistem, proses antar node harus **berkomunikasi**.
- Contoh: game multiplayer; Gojek/Grab (UI, layanan driver, location tracking, perhitungan biaya); chat/video call WhatsApp (point-to-point pada infrastruktur terdistribusi).

## 3. Model Komunikasi Pesan
- **Langsung:** memakai socket.
- **Tidak langsung:** mailbox / message queue.
- **Pub/Sub:** publisher mengirim ke topik, subscriber menerima.

## 4. Proses pada Sistem Terdistribusi dan IPC
Proses bisa berada di node berbeda dan **tidak ada shared memory**, sehingga diperlukan IPC khusus. Ada 4 jenis IPC:
1. Shared Memory (tidak cocok untuk sistem terdistribusi karena memori antar node tidak bisa diakses langsung)
2. Message Passing
3. Remote Procedure Call (RPC)
4. Middleware

### Message Passing
Standar: **MPI (Message Passing Interface)**; versi terakhir 5.0 (per Oktober 2026 menurut slide). Implementasi Python: `mpi4py`.

### RPC (Remote Procedure Call)
Memanggil prosedur di mesin lain seolah-olah fungsi lokal. Contoh: **gRPC**.

### MOM (Message-Oriented Middleware)
Infrastruktur untuk pertukaran data lewat pesan mandiri (*message*) secara **asinkron**. Standar/protokol:
- X/Open XATMI
- AMQP (umumnya aplikasi enterprise)
- HLA (IEEE 1516)
- MQTT (perangkat dengan sumber daya terbatas)
- DDS (OMG)
- XMPP

## 5. Studi Kasus: Microservices
Aplikasi dipecah menjadi layanan-layanan kecil yang saling berkomunikasi lewat jaringan (REST/gRPC/message broker).

## 6. Komunikasi Sinkron dan Asinkron
- **Sinkron:** pengirim menunggu respons sebelum lanjut.
- **Asinkron:** pengirim lanjut bekerja tanpa menunggu respons (mis. lewat message queue).

## 7. Masalah pada Komunikasi Antar Proses
| Masalah | Penjelasan | Solusi |
|---|---|---|
| Reliability | Pesan harus terkirim lengkap dan berurutan walau ada packet loss/crash | Acknowledgement, retransmisi, TCP |
| Ordering | Pesan diproses dalam urutan logis | Pipes (FIFO), message queue, shared memory, socket/message passing, RPC |
| Security | Kerahasiaan dan keaslian | Enkripsi, autentikasi |
| Performance | Latency dan bandwidth | Optimasi protokol/jaringan |

## 8. Protokol Komunikasi
- Dasar: TCP, UDP, HTTP.
- Utama (berjalan di atas HTTP sebagai transport): gRPC, GraphQL, REST.

## 9. Konsep Sinkronisasi (Clock)
- Tiap node punya clock lokal sendiri; karena latensi jaringan dan *clock drift*, clock menyimpang sehingga perlu sinkronisasi (untuk urutan kejadian akurat, konsistensi data, dan operasi terkoordinasi).
- Metode umum: sinkronisasi eksternal dengan atomic clock (TAI), GPS, dan UTC, lalu protokol **NTP** atau **PTP** (PTP lebih presisi daripada NTP).

## 10. Logical Clock (Lamport)
Logical clock tidak terkait waktu nyata. Aturan:
1. Setiap kejadian, counter bertambah 1.
2. Saat mengirim pesan, sertakan nilai counter.
3. Saat menerima pesan: ambil nilai terbesar antara counter lokal dan counter pesan, lalu tambah 1.

**Contoh (Sales → Product → Invoice Service):** semua mulai dari 0.
1. Sales mencatat transaksi → clock 1, kirim ke Product dengan timestamp 1.
2. Product menerima: max(0, 1) = 1, lalu +1 = 2, kirim ke Invoice dengan 2.
3. Invoice menerima: max(0, 2) = 2, lalu +1 = 3, membuat faktur.

Urutan benar: Sales (1) → Product (2) → Invoice (3), meski timestamp jam nyata bisa bergeser karena latensi/masalah jaringan/hardware.

Lamport clock masih berupa *scalar clock*. Pengembangan berikutnya: Vector Clocks (Fidge & Mattern, 1988), Plausible Clocks (1999), Chain Clocks (2005), Interval Tree Clocks (2008), Blum Clocks (2019).

## 11. Koordinasi Proses
Mekanisme agar proses independen di banyak komputer bekerja harmonis (komunikasi, sinkronisasi, akses sumber daya bersama) tanpa deadlock atau race condition. Contoh: Apache ZooKeeper, etcd, Consul, ClickHouse Keeper, Oxia, KRaft.

## 12. Tantangan dan Solusi
| Tantangan | Solusi |
|---|---|
| Message loss | Retransmission dan acknowledgement |
| Deadlock | Timeout, algoritma deteksi |
| Data inconsistency | Protokol konsensus (Raft, Paxos) |
| Security | SSL/TLS, OAuth2 |

## 13. Current Issues
Edge & Fog Computing, observability dan tracing (telemetry), orkestrasi container (Kubernetes, Docker Swarm).

## 14. Diskusi
1. Apakah GraphQL contoh RPC? _[isi pendapat Anda]_
2. Apakah REST contoh RPC? _[isi pendapat Anda]_
3. Apakah telemetry solusi yang etis? _[isi pendapat Anda]_
