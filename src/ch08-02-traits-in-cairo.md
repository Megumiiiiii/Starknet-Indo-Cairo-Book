# Sifat-sifat di Cairo

Sebuah sifat menentukan serangkaian metode yang dapat diimplementasikan oleh suatu tipe. Metode-metode ini dapat dipanggil pada instansi dari tipe tersebut ketika sifat ini diimplementasikan.
Sebuah sifat yang digabungkan dengan tipe generik menentukan fungsionalitas yang dimiliki oleh suatu tipe tertentu dan dapat dibagikan dengan tipe-tipe lain. Kita dapat menggunakan sifat untuk menentukan perilaku bersama secara abstrak.
Kita dapat menggunakan _batasan sifat_ untuk menentukan bahwa suatu tipe generik dapat menjadi tipe apa pun yang memiliki perilaku tertentu.

> Catatan: Sifat serupa dengan fitur yang sering disebut antarmuka dalam bahasa pemrograman lain, meskipun dengan beberapa perbedaan.

Meskipun sifat dapat ditulis untuk tidak menerima tipe generik, mereka paling berguna ketika digunakan dengan tipe generik. Kita sudah membahas generik pada [bab sebelumnya](./ch08-01-generic-data-types.md), dan kita akan menggunakannya dalam bab ini untuk menunjukkan bagaimana sifat dapat digunakan untuk menentukan perilaku bersama untuk tipe generik.

## Menentukan Sifat

Perilaku suatu tipe terdiri dari metode-metode yang dapat kita panggil pada tipe tersebut. Tipe-tipe yang berbeda berbagi perilaku yang sama jika kita dapat memanggil metode yang sama pada semua tipe tersebut. Definisi sifat adalah cara untuk mengelompokkan tanda tangan metode bersama untuk menentukan serangkaian perilaku yang diperlukan untuk mencapai suatu tujuan.

Sebagai contoh, katakanlah kita memiliki sebuah struktur `NewsArticle` yang menyimpan berita di lokasi tertentu. Kita dapat menentukan sifat `Summary` yang menjelaskan perilaku sesuatu yang dapat merangkum tipe `NewsArticle`.

```rust,noplayground
{{#rustdoc_include ../listings/ch08-generic-types-and-traits/no_listing_14_simple_trait/src/lib.cairo:trait}}
```

<!-- TODO tambahkan visibilitas `pub` -->

Di sini, kita mendeklarasikan suatu sifat menggunakan kata kunci `trait` dan kemudian nama sifatnya, yaitu `Summary` dalam hal ini.

<!-- Kita juga telah mendeklarasikan sifat sebagai pub agar kerangka yang bergantung pada kerangka ini dapat menggunakan sifat ini juga, seperti yang akan kita lihat dalam beberapa contoh.  -->

Di dalam kurung kurawal, kita mendeklarasikan tanda tangan metode yang menjelaskan perilaku dari tipe-tipe yang mengimplementasikan sifat ini, yang dalam hal ini adalah `fn summarize(self: @NewsArticle) -> ByteArray`. Setelah tanda tangan metode, alih-alih memberikan implementasi di dalam kurung kurawal, kita menggunakan titik koma.

> Catatan: Tipe `ByteArray` adalah tipe yang digunakan untuk merepresentasikan String di Cairo.

Karena sifat tidak bersifat generik, parameter `self` juga tidak bersifat generik dan bertipe `@NewsArticle`. Ini berarti bahwa metode `summarize` hanya dapat dipanggil pada instansi `NewsArticle`.

Sekarang, pertimbangkan bahwa kita ingin membuat kerangka perpustakaan pengumpul media yang dinamai `aggregator` yang dapat menampilkan ringkasan data yang mungkin disimpan dalam instansi `NewsArticle` atau `Tweet`. Untuk melakukannya, kita memerlukan ringkasan dari setiap tipe, dan kita akan meminta ringkasan tersebut dengan memanggil metode `summarize` pada suatu instansi. Dengan menentukan sifat `Summary` pada tipe generik `T`, kita dapat mengimplementasikan metode `summarize` pada tipe apa pun yang ingin kita rangkum.

