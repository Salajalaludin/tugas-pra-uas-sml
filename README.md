PAMAP2 Physical Activity Monitoring (ID: 231)

Deskripsi
--------
Dataset ini berasal dari sensor yang ditempelkan pada tubuh 9 subjek saat melakukan berbagai aktivitas fisik (misal: jalan, lari, naik/turun tangga, dll.). Data mentah mencakup sinyal IMU (akselerometer, giroskop, magnetometer) dan data detak jantung.

Tantangan
------------------------
Tantangan unsupervised di sini adalah mencoba mengelompokkan sinyal-sinyal sensor sehingga algoritma dapat membedakan aktivitas berat, sedang, dan ringan tanpa label aktivitas. Fokusnya adalah eksplorasi klaster pada fitur sensor (≈52 fitur).

Algoritma
-------------------------------
- K-Means (atau Mini-Batch K-Means) dikombinasikan dengan reduksi dimensi.
- Untuk skala besar, gunakan Mini-Batch K-Means agar penggunaan RAM lebih ramah.

Alur Implementasi
---------------------------------------
1. Load data secara efisien
	- Gunakan paket `data.table` dan fungsi `fread()` untuk membaca file .dat per-subjek (subject101.dat … subject109.dat).
	- Gabungkan file dengan `rbindlist()` daripada `rbind()` demi efisiensi.
2. Pembersihan & filter aktivitas
	- Buang baris dengan `activityId == 0` (transisi / tidak berlabel).
	- Tangani missing values (NA), khususnya pada kolom heart rate yang sering kosong: imputasi atau buang kolom/baris sesuai kebutuhan eksperimen.
3. Standardisasi fitur
	- Gunakan `scale()` untuk menormalisasi semua fitur sebelum klastering.
4. Eksekusi Mini-Batch K-Means
	- Gunakan paket `ClusterR` (MiniBatchKMeans) untuk menghindari penggunaan memori yang besar.
	- Pilih batch size kecil (mis. 500–5000) dan jumlah klaster yang sesuai (mis. 3 untuk ringan/sedang/berat).
5. Evaluasi & visualisasi
	- Gunakan PCA / UMAP untuk reduksi dimensi dan plot klaster.
	- Hitung metrik internal (Silhouette, Davies–Bouldin) untuk menilai kualitas klaster.

Catatan Teknis & Tips RAM
------------------------
R bersifat in-memory: memproses ~3,8 juta baris × 52 fitur bisa membuat laptop lambat atau freeze. Untuk mengatasi ini:
- Baca data per-subjek, bersihkan lalu gabungkan secara efisien.
- Gunakan `data.table::fread()` untuk kecepatan dan efisiensi memori.
- Gunakan `ClusterR::MiniBatchKmeans()` untuk klastering berbasis batch.

Struktur Data
--------------
- Set data PAMAP2 memiliki sekitar 54 kolom (termasuk timestamp, subject id, activity id, dan fitur sensor). Jika ingin menggabungkan semua file subjek, lakukan dengan `rbindlist()`.

Contoh potongan kode (ringkas)
-----------------------------
```r
library(data.table)
files <- list.files("pamap2 physical activity monitoring/PAMAP2_Dataset", pattern = "*.dat", full.names = TRUE)
dt_list <- lapply(files, fread)
dt <- rbindlist(dt_list)
dt <- dt[activityId != 0]
## Tangani NA, pilih fitur, lalu skala
dt_clean <- na.omit(dt) # atau strategi imputasi
features <- as.matrix(scale(dt_clean[, ..feature_cols]))

library(ClusterR)
# Pastikan nama fungsi sesuai versi paket; contoh penggunaan Mini-Batch K-Means:
mbkm <- MiniBatchKmeans(features, clusters = 3, batch_size = 2000, num_init = 1, max_iters = 100)
```

Referensi & File
----------------
- Data asli dipisah per-subjek (subject101.dat … subject109.dat) di folder `pamap2 physical activity monitoring/PAMAP2_Dataset/`.
- ID dataset: 231
- link dataset: `https://archive.ics.uci.edu/dataset/231/pamap2+physical+activity+monitoring`

