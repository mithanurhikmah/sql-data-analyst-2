# Analisis Penjualan Showroom Mobil dengan SQL (BigQuery)

Latihan analisis data penjualan showroom mobil menggunakan **SQL di Google BigQuery**. Ada 10 pertanyaan bisnis yang dijawab dengan `DISTINCT`, `WHERE`, `ORDER BY` + `LIMIT`, `GROUP BY`, `HAVING`, subquery, dan CTE.

> Dikerjakan dalam bootcamp **Data Analyst** (KarirNex, 2, 4, 7, 9 September 2026; 12 jam belajar + 3 jam mini project). Dataset yang dipakai adalah **data dummy** dari bootcamp, bukan data perusahaan sungguhan.

## Ringkasan Temuan

| # | Pertanyaan | Hasil |
|---|---|---|
| 1 | Daftar nama pelanggan unik | 3.977 pelanggan unik |
| 2 | Rekap transaksi kredit | 4.253 transaksi |
| 3 | 5 transaksi dengan quantity terbanyak | Semua bernilai 3 unit (Daihatsu Xenia, Mitsubishi Xpander, Hyundai Creta) |
| 4 | Model mobil warna Hitam Metalik | 16 model |
| 5 | Komposisi metode pembayaran | Kredit terbanyak: 4.253 transaksi (42,5%) |
| 6 | Cabang dengan total pendapatan di atas Rp50 miliar | Seluruh 10 cabang; tertinggi Malang Rp325,3 miliar |
| 7 | Rata-rata diskon per kategori | Hatchback tertinggi: Rp55.877 |
| 8 | Transaksi dengan diskon di atas rata-rata | 318 transaksi |
| 9 | Rata-rata pendapatan per kategori (completed) | Pickup tertinggi: sekitar Rp472,8 juta |
| 10 | 3 model mobil terlaris | Honda HR-V 1.5 E CVT (557 transaksi) |

## Dataset

- Tabel: `showroom_car.showroom_mobil` (BigQuery)
- Isi: 10.000 transaksi penjualan sepanjang 2025 (1 baris = 1 transaksi), 10 cabang, 5 kategori mobil
- Kolom (16): `sales_date`, `order_id`, `customer_name`, `branch`, `product_name`, `category`, `color`, `price`, `quantity`, `payment_type`, `trade_in`, `discount`, `total`, `total_sales`, `status`, `branch_address`
- Cabang: Jakarta, Padang, Malang, Bandung, Batam, Pontianak, Palembang, Kendari, Karawang, Semarang
- Kategori: SUV, MPV, Hatchback, Pickup, City Car (EV)
- Metode pembayaran: Kredit, Cash, Kredit + Trade In, Cash + Trade In
- Status: completed, cancelled, refund
- Data mentah tidak disertakan di repositori ini. Hanya query dan hasilnya.

## Skill SQL yang Dipakai

`SELECT DISTINCT` · `WHERE` · `ORDER BY` + `LIMIT` · `COUNT` / `SUM` / `AVG` · `GROUP BY` · `HAVING` · subquery di `WHERE` dan di `FROM` · CTE (`WITH`)

## Query dan Hasil

### 1. Daftar nama pelanggan unik

Skenario: mengirim newsletter promo tanpa nama ganda.

```sql
SELECT DISTINCT customer_name
FROM `karirnex-day-2-batch-7.showroom_car.showroom_mobil`;
```

Hasil: **3.977** nama pelanggan unik.

### 2. Rekap transaksi kredit

Skenario: tim finance merekap transaksi kredit untuk laporan cicilan.

```sql
SELECT
  order_id,
  branch,
  total_sales
FROM `karirnex-day-2-batch-7.showroom_car.showroom_mobil`
WHERE payment_type = 'Kredit';
```

Hasil: **4.253** transaksi. Contoh baris:

| order_id | branch | total_sales |
|---|---|---|
| TRX00081 | Bandung | 290.000.000 |
| TRX01132 | Bandung | 290.000.000 |
| TRX01390 | Bandung | 290.000.000 |

### 3. Lima transaksi dengan quantity terbanyak