```rust,noplayground
{{#rustdoc_include ../listings/ch08-generic-types-and-traits/no_listing_15_traits/src/lib.cairo:trait}}
```

<span class="caption">Sebuah sifat `Summary` yang terdiri dari perilaku yang diberikan oleh metode `summarize`</span>

Setiap tipe generik yang mengimplementasikan sifat ini harus menyediakan perilaku kustomnya sendiri untuk tubuh metode. Kompiler akan menegakkan bahwa setiap tipe yang memiliki sifat Summary akan memiliki metode summarize yang didefinisikan dengan tanda tangan ini secara tepat.

Sebuah sifat dapat memiliki beberapa metode di dalam tubuhnya: tanda tangan metode terdaftar satu per baris dan setiap baris diakhiri dengan titik koma.

## Mengimplementasikan Sifat pada Sebuah Tipe

Sekarang bahwa kita telah menentukan tanda tangan yang diinginkan dari metode-metode sifat `Summary`,
kita dapat mengimplementasikannya pada tipe-tipe di pengumpul media kita. Potongan kode berikut menunjukkan
implementasi sifat `Summary` pada struktur `NewsArticle` yang menggunakan
judul, penulis, dan lokasi untuk membuat nilai kembalian dari
`summarize`. Untuk struktur `Tweet`, kita mendefinisikan `summarize` sebagai nama pengguna
diikuti oleh seluruh teks tweet, dengan asumsi konten tweet
sudah terbatas pada 280 karakter.

```rust,noplayground
{{#rustdoc_include ../listings/ch08-generic-types-and-traits/no_listing_15_traits/src/lib.cairo:impl}}
```

Mengimplementasikan sifat pada suatu tipe mirip dengan mengimplementasikan metode biasa. Perbedaannya adalah setelah `impl`, kita menentukan nama implementasi,
kemudian menggunakan kata kunci `of`, dan kemudian menentukan nama sifat untuk implementasi tersebut.
Jika implementasinya adalah untuk tipe generik, kita menempatkan nama tipe generik di dalam tanda kurung sudut setelah nama sifat.

Di dalam blok `impl`, kita menempatkan tanda tangan metode
yang telah ditentukan oleh definisi sifat. Alih-alih menambahkan titik koma setelah setiap
tanda tangan, kita menggunakan kurung kurawal dan mengisi tubuh metode dengan perilaku spesifik
yang kita inginkan agar metode-metode sifat memiliki perilaku tertentu untuk tipe tertentu.

Sekarang bahwa perpustakaan telah mengimplementasikan sifat `Summary` pada `NewsArticle` dan
`Tweet`, pengguna dari kerangka ini dapat memanggil metode sifat pada instansi
`NewsArticle` dan `Tweet` dengan cara yang sama seperti memanggil metode biasa. Satu-satunya
perbedaan adalah bahwa pengguna harus mengimpor sifat ke dalam lingkup serta tipe-tipe tersebut.
Berikut adalah contoh bagaimana kerangka dapat digunakan:

```rust
{{#rustdoc_include ../listings/ch08-generic-types-and-traits/no_listing_15_traits/src/lib.cairo:main}}
```

Kode ini mencetak keluar:

```text
Berita baru tersedia! Cairo telah menjadi bahasa paling populer bagi pengembang oleh Cairo Digger (Seluruh Dunia)

1 tweet baru: EliBenSasson: Crypto is full of short-term maximizing projects.
 @Starknet and @StarkWareLtd are about long-term vision maximization.
```

Kerangka lain yang bergantung pada kerangka `aggregator` juga dapat mengimpor sifat `Summary`
ke dalam lingkup untuk mengimplementasikan `Summary` pada tipe-tipe mereka sendiri.

<!-- TODO: move traits as parameters here -->
<!-- ## Traits as parameters

