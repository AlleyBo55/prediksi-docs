# Mekanisme Pasar Prediksi

## Gambaran Umum

Predixi adalah platform pasar prediksi terdesentralisasi yang memungkinkan pengguna membuat dan memperdagangkan pasar dengan hasil biner. Sistem ini menggabungkan automated market making (AMM) dengan mekanisme penyelesaian sengketa yang kuat.

## Pembuatan Pasar

### Persyaratan
- **Jaminan Resolusi Minimum**: 100.000 IDR
- **Durasi Minimum**: 1 jam dari pembuatan
- **Biaya Pembuatan**: Biaya dinamis yang dihitung oleh layanan AMM

### Jenis Pasar
1. **Pasar Biner**: Hasil YA/TIDAK
2. **Pasar Multi-hasil**: Beberapa kemungkinan hasil

### Jenis Resolusi
1. **Resolusi Pembuat**: Hanya pembuat pasar yang dapat menyelesaikan
2. **Resolusi Oracle**: Oracle eksternal memberikan resolusi
3. **Resolusi Komunitas**: Voting komunitas menentukan hasil

## Mekanisme Trading

### Automated Market Maker (AMM)
- **Algoritma**: Constant Product Market Maker (x × y = k)
- **Pool Awal**: 1.000.000 IDR masing-masing untuk YA dan TIDAK
- **Harga Dinamis**: Harga menyesuaikan otomatis berdasarkan rasio pool
- **Selalu Likuid**: Tidak pernah berhenti menerima taruhan, self-balancing
- **Struktur Fee**: Fee trading 2% (1,5% platform + 0,5% creator)

### Persyaratan Trading
- **Taruhan Minimum**: 10.000 IDR per transaksi
- **Taruhan Maksimum**: 10.000.000 IDR per transaksi
- **Maksimum Per User Per Pasar**: 10.000.000 IDR total
- **Tidak Bisa Dibatalkan**: Taruhan final setelah ditempatkan

### Sistem Harga Berbasis AMM (Constant Product Market Maker)
- **Formula**: yes_pool × no_pool = k (konstan)
- **Pool Awal**: 1.000.000 IDR masing-masing (YA dan TIDAK)
- **Harga Dinamis**: Harga menyesuaikan otomatis berdasarkan volume taruhan
- **Sistem Saham**: Anda menerima saham berdasarkan kalkulasi AMM, bukan jumlah tetap

### Cara Bertaruh

#### Pasar Biner (YA/TIDAK)
- **Jumlah Taruhan**: Anda pilih berapa IDR yang ingin ditaruhkan (10k-10M)
- **Saham Diterima**: Dihitung oleh AMM berdasarkan pool saat ini
- **Tidak Ada Short Selling**: Anda hanya bisa beli posisi YA atau TIDAK
- **Contoh**: Taruhan 100.000 IDR pada YA → Dapat ~89.254 saham (bervariasi sesuai kondisi pool)

#### Pasar Multi-Hasil
- **Sistem Pool Sederhana**: Setiap hasil memiliki pool sendiri
- **Saham Sama**: 1 IDR taruhan = 1 saham (lebih sederhana dari AMM biner)
- **Contoh**: Taruhan 100.000 IDR pada hasil A → Dapat 100.000 saham hasil A

### Struktur Fee

#### Fee Trading (Dikenakan Saat Bertaruh)
- **Total Fee**: 2% dari jumlah taruhan
- **Fee Platform**: 1,5% ke platform
- **Fee Creator**: 0,5% ke pembuat pasar
- **Contoh**: Taruhan 100.000 IDR → Fee 2.000 IDR → 98.000 IDR masuk pool

#### Fee Klaim (Dikenakan Saat Menang)
- **Fee Klaim**: 1% dari profit saja (bukan principal)
- **Contoh**: Menang 150.000 IDR dari taruhan 100.000 IDR → Profit: 50.000 IDR → Fee: 500 IDR

