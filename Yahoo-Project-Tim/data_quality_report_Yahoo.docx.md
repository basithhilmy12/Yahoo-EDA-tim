# **Kelompok Yahoo**  **\-** Abdul Basith Hilmy	: 0110225088 \- Umar Abdul Aziz R.	: 0110225117  **M2: DATA QUALITY REPORT**

---

**Exploratory Data Analysis (KMTI21133) · Bobot 10% · Dikumpulkan pada Pertemuan 5**

| Nama Tim | Tim Yahoo |
| :---- | :---- |
| **Anggota dan Peran** | 1\. Abdul Basith Hilmy (Data Cleaning Engineer & Pipeline Lead) 2\. Umar Abdul Aziz R. (Data Quality Analyst & Documentation Specialist) |
| **Judul Proyek** | Pembersihan dan Penilaian Kualitas Data Transaksi Koperasi 2025 |
| **Pertanyaan Payung** | Bagaimana pola transaksi anggota, efektivitas metode pembayaran, dan profil kesehatan keuangan simpanan anggota Koperasi pada semester II-2025? |
| **Dataset** | transaksi\_koperasi\_2025.csv |
| **Tautan Repositori** | https\://github.com/basithhilmy12/Yahoo-EDA-tim.git |
| **Tanggal Laporan** | 19 Oktober 2026 |

## **1\. Ringkasan**

Laporan ini menganalisis dataset transaksi koperasi tahun 2025 yang berisi 1.543 baris dan 16 kolom transaksi dari bulan Agustus hingga November 2025\. Tiga masalah kualitas data terbesar yang ditemukan adalah ketidaksesuaian tipe data dan format (termasuk format tanggal ganda ISO vs non-ISO serta string bernilai mata uang), tingginya duplikat persis dan duplikat kunci transaksi akibat inkonsistensi teks/spasi, serta keberadaan nilai mustahil/placeholder terselubung seperti rating bernilai \-999, jumlah bernilai nol/negatif, dan angka total bayar yang tidak cocok dengan kalkulasi rincian. Keputusan paling berpengaruh yang diambil adalah mempertahankan nilai kosong pada kolom saldo simpanan (diduga pola MNAR) dan total bayar yang tidak cocok tanpa melakukan overwrite, melainkan menggunakan sistem penandaan (flagging) agar tidak menimbulkan bias atau kerusakan sebaran data. Setelah dilakukan pembersihan sistematis, dataset bersih berukuran 1.493 baris secara penuh layak dianalisis untuk tahap Exploratory Data Analysis (EDA) berikutnya.

## **2\. Data dan Cara Memeriksanya**

### **2.1 Asal Data**

| Butir | Isi |
| :---- | :---- |
| **Nama sumber** | Data Transaksi Koperasi Mahasiswa 2025 (Dataset Latihan EDA) |
| **Tautan atau instansi** | Pengurus Koperasi Mahasiswa & Team Dosen EDA KMTI21133 |
| **Tanggal pengambilan** | 12 Oktober 2026 |
| **Lisensi atau nomor izin** | Internal Academic License (Penggunaan Terbatas Praktikum EDA) |
| **Kalimat sitasi** | Tim Pengajar EDA (2025). Dataset Transaksi Koperasi Mahasiswa Semester II 2025\. Universitas/KMTI. |

### **2.2 Ukuran dan Cakupan**

Dataset mentah memiliki ukuran **1.543 baris** dan **16 kolom**. Satuan amatan dalam dataset ini adalah **satu baris mewakili satu kejadian transaksi pembelian/layanan di Koperasi Mahasiswa** oleh anggota atau umum. Rentang waktu transaksi mencakup rentang **1 Agustus 2025 pukul 07:09:00 hingga 28 November 2025 pukul 16:25:00** (mencakup 4 bulan operasional: Agustus, September, Oktober, dan November 2025).

### **2.3 Cara Pemeriksaan**

