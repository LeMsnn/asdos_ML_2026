# Hierarchical Clustering — Panduan Lengkap Praktikum Pertemuan 6

> Dokumen ini dibuat untuk membantu asisten praktikum memahami notebook `Hierarchical Clustering.ipynb`, cara membaca file data `data/data_hierarchical_clustering.csv`, dan cara membaca grafik (dendrogram & scatter) secara lengkap.

---

## A. Definisi Lengkap (Istilah & Kepanjangan)

| Istilah | Kepanjangan / Definisi |
|---|---|
| **Hierarchical Clustering** | Metode clustering yang membangun hierarki (pohon) antar data secara bertahap. Hasil akhirnya divisualisasikan sebagai **dendrogram**. Jumlah cluster tidak perlu ditentukan di awal. |
| **Agglomerative Clustering** | Pendekatan *bottom-up*: setiap data awalnya adalah cluster sendiri (6 data → 6 cluster), lalu digabungkan secara berulang hingga tersisa 1 cluster. Inilah yang dipakai di notebook ini. |
| **Divisive Clustering** | Pendekatan sebaliknya (*top-down*): semua data mulai dari 1 cluster lalu dipecah. (Tidak dipakai di notebook.) |
| **Linkage** | Aturan untuk menghitung **jarak antar cluster** (bukan antar titik tunggal). |
| **scipy** | **Sci**entific **Py**thon — library komputasi ilmiah. Fungsi `linkage()` dan `dendrogram()` ada di `scipy.cluster.hierarchy`. |
| **sklearn** | **scikit-learn** — library machine learning. Kelas `AgglomerativeClustering` ada di `sklearn.cluster`. |
| **pandas (pd)** | Library manipulasi data tabular (DataFrame). |
| **NumPy (np)** | Library komputasi numerik (array). |
| **matplotlib (plt)** | Library plotting. `plt` = alias dari `matplotlib.pyplot`. |
| **CSV** | **C**omma-**S**eparated **V**alues. Di file ini pemisah kolomnya bukan koma, melainkan **titik koma (`;`)**. |
| **Euclidean distance** | Jarak lurus antara dua titik: `√((x₁−x₂)² + (y₁−y₂)²)`. |
| **Dendrogram** | Diagram pohon yang menunjukkan urutan dan jarak penggabungan cluster. |
| **Threshold (max_d)** | Garis batas horizontal pada dendrogram untuk menentukan jumlah cluster. |

### Metode Linkage (dari markdown cell notebook)

Tanda tangan fungsi: `scipy.cluster.hierarchy.linkage(y, method='single', metric='euclidean', optimal_ordering=False)`

| method | Cara menghitung jarak antar cluster | Karakter |
|---|---|---|
| `single` | Jarak **terdekat** antar anggota dua cluster | Membentuk rantai panjang (*chaining effect*) |
| `complete` | Jarak **terjauh** antar anggota dua cluster | Menghasilkan cluster padat & bulat |
| `average` | **Rata-rata** jarak semua pasangan anggota | Kompromi single & complete |
| `ward` | Meminimalkan **total varians** dalam cluster | Cluster cenderung berukuran seimbang |
| `weighted` | Rata-rata tertimbang (WPGMA) | — |
| `centroid` | Jarak antar **centroid** cluster | Bisa terjadi inversi |
| `median` | Jarak antar **median** cluster (WPGMC) | — |

---

## B. Cara Membaca File `data/data_hierarchical_clustering.csv`

Isi file:

```
Objek;X1;X2
A1;4.5;3.5
A2;4;4
A3;4.25;4.25
A4;2.75;3.25
A5;2;3
A6;2.5;2.5
```

**Cara membacanya:**

1. **Baris pertama = header (nama kolom)**: `Objek`, `X1`, `X2`.
2. **Pemisah kolom = titik koma (`;`)**, bukan koma. Karena itu di kode ditulis `pd.read_csv(..., sep=";")`.
3. **Setiap baris berikutnya = satu objek data** dengan 2 fitur (koordinat X1 dan X2).
4. Kolom `Objek` berisi **label/nama** (A1–A6), kolom `X1` dan `X2` berisi **nilai numerik** (koordinat titik di bidang 2 dimensi).