### Kalkulasi Pembayaran

#### Pasar Biner (Berbasis AMM)
- **Formula**: (saham_anda ÷ pool_menang) × total_pool
- **Contoh**: Anda punya 89.254 saham, YA menang, pool akhir: 2M YA + 1M TIDAK
- **Kalkulasi**: (89.254 ÷ 2.000.000) × 3.000.000 = 133.881 IDR
- **Minus Fee Klaim**: 1% dari profit = ~339 IDR fee

#### Pasar Multi-Hasil (Sederhana)
- **Formula**: (saham_anda ÷ pool_hasil_menang) × total_semua_pool
- **Lebih Sederhana**: Jika hasil A menang dan Anda punya saham A, dapat pembayaran proporsional

### Penemuan Harga

#### Bagaimana Harga Berubah
- **Efek AMM**: Taruhan besar menyebabkan pergerakan harga lebih besar (slippage)
- **Contoh**: Taruhan 100k menggerakkan harga dari 50% ke 54,7% di pasar seimbang
- **Contoh**: Taruhan 100k yang sama menggerakkan harga dari 70% ke 72% di pasar tidak seimbang
- **Anti-Manipulasi**: Secara eksponensial mahal untuk menggerakkan harga secara signifikan

## Siklus Hidup Pasar

### 1. Fase Aktif
- Pengguna dapat memperdagangkan posisi YA/TIDAK
- Harga berfluktuasi berdasarkan aktivitas trading
- Pasar tetap terbuka hingga waktu berakhir

### 2. Fase Berakhir
- Trading berhenti pada waktu yang ditentukan
- Pasar menunggu resolusi
- Hanya pihak yang berwenang dapat menyelesaikan berdasarkan jenis resolusi

### 3. Fase Terselesaikan
- Hasil ditentukan (YA atau TIDAK)
- Periode sengketa 24 jam dimulai
- Posisi menang dapat diklaim
- Posisi kalah menjadi tidak berharga

### 4. Fase Disengketakan (Opsional)
- Pengguna mana pun (kecuali pembuat) dapat menyengketakan resolusi
- Memerlukan jaminan sengketa sama dengan jaminan resolusi
- Periode voting komunitas: 72 jam
- Minimum 10 suara diperlukan untuk finalisasi

### 5. Fase Final
- Hasil akhir dikonfirmasi
- Semua posisi diselesaikan
- Jaminan didistribusikan ke pihak yang tepat

## Sistem Penyelesaian Sengketa

### Persyaratan Sengketa
- Pasar harus dalam status "terselesaikan"
- Dalam jendela sengketa 24 jam
- Penyengketa tidak boleh pembuat pasar
- Harus membayar jaminan sengketa (sama dengan jaminan resolusi)

### Voting Komunitas
- **Periode Voting**: 72 jam
- **Suara Minimum**: 10 suara diperlukan
- **Metode Keputusan**: Mayoritas sederhana (>50%)
- **Kekuatan Voting**: Bobot sama per pengguna

### Distribusi Jaminan
- **Resolusi Tidak Disengketakan**: Pembuat menerima jaminan resolusi kembali
- **Sengketa Berhasil**: Penyengketa menerima kedua jaminan (pembuat + penyengketa)
- **Sengketa Gagal**: Pembuat menerima kedua jaminan (pembuat + penyengketa)

## Insentif Ekonomi

### Untuk Pembuat Pasar
- **Pendapatan**: Fee trading dari aktivitas pasar
- **Pemulihan Jaminan**: Jaminan resolusi dikembalikan jika tidak disengketakan
- **Risiko**: Kehilangan jaminan jika disengketakan dan komunitas tidak setuju

### Untuk Trader
- **Keuntungan**: Beli rendah, jual tinggi atau tahan hingga kedaluwarsa
- **Risiko**: Kehilangan seluruh posisi jika hasil salah

