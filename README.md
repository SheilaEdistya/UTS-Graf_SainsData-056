# UTS-Graf_SainsData-056

# UTS Graf Sains Data — Sistem Rekomendasi Item Berbasis Graf

## Informasi Proyek

| Keterangan | Detail |
|---|---|
| Nama | Sheila Edistya Salsabilla |
| NIM | 23031554056 |
| Kelas | 2023A |
| Mata Kuliah | Graf Sains Data |
| Jenis Tugas | Ujian Tengah Semester (UTS) |
| Program Studi | S1 Sains Data |
| Institusi | Universitas Negeri Surabaya |

## Gambaran Proyek

Proyek ini mengimplementasikan sistem rekomendasi item pada data interaksi e-commerce menggunakan pendekatan **Graph Data Science**. Dataset yang digunakan adalah RetailRocket, yang berisi aktivitas pengunjung terhadap item, seperti melihat produk, menambahkan produk ke keranjang, dan melakukan pembelian.

Data interaksi direpresentasikan sebagai graf menggunakan **Neo4j**, sedangkan Python digunakan untuk pengolahan data, pembentukan *Vector Space Model* (VSM), perhitungan kemiripan antaritem, serta penyusunan rekomendasi menggunakan metode *K-Nearest Neighbors* (KNN).

Hasil rekomendasi kemudian dievaluasi menggunakan metrik **Hit@5** dan **NDCG@5** dengan membandingkan item rekomendasi terhadap data interaksi pada periode pengujian.

