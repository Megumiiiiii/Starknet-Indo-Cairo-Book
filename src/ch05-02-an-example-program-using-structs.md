# Contoh Program Menggunakan Structs

Untuk memahami kapan kita sebaiknya menggunakan structs, mari kita tulis sebuah program yang menghitung luas dari sebuah persegi panjang. Kita akan mulai dengan menggunakan variabel tunggal, dan kemudian merefactor program tersebut hingga kita menggunakan structs.

Mari buat sebuah proyek baru dengan Scarb yang disebut _rectangles_ yang akan mengambil lebar dan tinggi dari sebuah persegi panjang yang ditentukan dalam piksel dan menghitung luas dari persegi panjang tersebut. Listing 5-6 menunjukkan sebuah program pendek dengan salah satu cara untuk melakukan hal tersebut dalam proyek kita pada _src/lib.cairo_.

<span class="filename">Nama File: src/lib.cairo</span>

```rust
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing_04_06_no_struct/src/lib.cairo:all}}
```

<span class="caption">Listing 5-6: Menghitung luas dari sebuah persegi panjang yang ditentukan oleh variabel lebar dan tinggi terpisah</span>

Sekarang jalankan program dengan `scarb cairo-run --available-gas=200000000`:

```bash
$ scarb cairo-run --available-gas=200000000
[DEBUG] ,                               (raw: 300)

Run completed successfully, returning []
```

Kode ini berhasil menghitung luas dari persegi panjang dengan memanggil fungsi `area` dengan setiap dimensi, tetapi kita dapat melakukan lebih banyak lagi untuk membuat kode ini lebih jelas dan mudah dibaca.

Permasalahan dengan kode ini terlihat pada tanda tangan dari `area`:

```rust,noplayground
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing_04_06_no_struct/src/lib.cairo:here}}
```

Fungsi `area` seharusnya menghitung luas dari satu persegi panjang, namun fungsi yang kita tulis memiliki dua parameter, dan tidak jelas di mana pun dalam program kita bahwa parameter tersebut saling terkait. Akan lebih mudah dibaca dan dikelola jika kita mengelompokkan lebar dan tinggi bersama-sama. Kami telah membahas satu cara yang mungkin kita lakukan dalam [Bab 2](ch02-02-data-types.html#the-tuple-type): menggunakan tuples.

## Merefactor dengan Tuples

Listing 5-7 menunjukkan versi lain dari program kita yang menggunakan tuples.

<span class="filename">Nama File: src/lib.cairo</span>

```rust
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing_04_07_w_tuples/src/lib.cairo}}
```

<span class="caption">Listing 5-7: Menentukan lebar dan tinggi dari persegi panjang dengan sebuah tuple</span>

Dalam satu aspek, program ini lebih baik. Tuples memungkinkan kita untuk menambah sedikit struktur, dan sekarang kita hanya melewatkan satu argumen. Namun dalam aspek lain, versi ini kurang jelas: tuples tidak memberi nama pada elemennya, sehingga kita harus mengindeks ke bagian-bagian dari tuple, membuat perhitungan kita kurang jelas.

Membingungkan lebar dan tinggi tidak akan masalah untuk perhitungan luas, tetapi jika kita ingin menghitung perbedaannya, itu akan masalah! Kita harus ingat bahwa `lebar` adalah indeks tuple `0` dan `tinggi` adalah indeks tuple `1`. Ini akan menjadi lebih sulit bagi orang lain untuk memahami dan diingat jika mereka menggunakan kode kita. Karena kita tidak menyampaikan makna dari data kita dalam kode kita, sekarang lebih mudah untuk memperkenalkan kesalahan.

## Merefactor dengan Structs: Menambah Makna Lebih Banyak

Kita menggunakan structs untuk menambah makna dengan memberi label pada data tersebut. Kita dapat mengubah tuple yang kita gunakan menjadi sebuah struct dengan nama untuk keseluruhannya serta nama untuk bagian-bagiannya.

<span class="filename">Nama File: src/lib.cairo</span>

```rust
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing_04_08_w_structs/src/lib.cairo}}
```

<span class="caption">Listing 5-8: Mendefinisikan sebuah struct `Rectangle`</span>

Di sini kita telah mendefinisikan sebuah struct dan memberinya nama `Rectangle`. Di dalam kurung kurawal, kita mendefinisikan fields sebagai `lebar` dan `tinggi`, kedua-duanya memiliki tipe `u64`. Kemudian, di `main`, kita membuat sebuah instance tertentu dari `Rectangle` yang memiliki lebar `30` dan tinggi `10`. Fungsi `area` kita sekarang didefinisikan dengan satu parameter, yang kita namakan `persegiPanjang` yang merupakan tipe struct `Rectangle`. Kita dapat mengakses fields dari instance dengan notasi titik, dan ini memberikan nama yang deskriptif pada nilai-nilai tersebut daripada menggunakan nilai indeks tuple `0` dan `1`.

## Menambah Fungsionalitas Berguna dengan Trait

Akan berguna untuk dapat mencetak sebuah instance dari `Rectangle` saat kita sedang melakukan debugging pada program kita dan melihat nilai-nilai untuk semua fieldsnya. Listing 5-9 mencoba menggunakan `print` seperti yang telah kita gunakan pada bab-bab sebelumnya. Ini tidak akan berhasil.

<span class="filename">Nama File: src/lib.cairo</span>

<!-- TODO implement debug instead -->

```rust
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing_04_10_print_rectangle/src/lib.cairo:here}}
```

<span class="caption">Listing 5-9: Mencoba untuk mencetak sebuah instance `Rectangle`</span>

Ketika kita mengompilasi kode ini, kita mendapatkan sebuah error dengan pesan ini:

```text
$ cairo-compile src/lib.cairo
error: Method `print` not found on type "../src::Rectangle". Did you import the correct trait and impl?
 --> lib.cairo:16:15
    rectangle.print();
              ^***^

Error: Compilation failed.
```

Trait `print` diimplementasikan untuk banyak tipe data, tetapi tidak untuk struct `Rectangle`. Kita dapat memperbaikinya dengan mengimplementasikan trait `PrintTrait` pada `Rectangle` seperti yang ditunjukkan pada Listing 5-10.
Untuk mempelajari lebih lanjut tentang traits, lihat [Traits in Cairo](ch08-02-traits-in-cairo.md).

<span class="filename">Nama File: src/lib.cairo</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing_04_10_print_rectangle/src/lib.cairo:all}}
```

<span class="caption">Listing 5-10: Mengimplementasikan trait `PrintTrait` pada `Rectangle`</span>

Bagus! Ini bukanlah output yang paling cantik, tetapi ini menunjukkan nilai-nilai dari semua fields untuk instance ini, yang pasti akan membantu saat debugging.