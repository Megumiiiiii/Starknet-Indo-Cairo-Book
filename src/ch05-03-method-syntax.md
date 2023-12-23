## Sintaks Metode

_Metode_ mirip dengan fungsi: kita mendeklarasikannya dengan kata kunci `fn` dan sebuah nama, mereka dapat memiliki parameter dan nilai kembali, dan mereka berisi kode yang dijalankan saat metode tersebut dipanggil dari tempat lain. Berbeda dengan fungsi, metode didefinisikan dalam konteks suatu tipe dan parameter pertamanya selalu `self`, yang mewakili instansi dari tipe yang metode itu dipanggil. Bagi yang akrab dengan Rust, pendekatan Cairo mungkin membingungkan, karena metode tidak dapat didefinisikan langsung pada tipe. Sebagai gantinya, Anda harus mendefinisikan sebuah [trait](./ch08-02-traits-in-cairo.md) dan implementasi yang terkait dengan tipe yang dimaksudkan untuk metode tersebut.

### Mendefinisikan Metode

Mari ubah fungsi `area` yang memiliki sebuah instansi `Rectangle` sebagai parameter menjadi sebuah metode `area` yang didefinisikan pada trait `RectangleTrait`, seperti yang ditunjukkan pada Listing 5-13.

<span class="filename">Nama File: src/lib.cairo</span>

```rust
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing_04_13_area_method/src/lib.cairo}}
```

<span class="caption">Listing 5-13: Mendefinisikan metode `area` untuk digunakan pada `Rectangle` </span>

Untuk mendefinisikan fungsi dalam konteks `Rectangle`, kita mulai dengan mendefinisikan blok `trait` dengan tanda tangan dari metode yang ingin kita implementasikan. Trait tidak terkait dengan tipe spesifik; hanya parameter `self` dari metode yang menentukan tipe mana yang bisa digunakan dengannya. Kemudian, kita mendefinisikan blok `impl` (implementasi) untuk `RectangleTrait`, yang mendefinisikan perilaku dari metode yang diimplementasikan. Segala sesuatu dalam blok `impl` ini akan terkait dengan tipe dari parameter `self` dari metode yang dipanggil. Meskipun teknisnya memungkinkan untuk mendefinisikan metode untuk banyak tipe dalam satu blok `impl`, ini bukanlah praktik yang disarankan, karena dapat menyebabkan kebingungan. Kami menyarankan agar tipe dari parameter `self` tetap konsisten dalam satu blok `impl`. Kemudian, kita memindahkan fungsi `area` ke dalam kurung kurawal `impl` dan mengubah parameter pertama (dan dalam kasus ini, hanya satu) menjadi `self` dalam tanda tangan dan di semua bagian dalam badan fungsi. Di `main`, di mana kita memanggil fungsi `area` dan meneruskan `rect1` sebagai argumen, kita dapat menggunakan _syntax metode_ untuk memanggil metode `area` pada instansi `Rectangle` kita. Syntax metode digunakan setelah sebuah instansi: kita tambahkan titik diikuti oleh nama metode, tanda kurung, dan argumen apapun.

Metode harus memiliki parameter bernama `self` dari tipe yang akan diterapkan untuk parameter pertamanya. Perhatikan bahwa kami menggunakan operator `@` di depan tipe `Rectangle` pada tanda tangan fungsi. Dengan melakukannya, kami menunjukkan bahwa metode ini mengambil snapshot yang tidak dapat diubah dari instansi `Rectangle`, yang otomatis dibuat oleh kompiler saat melewatkan instansi ke metode. Metode dapat memiliki kepemilikan dari `self`, menggunakan `self` dengan snapshot seperti yang telah kita lakukan di sini, atau menggunakan referensi yang dapat diubah dari `self` menggunakan sintaks `ref self: T`.

Kami memilih `self: @Rectangle` di sini dengan alasan yang sama kami menggunakan `@Rectangle` pada versi fungsi: kami tidak ingin memiliki kepemilikan, dan kami hanya ingin membaca data dalam struct, bukan menulisnya. Jika kami ingin mengubah instansi yang telah kita panggil metode tersebut sebagai bagian dari apa yang dilakukan metode, kami akan menggunakan `ref self: Rectangle` sebagai parameter pertama. Memiliki sebuah metode yang memiliki kepemilikan dari instansi dengan hanya menggunakan `self` sebagai parameter pertama adalah hal yang jarang terjadi; teknik ini biasanya digunakan saat metode mengubah `self` menjadi sesuatu yang lain dan Anda ingin mencegah pemanggil untuk menggunakan instansi asli setelah transformasi tersebut.

