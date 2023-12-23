# Mendefinisikan dan Membuat Instance Struct

Struct mirip dengan tuple, yang dibahas dalam bagian [Tipe Data](ch02-02-data-types.md), karena keduanya menyimpan beberapa nilai yang berhubungan. Seperti tuple, bagian-bagian dari sebuah struct dapat memiliki tipe data yang berbeda. Berbeda dengan tuple, pada struct, Anda akan memberikan nama pada setiap bagian data sehingga jelas apa makna dari nilai-nilai tersebut. Menambahkan nama-nama ini berarti bahwa struct lebih fleksibel daripada tuple: Anda tidak harus bergantung pada urutan data untuk menentukan atau mengakses nilai dari sebuah instance.

Untuk mendefinisikan sebuah struct, kita memasukkan kata kunci `struct` dan memberi nama pada seluruh struct. Nama dari sebuah struct seharusnya menjelaskan pentingnya bagian-bagian data yang dikelompokkan bersama. Kemudian, di dalam kurung kurawal, kita mendefinisikan nama dan tipe data dari setiap bagian data, yang disebut sebagai fields. Sebagai contoh, Listing 5-1 menunjukkan sebuah struct yang menyimpan informasi tentang akun pengguna.

<span class="filename">Nama File: src/lib.cairo</span>

```rust, noplayground
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing_04_01_user_struct/src/lib.cairo:user}}
```

<span class="caption">Listing 5-1: Definisi struct `User`</span>

Untuk menggunakan sebuah struct setelah kita mendefinisikannya, kita membuat sebuah _instance_ dari struct tersebut dengan menentukan nilai konkret untuk setiap field.
Kita membuat sebuah instance dengan menyebutkan nama struct dan kemudian menambahkan kurung kurawal yang berisi pasangan _key: value_, di mana kunci adalah nama-nama dari fields dan nilai adalah data yang ingin kita simpan pada fields tersebut. Kita tidak harus menyebutkan fields dalam urutan yang sama dengan yang kita deklarasikan pada struct. Dengan kata lain, definisi struct adalah seperti template umum untuk tipe tersebut, dan instance mengisi template tersebut dengan data tertentu untuk membuat nilai dari tipe tersebut.

Sebagai contoh, kita dapat mendeklarasikan seorang pengguna tertentu seperti yang ditunjukkan pada Listing 5-2.

<span class="filename">Nama File: src/lib.cairo</span>

```rust
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing_04_01_user_struct/src/lib.cairo:all}}
```

<span class="caption">Listing 5-2: Membuat sebuah instance dari struct `User`</span>

Untuk mendapatkan nilai spesifik dari sebuah struct, kita menggunakan notasi titik (dot notation). Sebagai contoh, untuk mengakses alamat email pengguna ini, kita menggunakan `user1.email`. Jika instance bersifat mutable, kita dapat mengubah nilai dengan menggunakan notasi titik dan mengassign ke field tertentu. Listing 5-3 menunjukkan bagaimana cara mengubah nilai pada field `email` dari sebuah instance `User` yang mutable.

<span class="filename">Nama File: src/lib.cairo</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing_04_03_mut_struct/src/lib.cairo:main}}
```

<span class="caption">Listing 5-3: Mengubah nilai pada field email dari sebuah instance `User`</span>

Perhatikan bahwa seluruh instance harus bersifat mutable; Cairo tidak mengizinkan kita untuk menandai hanya beberapa field tertentu sebagai mutable.

Seperti halnya dengan ekspresi lainnya, kita dapat membuat sebuah instance baru dari struct sebagai ekspresi terakhir dalam tubuh fungsi untuk secara implisit mengembalikan instance baru tersebut.

Listing 5-4 menunjukkan sebuah fungsi `build_user` yang mengembalikan sebuah instance `User` dengan email dan username yang diberikan. Field `active` mendapatkan nilai `true`, dan field `sign_in_count` mendapatkan nilai `1`.

<span class="filename">Nama File: src/lib.cairo</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing_04_03_mut_struct/src/lib.cairo:build_user}}
```

<span class="caption">Listing 5-4: Sebuah fungsi `build_user` yang mengambil email dan username dan mengembalikan sebuah instance `User`</span>

Masuk akal untuk memberi nama parameter fungsi dengan nama yang sama dengan fields struct, namun harus mengulang nama-nama `email` dan `username` serta variabel adalah sedikit membosankan. Jika struct memiliki lebih banyak fields, mengulangi setiap nama akan menjadi lebih menjengkelkan. Untungnya, ada singkatan yang nyaman!

## Menggunakan Singkatan Inisialisasi Field (Field Init Shorthand)

Karena nama parameter dan nama field struct sama persis dalam Listing 5-4, kita dapat menggunakan sintaks singkatan inisialisasi field untuk menulis ulang `build_user` agar berperilaku sama namun tanpa pengulangan `username` dan `email`, seperti yang ditunjukkan pada Listing 5-5.

<span class="filename">Nama File: src/lib.cairo</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing_04_03_mut_struct/src/lib.cairo:build_user2}}
```

<span class="caption">Listing 5-5: Sebuah fungsi `build_user` yang menggunakan singkatan inisialisasi field karena parameter `username` dan `email` memiliki nama yang sama dengan fields struct</span>

Di sini, kita membuat sebuah instance baru dari struct `User`, yang memiliki field bernama `email`. Kita ingin mengatur nilai field `email` ke nilai pada parameter `email` dari fungsi `build_user`. Karena field `email` dan parameter `email` memiliki nama yang sama, kita hanya perlu menuliskan `email` daripada `email: email`.