| Objek | X1 | X2 | Artinya (koordinat titik) |
|---|---|---|---|
| A1 | 4.50 | 3.50 | titik di (4.5 ; 3.5) |
| A2 | 4.00 | 4.00 | titik di (4 ; 4) |
| A3 | 4.25 | 4.25 | titik di (4.25 ; 4.25) |
| A4 | 2.75 | 3.25 | titik di (2.75 ; 3.25) |
| A5 | 2.00 | 3.00 | titik di (2 ; 3) |
| A6 | 2.50 | 2.50 | titik di (2.5 ; 2.5) |

**Interpretasi data:** secara visual ada **2 kelompok alami**:
- Kelompok kanan-atas: **A1, A2, A3** (nilai X1 ≈ 4–4.5)
- Kelompok kiri-bawah: **A4, A5, A6** (nilai X1 ≈ 2–2.75)

**Cara pandas membacanya di notebook (Cell 2):**

```python
df = pd.read_csv("data_hierarchical_clustering.csv", sep=";")  # baca CSV, pemisah ';'
dataku = df.iloc[:, [1, 2]].values  # ambil kolom index 1 (X1) dan 2 (X2) → array numpy
X = df.iloc[:, [1]].values          # kolom X1 saja
Y = df.iloc[:, [2]].values          # kolom X2 saja
```

> ⚠️ **Catatan penting untuk asisten:** di dalam notebook, Cell 6 dan Cell 5 **tidak memakai CSV**, melainkan data *hardcode*: `X=[4.5,4,4.25,2.75,2,2.5]` dan `Y=[3.4,4,4.25,3.25,3,2.5]`. Perhatikan nilai A1 pada Y: **CSV = 3.5, hardcode = 3.4**. Karena itu hasil dendrogram di notebook sedikit berbeda dengan hasil dari file CSV. Semua perhitungan pada dokumen ini menggunakan **data CSV (A1 = 3.5)**.

---

## C. Cara Membaca Data (Jarak antar Titik)

Jarak antar titik dihitung dengan rumus Euclidean. Pasangan terdekat:

| Pasangan | Perhitungan | Jarak |
|---|---|---|
| A2 – A3 | √(0.25² + 0.25²) = √0.125 | **≈ 0.354** (terdekat) |
| A1 – A2 | √(0.5² + 0.5²) = √0.5 | ≈ 0.707 |
| A5 – A6 | √(0.5² + 0.5²) = √0.5 | ≈ 0.707 |
| A4 – A5 | √(0.75² + 0.25²) = √0.625 | ≈ 0.791 |
| A4 – A6 | √(0.25² + 0.75²) = √0.625 | ≈ 0.791 |

Agak jauh: A2–A4 ≈ 1.458 (inilah yang menjadi "jembatan" kedua kelompok pada single linkage).

---

## D. Cara Membaca Grafik Dendrogram (Lengkap)

Dendrogram adalah grafik **pohon terbalik**. Cara membacanya:

1. **Sumbu X (horizontal)** = objek data (A1 … A6). Objek yang sering bergabung akan berdekatan di sumbu X.
2. **Sumbu Y (vertikal)** = **jarak** (tinggi) saat dua cluster digabungkan.
3. **Setiap bentuk "U terbalik" (∩) = satu penggabungan** dua cluster.
4. **Tinggi huruf U = jarak saat penggabungan itu terjadi**. Semakin tinggi, semakin berbeda kedua cluster.
5. **Baca dari bawah ke atas**: penggabungan paling bawah = cluster paling mirip.
6. **Garis horizontal hitam (`plt.axhline(y=max_d, c='k')`) = threshold**. Cara menentukan jumlah cluster:
   - Tarik garis threshold melintang.
   - Hitung **jumlah garis vertikal dendrogram yang terpotong** oleh garis itu → itulah jumlah cluster.

**Contoh (single linkage, threshold max_d = 1):**
Garis y = 1 memotong **2 garis vertikal** (cabang {A1,A2,A3} dan cabang {A4,A5,A6}) → **diperoleh 2 cluster**.

**Membandingkan 4 metode linkage pada data ini:**

| Metode | Urutan penggabungan (jarak) | Jarak gabungan TERAKHIR (2 kelompok besar) |
|---|---|---|
| **Single** | A2+A3 (0.354) → A1+{A2,A3} (0.707) → A5+A6 (0.707) → A4+{A5,A6} (0.791) → 2 kelompok (1.458) | 1.458 |
| **Complete** | A2+A3 (0.354) → A5+A6 (0.707) → A1+{A2,A3} (0.791) → A4+{A5,A6} (0.791) → 2 kelompok (2.574) | 2.574 |
| **Average** | A2+A3 (0.354) → A5+A6 (0.707) → A1+{A2,A3} (0.749) → A4+{A5,A6} (0.791) → 2 kelompok (2.136) | 2.136 |
| **Ward** | A2+A3 (0.354) → A5+A6 (0.707) → A4+{A5,A6} (0.816) → A1+{A2,A3} (0.842) → 2 kelompok (3.617) | 3.617 |