Perhatikan penggunaan operator desnap `*` dalam metode area saat mengakses member dari struct. Ini diperlukan karena struct tersebut dilewatkan sebagai snapshot, dan semua nilai field-nya memiliki tipe `@T`, yang mengharuskan untuk didesnap agar bisa dimanipulasi.

Alasan utama menggunakan metode daripada fungsi adalah untuk organisasi dan kejelasan kode. Kami telah menempatkan semua hal yang dapat kita lakukan dengan sebuah instansi dari suatu tipe dalam satu kombinasi blok `trait` & `impl`, daripada membuat pengguna masa depan dari kode kita mencari kemampuan dari `Rectangle` di berbagai tempat dalam pustaka yang kami sediakan. Namun, kita dapat mendefinisikan beberapa kombinasi blok `trait` & `impl` untuk tipe yang sama di tempat yang berbeda, yang dapat berguna untuk organisasi kode yang lebih terperinci. Sebagai contoh, Anda bisa mengimplementasikan trait `Add` untuk tipe Anda dalam satu blok `impl`, dan trait `Sub` dalam blok lainnya.

Catatan bahwa kita dapat memilih memberikan nama metode sama dengan salah satu field dari struct tersebut. Sebagai contoh, kita dapat mendefinisikan sebuah metode pada `Rectangle` yang juga diberi nama `width`:

<span class="filename">Nama File: src/lib.cairo</span>

```rust
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing_04_14_width_method/src/lib.cairo}}
```

Di sini, kita memilih untuk membuat metode `width` mengembalikan `true` jika nilai dalam field `width` dari instansi tersebut lebih besar dari `0` dan `false` jika nilainya adalah `0`: kita dapat menggunakan field dalam sebuah metode dengan nama yang sama untuk tujuan apa pun. Di `main`, ketika kita menggunakan `rect1.width` dengan tanda kurung, Cairo mengetahui bahwa yang dimaksud adalah metode `width`. Ketika kita tidak menggunakan tanda kurung, Cairo tahu bahwa yang dimaksud adalah field `width`.

### Metode dengan Lebih Banyak Parameter

Mari kita latihan menggunakan metode dengan mengimplementasikan sebuah metode kedua pada struct `Rectangle`. Kali ini kita ingin sebuah instansi dari `Rectangle` untuk menerima instansi lain dari `Rectangle` dan mengembalikan `true` jika `Rectangle` kedua dapat benar-benar masuk dalam `self` (Rectangle pertama); jika tidak, maka seharusnya mengembalikan `false`. Artinya, setelah kita mendefinisikan metode `can_hold`, kita ingin dapat menulis program yang ditunjukkan pada Listing 5-14.

<span class="filename">Nama File: src/lib.cairo</span>

```rust,does_not_compile
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing_04_15_can_hold/src/lib.cairo:no_method}}
```

<span class="caption">Listing 5-14: Menggunakan metode `can_hold` yang belum ditulis</span>

Output yang diharapkan akan terlihat seperti berikut karena kedua dimensi dari `rect2` lebih kecil dari dimensi dari `rect1`, tetapi `rect3` lebih lebar dari `rect1`:

```text
$ scarb cairo-run --available-gas=200000000
[DEBUG]	Can rec1 hold rect2?           	(raw: 384675147322001379018464490539350216396261044799)

[DEBUG]	true                           	(raw: 1953658213)

[DEBUG]	Can rect1 hold rect3?          	(raw: 384675147322001384331925548502381811111693612095)

[DEBUG]	false                          	(raw: 439721161573)

```

Kita tahu kita ingin mendefinisikan sebuah metode, sehingga metodenya akan berada dalam blok `trait RectangleTrait` dan `impl RectangleImpl of RectangleTrait`. Nama metodenya akan menjadi `can_hold`, dan akan menerima sebuah snapshot dari `Rectangle` lain sebagai parameter. Kita dapat mengetahui tipe dari parameter tersebut dengan melihat kode yang memanggil metodenya: `rect1.can_hold(@rect2)` meneruskan `@rect2`, yang merupakan snapshot ke `rect2`, sebuah instansi dari `Rectangle`. Hal ini masuk akal karena kita hanya perlu membaca `rect2` (daripada menulis, yang berarti kita akan memerlukan pinjaman yang dapat diubah), dan kita ingin `main` mempertahankan kepemilikan dari `rect2` sehingga kita dapat menggunakannya kembali setelah memanggil metode `can_hold`. Nilai kembalian dari `can_hold` akan berupa Boolean, dan implementasinya akan memeriksa apakah lebar dan tinggi dari `self` lebih besar dari lebar dan tinggi dari `Rectangle` lain, secara berturut-turut. Mari tambahkan metode `can_hold` baru ke dalam blok `trait` dan `impl` dari Listing 5-13, seperti yang ditunjukkan pada Listing 5-15.

