# Macros

Bahasa Cairo memiliki beberapa plugin yang memungkinkan pengembang menyederhanakan kode mereka. Mereka disebut `inline_macros` dan merupakan cara penulisan kode yang menghasilkan kode lain. Dalam Cairo, hanya ada dua `macros`: `array![]` dan `consteval_int!()`.

### Mari kita mulai dengan `array!`

Kadang-kadang, kita perlu membuat array dengan nilai yang sudah diketahui pada saat kompilasi. Cara dasar untuk melakukannya redundan. Pertama, Anda akan mendeklarasikan array dan kemudian menambahkan setiap nilai satu per satu. `array!` adalah cara yang lebih sederhana untuk melakukan tugas ini dengan menggabungkan dua langkah.
Pada saat kompilasi, kompiler akan membuat array dan menambahkan semua nilai yang dilewatkan ke makro `array!` secara berurutan.

Tanpa `array!`:

```rust
{{#include ../listings/ch11-advanced-features/no_listing_02_array_macro/src/lib.cairo:no_macro}}
```

Dengan `array!`:

```rust
{{#include ../listings/ch11-advanced-features/no_listing_02_array_macro/src/lib.cairo:array_macro}}
```

### `consteval_int!`

Dalam beberapa situasi, seorang pengembang mungkin perlu mendeklarasikan konstanta yang merupakan hasil dari perhitungan bilangan bulat. Untuk menghitung ekspresi konstan dan menggunakan hasilnya pada waktu kompilasi, diperlukan penggunaan makro `consteval_int!`.

Berikut adalah contoh dari `consteval_int!`:

```rust
const a: felt252 = consteval_int!(2 * 2 * 2);
```

Ini akan diinterpretasikan sebagai `const a: felt252 = 8;` oleh kompiler.