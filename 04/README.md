# Materi Minggu 4: Konsistensi dan Replikasi pada Sistem Terdistribusi

**AI yang digunakan:** Claude (Sonnet 5.5, Anthropic)

**Prompt:** "Buatkan rangkuman materi minggu 4 sesuai agenda pembahasan pada file `04--konsistensi-dan-replikasi.pdf`, dalam format markdown untuk README.md di direktori `04` repo `disdec-teori`."

---

## 1. Konsistensi pada Sistem Terdistribusi
Menjamin semua node punya data yang sama (atau tampak sama) pada waktu tertentu walau data tersebar di banyak server dan bisa diubah bersamaan. Jika satu server diubah, server lain harus akhirnya memiliki data yang sama.

**Mengapa penting:** tanpa konsistensi, pengguna bisa melihat versi data berbeda tergantung server tempat membaca. Contoh: transfer rekening A ke B, server 1 sudah mengurangi saldo A tetapi server 2 belum menambah saldo B, sehingga transaksi dari B bisa gagal karena saldo "kurang".

## 2. Teorema CAP
Partisi jaringan itu wajar terjadi, sehingga kondisi ideal CAP sekaligus tidak mungkin.
- **Consistency:** semua node punya data sama.
- **Availability:** data tersedia saat dibutuhkan.
- **Partition Tolerance:** sistem tetap jalan walau ada komponen jaringan gagal/terputus.

**Contoh e-commerce (server A melayani Indonesia barat, B timur) saat jaringan keduanya terputus:**
- Pilih **AP**: pengguna tetap bisa belanja di kedua wilayah, tetapi stok sementara tidak sinkron.
- Pilih **CP**: transaksi ditolak sampai jaringan pulih agar stok konsisten (mengorbankan availability).

## 3. Model Konsistensi
Aturan/jaminan tentang bagaimana dan kapan perubahan data terlihat oleh node atau klien lain.

| # | Model | Penjelasan | Contoh |
|---|---|---|---|
| 1 | Strong | Setelah update, semua node langsung melihat versi terbaru; biasanya memakai Two-Phase Commit atau Paxos | Perbankan |
| 2 | Eventual | Tidak langsung sama, tetapi akhirnya sama jika tidak ada update baru; cocok untuk availability dan kecepatan | DynamoDB, Cassandra |
| 3 | Causal | Operasi yang saling bergantung dijaga urutannya; yang tidak terkait boleh terlihat berbeda urutan | Komentar tidak boleh muncul sebelum postingan |
| 4 | Read-Your-Writes | Setelah menulis, pembacaan berikutnya oleh penulis yang sama melihat tulisannya; orang lain belum tentu | Pesan langsung tampil di layar pengirim |
| 5 | Monotonic Read | Pembacaan berulang tidak pernah melihat versi lebih lama dari yang sudah dilihat | Timeline tidak "mundur" saat refresh |
| 6 | Sequential | Semua operasi tampak dieksekusi dalam urutan sama untuk semua node, tidak harus real-time | Semua pengguna melihat urutan postingan sama |
| 7 | Linearizability | Varian terkuat strong consistency; operasi tampak atomik dan instan, sistem tampak seperti satu node tunggal | etcd, ZooKeeper, Spanner; cek keunikan username/ID |

**BASE** (pada eventual consistency): *Basically Available* (tetap bisa dibaca/ditulis saat kegagalan sebagian), *Soft state* (replika tidak harus konsisten setiap saat), *Eventual consistency*.

## 4. Trade-off Latensi dan Konsistensi
Konsistensi ↑ → latensi ↑; konsistensi ↓ → latensi ↓.

Contoh dua pusat data (Jakarta dan Yogyakarta), pengguna mengubah data di Jakarta:
- **Strong:** tunggu Jakarta dan Yogyakarta sama persis sebelum menjawab. Data konsisten, respons lambat.
- **Eventual:** jawab langsung, update ke Yogyakarta menyusul. Respons cepat, data bisa sementara berbeda.

## 5. Replikasi Data
Membuat dan memelihara salinan data yang sama di beberapa node/server.

**Tujuan:** ketersediaan (availability), kinerja (baca dari server terdekat), reliabilitas (risiko kehilangan data turun), skalabilitas (beban baca dibagi).

