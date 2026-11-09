# Cleaning Log

Dataset: transaksi_koperasi_2025.csv  
Baris mentah: 1543 -> baris bersih: 1493  

| # | Langkah | Baris terdampak | Alasan |
|---|---|---|---|
| 1 | Konversi total_bayar ke angka | 250 | Buang "Rp" dan pemisah ribuan agar kolom dapat dihitung |
| 2 | Konversi tanggal per format | 1543 | format="mixed" dengan dayfirst merusak 483 baris berformat ISO tanpa memunculkan galat |
| 3 | Buang spasi awal dan akhir kolom teks | 160 | Spasi tersembunyi memecah groupby dan melambungkan nunique |
| 4 | Placeholder teks menjadi kosong | 19 | N/A, -, null, dan teks kosong bukan nilai sungguhan |
| 5 | Placeholder -999 pada rating menjadi kosong | 49 | Angka -999 merusak rata-rata tanpa pernah terdeteksi isna() |
| 6 | Seragamkan kategori | 2 | Huruf dan spasi oleh kode; sinonim dan salah ketik diputuskan manual lalu dicatat di peta |
| 7 | Hapus duplikat persis | 39 | Seluruh kolom identik; satu baris disimpan |
| 8 | Satukan duplikat menurut id_transaksi | 15 | Satu transaksi hanya boleh satu baris; dipilih baris yang catatannya terisi |
| 9 | Buang baris tanpa id_transaksi | 1 | Kunci utama kosong tidak dapat ditelusuri kembali ke sumber |
| 10 | jumlah <= 0 menjadi kosong | 29 | Belum jelas retur atau salah input; barisnya masih berguna untuk kolom lain |
| 11 | Diskon di luar 0-100 menjadi kosong | 9 | Nilai mustahil; menunggu konfirmasi pengurus |
| 12 | Tandai total yang tidak cocok | 50 | Ditandai, tidak ditimpa, karena belum diketahui kolom mana yang keliru |
| 13 | saldo_simpanan dibiarkan kosong | 156 | Diduga MNAR; imputasi apa pun akan membuat rata-rata saldo tampak lebih kecil daripada kenyataan |
| 14 | Tandai keberlakuan berat_kg | 290 | Kosong karena tidak berlaku, bukan karena hilang; diberi kolom penanda, tidak diisi |
| 15 | Tandai keberadaan catatan kasir | 1004 | Kosong berarti kasir tidak menulis apa pun; dijadikan penanda biner |