Skenario: inventory memantau transaksi borongan.

```sql
SELECT
  order_id,
  customer_name,
  product_name,
  quantity
FROM `karirnex-day-2-batch-7.showroom_car.showroom_mobil`
ORDER BY quantity DESC
LIMIT 5;
```

| order_id | customer_name | product_name | quantity |
|---|---|---|---|
| TRX01301 | Customer_1271 | Daihatsu Xenia 1.5 R CVT | 3 |
| TRX08323 | Customer_909 | Mitsubishi Xpander Ultimate CVT | 3 |
| TRX09440 | Customer_1221 | Hyundai Creta Prime AT | 3 |
| TRX09088 | Customer_3258 | Daihatsu Xenia 1.5 R CVT | 3 |
| TRX07623 | Customer_4268 | Daihatsu Xenia 1.5 R CVT | 3 |

**Catatan:** quantity tertinggi adalah 3 dan kelima baris bernilai sama. Jika ada lebih dari 5 transaksi dengan quantity 3, urutan antar-transaksi yang seri tidak ditentukan oleh query ini.

### 4. Model mobil warna Hitam Metalik

Skenario: sales manager ingin tahu model Hitam Metalik yang pernah terjual.

```sql
SELECT DISTINCT product_name
FROM `karirnex-day-2-batch-7.showroom_car.showroom_mobil`
WHERE color = 'Hitam Metalik';
```

Hasil: **16** model, antara lain Daihatsu Terios 1.5 R CVT, Daihatsu Xenia 1.5 R CVT, Honda BR-V 1.5 E CVT, Honda CR-V 1.5 Turbo RS, Honda HR-V 1.5 E CVT, dan Isuzu D-Max Double Cabin 4x4 AT.

### 5. Komposisi metode pembayaran

Skenario: melihat jumlah transaksi untuk setiap metode pembayaran.

```sql
SELECT
  payment_type,
  COUNT(*) AS jumlah_transaksi
FROM `karirnex-day-2-batch-7.showroom_car.showroom_mobil`
GROUP BY payment_type
ORDER BY jumlah_transaksi DESC;
```

| payment_type | jumlah_transaksi | Porsi |
|---|---|---|
| Kredit | 4.253 | 42,5% |
| Cash | 2.224 | 22,2% |
| Kredit + Trade In | 1.999 | 20,0% |
| Cash + Trade In | 1.524 | 15,2% |

**Insight:** kredit adalah metode yang paling banyak dipakai, dan sekitar 35% transaksi (3.523 dari 10.000) menggunakan trade-in.

### 6. Cabang dengan total pendapatan di atas Rp50 miliar

Skenario: direktur ingin memberi apresiasi pada cabang dengan pendapatan tinggi.

```sql
SELECT
  branch,
  SUM(total_sales) AS total_pendapatan
FROM `karirnex-day-2-batch-7.showroom_car.showroom_mobil`
GROUP BY branch
HAVING SUM(total_sales) > 50000000000
ORDER BY total_pendapatan DESC;
```

| branch | total_pendapatan (Rp) |
|---|---|
| Malang | 325.311.000.000 |
| Jakarta | 322.346.000.000 |
| Padang | 321.583.000.000 |
| Palembang | 314.018.000.000 |
| Bandung | 309.919.000.000 |
| Batam | 307.843.000.000 |
| Kendari | 304.402.000.000 |
| Semarang | 300.335.000.000 |

Query mengembalikan 10 baris. Dua cabang lainnya (Pontianak dan Karawang) berada di bawah Semarang.

**Insight:** semua cabang melampaui Rp50 miliar, jadi ambang tersebut tidak membedakan cabang. Performa antar cabang relatif merata, dengan selisih Malang (tertinggi) dan Semarang sekitar Rp25 miliar.

### 7. Rata-rata diskon per kategori

Skenario: tim pricing mengevaluasi kategori yang paling sering diberi diskon besar.

```sql
SELECT
  category,
  AVG(discount) AS rata_rata_discount
FROM `karirnex-day-2-batch-7.showroom_car.showroom_mobil`
GROUP BY category
ORDER BY rata_rata_discount DESC;
```

