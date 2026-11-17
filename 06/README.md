# Materi Minggu 6: Distributed File System

**AI yang digunakan:** Claude (Sonnet 5.5, Anthropic)

**Prompt:** "Buatkan rangkuman materi minggu 6 sesuai agenda pembahasan pada file `06--distributed-filesystem.pdf`, dalam format markdown untuk README.md di direktori `06` repo `disdec-teori`."

---

## 1. Pengertian File System (FS)
Metode yang dipakai OS untuk mengelola dan mengatur file pada media penyimpan (HDD, flashdisk, SSD).
- Linux: ext4, XFS, btrfs, ReiserFS. Windows/DOS: FAT16, FAT32, NTFS. Mac: APFS.
- FS diakses lewat driver di kernel yang menyediakan *virtual file system* untuk aplikasi di user space.
- Alur: Aplikasi → System Call Interface → Virtual FS → driver kernel → hardware.
- FS berbeda dengan **hirarki FS** (pengaturan file dan direktori di OS, mis. standar FHS); pengguna hanya melihat hirarkinya.

## 2. Pengertian Distributed File System (DFS)
Bukan sekadar "FS di banyak mesin". Secara konseptual, DFS adalah abstraksi terkoordinasi dari sekumpulan media penyimpan yang menyembunyikan heterogenitas, kegagalan, dan konkurensi, sehingga menjadi penyimpanan terintegrasi, andal, dan berkinerja tinggi. Dari sisi pengguna tampak sebagai **satu** media penyimpan.

### DAS, NAS, SAN, dan DFS
- **DAS (Direct-Attached Storage):** penyimpanan terhubung langsung ke satu server.
- **NAS (Network-Attached Storage):** penyimpanan berbasis file yang diakses lewat jaringan.
- **SAN (Storage Area Network):** menyajikan penyimpanan sebagai block device yang dihubungkan langsung oleh server dan diformat dengan FS sendiri. Umum untuk backend database besar, editing video/audio, dan virtualisasi skala besar. Secara konsep bisa menjadi storage DFS, tetapi tidak disarankan.
- **DFS:** abstraksi level *software* yang mengelola data sebagai file dan direktori yang tersebar di banyak node; lebih mengarah ke DAS pada setiap node.

## 3. DFS yang Umum
Google File System, Colossus (pengganti GFS; dipakai YouTube, Gmail, Google Docs), CephFS, Gluster, HDFS (Hadoop), Lustre.

## 4. Studi Kasus: HDFS
- FS open source, scalable, terdistribusi, dan fault-tolerant untuk menyimpan dataset besar di klaster **commodity hardware** (perangkat massal, murah, mudah dipertukarkan).
- Komponen utama Apache Hadoop untuk pemrosesan data skala besar: file **dipecah menjadi blok**, disebar ke beberapa server, dan **direplikasi** untuk high availability.
- Menurut slide, Apache Hadoop 3.5.0 dikompilasi dengan JDK 17; server berjalan di Java 17, client di Java 17 atau 21.
- Praktik penggunaan HDFS mengikuti dokumen di EL-UTDI.

## 5. Cloud Storage
Layanan online untuk menyimpan dan mengakses data di server jarak jauh lewat internet, bukan di perangkat lokal. Dikelola sebagai pool logis, penyimpanan fisik di banyak server (kadang banyak lokasi) yang dimiliki dan dikelola penyedia cloud. Contoh: Amazon S3, Azure Blob Storage, Google Cloud Storage (GCS).

## 6. Relasi Cloud Storage dengan DFS
Cloud storage adalah **tempat** penyimpanan data, sedangkan pengorganisasian (operasi FS) dilakukan oleh DFS. Contoh: S3 bisa dipakai HDFS; S3 sebagai cloud storage, operasi FS oleh HDFS, dikonfigurasi lewat `core-site.xml`.

## 7. Tipe Cloud Storage
- **Object storage**
- **File storage**
- **Block storage**

## 8. Studi Kasus Cloud Storage
Slide menampilkan arsitektur CubeFS (cubefs.io) sebagai contoh. Instalasi salah satu object storage dilakukan sebagai praktik: _[tuliskan hasil praktik Anda di sini bila diminta]_