### Untuk Penyengketa
- **Hadiah**: Menang kedua jaminan jika komunitas setuju
- **Risiko**: Kehilangan jaminan sengketa jika komunitas tidak setuju

### Untuk Pemilih
- **Insentif**: Menjaga integritas pasar
- **Tanggung Jawab**: Penentuan hasil yang akurat

## Fitur Keamanan

### Kontrol Berbasis Waktu
- Durasi pasar minimum (1 jam)
- Periode sengketa tetap (24 jam)
- Periode voting tetap (72 jam)

### Keamanan Ekonomi
- Jaminan resolusi mencegah pembuatan pasar sembarangan
- Jaminan sengketa mencegah spam sengketa
- Voting komunitas mencegah manipulasi

### Kontrol Akses
- Resolusi khusus pembuat untuk pasar pembuat
- Resolusi khusus oracle untuk pasar oracle
- Resolusi khusus komunitas untuk pasar yang disengketakan

### Perlindungan Teknis
- Operasi saldo atomik
- Perlindungan overflow
- Pemeriksaan validasi status

## Kategori Pasar

### Kategori yang Didukung
- **Olahraga**: Acara dan kompetisi olahraga
- **Politik**: Pemilu dan acara politik
- **Kripto**: Prediksi harga cryptocurrency
- **Cuaca**: Hasil terkait cuaca
- **Hiburan**: Acara industri hiburan
- **Ekonomi**: Indikator dan acara ekonomi

## Titik Integrasi

### Integrasi Oracle
- **Chainlink**: Untuk feed harga dan data eksternal
- **Pyth Network**: Untuk data pasar real-time
- **Oracle Kustom**: Untuk sumber data khusus

### Integrasi Wallet
- **Solana Wallet Adapter**: Dukungan multi-wallet
- **Integrasi Web3**: Interaksi blockchain yang mulus
- **Mobile Wallets**: Kompatibilitas aplikasi mobile

## Manajemen Risiko

### Risiko Pasar
- **Risiko Likuiditas**: Pasar dengan volume trading rendah
- **Risiko Resolusi**: Hasil yang disengketakan atau tidak jelas
- **Risiko Oracle**: Kegagalan sumber data eksternal

### Risiko Pengguna
- **Risiko Trading**: Volatilitas harga pasar
- **Risiko Waktu**: Timing kedaluwarsa pasar
- **Risiko Sengketa**: Ketidaksepakatan komunitas

### Risiko Platform
- **Risiko Smart Contract**: Kerentanan kode
- **Risiko Governance**: Akurasi penyelesaian sengketa
- **Risiko Regulasi**: Persyaratan kepatuhan

## Metrik Kinerja

### Indikator Kesehatan Pasar
- **Total Volume**: Volume trading kumulatif
- **Trader Aktif**: Jumlah peserta unik
- **Akurasi Resolusi**: Persentase resolusi tidak disengketakan
- **Tingkat Sengketa**: Persentase pasar yang disengketakan

### Metrik Platform
- **Total Pasar**: Jumlah pasar yang dibuat
- **Tingkat Keberhasilan**: Persentase pasar yang berhasil diselesaikan
- **Retensi Pengguna**: Pertumbuhan pengguna aktif
- **Pendapatan**: Fee platform yang dikumpulkan

## Peningkatan Masa Depan

### Fitur yang Direncanakan
- **Jenis Order Lanjutan**: Limit order, stop loss
- **Pelacakan Portfolio**: Manajemen posisi pengguna
- **Fitur Sosial**: Diskusi dan analisis pasar
- **Aplikasi Mobile**: Pengalaman mobile native

### Solusi Scaling
- **Integrasi Layer 2**: Biaya transaksi yang dikurangi
- **Dukungan Cross-chain**: Kompatibilitas multi-blockchain
- **AMM Lanjutan**: Mekanisme penemuan harga yang ditingkatkan