**Cara membaca perbedaannya:**
- **Single linkage** menghasilkan dendrogram "merambat" (chaining) — gabungan terjadi pada jarak kecil-kecil terus.
- **Complete/Average/Ward** menunjukkan **loncatan besar** di gabungan terakhir (2.57 / 2.14 / 3.62) → sinyal kuat bahwa data memang terdiri dari **2 cluster** yang terpisah jelas.
- Semakin besar selisih tinggi antar tangga di dendrogram, semakin "natural" jumlah cluster tersebut.

---

## E. Komentar Per-Line Kode Notebook

### Cell 1 — Import library
```python
import numpy as np                      # import NumPy, alias np (array & perhitungan numerik)
import matplotlib.pyplot as plt         # import modul plotting matplotlib, alias plt
import pandas as pd                     # import pandas, alias pd (DataFrame)
```

### Cell 2 — Load data dari CSV & plot awal
```python
#importing the dataset into df          # komentar: mengimpor dataset ke dalam DataFrame

df = pd.read_csv("data_hierarchical_clustering.csv",sep=";")  # baca file CSV (pemisah ';') jadi DataFrame df

dataku=df.iloc[:,[1,2]].values         # ambil kolom index 1 (X1) dan 2 (X2), semua baris → array NumPy
dataku                                 # menampilkan isi dataku (ekspresi terakhir otomatis ditampilkan)
X=df.iloc[:,[1]].values                # ambil kolom X1 saja → array 2D
Y=df.iloc[:,[2]].values                # ambil kolom X2 saja → array 2D

#X=[4.5,4,4.25,2.75,2,2.5]            # komentar: contoh data X kalau tidak pakai CSV
#Y=[3.4,4,4.25,3.25,3,2.5]            # komentar: contoh data Y (perhatikan: 3.4, bukan 3.5 seperti CSV)

plt.scatter(X, Y)                      # buat scatter plot X vs Y
plt.show()                             # tampilkan plot
```

### Cell 6 — Dendrogram dengan 4 metode linkage (scipy)
```python
import numpy as np                                     # import NumPy
import matplotlib.pyplot as plt                        # import pyplot
from scipy.cluster.hierarchy import dendrogram, linkage  # import fungsi dendrogram & linkage

X=[4.5,4,4.25,2.75,2,2.5]        # list koordinat x dari 6 titik data
Y=[3.4,4,4.25,3.25,3,2.5]        # list koordinat y dari 6 titik data

data = list(zip(X, Y))           # gabungkan X & Y jadi list of tuple: [(4.5,3.4), (4,4), ...]
print(data)                      # cetak isi data ke console
linkage_data = linkage(data, method='single', metric='euclidean')  # hitung linkage matrix: single linkage, jarak euclidean
dendrogram(linkage_data)         # gambar dendrogram dari linkage matrix
plt.title("Single Linkage")      # beri judul plot

#batas threshold misalkan = 1     # komentar: menetapkan threshold = 1
max_d=1                          # variabel max_d menyimpan nilai threshold
plt.axhline(y=max_d, c='k')      # gambar garis horizontal di y=1, warna hitam ('k' = black)
plt.show()                       # tampilkan plot

linkage_data1 = linkage(data, method='complete', metric='euclidean')  # linkage dengan metode complete linkage
plt.title("Complete Linkage")    # judul plot
dendrogram(linkage_data1)        # dendrogram complete linkage
plt.show()                       # tampilkan

linkage_data2 = linkage(data, method='average', metric='euclidean')  # linkage dengan metode average linkage
plt.title("Average Linkage")     # judul plot
dendrogram(linkage_data2)        # dendrogram average linkage
plt.show()                       # tampilkan

linkage_data3 = linkage(data, method='ward', metric='euclidean')  # linkage dengan metode ward (minimalisasi varians)
plt.title("Ward Linkage")        # judul plot
dendrogram(linkage_data3)        # dendrogram ward
plt.show()                       # tampilkan
```

