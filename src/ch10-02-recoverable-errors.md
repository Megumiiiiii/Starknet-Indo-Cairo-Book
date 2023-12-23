# Kesalahan yang Dapat Diperbaiki dengan `Result`

<br />

Sebagian besar kesalahan tidak cukup serius untuk memerlukan program berhenti sepenuhnya. Terkadang, ketika suatu fungsi gagal, itu disebabkan oleh suatu alasan yang dapat Anda interpretasikan dan tanggapi dengan mudah. Sebagai contoh, jika Anda mencoba menambahkan dua bilangan bulat besar dan operasinya melampaui batas nilai yang dapat direpresentasikan, Anda mungkin ingin mengembalikan kesalahan atau hasil yang dibungkus daripada menyebabkan perilaku yang tidak terdefinisi atau menghentikan proses.

## Enum `Result`

Ingat dari [“Tipe data generik”](ch08-01-generic-data-types.md#enums) di Bab 8 bahwa enum `Result` didefinisikan memiliki dua varian, `Ok` dan `Err`, sebagai berikut:

```rust,noplayground
{{#include ../listings/ch10-error-handling/no_listing_07_result_enum/src/lib.cairo}}
```

Enum `Result<T, E>` memiliki dua tipe generik, `T` dan `E`, dan dua varian: `Ok` yang menyimpan nilai bertipe `T` dan `Err` yang menyimpan nilai bertipe `E`. Definisi ini memudahkan penggunaan enum `Result` di mana pun kita memiliki operasi yang mungkin berhasil (dengan mengembalikan nilai bertipe `T`) atau gagal (dengan mengembalikan nilai bertipe `E`).

## Trait `ResultTrait`

Trait `ResultTrait` menyediakan metode-metode untuk bekerja dengan enum `Result<T, E>`, seperti membuka nilai, memeriksa apakah `Result` adalah `Ok` atau `Err`, dan memicu panic dengan pesan kustom. Implementasi `ResultTraitImpl` mendefinisikan logika dari metode-metode ini.

```rust,noplayground
{{#include ../listings/ch10-error-handling/no_listing_08_result_trait/src/lib.cairo}}
```

Metode `expect` dan `unwrap` serupa dalam hal keduanya mencoba mengekstrak nilai bertipe `T` dari `Result<T, E>` ketika berada dalam varian `Ok`. Jika `Result` adalah `Ok(x)`, keduanya mengembalikan nilai `x`. Namun, perbedaan kunci antara kedua metode tersebut terletak pada perilaku mereka saat `Result` berada dalam varian `Err`. Metode `expect` memungkinkan Anda menyediakan pesan kesalahan kustom (sebagai nilai `felt252`) yang akan digunakan saat terjadi panic, memberi Anda lebih banyak kontrol dan konteks atas panic tersebut. Di sisi lain, metode `unwrap` memicu panic dengan pesan kesalahan default, memberikan informasi yang lebih sedikit tentang penyebab panic.

Metode `expect_err` dan `unwrap_err` memiliki perilaku yang sama persis, tetapi kebalikannya. Jika `Result` adalah `Err(x)`, keduanya mengembalikan nilai `x`. Namun, perbedaan kunci antara keduanya terletak pada kasus `Result::Ok()`. Metode `expect_err` memungkinkan Anda menyediakan pesan kesalahan kustom (sebagai nilai `felt252`) yang akan digunakan saat terjadi panic, memberi Anda lebih banyak kontrol dan konteks atas panic tersebut. Di sisi lain, metode `unwrap_err` memicu panic dengan pesan kesalahan default, memberikan informasi yang lebih sedikit tentang penyebab panic.

Seorang pembaca yang teliti mungkin telah memperhatikan `<+Drop<T>>` dan `<+Drop<E>>` dalam empat tanda tangan metode pertama. Sintaks ini mewakili kendala tipe generik dalam bahasa Cairo. Kendala ini menunjukkan bahwa fungsi terkait memerlukan implementasi dari trait `Drop` untuk tipe generik `T` dan `E` masing-masing.

Terakhir, metode `is_ok` dan `is_err` adalah fungsi utilitas yang disediakan oleh trait `ResultTrait` untuk memeriksa varian dari nilai enum `Result`.

`is_ok` mengambil cuplikan nilai `Result<T, E>` dan mengembalikan `true` jika `Result` adalah varian `Ok`, yang berarti operasi berhasil. Jika `Result` adalah varian `Err`, metode ini mengembalikan `false`.

`is_err` mengambil referensi ke nilai `Result<T, E>` dan mengembalikan `true` jika `Result` adalah varian `Err`, yang berarti operasi mengalami kesalahan. Jika `Result` adalah varian `Ok`, metode ini mengembalikan `false`.

Metode-metode ini berguna ketika Anda ingin memeriksa keberhasilan atau kegagalan suatu operasi tanpa mengonsumsi nilai Result, memungkinkan Anda melakukan operasi tambahan atau membuat keputusan berdasarkan varian tanpa membungkusnya.

Anda dapat menemukan implementasi dari `ResultTrait` [di sini](https://github.com/starkware-libs/cairo/blob/main/corelib/src/result.cairo#L20).

<br />

Selalu lebih mudah dipahami dengan contoh.

Lihatlah tanda tangan fungsi ini:

```rust,noplayground
fn u128_overflowing_add(a: u128, b: u128) -> Result<u128, u128>;
```

Fungsi ini mengambil dua bilangan bulat u128, a dan b, dan mengembalikan `Result<u128, u128>` di mana varian `Ok` menyimpan jumlah jika penambahan tidak melampaui batas, dan varian `Err` menyimpan nilai yang melampaui batas jika penambahan melampaui batas.

Sekarang, kita bisa menggunakan fungsi ini di tempat lain. Misalnya:

```rust,noplayground
fn u128_checked_add(a: u128, b: u128) -> Option<u128> {
    match u128_overflowing_add(a, b) {
        Result::Ok(r) => Option::Some(r),
        Result::Err(r) => Option::None,
    }
}
```

Di sini, fungsi ini menerima dua bilangan bulat u128, a dan b, dan mengembalikan `Option<u128>`. Ini menggunakan `Result` yang dikembalikan oleh `u128_overflowing_add` untuk menentukan keberhasilan atau kegagalan operasi penambahan. Ekspresi match memeriksa `Result` dari `u128_overflowing_add`. Jika hasilnya adalah `Ok(r)`, itu mengembalikan `Option::Some(r)` yang berisi jumlah. Jika hasilnya adalah `Err(r)`, itu mengembalikan `Option::None` untuk menunjukkan bahwa operasi telah gagal karena melampaui batas. Fungsi ini tidak memicu panic dalam kasus melampaui batas.

Mari kita lihat contoh lain yang menunjukkan penggunaan `unwrap`.

```rust,noplayground
{{#include ../listings/ch10-error-handling/listing_01/src/lib.cairo:function}}
```

<span class="caption">Listing 10-1: Menggunakan tipe Result</span>

Dalam contoh ini, fungsi `parse_u8` mengambil integer `felt252` dan mencoba mengonversinya menjadi integer `u8` menggunakan metode `try_into`. Jika berhasil, itu mengembalikan `Result::Ok(value)`, jika tidak mengembalikan `Result::Err('Invalid integer')`.

Dua kasus uji kita adalah:

```rust,noplayground
{{#rustdoc_include ../listings/ch10-error-handling/listing_01/src/lib.cairo:tests}}
```

Yang pertama menguji konversi yang valid dari `felt252` menjadi `u8`, mengharapkan metode `unwrap` tidak memicu panic. Fungsi uji kedua mencoba mengonversi nilai yang berada di luar rentang `u8`, mengharapkan metode `unwrap` memicu panic dengan pesan kesalahan 'Invalid integer'.

> Kita juga bisa menggunakan atribut #[should_panic] di sini.

### `?` operator ?

Operator terakhir yang akan kita bahas adalah operator `?`. Operator `?` digunakan untuk penanganan kesalahan yang lebih idiomatik dan ringkas. Ketika Anda menggunakan operator `?` pada tipe `Result` atau `Option`, itu akan melakukan hal berikut:

- Jika nilai adalah `Result::Ok(x)` atau `Option::Some(x)`, itu akan langsung mengembalikan nilai dalamnya, yaitu `x`.
- Jika nilai adalah `Result::Err(e)` atau `Option::None`, itu akan menyebarkan kesalahan atau `None` dengan segera keluar dari fungsi.

Operator `?` berguna ketika Anda ingin menangani kesalahan secara implisit dan membiarkan fungsi pemanggil mengatasinya.

Berikut adalah contoh penggunaannya.

```rust,noplayground
{{#include ../listings/ch10-error-handling/listing_02/src/lib.cairo:function}}
```

<span class="caption">Listing 10-1: Menggunakan operator `?`</span>

Fungsi `do_something_with_parse_u8` mengambil nilai `felt252` sebagai input dan memanggil `parse_u8`. Operator `?` digunakan untuk menyebarkan kesalahan, jika ada, atau membuka kemasan nilai yang berhasil.

Dan dengan sedikit kasus uji:

```rust,noplayground
{{#rustdoc_include ../listings/ch10-error-handling/listing_02/src/lib.cairo:tests}}
```

Konsol akan mencetak kesalahan "Invalid Integer".

<br/>

### Ringkasan

Kita melihat bahwa kesalahan yang dapat diperbaiki dapat ditangani di Cairo menggunakan enum Result, yang memiliki dua varian: `Ok` dan `Err`. Enum `Result<T, E>` bersifat generik, dengan tipe `T` dan `E` mewakili nilai yang berhasil dan nilai kesalahan, secara berturut-turut. `ResultTrait` menyediakan metode-metode untuk bekerja dengan `Result<T, E>`, seperti membuka nilai, memeriksa apakah hasilnya adalah `Ok` atau `Err`, dan memicu panic dengan pesan kustom.

Untuk menangani kesalahan yang dapat diperbaiki, sebuah fungsi dapat mengembalikan tipe `Result` dan menggunakan pola pencocokan untuk menangani keberhasilan atau kegagalan suatu operasi. Operator `?` dapat digunakan untuk menangani kesalahan secara implisit dengan menyebarkan kesalahan atau membuka kemasan nilai yang berhasil. Ini memungkinkan penanganan kesalahan yang lebih ringkas dan jelas, di mana pemanggil bertanggung jawab untuk mengelola kesalahan yang dibangkitkan oleh fungsi yang dipanggil.