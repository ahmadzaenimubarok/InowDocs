# Table Short URL Bloat: Analisis Akar Masalah & Penyelesaian

## Pernyataan Masalah

Sebuah scheduled task yang menghasilkan short URL menyebabkan **CPU MySQL mencapai 100%**. Process list menunjukkan **13+ query identik** berjalan secara bersamaan:

```sql
SELECT * FROM `short_urls` WHERE `original_url` = '...'
```

Semua query dalam status `Sending data` dengan waktu eksekusi 0 hingga 2 detik.

## Analisis Akar Masalah

### Masalah 1: Tidak Ada Index pada Kolom Lookup

Tabel `short_urls` melakukan query pada kolom `TEXT` **tanpa index**. Setiap pencarian melakukan **full table scan**.

```
Tanpa index: "Cari 'Budi' di buku telepon 1000 halaman dengan membaca setiap halaman"
Dengan index: "Cari 'Budi' di daftar isi, langsung ke halaman 42"
```

### Masalah 2: Query Dipanggil dalam Loop Berfrekuensi Tinggi

Sebuah cron job yang berjalan **setiap menit** menghasilkan short URL untuk setiap record yang cocok, mengeksekusi query pencarian tanpa cache setiap kali.

```
70 record × 1.440 menit/hari = 100.800 query/hari
```

### Masalah 3: Enkripsi ID Non-Deterministik

URL builder menggunakan `Crypt::encrypt()` dengan **random IV** (Initialization Vector). Setiap pemanggilan menghasilkan **output terenkripsi yang berbeda** untuk input yang sama:

```
Panggilan 1: encrypt(7258) → "ZXlKcGRpSTZJ..."
Panggilan 2: encrypt(7258) → "QwFtYmNkZWdo..."  (berbeda!)
Panggilan 3: encrypt(7258) → "R3N4aWp5bGt1..."  (berbeda lagi!)
```

Akibatnya `original_url` **selalu unik**, sehingga:

- Pengecekan keberadaan selalu miss (mengembalikan null)
- Row baru selalu diinsert
- Tidak ada duplikat terdeteksi — setiap row memiliki URL unik

### Masalah 4: Operasi Mahal Ditempatkan Terlalu Awal

Short URL dihasilkan **sebelum** diperiksa apakah benar-benar diperlukan:

```php
// SEBELUM: Dihasilkan untuk SEMUA record (boros)
$url = UrlShortener::shorten($route);

// Kemudian: Hanya digunakan jika kondisi terpenuhi
if ($needsAction) {
    // gunakan $url
}
```

### Akumulasi Data

Dalam 3 hari, tabel `short_urls` mengakumulasi **334.571 row** — semuanya dari satu cron job yang berjalan setiap menit.

## Penyelesaian

### 1. Tambah Index pada Kolom Lookup

```sql
ALTER TABLE short_urls ADD INDEX idx_original_url (original_url(191));
```

!!! note
    Untuk kolom `TEXT` di MySQL, kamu harus menggunakan **prefix index**. Pilih panjang yang cukup untuk menampung variasi URL yang umum (misal 191 karakter).

### 2. Generasi Slug Deterministik

Alih-alih slug acak, generate **slug stabil** dari unique business key menggunakan hash:

```php
// Acak (BURUK — slug berbeda setiap kali)
$slug = Str::random(6);

// Deterministik (BAGUS — slug sama untuk key yang sama)
$slug = substr(hash('crc32', $uniqueKey), 0, 8);
```

**Implementasi lengkap:**

```php
public static function shortenDeterministic(string $url, string $uniqueKey): string
{
    $slug = substr(hash('crc32', $uniqueKey), 0, 8);

    $existing = ShortUrl::where('slug', $slug)->first();

    if ($existing) {
        if ($existing->original_url === $url) {
            return $baseUrl . '/c/' . $slug;
        }
        // Collision: tambahkan counter
        $counter = 1;
        while (ShortUrl::where('slug', $slug . $counter)->exists()) {
            $counter++;
        }
        $slug = $slug . $counter;
    }

    ShortUrl::create(['slug' => $slug, 'original_url' => $url]);
    return $baseUrl . '/c/' . $slug;
}
```

| Pendekatan | Perilaku |
|-----------|----------|
| Slug acak | Slug berbeda setiap panggilan → row baru setiap kali |
| Slug deterministik | Slug sama untuk key yang sama → reuse row yang sudah ada |

### 3. Pindahkan Operasi Mahal ke Scope yang Tepat

```php
// SEBELUM: Boros — dihasilkan untuk semua record
$url = generateShortUrl($route);
if ($needsAction) {
    // gunakan $url
}

// SESUDAH: Efisien — hanya dihasilkan saat dibutuhkan
if ($needsAction) {
    $url = generateShortUrl($route, $recordId);
    // gunakan $url
}
```

### 4. In-Memory Cache untuk Batch Processing

Saat memproses record dalam loop, cache hasilnya untuk menghindari query database berulang dalam satu eksekusi:

```php
private array $cache = [];

// Di dalam loop:
if (!isset($this->cache[$recordId])) {
    $this->cache[$recordId] = UrlShortener::shortenDeterministic($url, $recordId);
}
$shortUrl = $this->cache[$recordId];
```

### 5. Cleanup Terjadwal

Buat command terjadwal untuk menghapus record lama:

```php
protected $signature = 'short-url:cleanup {--days=3 : Hapus record lebih tua dari N hari}';

public function handle(): int
{
    $days = (int) $this->option('days');
    $deleted = ShortUrl::where('created_at', '<', now()->subDays($days))->delete();
    $this->info("Berhasil menghapus {$deleted} record lebih tua dari {$days} hari.");
    return self::SUCCESS;
}
```

Daftarkan di scheduler:

```php
$schedule->command('short-url:cleanup --days=3')->daily();
```

## Hasil

| Metrik | Sebelum | Sesudah |
|--------|---------|---------|
| Row per record | N (satu per run cron) | 1 (sekali saja) |
| Rate insert harian | ~100.800/hari | ~70/hari (hanya record baru) |
| Query per run cron | N × 2 (check + insert) | 0 (cache hit) |
| Pertumbuhan table | Tidak terbatas | Terbatas + auto-cleanup |

### Verifikasi

Setelah deploy fix, jalankan cron job dua kali untuk record yang sama:

```
Run 1: INSERT → 1 row (slug: 4b7b5f49)
Run 2: SELECT by slug → ditemukan → reuse → 0 row baru
```

Total row: **1** (tidak berubah setelah run kedua).

## Pelajaran

1. **Selalu index kolom yang digunakan di `WHERE`** — terutama pada kolom `TEXT` di mana MySQL tidak bisa menggunakan B-tree index tanpa prefix length.
2. **Gunakan ID deterministik** ketika input yang sama harus menghasilkan output yang sama. Random IV pada enkripsi membuat input identik menghasilkan output berbeda.
3. **Tunda operasi mahal** — jangan generate resource (short URL, token, thumbnail) sampai benar-benar diperlukan.
4. **Cache dalam batch loop** — meskipun caching lintas eksekusi tidak mungkin, in-memory caching dalam satu batch run menghilangkan query redundan.
5. **Jadwalkan cleanup** — setiap tabel yang tumbuh tanpa batas perlu memiliki retensi policy.