**Bonus — struktur linkage matrix** (untuk materi tambahan): `linkage()` mengembalikan matriks (n−1) × 4. Tiap baris = satu penggabungan:
`[id_cluster_1, id_cluster_2, jarak, jumlah_anggota_cluster_baru]`. Cluster baru diberi id mulai dari n (6, 7, 8, …).

### Cell 5 — Clustering dengan sklearn (AgglomerativeClustering)
```python
import numpy as np                                    # import NumPy
import pandas as pd                                  # import pandas
import matplotlib.pyplot as plt                      # import pyplot
from sklearn.cluster import AgglomerativeClustering  # import kelas AgglomerativeClustering

asli={"x":[4.5,4,4.25,2.75,2,2.5],   # dictionary berisi data x
    "y":[3.4,4,4.25,3.25,3,2.5]}       # dan data y

data=pd.DataFrame(asli)              # ubah dictionary jadi DataFrame pandas
print(data)                          # cetak isi DataFrame
#data = list(zip(x, y))              # komentar: alternatif ubah ke list of tuple (x,y kecil tidak terdefinisi)

#print(data)                          # komentar non-aktif

linkage_data = linkage(data, method='single', metric='euclidean')  # linkage single linkage (fungsi linkage & dendrogram "dipinjam" dari Cell 6)
dendrogram(linkage_data)             # gambar dendrogram
#Threshold                          # komentar
plt.axhline(y=max_d, c='k')          # garis threshold di y=max_d (=1, dari Cell 6)
plt.show()                           # tampilkan

#default linkage ='ward'             # komentar: linkage default sklearn adalah 'ward'
hierarchical_cluster = AgglomerativeClustering(n_clusters=3, affinity='euclidean', linkage='single')  # objek clustering: 3 cluster, jarak euclidean, single linkage
labels = hierarchical_cluster.fit_predict(data)  # latih model & prediksi label cluster untuk tiap data
print(labels)                        # cetak label → output: [0 0 0 2 1 1]

plt.scatter(data['x'], data['y'], c=labels)  # scatter plot, warna titik mengikuti label cluster
plt.title("Agglomerative")           # judul plot
plt.xlabel("X")                      # label sumbu X
plt.ylabel("Y")                      # label sumbu Y
plt.show()                           # tampilkan plot
```

---

## F. Cara Membaca Hasil AgglomerativeClustering

Output Cell 5: `labels = [0 0 0 2 1 1]`

| Objek | Label | Artinya |
|---|---|---|
| A1 | 0 | cluster 0 |
| A2 | 0 | cluster 0 |
| A3 | 0 | cluster 0 |
| A4 | 2 | cluster 2 |
| A5 | 1 | cluster 1 |
| A6 | 1 | cluster 1 |

Jadi dengan `n_clusters=3` dan **single linkage**, terbentuk 3 cluster: **{A1, A2, A3}**, **{A5, A6}**, dan **{A4}** (A4 "terpisah" karena single linkage menggabungkan {A5,A6} lebih dulu pada jarak 0.707 sebelum A4 bergabung pada 0.791).

Pada **scatter plot berwarna**: titik dengan warna sama = cluster yang sama. Warna bukan berarti "lebih baik", hanya pembeda visual.

---

## G. Catatan Penting untuk Asisten Praktikum

1. **Urutan eksekusi cell**: Cell 5 bergantung pada variabel `linkage`, `dendrogram`, dan `max_d` dari Cell 6. Urutan run di notebook: 1 → 2 → 6 → 5. Jika Cell 5 dijalankan sendiri, akan error `NameError`.
2. **Data CSV vs hardcode**: CSV berisi A1 = (4.5, **3.5**), sedangkan kode hardcode memakai (4.5, **3.4**). Hasil dendrogram notebook karena itu sedikit berbeda dari hasil CSV. Jika diminta praktikan menggunakan data CSV, pastikan membaca dari file, bukan hardcode.
3. **Deprecation**: parameter `affinity=` di `AgglomerativeClustering` sudah digantikan `metric=` pada scikit-learn ≥ 1.2. Untuk versi baru: `AgglomerativeClustering(n_clusters=3, metric='euclidean', linkage='single')`.
4. **Menentukan jumlah cluster**: (a) potong dendrogram dengan garis threshold, hitung garis vertikal yang terpotong; atau (b) lihat "loncatan" tinggi terbesar pada dendrogram (gap terbesar = tempat paling natural untuk memotong).
5. **Single linkage rawan chaining effect** (cluster memanjang seperti rantai) — tunjukkan pada praktikan mengapa dendrogram single linkage terlihat "merambat" dibanding ward.