## 6. Jenis Replikasi
| Jenis | Penjelasan | Contoh |
|---|---|---|
| Master-Slave (Primary-Secondary) | Satu master menulis, slave hanya membaca. Konsistensi kuat tetapi master bisa jadi bottleneck | PostgreSQL streaming replication |
| Multi-Master (Peer-to-Peer) | Semua node bisa baca dan tulis. Fleksibel tetapi rawan konflik | PostgreSQL logical replication, Cassandra, MongoDB, Apache Ignite |
| Synchronous | Update ditulis ke semua node sebelum dianggap selesai. Konsisten tetapi lambat | Google Spanner |
| Asynchronous | Update dikirim setelah transaksi selesai di master. Cepat tetapi ada delay data | MySQL async, DynamoDB |

## 7. Strategi Sinkronisasi Replika
- **Berdasarkan waktu:** asynchronous (tercepat, risiko kehilangan data tertinggi), semi-synchronous (menunggu satu replika, seimbang; dipakai MySQL), synchronous (menunggu semua replika, konsistensi tertinggi, performa terendah).
- **Berdasarkan arah:** master-slave, master-master, master-master asymmetric.
- **Berdasarkan cara:** pull-based (replika memeriksa update secara periodik), push-based (sumber mengirim saat ada update, memakai CDC/Change Data Capture dengan WebSocket atau SSE), hybrid.
- **Berdasarkan penanganan konflik:** LWW, Vector Clock, CRDT, Operational Transformation.

### Penanganan Konflik
- **LWW (Last Write Wins):** timestamp terbaru yang dipakai, lainnya diabaikan.
- **Vector Clock:** pengurutan memakai vector clock (lihat minggu 3).
- **CRDT:** struktur data yang bisa digabung otomatis tanpa konflik. Contoh library: Yjs (editor kolaboratif seperti Google Docs).
  - Contoh hitung pengunjung: A mencatat `{A:5, B:0}`, B mencatat `{A:0, B:3}`. Saat sinkron, tiap entri diambil max → `{A:5, B:3}`, total = 8.
- **OT (Operational Transformation):** operasi ditransformasi sesuai kondisi dokumen lokal.
  - Contoh dokumen "abc": A menyisipkan "x" di indeks 0 (jadi "xabc"), B menghapus "c" di indeks 2. Tanpa transformasi hasilnya berbeda ("xac" vs "xab"). Dengan OT, perintah hapus milik B ditransformasi menjadi indeks 3 pada sisi A sehingga keduanya menghasilkan "xab".

## 8. Protokol Replikasi
1. **Primary-based (Master-Slave):** satu node baca-tulis, lainnya baca.
2. **Quorum-based:** operasi dianggap berhasil jika dikonfirmasi sejumlah node tertentu. Syarat: **W + R > N**.
   - Contoh N=3, W=2, R=2 (2+2>3). Tulis `x=100` ke A, B, C paralel; A dan B ACK, C lambat. Karena sudah 2 ACK (W=2), tulis sukses. Saat membaca dari 2 node, minimal satu pasti ikut dalam penulisan terbaru (irisan), sehingga data terbaru terbaca.
3. **State Machine Replication:** semua replika mengeksekusi perintah sama dalam urutan sama. Leader memberi nomor urut, mencatat di log, mengirim ke replika; setelah dikonfirmasi ia dianggap *committed*, lalu semua replika mengeksekusi (contoh `SET x = 20`) dan leader membalas ke klien.
4. **Gossip Protocol:** tiap node secara acak memilih 1-3 peer setiap detik dan membagikan statusnya; melalui beberapa putaran semua node tahu perubahan. Dipakai Cassandra untuk cek keberadaan node dan replikasi data.
5. **Chain Replication:** replika disusun berantai; tulis masuk dari kepala rantai, baca dari ekor.

## 9. Studi Kasus
PostgreSQL 18 (streaming replication) dan Apache Ignite 3 (master-master), mengacu pada lecture notes dan source code pendamping.

## 10. Diskusi
1. Kelebihan dan kekurangan strong vs eventual consistency: _[isi analisis Anda; poin dasar: strong = data selalu benar tetapi latensi tinggi; eventual = cepat dan available tetapi data bisa sementara usang]_
2. Jika replikasi gagal di satu node: _[isi jawaban Anda; petunjuk: lihat quorum, failover, dan resync/catch-up]_
