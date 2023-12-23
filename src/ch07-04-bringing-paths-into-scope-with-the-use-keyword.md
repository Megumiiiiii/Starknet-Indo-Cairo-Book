# Membawa Jalur ke dalam Lingkup dengan Kata Kunci `use`

Menulis jalur untuk memanggil fungsi dapat terasa merepotkan dan repetitif. Untungnya, ada cara untuk menyederhanakan proses ini: kita dapat membuat pintasan ke suatu jalur dengan kata kunci `use` sekali, dan kemudian menggunakan nama yang lebih pendek di mana pun di dalam lingkup.

Pada Listing 7-5, kita membawa modul `restaurant::front_of_house::hosting` ke dalam lingkup fungsi `eat_at_restaurant` sehingga kita hanya perlu menyebutkan `hosting::add_to_waitlist` untuk memanggil fungsi `add_to_waitlist` di dalam `eat_at_restaurant`.

<span class="filename">Nama File: src/lib.cairo</span>

```rust
{{#include ../listings/ch07-managing-cairo-projects-with-packages-crates-and-modules/listing_06_05/src/lib.cairo}}
```

<span class="caption">Listing 7-5: Membawa modul ke dalam lingkup dengan
`use`</span>

Menambahkan `use` dan suatu jalur dalam suatu lingkup mirip dengan membuat symbolic link dalam sistem file. Dengan menambahkan `use restaurant::front_of_house::hosting` di akar kerangka, `hosting` sekarang adalah nama yang valid di dalam lingkup tersebut, seolah-olah modul `hosting` telah didefinisikan di akar kerangka.

Perhatikan bahwa `use` hanya membuat pintasan untuk lingkup tertentu di mana `use` tersebut terjadi. Listing 7-6 memindahkan fungsi `eat_at_restaurant` ke dalam modul anak baru yang diberi nama `customer`, yang kemudian menjadi lingkup yang berbeda dari pernyataan `use`, sehingga tubuh fungsi tidak akan dikompilasi:

<span class="filename">Nama File: src/lib.cairo</span>

```rust
{{#include ../listings/ch07-managing-cairo-projects-with-packages-crates-and-modules/listing_06_06/src/lib.cairo}}
```

<span class="caption">Listing 7-6: Pernyataan `use` hanya berlaku dalam lingkup
yang ada di dalamnya</span>

Error kompilator menunjukkan bahwa pintasan tidak lagi berlaku di dalam modul `customer`:

```shell
❯ scarb build
error: Identifier not found.
 --> lib.cairo:11:9
        hosting::add_to_waitlist();
        ^*****^
```

## Membuat Jalur `use` yang Idiomatik

Pada Listing 7-5, Anda mungkin bertanya-tanya mengapa kita menentukan `use
restaurant::front_of_house::hosting` dan kemudian memanggil `hosting::add_to_waitlist` di
`eat_at_restaurant` daripada menentukan jalur `use` sampai ke
fungsi `add_to_waitlist` untuk mencapai hasil yang sama, seperti pada Listing 7-7.

<span class="filename">Nama File: src/lib.cairo</span>

```rust
{{#include ../listings/ch07-managing-cairo-projects-with-packages-crates-and-modules/listing_06_07/src/lib.cairo}}
```

<span class="caption">Listing 7-7: Membawa fungsi `add_to_waitlist` ke dalam lingkup dengan `use`, yang tidak idiomatik</span>

Meskipun keduanya, Listing 7-5 dan 6-7, mencapai tugas yang sama, Listing 7-5 adalah cara yang idiomatik untuk membawa fungsi ke dalam lingkup dengan `use`. Membawa modul induk fungsi ke dalam lingkup dengan `use` berarti kita harus menyebutkan modul induk saat memanggil fungsi. Menyebutkan modul induk saat memanggil fungsi membuat jelas bahwa fungsi tersebut tidak didefinisikan secara lokal sambil tetap meminimalkan pengulangan dari jalur lengkap. Kode pada Listing 7-7 tidak jelas di mana `add_to_waitlist` didefinisikan.

Di sisi lain, ketika membawa dalam struktur, enumerasi, trait, dan item lain dengan menggunakan `use`, cara idiomatik adalah dengan menentukan jalur lengkap. Listing 7-8 menunjukkan cara idiomatik membawa trait `ArrayTrait` dari pustaka inti ke dalam lingkup.

```rust
{{#include ../listings/ch07-managing-cairo-projects-with-packages-crates-and-modules/listing_06_08/src/lib.cairo}}
```

<span class="caption">Listing 7-8: Membawa `ArrayTrait` ke dalam lingkup dengan cara
idiomatik</span>

Tidak ada alasan kuat di balik idiom ini: ini hanya konvensi yang muncul dalam komunitas Rust, dan orang-orang telah terbiasa membaca dan menulis kode Rust dengan cara ini. Karena Cairo memiliki banyak idiom yang sama dengan Rust, kami mengikuti konvensi ini juga.