| category | rata_rata_discount (Rp) |
|---|---|
| Hatchback | 55.877 |
| City Car (EV) | 45.361 |
| Pickup | 38.760 |
| MPV | 29.630 |
| SUV | 28.697 |

### 8. Transaksi dengan diskon di atas rata-rata

Skenario: tim pricing menyelidiki transaksi dengan diskon yang dicurigai terlalu besar.

```sql
SELECT
  order_id,
  product_name,
  discount
FROM `karirnex-day-2-batch-7.showroom_car.showroom_mobil`
WHERE discount > (
  SELECT AVG(discount)
  FROM `karirnex-day-2-batch-7.showroom_car.showroom_mobil`
);
```

Hasil: **318** transaksi. Contoh baris:

| order_id | product_name | discount |
|---|---|---|
| TRX08940 | Daihatsu Terios 1.5 R CVT | 1.000.000 |
| TRX03889 | Daihatsu Xenia 1.5 R CVT | 1.000.000 |
| TRX09440 | Hyundai Creta Prime AT | 1.000.000 |

**Insight:** dari hasil nomor 7 dan 8, diskon yang tercatat tampaknya bernilai Rp1.000.000 pada 318 transaksi (3,18% dari 10.000). Karena sebagian besar transaksi tanpa diskon, rata-rata diskon sangat kecil dan setiap transaksi berdiskon otomatis berada di atasnya.

### 9. Rata-rata pendapatan per kategori (hanya transaksi completed)

Skenario: melihat rata-rata pendapatan per transaksi untuk setiap kategori, tanpa transaksi yang dibatalkan atau di-refund.

```sql
SELECT
  category,
  AVG(total_sales) AS rata_rata_pendapatan
FROM (
  SELECT
    category,
    total_sales
  FROM `karirnex-day-2-batch-7.showroom_car.showroom_mobil`
  WHERE status = 'completed'
)
GROUP BY category
ORDER BY rata_rata_pendapatan DESC;
```

| category | rata_rata_pendapatan (sekitar) |
|---|---|
| Pickup | Rp472,8 juta |
| SUV | Rp367,7 juta |
| MPV | Rp255,0 juta |
| City Car (EV) | Rp202,7 juta |
| Hatchback | Rp148,8 juta |

### 10. Tiga model mobil terlaris (CTE)

Skenario: tim produk menentukan strategi stok berdasarkan model yang paling banyak terjual.

```sql
WITH jumlah_transaksi AS (
  SELECT
    product_name,
    COUNT(*) AS total_transaksi
  FROM `karirnex-day-2-batch-7.showroom_car.showroom_mobil`
  GROUP BY product_name
)

SELECT
  product_name,
  total_transaksi
FROM jumlah_transaksi
ORDER BY total_transaksi DESC
LIMIT 3;
```

| product_name | total_transaksi |
|---|---|
| Honda HR-V 1.5 E CVT | 557 |
| Toyota Fortuner 2.4 VRZ 4x2 AT | 528 |
| Hyundai Creta Prime AT | 526 |

## Catatan Asumsi

- Pada baris contoh data, `total_sales` sama dengan `total` dikurangi `trade_in`, sehingga nilai penjualan di project ini sudah dipotong nilai tukar tambah.
- Query nomor 6, 7, 8, dan 10 tidak memfilter `status`, sehingga transaksi cancelled dan refund ikut dihitung. Hanya nomor 9 yang membatasi pada `completed`.
- Nilai `discount` dan `total_sales` dalam Rupiah.
- Pada nomor 9, BigQuery menampilkan hasil dalam notasi ilmiah; angka di tabel adalah pembulatan ke juta.

## Proyek Terkait

- Pivot table Excel (trade-in per pelanggan dan cabang): [tambahkan link]
- Pembersihan data dan analisis dengan Python (Google Colab): [tambahkan link]
- Dashboard Power BI (Dashboard Penjualan Otomotif): [tambahkan link]

## Tools

Google BigQuery (SQL)