**Dataset:** [RetailRocket E-commerce Dataset — Kaggle](https://www.kaggle.com/datasets/retailrocket/ecommerce-dataset)

## Teknologi yang Digunakan

- **Python** — pemrosesan data dan implementasi algoritma rekomendasi.
- **Pandas dan NumPy** — manipulasi data dan operasi numerik.
- **SciPy Sparse Matrix** — penyimpanan matriks interaksi yang jarang.
- **Neo4j dan Cypher** — pemodelan graf serta penyimpanan hubungan antarentitas.
- **Neo4j Python Driver** — koneksi dan eksekusi query Neo4j melalui Python.
- **Scikit-learn** — mendukung kebutuhan analisis dan perhitungan pada tahap pemodelan, jika digunakan pada notebook.
- **Matplotlib dan NetworkX** — visualisasi data atau graf, jika digunakan pada notebook.

## Alur Pengolahan dan Analisis

Secara umum, pengerjaan proyek dilakukan melalui tahapan berikut.

1. **Persiapan dataset** — memuat data interaksi dan memisahkan data berdasarkan periode yang telah ditentukan.
2. **Pemodelan graf** — merepresentasikan pengunjung, item, dan aktivitas interaksi menggunakan node serta relationship di Neo4j.
3. **Pembentukan VSM** — menyusun matriks interaksi User–Item dan Item–User dengan pembobotan aktivitas.
4. **Analisis proyeksi graf** — mengkaji hubungan pengguna dan item untuk menentukan pendekatan proyeksi yang digunakan.
5. **Perhitungan kemiripan item** — menghitung cosine similarity dan menyimpan hubungan kemiripan ke graf melalui relationship `SIMILAR_TO`.
6. **Rekomendasi KNN** — memilih user secara acak dengan seed tetap, mencari tetangga item berdasarkan kemiripan, lalu menyusun ranking rekomendasi.
7. **Evaluasi rekomendasi** — menghitung Hit@5 dan NDCG@5 menggunakan data pengujian.

## Representasi Graf

Graf dibangun dengan dua jenis node utama, yaitu `Visitor` dan `Item`. Interaksi pengunjung direpresentasikan melalui relationship yang sesuai dengan jenis aktivitasnya.

```cypher
(:Visitor {visitorId})
    -[:VIEWED {timestamp}]->
(:Item {itemId})

(:Visitor {visitorId})
    -[:ADDED_TO_CART {timestamp}]->
(:Item {itemId})

(:Visitor {visitorId})
    -[:BOUGHT {timestamp, transactionId}]->
(:Item {itemId})

(:Item)
    -[:SIMILAR_TO {score, common, method}]->
(:Item)
```

Relationship `SIMILAR_TO` digunakan untuk menyimpan kemiripan antaritem berdasarkan hasil perhitungan cosine similarity.

### Bobot Interaksi

Dalam pembentukan matriks interaksi, aktivitas pengunjung diberikan bobot yang berbeda.

| Jenis Aktivitas | Bobot |
|---|---:|
| `VIEWED` | 1 |
| `ADDED_TO_CART` | 3 |
| `BOUGHT` | 5 |

Pembobotan ini digunakan untuk merepresentasikan tingkat interaksi pengguna terhadap item dalam matriks VSM.

## Pembentukan VSM

VSM disimpan dalam format sparse matrix agar penggunaan memori lebih efisien untuk data interaksi yang memiliki banyak nilai nol.

| Berkas | Representasi |
|---|---|
| `VSM_User_P1.npz` | Matriks User × Item |
| `VSM_Item_P1.npz` | Matriks Item × User |
| `user_ids_P1.npy` | Pemetaan indeks baris ke ID user |
| `item_ids_P1.npy` | Pemetaan indeks kolom ke ID item |

Berdasarkan output program, matriks User × Item memiliki **1.407.580 user**, **235.061 item**, dan **2.145.179 interaksi nonzero**.

## Rekomendasi Menggunakan KNN

Rekomendasi dibentuk berdasarkan item yang pernah berinteraksi dengan user. Untuk setiap item dalam riwayat user, sistem mengambil maksimal lima item tetangga dengan nilai similarity tertinggi.

Skor kandidat rekomendasi dihitung dengan mengalikan nilai similarity antara item sumber dan item kandidat dengan bobot interaksi item sumber. Seluruh kontribusi skor dari item-item dalam riwayat user kemudian dijumlahkan untuk menentukan peringkat kandidat rekomendasi.

Item yang sudah terdapat dalam riwayat interaksi user dikeluarkan dari daftar kandidat. Selanjutnya, kandidat diurutkan berdasarkan skor tertinggi untuk menghasilkan lima rekomendasi teratas, kemudian dipilih tiga item sebagai rekomendasi iklan.

### Hasil Rekomendasi

Pemilihan user menggunakan `numpy.random.default_rng(42)` agar proses pengacakan dapat direproduksi.

| Keterangan | Hasil |
|---|---|
| Seed | 42 |
| User terpilih | `119834` |
| Jumlah item dalam riwayat | 3 |
| Jumlah tetangga per item | 5 |
| Jumlah rekomendasi yang ditampilkan | 5 |
| Jumlah item iklan | 3 |

Tiga item iklan yang diperoleh dari peringkat teratas adalah:

1. `90921`
2. `101698`
3. `443320`

Lima item rekomendasi berdasarkan ranking yang dihasilkan program:

| Peringkat | ID Item | Skor |
|---:|---:|---:|
| 1 | `90921` | 1.198874 |
| 2 | `101698` | 0.941690 |
| 3 | `443320` | 0.557226 |
| 4 | `399550` | 0.496054 |
| 5 | `464954` | 0.482082 |

Skor tersebut merupakan skor pemeringkatan internal algoritma, bukan probabilitas bahwa pengguna akan membeli item.

## Evaluasi Rekomendasi

Evaluasi menggunakan data `events_P2_testing.csv` sebagai sumber interaksi pada periode pengujian.

Metrik yang digunakan adalah:

- **Hit@5** — menunjukkan apakah setidaknya satu item rekomendasi Top-5 ditemukan dalam item relevan user pada data pengujian.
- **DCG@5** — mengukur perolehan item relevan dengan mempertimbangkan posisi item pada daftar rekomendasi.
- **IDCG@5** — nilai DCG ideal sebagai pembanding.
- **NDCG@5** — normalisasi DCG terhadap IDCG untuk menilai kualitas urutan rekomendasi.

### Hasil Evaluasi Awal

Pada pengujian awal, user `119834` tidak ditemukan dalam dataset P2. Oleh karena itu, belum tersedia ground truth untuk user tersebut.

| Metrik | Output Awal |
|---|---:|
| Hit@5 | 0 |
| DCG@5 | 0.000000 |
| IDCG@5 | 0.000000 |
| NDCG@5 | 0.000000 |

**Catatan interpretasi:** nilai tersebut bukan bukti bahwa algoritma rekomendasi berkinerja buruk. User yang dievaluasi tidak memiliki data interaksi pada periode pengujian, sehingga hasil tersebut belum dapat digunakan untuk menyimpulkan kualitas model. Evaluasi yang representatif perlu menggunakan user yang memiliki interaksi pada P2 serta ground truth yang sesuai.

## Struktur Berkas

Struktur berkas utama yang digunakan dalam pengerjaan proyek:

```text
Graf Sains Data/
├── events_P1_training.csv
├── events_P2_testing.csv
├── VSM_User_P1.npz
├── VSM_Item_P1.npz
├── user_ids_P1.npy
├── item_ids_P1.npy
├── hasil_similarity_top5_P1.csv
└── [berkas notebook Python]
```

Nama berkas notebook dapat disesuaikan dengan nama yang digunakan pada repositori GitHub.

## Cara Menjalankan

### 1. Persiapkan lingkungan

Pastikan Python, Jupyter Notebook, dan Neo4j sudah tersedia. Instal pustaka yang diperlukan:

```bash
pip install pandas numpy scipy neo4j scikit-learn matplotlib networkx
```

Sesuaikan daftar pustaka dengan import yang benar-benar digunakan pada notebook.

### 2. Siapkan dataset

Unduh dataset RetailRocket dari Kaggle, lalu tempatkan berkas hasil pemisahan periode pada direktori proyek:

- `events_P1_training.csv`
- `events_P2_testing.csv`

Pastikan juga berkas VSM dan pemetaan ID tersedia apabila notebook memuatnya secara langsung.

### 3. Jalankan Neo4j

Pastikan layanan Neo4j aktif dan konfigurasi koneksi pada notebook sesuai dengan lingkungan lokal.

Contoh konfigurasi:

```python
URI = "bolt://127.0.0.1:7687"
DATABASE = "neo4j"
```

Gunakan kredensial lokal milik masing-masing pengguna dan jangan menyimpan kata sandi di repositori publik.

### 4. Eksekusi notebook

Jalankan cell secara berurutan mulai dari pemuatan data, pengolahan graf, pembentukan VSM, perhitungan similarity, rekomendasi KNN, hingga evaluasi.

## Catatan Implementasi

- Seed pengacakan digunakan agar pemilihan user dapat direproduksi dengan populasi dan urutan kandidat yang sama.
- Relationship `SIMILAR_TO` harus sudah tersedia sebelum proses rekomendasi dijalankan.
- Proses impor ulang data perlu memperhatikan kemungkinan duplikasi node atau relationship.
- Dataset P2 digunakan sebagai data pengujian, bukan sebagai sumber untuk membangun similarity apabila tujuan eksperimen adalah menghindari kebocoran data.
- Hasil evaluasi harus ditafsirkan berdasarkan ketersediaan ground truth dan definisi relevansi yang digunakan.

## Referensi

1. [RetailRocket E-commerce Dataset — Kaggle](https://www.kaggle.com/datasets/retailrocket/ecommerce-dataset)
2. [Neo4j Documentation](https://neo4j.com/docs/)
3. [Neo4j Python Driver Manual](https://neo4j.com/docs/python-manual/current/)
4. [Scikit-learn Documentation](https://scikit-learn.org/stable/)