Pengecualian dari idiom ini adalah jika kita membawa dua item dengan nama yang sama ke dalam lingkup yang sama dengan pernyataan `use`, karena Cairo tidak mengizinkan hal tersebut.

### Memberikan Nama Baru dengan Kata Kunci `as`

Ada solusi lain untuk masalah membawa dua jenis dengan nama yang sama ke dalam lingkup yang sama dengan `use`: setelah jalur, kita dapat menentukan `as` dan nama lokal baru, atau _alias_, untuk jenis tersebut. Listing 7-9 menunjukkan cara Anda dapat mengganti nama impor dengan `as`:

<span class="filename">Nama File: src/lib.cairo</span>

```rust
{{#include ../listings/ch07-managing-cairo-projects-with-packages-crates-and-modules/listing_06_09/src/lib.cairo}}
```

<span class="caption">Listing 7-9: Mengganti nama trait ketika dibawa ke dalam
lingkup dengan kata kunci `as`</span>

Di sini, kami membawa `ArrayTrait` ke dalam lingkup dengan alias `Arr`. Sekarang kita dapat mengakses metode-metode trait dengan pengenal `Arr`.

### Mengimpor beberapa item dari modul yang sama

Ketika Anda ingin mengimpor beberapa item (seperti fungsi, struktur, atau enumerasi) dari modul yang sama di Cairo, Anda dapat menggunakan kurung kurawal `{}` untuk mencantumkan semua item yang ingin Anda impor. Ini membantu menjaga kode Anda bersih dan mudah dibaca dengan menghindari daftar panjang pernyataan `use` yang terpisah.

Syntax umum untuk mengimpor beberapa item dari modul yang sama adalah:

```rust
use module::{item1, item2, item3};
```

Berikut adalah contoh di mana kita mengimpor tiga struktur dari modul yang sama:

```rust
{{#include ../listings/ch07-managing-cairo-projects-with-packages-crates-and-modules/listing_06_10/src/lib.cairo}}
```

<span class="caption">Listing 7-10: Mengimpor beberapa item dari modul yang sama</span>

## Mengulang Nama dalam Berkas Modul

Ketika kita membawa suatu nama ke dalam lingkup dengan kata kunci `use`, nama yang tersedia dalam lingkup baru dapat diimpor seolah-olah telah didefinisikan dalam lingkup kode tersebut. Teknik ini disebut _re-exporting_ karena kita membawa suatu item ke dalam lingkup, tetapi juga membuat item tersebut tersedia bagi orang lain untuk membawanya ke dalam lingkup mereka.

Sebagai contoh, mari re-export fungsi `add_to_waitlist` pada contoh restoran:

<span class="filename">Nama File: src/lib.cairo</span>

```rust
{{#include ../listings/ch07-managing-cairo-projects-with-packages-crates-and-modules/listing_06_11/src/lib.cairo}}
```

<span class="caption">Listing 7-11: Membuat suatu nama tersedia untuk kode apa pun digunakan
dari lingkup baru dengan `pub use`</span>

Sebelum perubahan ini, kode eksternal harus memanggil fungsi `add_to_waitlist` dengan menggunakan jalur
`restaurant::front_of_house::hosting::add_to_waitlist()`. Sekarang bahwa `use`
ini telah mere-export modul `hosting` dari modul akar, kode eksternal
sekarang dapat menggunakan jalur `restaurant::hosting::add_to_waitlist()` sebagai gantinya.

Mere-export berguna ketika struktur internal kode Anda berbeda
dengan cara pemrogram yang memanggil kode Anda akan memikirkan domain tersebut. Sebagai
contoh, dalam metafora restoran ini, orang-orang yang menjalankan restoran berpikir
tentang "front of house" dan "back of house." Tetapi pelanggan yang mengunjungi restoran
mungkin tidak akan memikirkan bagian-bagian restoran dengan istilah tersebut. Dengan
`use`, kita dapat menulis kode kita dengan satu struktur tetapi mengekspos struktur
yang berbeda. Melakukan hal tersebut membuat perpustakaan kita terorganisir dengan baik untuk pemrogram yang bekerja pada
perpustakaan dan pemrogram yang memanggil perpustakaan tersebut.

## Menggunakan Paket Eksternal di Cairo dengan Scarb

Anda mungkin perlu menggunakan paket eksternal untuk memanfaatkan fungsionalitas yang disediakan oleh komunitas. Untuk menggunakan paket eksternal dalam proyek Anda dengan Scarb, ikuti langkah-langkah berikut:

> Sistem dependensi masih dalam tahap pengembangan. Anda dapat memeriksa [dokumentasi resmi](https://docs.swmansion.com/scarb/docs/guides/dependencies.html).