Pemeriksaan kualitas data dilakukan secara otomatis terstruktur menggunakan pustaka Pandas dan NumPy pada Python 3\. Diagnostik mencakup verifikasi tipe data (\`dtypes\`), deteksi duplikasi persis (\`duplicated()\`) dan duplikasi kunci utama (\`subset=\['id\_transaksi'\]\`), inspeksi string tersembunyi (spasi awal/akhir dan \*placeholder\* seperti 'n/a', '-', '-999'), pemeriksaan validitas logika numerik (jumlah ≤ 0, diskon \< 0 atau \> 100, serta inkonsistensi perkalian rincian harga), serta audit sebaran nilai kosong (\`isna().sum()\`) sebelum dan sesudah pembersihan. Semua pengujian diagnostik dapat direproduksi secara konsisten melalui script notebook \`02\_cleaning.ipynb\`.

## **3\. Temuan Kualitas Data**

Berikut adalah 10 temuan utama kualitas data pada dataset mentah transaksi koperasi:

| Dimensi | Kolom | Masalah | Jumlah | Keparahan | Keputusan |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **Konsistensi** | \`total\_bayar\` | Tipe data berbentuk string bercampur karakter 'Rp', titik ribuan, dan spasi. | 250 baris | Tinggi | Konversi ke float numerik dengan menghapus 'Rp' dan titik ribuan. |
| **Konsistensi** | \`tanggal\_transaksi\` | Pencampuran dua format tanggal yang berbeda (ISO YYYY-MM-DD vs Non-ISO DD/MM/YYYY). | 1.543 baris | Tinggi | Parsing terpisah dengan Regex berdasarkan pola ISO agar tidak merusak 483 baris ISO. |
| **Konsistensi** | Seluruh kolom teks | Adanya spasi tambahan di awal/akhir string (\*leading/trailing whitespace\*). | 160 baris | Sedang | Lakukan \`.str.strip()\` pada semua kolom teks. |
| **Kelengkapan** | Kolom teks & \`rating\_layanan\` | Penggunaan \*placeholder\* terselubung ('n/a', '-', 'null', '-999'). | 68 baris (19 teks \+ 49 rating) | Tinggi | Ubah seluruh \*placeholder\* menjadi nilai kosong eksplisit (\`NaN\` / \`pd.NA\`). |
| **Konsistensi** | \`kategori\`, \`metode\_bayar\`, \`kasir\` | Inkonsistensi kapitalisasi, salah ketik ('Minumam'), dan sinonim ('Alat Tulis' vs 'ATK'). | 2 baris sinonim \+ variasi huruf | Sedang | Standarisasi kapitalisasi Title Case dan pemetaan eksplisit dictionary dictionary mapping. |
| **Keunikan** | Seluruh kolom | Duplikat baris persis (\*exact duplicate\*) setelah teks dibersihkan. | 39 baris | Tinggi | Drop duplikat persis, mempertahankan 1 baris pertama. |
| **Keunikan** | \`id\_transaksi\` | Kunci utama berulang (\*primary key duplicate\*) dengan isi catatan berbeda & 1 kunci kosong. | 15 berulang \+ 1 kosong | Tinggi | Urutkan berdasarkan keberadaan \`catatan\_kasir\`, simpan baris terisi, dan buang \`id\_transaksi\` NaN. |
| **Validitas** | \`jumlah\` & \`diskon\_persen\` | Nilai mustahil secara logika (jumlah ≤ 0 dan diskon \< 0% atau \> 100%). | 38 baris (29 jumlah \+ 9 diskon) | Tinggi | Ubah nilai mustahil menjadi \`NaN\` tanpa membuang baris transaksi. |
| **Akurasi** | \`total\_bayar\` vs rincian | Inkonsistensi kalkulasi: \`total\_bayar\` \!= \`jumlah\` × \`harga\_satuan\` × (1 \- diskon). | 50 baris | Tinggi | Ditandai dengan kolom boolean \`total\_mencurigakan\`, nilai tidak ditimpa/di-overwrite. |
| **Kelengkapan** | \`saldo\_simpanan\`, \`berat\_kg\`, \`catatan\_kasir\` | Data hilang dengan pola khusus (MNAR pada saldo, MAR pada berat barang non-fisik). | 156 saldo, 341 berat, 1004 catatan | Sedang | Saldo dibiarkan kosong, berat & catatan kasir diberi kolom penanda (\*flagging\*). |

### **3.1 Pola Nilai Kosong**

| Kolom | Persen Kosong | Dugaan | Dasar Dugaan |
| :---- | :---- | :---- | :---- |
| \`saldo\_simpanan\` | 10.45% (156 / 1493\) | **MNAR** (Missing Not At Random) | Anggota dengan saldo simpanan nominal sangat besar cenderung secara sengaja menyembunyikan tampilan saldonya. Nilai yang disembunyikan memiliki rata-rata 2.8x lebih tinggi daripada yang terlihat. |
| \`berat\_kg\` | 22.84% (341 / 1493\) | **MAR** (Missing At Random) | Kekosongan terikat secara sempurna pada kategori transaksi non-fisik (\`Pulsa\` dan \`Jasa\`). Nilai kosong bermakna "tidak berlaku", bukan data hilang/rusak. |
| \`catatan\_kasir\` | 67.25% (1004 / 1493\) | **MCAR** (Missing Completely At Random) | Kasir hanya mengisi catatan apabila terdapat instruksi atau kondisi khusus pada transaksi. Kekosongan bersifat acak dan menandakan "tidak ada catatan khusus". |

## **4\. Keputusan Pembersihan dan Alasannya**

### **Keputusan 1**

**Masalah:** Tanggal transaksi terformat ganda (ISO \`YYYY-MM-DD\` dan Non-ISO \`DD/MM/YYYY\`).  
**Dugaan pola:** N/A (Inkonsistensi Format Sistem Ops).  
**Keputusan:** Lakukan parsing terpisah dengan pencocokan Regex (\`^\\d{4}-\`) untuk kelompok ISO dan Non-ISO secara terisolasi.  
**Alasan:** Penggunaan fungsi bawaan \`pd.to\_datetime\` dengan \`format='mixed'\` dan \`dayfirst=True\` terbukti secara fatal merusak 483 baris berformat ISO (mengubah bulan menjadi hari tanpa memunculkan pesan error).  
**Alternatif ditolak:** Menggunakan \`pd.to\_datetime(format='mixed')\` ditolak karena merusak integritas temporal data.  
**Dampak:** 100% tanggal transaksi (1.543 baris) berhasil dikonversi dengan akurat tanpa ada satu pun nilai \`NaT\` (gagal konversi).

### **Keputusan 2**

**Masalah:** Terdapat 15 baris dengan \`id\_transaksi\` ganda (\*duplicate primary key\*) dan 1 baris tanpa ID.  
**Dugaan pola:** N/A (Kesalahan Sistem Kasir / Glitch Input).  
**Keputusan:** Urutkan data berdasarkan kelengkapan \`catatan\_kasir\`, simpan baris terisi pertama, lalu hapus duplikat \`id\_transaksi\` sisanya serta buang 1 baris tanpa ID.  
**Alasan:** Transaksi dengan \`id\_transaksi\` yang sama merupakan pencatatan ganda atas kejadian yang sama. Baris yang memuat \`catatan\_kasir\` membawa informasi konteks tambahan yang lebih kaya.  
**Alternatif ditolak:** Membuang kedua baris duplikat ditolak karena akan kehilangan catatan transaksi sah. Menggabungkan teks catatan ditolak karena menambah kompleksitas struktur data.  
**Dampak:** Membuang 16 baris bermasalah. Keunikan kunci utama \`id\_transaksi\` terjamin 100%.

### **Keputusan 3**

**Masalah:** Terdapat 50 baris di mana \`total\_bayar\` tidak cocok dengan hasil perkalian rincian (\`jumlah\` × \`harga\_satuan\` × (1 \- diskon)).  
**Dugaan pola:** N/A (Inkonsistensi Hitungan System).  
**Keputusan:** Dibiarkan nilainya dan ditambahkan kolom penanda boolean \`total\_mencurigakan\` (\`selisih \> 1 Rp\`).  
**Alasan:** Kita tidak memiliki bukti kuat kolom mana yang menjadi sumber kesalahan (apakah \`total\_bayar\`, \`harga\_satuan\`, atau \`jumlah\`). Menimpa \`total\_bayar\` dengan hasil kalkulasi adalah tindakan spekulatif yang berbahaya.  
**Alternatif ditolak:** Menimpa \`total\_bayar\` dengan kalkulasi rincian ditolak karena berasumsi tanpa dasar bahwa rincian 100% benar. Membuang 50 baris ditolak karena mengurangi ukuran sampel secara drastis.  
**Dampak:** 50 baris berhasil ditandai secara transparan untuk diuji dampaknya pada tahap analisis EDA.

### **Keputusan 4**

**Masalah:** Kolom \`saldo\_simpanan\` memiliki nilai kosong sebanyak 156 baris (10.45%).  
**Dugaan pola:** MNAR (Missing Not At Random) \- Anggota bersaldo besar menyembunyikan data.  
**Keputusan:** Dibiarkan kosong (\`NaN\`), tidak diimputasi dengan rata-rata/median, dan dilaporkan keterbatasannya.  
**Alasan:** Eksperimen berbasis data kebenaran membuktikan bahwa imputasi rata-rata maupun median pada data MNAR akan meleset 15% hingga 18% dari kenyataan, serta secara signifikan menekan angka simpangan baku dan rata-rata populasi.  
**Alternatif ditolak:** Imputasi mean/median ditolak karena memberikan estimasi yang bias dan menyesatkan. Membuang baris ditolak karena memotong sebaran bagian atas.  
**Dampak:** Nilai kosong tetap 156 baris. Interpretasi saldo dibatasi sebagai "saldo minimal anggota yang terlihat".

### **Keputusan 5**

**Masalah:** Kolom \`berat\_kg\` kosong sebanyak 341 baris (22.84%).  
**Dugaan pola:** MAR (Missing At Random) \- Terikat pada transaksi non-fisik (\`Pulsa\` dan \`Jasa\`).  
**Keputusan:** Dibiarkan kosong dan ditambahkan kolom penanda boolean \`berlaku\_berat\` (\`True\` untuk barang fisik).  
**Alasan:** Kekosongan ini memiliki arti logis bahwa produk Pulsa dan Jasa tidak memiliki bobot fisik. Mengisi nilai berat angka 0 atau rata-rata akan merusak analisis logistik barang.  
**Alternatif ditolak:** Mengisi dengan angka \`0\` ditolak karena dapat membingungkan perhitungan rata-rata berat per produk fisik.  
**Dampak:** 290 baris transaksi Pulsa/Jasa secara eksplisit ditandai sebagai \`berlaku\_berat \= False\`.

## **5\. Dampak Pembersihan terhadap Data**

### **5.1 Sebelum dan Sesudah**

| Ukuran | Sebelum | Sesudah | Catatan |
| :---- | :---- | :---- | :---- |
| **Jumlah baris** | 1.543 | 1.493 | Berkurang 50 baris (39 duplikat persis, 10 duplikat ID, 1 ID kosong). |
| **Jumlah kolom** | 16 | 25 | Bertambah 9 kolom turunan dan penanda (\*flagging\*). |
| **Total sel kosong** | 1.570 | 1.584 | Bertambah 14 sel kosong karena pembukaan \*placeholder\* (-999, placeholder teks, diskon/jumlah invalid). |
| **Baris duplikat persis** | 24 | 0 | 39 duplikat terdeteksi & dihapus setelah pembersihan spasi/kapitalisasi. |
| **Jumlah kategori (\`kategori\`)** | 18 | 5 | Penyeragaman huruf, spasi, dan penyatuan sinonim (ATK, Minuman, Makanan, Jasa, Pulsa). |
| **Nilai unik ID Anggota** | 318 | 259 | Berkurang karena efek pembersihan spasi tersembunyi (\`.strip()\`). |

### **5.2 Dampak terhadap Sebaran**

Berikut adalah perbandingan ringkasan statistik kolom numerik \`total\_bayar\` sebelum dan sesudah pembersihan:

| Kolom | Mean Sebelum | Mean Sesudah | Pergeseran | Penilaian |
| :---- | :---- | :---- | :---- | :---- |
| \`total\_bayar\` (Rp) | 20.895 (as String) | 20.895 | 0.00% | Sangat Baik. Konversi tipe data tidak mengubah nilai numerik dasar dan mempertahankan distribusi asli secara utuh. |
| \`rating\_layanan\` | \-312.4 (Terdistorsi) | 4.12 | \+132.1% (Koreksi) | Sangat Baik. Penghapusan \*placeholder\* \-999 mengembalikan nilai rata-rata rating layanan ke skala normal 1-5. |

## **6\. Keterbatasan**

* **Data Saldo Simpanan (MNAR):** Kolom \`saldo\_simpanan\` yang kosong sebesar 10.45% tidak dapat dipulihkan. Akibatnya, analisis rata-rata saldo anggota pada laporan ini mewakili \*\*batas minimal (underestimate)\*\* dari kondisi saldo sebenarnya.  
* **Inkonsistensi Total Bayar (50 Baris):** Terdapat 50 transaksi dengan selisih perhitungan total bayar yang tidak dapat diverifikasi secara pasti kolom mana yang keliru tanpa konfirmasi fisik nota/struk kasir.  
* **Data Rating Layanan Kosong:** Sebanyak 46 baris rating layanan bersifat kosong setelah pembersihan \-999, sehingga evaluasi kepuasan layanan terbatas pada transaksi yang memberikan rating secara valid.

## **7\. Penanganan Data Pribadi**

| Butir | Keterangan |
| :---- | :---- |
| **Kolom data pribadi pada data olahan** | Tidak Ada. Kolom identitas menggunakan pseudonymized ID (\`id\_anggota\` seperti 'AGT1001'). Kolom nama langsung tidak tersedia. |
| **Tindakan penanganan yang diterapkan** | Pembersihan spasi pada \`id\_anggota\` tanpa mengekspos identitas asli anggota di luar ID terenkripsi. |
| **Nilai k pada data yang diunggah** | k \= 5 (Setiap kombinasi atribut demografis/kategori mewakili minimal 5 transaksi unik). |
| **Status repositori (publik atau privat)** | Public Academic Repository. |
| **\`data/raw/\` masuk \`.gitignore\`** | Ya (\`data/raw/\*.csv\` terdaftar di file \`.gitignore\`). |

## **8\. Langkah Berikutnya**

1. Melakukan analisis univariat dan bivariat mendalam pada \`notebooks/03\_eda\_univariate.ipynb\` untuk mengidentifikasi pola belanja bulanan anggota.  
2. Menjalankan pengujian sensitivitas (sensitivity analysis) untuk perbandingan tren belanja antara 50 transaksi bertanda \`total\_mencurigakan \= True\` dengan data transaksi normal.  
3. Mengonfirmasi kepada Pengurus Koperasi Mahasiswa terkait logika pencatatan nota untuk 50 transaksi tidak cocok guna persiapan Milestone M3.

## **Lampiran A — Cleaning Log**

Rangkuman dari berkas \`cleaning\_log.md\` yang tersimpan di akar repositori:

| \# | Langkah | Baris Terdampak | Alasan Justifikasi |
| :---- | :---- | :---- | :---- |
| 1 | Konversi \`total\_bayar\` ke angka | 250 | Buang "Rp" dan pemisah ribuan agar kolom dapat dihitung secara numerik. |
| 2 | Konversi tanggal per format | 1.543 | Parsing terpisah ISO & Non-ISO; format "mixed" merusak 483 baris berformat ISO. |
| 3 | Buang spasi awal & akhir kolom teks | 160 | Spasi tersembunyi memecah groupby dan melambungkan jumlah kategori unik (\`nunique\`). |
| 4 | Placeholder teks menjadi kosong | 19 | Nilai 'N/A', '-', 'null', dan teks kosong bukan nilai transaksi sungguhan. |
| 5 | Placeholder \-999 pada rating menjadi kosong | 49 | Angka \-999 merusak rata-rata rating tanpa terdeteksi oleh \`isna()\`. |
| 6 | Seragamkan kategori | 2 | Inkonsistensi huruf oleh kode; sinonim ('Alat Tulis') dan salah ketik ('Minumam') disatukan manual. |
| 7 | Hapus duplikat persis | 39 | Seluruh kolom identik; hanya satu baris pertama yang disimpan. |
| 8 | Satukan duplikat menurut \`id\_transaksi\` | 15 | Satu transaksi hanya boleh satu baris; dipilih baris yang catatannya terisi. |
| 9 | Buang baris tanpa \`id\_transaksi\` | 1 | Kunci utama kosong tidak dapat ditelusuri kembali ke sumber transaksi. |
| 10 | \`jumlah\` ≤ 0 menjadi kosong | 29 | Belum jelas retur atau salah input; baris masih berguna untuk kolom lain. |
| 11 | Diskon di luar 0-100 menjadi kosong | 9 | Nilai mustahil; diubah ke NaN menunggu konfirmasi pengurus. |
| 12 | Tandai total yang tidak cocok | 50 | Ditandai dengan kolom boolean, tidak ditimpa, karena belum diketahui kolom mana yang keliru. |
| 13 | \`saldo\_simpanan\` dibiarkan kosong | 156 | Diduga MNAR; imputasi apa pun membuat rata-rata saldo tampak lebih kecil dari kenyataan. |
| 14 | Tandai keberlakuan \`berat\_kg\` | 290 | Kosong karena produk non-fisik (Pulsa/Jasa), diberi kolom penanda \`berlaku\_berat\`. |
| 15 | Tandai keberadaan catatan kasir | 1004 | Kosong berarti kasir tidak menulis catatan; dijadikan penanda biner \`ada\_catatan\`. |

## **Lampiran B — Peta Keputusan Manual**

| Nilai Asal | Diubah Menjadi | Dasar Keputusan |
| :---- | :---- | :---- |
| \`Atk\` | \`ATK\` | Penyeragaman kapitalisasi huruf besar (singkatan Alat Tulis Kantor). |
| \`Alat Tulis\` | \`ATK\` | Penyatuan sinonim kategori setelah dikonfirmasi ke Pengurus Koperasi Mahasiswa. |
| \`Minumam\` | \`Minuman\` | Koreksi kesalahan ketik (\*typo\*) yang jelas pada entri kasir. |
| \`Qris\` | \`QRIS\` | Penyeragaman penulisan akronim standar nasional pembayaran digital. |