<span class="filename">Nama File: src/lib.cairo</span>

```rust
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing_04_15_can_hold/src/lib.cairo:trait_impl}}
```

<span class="caption">Listing 5-15: Mengimplementasikan metode `can_hold` pada `Rectangle` yang menerima instansi `Rectangle` lain sebagai parameter</span>

Ketika kita menjalankan kode ini dengan fungsi `main` pada Listing 5-14, kita akan mendapatkan output yang diinginkan. Metode dapat memiliki beberapa parameter yang kita tambahkan ke dalam tanda tangan setelah parameter `self`, dan parameter-parameter tersebut berfungsi seperti parameter dalam fungsi.

### Mengakses fungsi implementasi

Semua fungsi yang didefinisikan dalam blok `trait` dan `impl` dapat diakses langsung menggunakan operator `::` pada nama implementasi. Fungsi-fungsi dalam trait yang bukan merupakan metode sering digunakan untuk konstruktor yang akan mengembalikan instansi baru dari struct. Biasanya ini disebut `new`, tetapi `new` bukanlah nama khusus dan bukan bagian dari bahasa. Sebagai contoh, kita bisa memilih untuk menyediakan fungsi terkait yang dinamai `square` yang akan memiliki satu parameter dimensi dan menggunakan nilai tersebut sebagai lebar dan tinggi, sehingga memudahkan pembuatan `Rectangle` persegi daripada harus menentukan nilai yang sama dua kali:

<span class="filename">Nama File: src/lib.cairo</span>

```rust,noplayground
{{#include ../listings/ch05-using-structs-to-structure-related-data/no_listing_01_implementation_functions/src/lib.cairo:here}}
```

Untuk memanggil fungsi ini, kita menggunakan sintaks `::` dengan nama implementasi; `let square = RectangleImpl::square(10);` adalah contohnya. Fungsi ini berada di dalam ruang nama oleh implementasi; sintaks `::` digunakan baik untuk fungsi trait maupun ruang nama yang dibuat oleh modul. Kita akan membahas modul dalam [Chapter 8][modules].

> Catatan: Juga memungkinkan untuk memanggil fungsi ini menggunakan nama trait, dengan `RectangleTrait::square(10)`.

### Beberapa Blok `impl`

Setiap struct diizinkan memiliki beberapa blok `trait` dan `impl`. Sebagai contoh, Listing 5-15 setara dengan kode yang ditunjukkan pada Listing 5-16, di mana setiap metode berada di dalam blok `trait` dan `impl` yang terpisah.

```rust,noplayground
{{#include ../listings/ch05-using-structs-to-structure-related-data/no_listing_02_multiple_impls/src/lib.cairo:here}}
```

<span class="caption">Listing 5-16: Menulis ulang Listing 5-15 menggunakan beberapa blok `impl`</span>

Tidak ada alasan untuk memisahkan metode-metode ini ke dalam beberapa blok `trait` dan `impl` di sini, namun ini adalah sintaks yang valid. Kita akan melihat kasus di mana blok-blok multiple berguna dalam [Chapter 8](ch08-00-generic-types-and-traits.md), di mana kita membahas tipe-tipe generic dan trait.

## Ringkasan

Struct memungkinkan Anda membuat tipe kustom yang bermakna untuk domain Anda. Dengan menggunakan struct, Anda dapat menyatukan bagian-bagian data yang terkait satu sama lain dan memberi nama pada setiap bagian untuk membuat kode Anda jelas. Dalam blok `trait` dan `impl`, Anda dapat mendefinisikan metode, yang merupakan fungsi terkait dengan tipe dan memungkinkan Anda menentukan perilaku yang dimiliki oleh instansi tipe Anda.

Namun, struct bukanlah satu-satunya cara Anda dapat membuat tipe kustom: mari beralih ke fitur enum dalam Cairo untuk menambahkan alat lain dalam kotak alat Anda.