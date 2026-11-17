# Materi Minggu 1: Pengenalan Sistem Terdistribusi dan Terdesentralisasi

**AI yang digunakan:** Claude (Sonnet 5.5, Anthropic)

**Prompt:** "Buatkan rangkuman materi minggu 1 sesuai agenda pembahasan pada file `01--pengenalan.pdf` (Sistem Terdistribusi dan Terdesentralisasi), dalam format markdown untuk README.md di direktori `01` repo `disdec-teori`."


---

## 1. Arti Penting Sistem Terdistribusi dan Terdesentralisasi
- Data dan pengguna tumbuh sangat besar, sehingga layanan modern butuh **skalabilitas**.
- Sistem harus **fault-tolerant**, dengan performa dan ketersediaan 24/7.
- Tanpa sistem terdistribusi/terdesentralisasi, layanan seperti Google, Netflix, dan Blockchain tidak mungkin ada.
- **Skalabilitas**: kemampuan sistem menangani peningkatan beban kerja, pengguna, atau permintaan tanpa menurunkan kinerja/kualitas.
- **Fault-tolerant**: sistem tetap beroperasi normal meski sebagian komponen (hardware/software) gagal.

## 2. Pengertian Sistem Terdistribusi
> *A collection of independent computers that appears to its users as a single coherent system* (Tanenbaum & van Steen, 2007).

Ciri utama: komponen independen, berkomunikasi lewat jaringan, dan menawarkan **transparansi** (lokasi, akses, replikasi, kegagalan).

## 3. Arsitektur Sistem Dasar
| Arsitektur | Penjelasan |
|---|---|
| Client-Server | Klien meminta layanan, server menyediakan. |
| Peer-to-Peer (P2P) | Setiap node setara, bisa menjadi klien sekaligus server. |
| Hybrid | Gabungan keduanya (mis. server pusat untuk pencarian, transfer data P2P). |

## 4. Karakteristik Sistem Terdistribusi
Transparansi lokasi, skalabilitas, fault tolerance, kinerja tinggi, dan heterogenitas (perangkat/OS/jaringan yang beragam).

## 5. Pengertian Sistem Terdesentralisasi
- Sentralisasi: satu titik kontrol. Desentralisasi: kontrol didistribusikan, **tidak ada single point of failure**.
- Keputusan diambil kolektif melalui **konsensus**.
- Contoh: Blockchain, DNS, BitTorrent.

## 6. Terdistribusi vs Terdesentralisasi
| | Terdistribusi | Terdesentralisasi |
|---|---|---|
| Fokus | Pembagian **beban kerja** | Pembagian **kontrol** |
| Contoh | Google File System, MinIO Object Storage | Blockchain |

Keduanya saling terkait tetapi berbeda fokus, dan keduanya menjadi fondasi teknologi modern.

## 7. Studi Kasus
- **Terdistribusi:** Google File System / HDFS (penyimpanan data besar), Apache Kafka dan Apache Pulsar (messaging & streaming), Netflix, Twitch, YouTube, Instagram, Threads, Facebook, TikTok.
- **Terdesentralisasi:** Blockchain (Bitcoin, Ethereum, Algorand, Sui, Near, Aptos, dll.), DNS, BitTorrent.

## 8. CAP Theorem (Eric Brewer, 2000)
- **C (Consistency):** setiap pembacaan mengembalikan penulisan terbaru atau error.
- **A (Availability):** setiap permintaan ke node yang tidak gagal mendapat respons non-error, meski bukan data terbaru.
- **P (Partition Tolerance):** sistem tetap berfungsi walau komunikasi antar node terputus/terfragmentasi.
- Pada distributed data store, C dan A tidak bisa dijamin bersamaan karena partisi pasti bisa terjadi. Saat partisi, perancang harus memilih:
  - **AP**: permintaan selalu dilayani, tetapi data bisa tidak sinkron (mengorbankan konsistensi).
  - **CP**: read/write ditolak demi menjaga konsistensi (mengorbankan availability).
- Slide menyebut MongoDB sebagai contoh AP dan Apache Cassandra sebagai contoh CP.

> **Catatan AI (di luar slide):** di literatur dan dokumentasi vendor, penggolongan ini umumnya terbalik: Cassandra lazim disebut AP (tunable consistency) dan MongoDB dengan konfigurasi default lazim disebut CP. Slide minggu 4 juga menyebut Cassandra sebagai contoh *eventual consistency*. Sebaiknya Anda verifikasi sendiri sebelum dikumpulkan.

## 9. Konsistensi Data
- **Strong consistency:** hasil selalu sama (contoh: perbankan).
- **Eventual consistency:** lama-lama konsisten (contoh: media sosial).

## 10. Tantangan
- **Latency** (lama pemrosesan) dan **throughput** (jumlah yang diproses).
- Keamanan: DDoS dan **Sybil attack** (membuat banyak identitas palsu untuk mendominasi pengaruh/suara/reputasi, terutama pada sistem P2P/blockchain).
- Kompleksitas manajemen node.
- Governance (tata kelola) pada sistem terdesentralisasi.

## 11. Tren Masa Depan
Cloud-native (Kubernetes, Docker), Edge & Fog Computing, Web3 & DeFi, Federated Learning (AI terdistribusi).

## 12. Ringkasan
- Terdistribusi = berbagi beban kerja; terdesentralisasi = berbagi kontrol.
- Keduanya terkait namun berbeda fokus, dan menjadi fondasi teknologi modern.

## 13. Diskusi
1. Mengapa dunia semakin bergerak menuju distribusi dan desentralisasi?
2. Contoh sehari-hari yang paling terasa: _[isi dengan pendapat Anda sendiri]_
