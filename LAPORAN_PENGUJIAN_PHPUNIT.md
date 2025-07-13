# LAPORAN PENGUJIAN APLIKASI CATATAN KEUANGAN MENGGUNAKAN PHPUNIT

## 1. Pendahuluan

Aplikasi **Catatan Keuangan** merupakan sistem berbasis Laravel yang digunakan untuk mencatat transaksi harian pengguna, baik pemasukan maupun pengeluaran. Untuk memastikan aplikasi berjalan dengan baik dan sesuai fungsinya, dilakukan **pengujian otomatis menggunakan PHPUnit**. Pengujian ini penting untuk mendeteksi kesalahan (bug) lebih awal dan menjaga kestabilan sistem.

## 2. Tujuan Pengujian

- Memastikan halaman transaksi tidak bisa diakses oleh user yang belum login.
- Memastikan user yang login dapat mengakses halaman transaksi.
- Memastikan user dapat menambahkan data transaksi baru ke dalam database.
- Memastikan integritas data tetap terjaga setelah proses insert transaksi.

## 3. Tools yang Digunakan

| No | Tool / Library           | Keterangan                               |
|----|--------------------------|-------------------------------------------|
| 1  | Laravel 12               | Framework utama aplikasi                  |
| 2  | PHPUnit (Built-in)       | Framework untuk testing                   |
| 3  | Laravel TestCase         | Base class untuk pengujian feature/unit   |
| 4  | RefreshDatabase          | Trait Laravel untuk reset database        |
| 5  | UserFactory              | Untuk membuat data user palsu otomatis    |

## 4. Struktur Pengujian

File pengujian dibuat di:
tests/Feature/TransactionTest.php

php
Salin
Edit

Pengujian difokuskan pada:
- Akses halaman transaksi
- Autentikasi user
- Penambahan transaksi

## 5. Implementasi Kode Test

```php
<?php

namespace Tests\Feature;

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class TransactionTest extends TestCase
{
    use RefreshDatabase;

    /** @test */
    public function guest_redirected_from_transactions()
    {
        $response = $this->get('/transactions');
        $response->assertRedirect('/login');
    }

    /** @test */
    public function user_can_access_transactions()
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)->get('/transactions');

        $response->assertStatus(200);
    }

    /** @test */
    public function user_can_create_transaction()
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)->post('/transactions', [
            'description' => 'Bayar Listrik',
            'amount' => 100000,
            'type' => 'expense'
        ]);

        $response->assertRedirect('/transactions');
        $this->assertDatabaseHas('transactions', [
            'description' => 'Bayar Listrik',
            'amount' => 100000,
            'type' => 'expense',
            'user_id' => $user->id
        ]);
    }
}
## **6. Hasil Pengujian**
Pengujian dijalankan dengan perintah:

bash
Salin
Edit
php artisan test
Hasil:
bash
Salin
Edit
PASS  Tests\Feature\TransactionTest
✓ guest_redirected_from_transactions
✓ user_can_access_transactions
✓ user_can_create_transaction

Tests: 3 passed (8 assertions)

## **7. Tabel Test Case**
No	Nama Pengujian	Input / Aksi	Expected Output	Status
1	guest_redirected_from_transactions	Akses /transactions tanpa login	Redirect ke /login	✅
2	user_can_access_transactions	Login → akses /transactions	Status 200 OK, halaman transaksi tampil	✅
3	user_can_create_transaction	Kirim POST data transaksi baru	Redirect + Data masuk ke database	✅

## **8. Kesimpulan**
Berdasarkan hasil pengujian otomatis menggunakan PHPUnit, seluruh fitur utama yang diuji berjalan dengan baik dan sesuai fungsinya. Proses autentikasi, akses halaman, dan penyimpanan data transaksi telah tervalidasi. Dengan adanya pengujian ini, aplikasi Catatan Keuangan menjadi lebih andal dan siap digunakan dalam lingkungan produksi.

## **9. Saran Pengembangan**
Menambahkan pengujian validasi form (input kosong, format tidak sesuai)

Pengujian fitur export (PDF dan Excel)

Unit testing untuk model dan helper (jika ada)

Pengujian skenario gagal login atau data tidak ditemukan
