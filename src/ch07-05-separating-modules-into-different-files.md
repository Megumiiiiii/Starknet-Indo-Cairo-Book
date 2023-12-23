## Memisahkan Modul ke dalam Berkas yang Berbeda

Sejauh ini, semua contoh dalam bab ini mendefinisikan beberapa modul dalam satu file. Ketika modul menjadi besar, Anda mungkin ingin memindahkan definisinya ke file terpisah untuk membuat kode lebih mudah dinavigasi.

Sebagai contoh, mari mulai dari kode pada Listing 7-11 yang memiliki beberapa modul restoran. Kita akan mengekstrak modul ke dalam file daripada mendefinisikan semua modul di dalam file akar kerangka. Dalam hal ini, file akar kerangka adalah _src/lib.cairo_.

Pertama, kita akan mengekstrak modul `front_of_house` ke dalam file tersendiri. Hapus kode di dalam kurung kurawal untuk modul `front_of_house`, hanya meninggalkan deklarasi `mod front_of_house;`, sehingga _src/lib.cairo_ berisi kode seperti yang terlihat pada Listing 7-12. Perlu diingat bahwa ini tidak akan dikompilasi hingga kita membuat file _src/front_of_house.cairo_ seperti yang terlihat pada Listing 7-13.

<span class="filename">Nama File: src/lib.cairo</span>

```rust,noplayground
{{#include ../listings/ch07-managing-cairo-projects-with-packages-crates-and-modules/listing_06_12/src/lib.cairo}}
```

<span class="caption">Listing 7-12: Mendeklarasikan modul `front_of_house` yang
tubuhnya akan berada di _src/front_of_house.cairo_</span>

Selanjutnya, letakkan kode yang berada di dalam kurung kurawal ke dalam file baru bernama _src/front_of_house.cairo_, seperti yang ditunjukkan pada Listing 7-13. Kompiler tahu untuk mencari di file ini karena menemui deklarasi modul di akar kerangka dengan nama `front_of_house`.

<span class="filename">Nama File: src/front_of_house.cairo</span>

```rust,noplayground
{{#include ../listings/ch07-managing-cairo-projects-with-packages-crates-and-modules/listing_06_13/src/lib.cairo}}
```

<span class="caption">Listing 7-13: Definisi di dalam modul `front_of_house`
di _src/front_of_house.cairo_</span>

Perhatikan bahwa Anda hanya perlu memuat suatu file menggunakan deklarasi `mod` _sekali_ dalam pohon modul Anda. Setelah kompiler tahu bahwa file itu bagian dari proyek (dan tahu di mana kode itu berada dalam pohon modul karena di mana Anda menempatkan pernyataan `mod`), file lain dalam proyek Anda harus merujuk ke kode file yang dimuat menggunakan jalur ke tempat itu dideklarasikan, seperti yang dijelaskan dalam [“Paths for Referring
to an Item in the Module Tree”][paths]<!-- ignore -->. Dengan kata lain, `mod` _bukan_ operasi "include" yang mungkin telah Anda lihat dalam bahasa pemrograman lain.

Selanjutnya, kita akan mengekstrak modul `hosting` ke dalam file terpisah. Proses ini sedikit berbeda karena `hosting` adalah modul anak dari `front_of_house`, bukan dari modul akar. Kita akan meletakkan file untuk `hosting` dalam direktori baru yang akan dinamai sesuai dengan leluhurnya dalam pohon modul, dalam hal ini _src/front_of_house/_.

Untuk memulai memindahkan `hosting`, kita ubah _src/front_of_house.cairo_ untuk hanya berisi deklarasi modul `hosting`:

<span class="filename">Nama File: src/front_of_house.cairo</span>

```rust,noplayground
mod hosting;
```

Kemudian, kita membuat direktori _src/front_of_house_ dan sebuah file _hosting.cairo_ untuk berisi definisi yang dibuat di dalam modul `hosting`:

<span class="filename">Nama File: src/front_of_house/hosting.cairo</span>

```rust,noplayground
fn add_to_waitlist() {}
```

Jika kita malah meletakkan _hosting.cairo_ di dalam direktori _src_, kompiler akan mengharapkan kode _hosting.cairo_ berada dalam modul `hosting` yang dideklarasikan di akar kerangka, dan tidak dideklarasikan sebagai anak dari modul `front_of_house`. Aturan kompiler untuk file mana yang akan diperiksa untuk kode modul mana berarti direktori dan file lebih mendekati pohon modul.

Kita telah memindahkan kode setiap modul ke dalam file terpisah, dan pohon modul tetap sama. Panggilan fungsi di dalam `eat_at_restaurant` akan berfungsi tanpa modifikasi apa pun, meskipun definisinya berada di file yang berbeda. Teknik ini memungkinkan Anda memindahkan modul ke file baru seiring pertumbuhannya.

Perlu diingat bahwa pernyataan `use restaurant::front_of_house::hosting` di
_src/lib.cairo_ juga tidak berubah, dan `use` tidak memiliki dampak pada file mana
yang dikompilasi sebagai bagian dari crate. Kata kunci `mod` mendeklarasikan modul, dan Cairo mencari di file dengan nama yang sama dengan modul untuk kode yang masuk ke dalam modul tersebut.

## Ringkasan

Cairo memungkinkan Anda membagi paket menjadi beberapa crate dan crate menjadi modul
sehingga Anda dapat merujuk pada item yang didefinisikan di satu modul dari modul lainnya. Anda dapat
melakukan ini dengan menentukan jalur absolut atau relatif. Jalur-jalur ini dapat dibawa
ke dalam lingkup dengan pernyataan `use` sehingga Anda dapat menggunakan jalur yang lebih pendek untuk penggunaan
item tersebut dalam lingkup tersebut. Kode modul secara default bersifat publik.

[paths]: ch06-03-paths-for-referring-to-an-item-in-the-module-tree.html