Now that you know how to define and implement traits, we can explore how to use
traits to define functions that accept many different types. We'll use the
`Summary` trait we implemented on the `NewsArticle` and `Tweet` types to define a `notify` function that calls the `summarize` method
on its `item` parameter, which is of some type that implements the `Summary`
trait. To do this, we use the `impl Trait` syntax, like this:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-04-traits-as-parameters/src/lib.rs:here}}
```

Instead of a concrete type for the `item` parameter, we specify the `impl`
keyword and the trait name. This parameter accepts any type that implements the
specified trait. In the body of `notify`, we can call any methods on `item`
that come from the `Summary` trait, such as `summarize`. We can call `notify`
and pass in any instance of `NewsArticle` or `Tweet`. Code that calls the
function with any other type, such as a `String` or an `i32`, won’t compile
because those types don’t implement `Summary`. -->

<!-- TODO trait bound syntax -->

<!-- TODO multiple trait bounds -->

<!-- TODO: Using trait bounds to conditionally implement methods -->

## Mengimplementasikan Sifat, Tanpa Menulis Deklarasinya.

Anda dapat menulis implementasi langsung tanpa mendefinisikan sifat yang sesuai. Ini dimungkinkan dengan menggunakan atribut `#[generate_trait]` dalam implementasi, yang akan membuat kompiler secara otomatis menghasilkan sifat yang sesuai dengan implementasi tersebut. Ingatlah untuk menambahkan `Trait` sebagai akhiran dari nama sifat Anda, karena kompiler akan membuat sifat tersebut dengan menambahkan akhiran `Trait` ke nama implementasi.

```rust,noplayground
{{#include ../listings/ch08-generic-types-and-traits/no_listing_16_generate_trait/src/lib.cairo}}
```

Dalam kode tersebut, tidak perlu mendefinisikan sifat secara manual. Kompiler akan menangani definisinya secara otomatis, secara dinamis menghasilkan dan memperbarui sifat saat fungsi-fungsi baru diperkenalkan.

<!-- TODO: rework this section -->

## Mengelola dan Menggunakan Implementasi Sifat Eksternal

Untuk menggunakan metode-metode sifat, Anda perlu memastikan bahwa sifat/implemetasi yang benar diimpor. Dalam kode di atas, kita mengimpor `PrintTrait` dari `debug` dengan `use core::debug::PrintTrait;` untuk menggunakan metode `print()` pada tipe-tipe yang didukung. Semua sifat yang termasuk dalam prelude tidak perlu diimpor secara eksplisit dan dapat diakses secara bebas.

Dalam beberapa kasus, Anda mungkin perlu mengimpor tidak hanya sifat tetapi juga implementasinya jika mereka dideklarasikan dalam modul terpisah.
Jika `CircleGeometry` berada di modul/berkas terpisah `circle`, maka untuk menggunakan `boundary` pada `circ: Circle`, kita perlu mengimpor `CircleGeometry` selain `ShapeGeometry`.

Jika kode diorganisir ke dalam modul seperti ini, di mana implementasi sifat didefinisikan dalam modul yang berbeda dengan sifat itu sendiri, mengimpor implementasi yang relevan secara eksplisit diperlukan.

```rust,noplayground
use core::debug::PrintTrait;

// struct Circle { ... } and struct Rectangle { ... }

mod geometry {
    use super::Rectangle;
    trait ShapeGeometry<T> {
        // ...
    }

    impl RectangleGeometry of ShapeGeometry<Rectangle> {
        // ...
    }
}

// Bisa berada di berkas yang berbeda
mod circle {
    use super::geometry::ShapeGeometry;
    use super::Circle;
    impl CircleGeometry of ShapeGeometry<Circle> {
        // ...
    }
}

fn main() {
    let rect = Rectangle { height: 5, width: 7 };
    let circ = Circle { radius: 5 };
    // Gagal dengan kesalahan ini
    // Metode `area` tidak ditemukan pada... Apakah Anda mengimpor sifat dan impl yang benar?
    rect.area().print();
    circ.area().print();
}
```

Untuk membuatnya berfungsi, selain dari,

```rust
use geometry::ShapeGeometry;
```

Anda perlu mengimpor `CircleGeometry` secara eksplisit. Perhatikan bahwa Anda tidak perlu mengimpor `RectangleGeometry`, karena itu didefinisikan dalam modul yang sama dengan sifat yang diimpor, dan dengan demikian secara otomatis dipecahkan.

```rust
use circle::CircleGeometry
```
