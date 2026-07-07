# Testing

Alur testing sebelum fitur dianggap siap merge.

## Setup Test Database

Jangan jalankan test di DB development. Pisahkan dengan DB khusus testing.

**Opsi 1: SQLite in-memory (paling cepat)**

Di `phpunit.xml`:

```xml
<env name="DB_CONNECTION" value="sqlite"/>
<env name="DB_DATABASE" value=":memory:"/>
```

**Opsi 2: DB MySQL terpisah**

```dotenv
# .env.testing
DB_DATABASE=nama_db_test
```

Jalankan dengan:

```bash
php artisan test --env=testing
```

## Level Testing

### 1. Unit Test — logika terisolasi

Test model, helper, atau service class tanpa menyentuh DB atau HTTP. Cocok untuk fungsi kalkulasi, transformasi data, atau validasi logika bisnis.

```bash
php artisan test --filter=NamaUnitTest
```

Contoh:

```php
public function test_diskon_tidak_boleh_negatif(): void
{
    $hasil = hitungDiskon(harga: 100000, persen: -10);
    $this->assertEquals(0, $hasil);
}
```

### 2. Feature Test — alur HTTP / endpoint

Test request → response lengkap, termasuk middleware, auth, dan DB. Gunakan `RefreshDatabase` agar setiap test mulai dari DB bersih.

```bash
php artisan test --filter=NamaFeatureTest
```

```php
use Illuminate\Foundation\Testing\RefreshDatabase;

class LoginTest extends TestCase
{
    use RefreshDatabase;

    public function test_user_bisa_login(): void
    {
        $user = User::factory()->create();

        $response = $this->postJson('/api/login', [
            'email'    => $user->email,
            'password' => 'password',
        ]);

        $response->assertStatus(200);
        $response->assertJsonStructure(['data' => ['token']]);
    }
}
```

Assertion wajib untuk setiap endpoint baru:

```php
$response->assertStatus(200);
$response->assertJsonStructure(['data' => ['id', 'name']]);
$this->assertDatabaseHas('users', ['email' => 'test@example.com']);
```

### 3. Manual QA — verifikasi di browser

Dilakukan setelah unit & feature test hijau.

Checklist QA manual:

- [ ] Flow utama berjalan (happy path)
- [ ] Validasi error muncul dengan benar
- [ ] Tidak ada console error di browser
- [ ] Tampilan oke di mobile (jika relevan)

### 4. API Testing (jika ada endpoint API)

Gunakan Postman atau Insomnia. Simpan collection di repo jika memungkinkan.

Yang harus ditest per endpoint:

- [ ] Request valid → response benar
- [ ] Request tanpa auth → `401 Unauthorized`
- [ ] Request dengan data invalid → `422 Unprocessable`
- [ ] Request ke resource orang lain → `403 Forbidden`

## Menjalankan Semua Test

```bash
php artisan test
```

Dengan output detail:

```bash
php artisan test --verbose
```

Dengan coverage (butuh Xdebug atau PCOV):

```bash
php artisan test --coverage
```

## Check Before Merge

- [ ] Semua test hijau (`php artisan test`)
- [ ] Tidak ada `.env` / secret ter-commit
- [ ] Migration sudah diuji di DB bersih (`php artisan migrate:fresh`)
- [ ] Sudah di-review minimal 1 orang (lihat [Code Review SOP](code-review.md))
- [ ] Manual QA sudah dilakukan untuk perubahan UI

!!! warning "Jangan skip"
    Kalau test merah, jangan di-merge. Perbaiki dulu sebelum lanjut.

!!! tip "Test setelah pull"
    Setiap kali pull dari `develop`, jalankan `php artisan test` untuk memastikan
    tidak ada konflik yang merusak test orang lain.
