# Struktur Data Kustom

Ketika Anda pertama kali mulai memprogram di Cairo, kemungkinan besar Anda ingin menggunakan array (`Array<T>`) untuk menyimpan kumpulan data. Namun, Anda akan segera menyadari bahwa array memiliki satu keterbatasan besar - data yang disimpan di dalamnya bersifat tidak dapat diubah. Begitu Anda menambahkan nilai ke dalam array, Anda tidak dapat memodifikasinya.

Hal ini dapat menjadi frustrasi ketika Anda ingin menggunakan struktur data yang dapat diubah. Misalnya, katakanlah Anda membuat sebuah game di mana para pemain memiliki level, dan mereka dapat naik level. Anda mungkin mencoba menyimpan level pemain dalam sebuah array:

```rust,noplayground
let mut level_pemain = Array::new();
level_pemain.append(5);
level_pemain.append(1);
level_pemain.append(10);
```

Namun, kemudian Anda menyadari bahwa Anda tidak dapat meningkatkan level di indeks tertentu setelah diatur. Jika seorang pemain mati, Anda tidak dapat menghapusnya dari array kecuali dia kebetulan berada di posisi pertama.

Untungnya, Cairo menyediakan tipe [kamus bawaan](./ch03-02-dictionaries.md) yang praktis disebut `Felt252Dict<T>` yang memungkinkan kita mensimulasikan perilaku struktur data yang dapat diubah. Mari pertama-tama jelajahi cara menggunakannya untuk membuat implementasi array dinamis.

Catatan: Beberapa konsep yang digunakan dalam bab ini disajikan di bagian-bagian berikutnya dari buku ini. Kami menyarankan Anda untuk membaca bab berikut terlebih dahulu:
[Strukt](ch05-00-using-structs-to-structure-related-data),
[Metode](./ch05-03-method-syntax.md), [Tipe Generik](./ch08-00-generic-types-and-traits.md),
[Traits](./ch08-02-traits-in-cairo.md)

## Mensimulasikan array dinamis dengan kamus

Pertama, mari pikirkan bagaimana kita ingin array dinamis kita bersikap. Operasi apa yang harus didukung?

Array dinamis kita harus:

- Memungkinkan kita untuk menambahkan item di akhir
- Memungkinkan kita mengakses item apa pun berdasarkan indeks
- Memungkinkan menetapkan nilai item di indeks tertentu
- Mengembalikan panjang saat ini

Kita dapat mendefinisikan antarmuka ini di Cairo seperti:

```rust
{{#include ../listings/ch03-common-collections/no_listing_13_cust_struct_vect/src/lib.cairo:trait}}
```

Ini memberikan blueprint untuk implementasi array dinamis kita. Kami menamainya Vec karena mirip dengan struktur data `Vec<T>` di Rust.

### Mengimplementasikan array dinamis di Cairo

Untuk menyimpan data kita, kita akan menggunakan `Felt252Dict<T>` yang memetakan nomor indeks (felts) ke nilai. Kita juga akan menyimpan bidang `len` terpisah untuk melacak panjang.

Inilah tampilan struct kita. Kami membungkus tipe `T` di dalam pointer `Nullable` untuk memungkinkan penggunaan setiap tipe `T` dalam struktur data kita, sebagaimana dijelaskan dalam bagian [Kamus](./ch03-02-dictionaries.md#dictionaries-of-types-not-supported-natively):

```rust
{{#include ../listings/ch03-common-collections/no_listing_13_cust_struct_vect/src/lib.cairo:struct}}
```

Hal utama yang membuat vektor ini dapat diubah adalah kita dapat menyisipkan nilai ke dalam kamus untuk menetapkan atau memperbarui nilai dalam struktur data kita. Misalnya, untuk memperbarui nilai di indeks tertentu, kita melakukan:

```rust,noplayground
{{#include ../listings/ch03-common-collections/no_listing_13_cust_struct_vect/src/lib.cairo:set}}
```

Ini mengganti nilai yang sudah ada sebelumnya di indeks tersebut dalam kamus.

Sementara array bersifat tidak dapat diubah, kamus memberikan fleksibilitas yang kita butuhkan untuk struktur data yang dapat dimodifikasi seperti vektor.

Implementasi sisa antarmuka ini mudah dimengerti. Implementasi semua metode yang didefinisikan dalam antarmuka kami dapat dilakukan sebagai berikut:

```rust
{{#include ../listings/ch03-common-collections/no_listing_13_cust_struct_vect/src/lib.cairo:implem}}
```

Implementasi lengkap struktur `Vec` dapat ditemukan dalam perpustakaan yang dijaga oleh komunitas [Alexandria](https://github.com/keep-starknet-strange/alexandria/tree/main/src/data_structures).

## Mensimulasikan Stack dengan kamus

Sekarang kita akan melihat contoh kedua dan detail implementasinya: sebuah Stack.

Stack adalah koleksi LIFO (Last-In, First-Out). Penyisipan elemen baru dan penghapusan elemen yang ada terjadi di ujung yang sama, yang diwakili sebagai puncak tumpukan.

Mari kita tentukan operasi apa yang kita perlukan untuk membuat stack:

- Dorong item ke bagian atas tumpukan
- Pop item dari bagian atas tumpukan
- Periksa apakah masih ada elemen di tumpukan.

Dari spesifikasi ini, kita dapat menentukan antarmuka berikut:

```rust
{{#include ../listings/ch03-common-collections/no_listing_14_cust_struct_stack/src/lib.cairo:trait}}
```

### Mengimplementasikan Stack yang Dapat Diubah di Cairo

Untuk membuat struktur data tumpukan di Cairo, kita bisa lagi menggunakan `Felt252Dict<T>` untuk menyimpan nilai-nilai tumpukan bersama dengan bidang `usize` untuk melacak panjang tumpukan agar bisa diiterasi.

Struktur Stack didefinisikan sebagai berikut:

```rust
{{#include ../listings/ch03-common-collections/no_listing_14_cust_struct_stack/src/lib.cairo:struct}}
```

Selanjutnya, mari lihat bagaimana fungsi utama kami `push` dan `pop` diimplementasikan.

```rust
{{#include ../listings/ch03-common-collections/no_listing_14_cust_struct_stack/src/lib.cairo:implem}}
```

Kode menggunakan metode `insert` dan `get` untuk mengakses nilai-nilai dalam `Felt252Dict<T>`. Untuk mendorong elemen di bagian atas tumpukan, fungsi `push` menyisipkan elemen ke dalam kamus di indeks `len` - dan meningkatkan bidang `len` dari tumpukan untuk melacak posisi puncak tumpukan. Untuk menghapus sebuah nilai, fungsi `pop` mengambil nilai terakhir di posisi `len-1` kemudian mengurangi nilai `len` untuk memperbarui posisi puncak tumpukan sesuai.

Implementasi lengkap dari Stack, bersama dengan struktur data lain yang dapat Anda gunakan dalam kode Anda, dapat ditemukan dalam perpustakaan yang dijaga oleh komunitas [Alexandria](https://github.com/keep-starknet-strange/alexandria/tree/main/src/data_structures), di dalam crate "data_structures".

## Ringkasan

Meskipun model memori Cairo bersifat tidak dapat diubah dan dapat membuat sulit untuk mengimplementasikan struktur data yang dapat diubah, kita untungnya dapat menggunakan tipe `Felt252Dict<T>` untuk mensimulasikan struktur data yang dapat diubah. Ini memungkinkan kita untuk mengimplementasikan berbagai struktur data yang berguna untuk banyak aplikasi, efektif menyembunyikan kompleksitas model memori yang mendasarinya.