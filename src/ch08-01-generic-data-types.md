# Jenis Data Generik

Kami menggunakan generik untuk membuat definisi deklarasi item, seperti structs dan functions, yang kemudian dapat kita gunakan dengan banyak jenis data konkret yang berbeda. Di Cairo, kita dapat menggunakan generik saat mendefinisikan functions, structs, enums, traits, implementasi, dan metode! Pada bab ini, kita akan melihat bagaimana cara menggunakan tipe generik secara efektif dengan semua hal tersebut.

## Fungsi Generik

Saat mendefinisikan fungsi yang menggunakan generik, kita menempatkan generik pada tanda tangan fungsi, di mana kita biasanya menentukan jenis data dari parameter dan nilai kembali. Sebagai contoh, bayangkan kita ingin membuat sebuah fungsi yang, dengan dua `Array` item, akan mengembalikan yang terbesar. Jika kita perlu melakukan operasi ini untuk daftar jenis yang berbeda, maka kita harus mendefinisikan kembali fungsi tersebut setiap kali. Untungnya, kita dapat mengimplementasikan fungsi ini sekali menggunakan generik dan melanjutkan ke tugas lain.

```rust
{{#include ../listings/ch08-generic-types-and-traits/no_listing_01_missing_tdrop/src/lib.cairo}}
```

Fungsi `largest_list` membandingkan dua daftar dengan tipe yang sama dan mengembalikan yang memiliki lebih banyak elemen serta menghapus yang lain. Jika Anda mengompilasi kode sebelumnya, Anda akan melihat bahwa itu akan gagal dengan kesalahan yang mengatakan bahwa tidak ada traits yang ditentukan untuk menghapus array dari jenis generik. Hal ini terjadi karena kompiler tidak memiliki cara untuk menjamin bahwa `Array<T>` dapat dihapus saat menjalankan fungsi `main`. Untuk menghapus array dari `T`, kompiler harus tahu cara menghapus `T`. Ini dapat diperbaiki dengan menentukan dalam tanda tangan fungsi `largest_list` bahwa `T` harus mengimplementasikan trait drop. Definisi fungsi `largest_list` yang benar adalah sebagai berikut:

```rust
{{#rustdoc_include ../listings/ch08-generic-types-and-traits/no_listing_02_with_tdrop/src/lib.cairo}}
```

Fungsi `largest_list` baru mencakup dalam definisinya persyaratan bahwa jenis generik apa pun yang ditempatkan di sana, itu harus dapat dihapus. Fungsi `main` tetap tidak berubah, kompiler cukup cerdas untuk menyimpulkan jenis konkret apa yang digunakan dan apakah itu mengimplementasikan trait `Drop`.

### Kendala untuk Jenis Generik

Saat mendefinisikan jenis generik, berguna untuk memiliki informasi tentang mereka. Mengetahui trait mana yang diimplementasikan oleh jenis generik memungkinkan kita menggunakan mereka secara lebih efektif dalam logika fungsi dengan biaya membatasi jenis generik yang dapat digunakan dengan fungsi tersebut. Kita melihat contoh ini sebelumnya dengan menambahkan implementasi `TDrop` sebagai bagian dari argumen generik dari `largest_list`. Sementara `TDrop` ditambahkan untuk memenuhi persyaratan kompiler, kita juga dapat menambahkan kendala untuk memperkaya logika fungsi kita.

Bayangkan kita ingin, diberikan sebuah daftar elemen dari suatu jenis generik `T`, mencari elemen terkecil di antara mereka. Awalnya, kita tahu bahwa untuk elemen jenis `T` dapat dibandingkan, itu harus mengimplementasikan trait `PartialOrd`. Fungsi yang dihasilkan akan menjadi:

```rust
{{#include ../listings/ch08-generic-types-and-traits/no_listing_03_missing_tcopy/src/lib.cairo}}
```

Fungsi `smallest_element` menggunakan jenis generik `T` yang mengimplementasikan trait `PartialOrd`, mengambil snapshot dari `Array<T>` sebagai parameter, dan mengembalikan salinan elemen terkecil. Karena parameter berjenis `@Array<T>`, kita tidak perlu lagi menghapusnya pada akhir eksekusi dan oleh karena itu tidak memerlukan implementasi trait `Drop` untuk `T` juga. Mengapa ini tidak dikompilasi?

Saat mengindeks pada `list`, nilai menghasilkan snap dari elemen yang diindeks, kecuali jika `PartialOrd` diimplementasikan untuk `@T`, kita perlu melepaskan elemen tersebut menggunakan `*`. Operasi `*` memerlukan salinan dari `@T` ke `T`, yang berarti bahwa `T` harus mengimplementasikan trait `Copy`. Setelah menyalin elemen bertipe `@T` menjadi `T`, sekarang ada variabel dengan tipe `T` yang perlu dihapus, memerlukan `T` untuk mengimplementasikan trait `Drop` juga. Kita harus menambahkan implementasi trait `Drop` dan `Copy` untuk fungsi menjadi benar. Setelah memperbarui fungsi `smallest_element`, kode yang dihasilkan akan menjadi:

```rust
{{#rustdoc_include ../listings/ch08-generic-types-and-traits/no_listing_04_with_tcopy/src/lib.cairo}}
```

### Parameter Implementasi Generik Anonim (`+` operator)

Sampai sekarang, kita selalu menentukan nama untuk setiap implementasi trait generik yang diperlukan: `TPartialOrd` untuk `PartialOrd<T>`, `TDrop` untuk `Drop<T>`, dan `TCopy` untuk `Copy<T>`.

Namun, sebagian besar waktu, kita tidak menggunakan implementasi dalam tubuh fungsi; kita hanya menggunakannya sebagai batasan. Dalam kasus-kasus ini, kita dapat menggunakan operator `+` untuk menentukan bahwa jenis generik harus mengimplementasikan suatu trait tanpa memberi nama implementasinya. Ini disebut sebagai *parameter implementasi generik anonim*.

Sebagai contoh, `+PartialOrd<T>` setara dengan `impl TPartialOrd: PartialOrd<T>`.

Kita dapat menulis ulang tanda tangan fungsi `smallest_element` sebagai berikut:

```rust
{{#rustdoc_include ../listings/ch08-generic-types-and-traits/no_listing_05_with_anonymous_impl/src/lib.cairo:1}}
```

## Structs

Kita juga dapat mendefinisikan structs untuk menggunakan parameter jenis generik untuk satu atau lebih field menggunakan sintaks `<>`, mirip dengan definisi fungsi. Pertama, kita mendeklarasikan nama parameter jenis di dalam tanda kurung sudut setelah nama struct. Kemudian kita menggunakan jenis generik dalam definisi struct di mana kita seharusnya menentukan jenis data konkret. Contoh kode berikut menunjukkan definisi `Wallet<T>` yang memiliki field `balance` bertipe `T`.

```rust
{{#include ../listings/ch08-generic-types-and-traits/no_listing_06_derive_generics/src/lib.cairo}}
```

Kode di atas menghasilkan trait `Drop` untuk tipe `Wallet` secara otomatis. Ini setara dengan menulis kode berikut:

```rust
{{#include ../listings/ch08-generic-types-and-traits/no_listing_07_drop_explicit/src/lib.cairo}}
```

Kita menghindari menggunakan macro `derive` untuk implementasi `Drop` dari `Wallet` dan sebaliknya menentukan implementasi `WalletDrop` kita sendiri. Perhatikan bahwa kita harus mendefinisikan, seperti fungsi, jenis generik tambahan untuk `WalletDrop` yang menyatakan bahwa `T` juga mengimplementasikan trait `Drop`. Kita basically mengatakan bahwa struct `Wallet<T>` dapat dihapus selama `T` juga dapat dihapus.

Terakhir, jika kita ingin menambahkan field ke `Wallet` yang mewakili Addressnya dan kita ingin field tersebut berbeda dari `T` tetapi juga generik, kita dapat dengan mudah menambahkan jenis generik lain di antara `<>`:

```rust
{{#include ../listings/ch08-generic-types-and-traits/no_listing_08_two_generics/src/lib.cairo}}
```

Kita menambahkan definisi jenis generik `U` ke dalam struct `Wallet` dan kemudian mengassign jenis ini ke member field `address` yang baru. Perhatikan bahwa atribut `derive` untuk trait `Drop` juga berfungsi untuk `U`. 

## Enums

Seperti yang kita lakukan dengan structs, kita dapat mendefinisikan enums untuk menyimpan jenis data generik dalam variasi mereka. Contohnya adalah enum `Option<T>` yang disediakan oleh perpustakaan inti Cairo:

```rust,noplayground
{{#include ../listings/ch08-generic-types-and-traits/no_listing_09_option/src/lib.cairo}}
```

Enum `Option<T>` adalah generik terhadap jenis `T` dan memiliki dua variasi: `Some`, yang menyimpan satu nilai bertipe `T`, dan `None` yang tidak menyimpan nilai apa pun. Dengan menggunakan enum `Option<T>`, kita dapat menyatakan konsep abstrak dari nilai opsional dan karena nilai tersebut memiliki jenis generik `T`, kita dapat menggunakan abstraksi ini dengan jenis apa pun.

Enums juga dapat menggunakan beberapa jenis generik, seperti definisi enum `Result<T, E>` yang disediakan oleh perpustakaan inti:

```rust,noplayground
{{#include ../listings/ch08-generic-types-and-traits/no_listing_10_result/src/lib.cairo}}
```

Enum `Result<T, E>` memiliki dua jenis generik, `T` dan `E`, dan dua variasi: `Ok` yang menyimpan nilai bertipe `T` dan `Err` yang menyimpan nilai bertipe `E`. Definisi ini memudahkan penggunaan enum `Result` di mana pun kita memiliki operasi yang mungkin berhasil (dengan mengembalikan nilai bertipe `T`) atau gagal (dengan mengembalikan nilai bertipe `E`).

## Metode Generik

Kita dapat mengimplementasikan metode pada structs dan enums, dan menggunakan jenis generik dalam definisinya juga. Dengan menggunakan definisi sebelumnya dari struct `Wallet<T>`, kita mendefinisikan metode `balance` untuknya:

```rust
{{#include ../listings/ch08-generic-types-and-traits/no_listing_11_generic_methods/src/lib.cairo}}
```

Pertama, kita mendefinisikan trait `WalletTrait<T>` menggunakan jenis generik `T` yang mendefinisikan metode yang mengembalikan snapshot dari field `balance` dari `Wallet`. Kemudian kita memberikan implementasi untuk trait tersebut pada `WalletImpl<T>`. Perhatikan bahwa Anda perlu menyertakan jenis generik di kedua definisi trait dan implementasi.

Kita juga dapat menentukan kendala pada jenis generik saat mendefinisikan metode pada jenis tersebut. Kita bisa, misalnya, mengimplementasikan metode hanya untuk instance `Wallet<u128>` daripada `Wallet<T>`. Pada contoh kode, kita mendefinisikan implementasi untuk Wallet yang memiliki tipe konkret `u128` untuk field `balance`.

```rust
{{#include ../listings/ch08-generic-types-and-traits/no_listing_12_constrained_generics/src/lib.cairo}}
```

Metode baru `receive` menambah ukuran saldo dari setiap instance `Wallet<u128>`. Perhatikan bahwa kita mengubah fungsi `main` membuat `w` menjadi variabel yang dapat diubah agar dapat memperbarui saldo. Jika kita mengubah inisialisasi `w` dengan mengubah tipe `balance`, kode sebelumnya tidak akan dikompilasi.

Cairo memungkinkan kita mendefinisikan metode generik di dalam trait generik juga. Menggunakan implementasi sebelumnya dari `Wallet<U, V>`, kita akan mendefinisikan sebuah trait yang memilih dua Wallet dari jenis generik yang berbeda dan membuat yang baru dengan jenis generik masing-masing. Pertama, mari kita menulis ulang definisi struct:

```rust,noplayground
{{#include ../listings/ch08-generic-types-and-traits/no_listing_13_not_compiling/src/lib.cairo:struct}}
```

Selanjutnya, kita akan secara naif mendefinisikan trait dan implementasinya:

```rust,noplayground
{{#include ../listings/ch08-generic-types-and-traits/no_listing_13_not_compiling/src/lib.cairo:trait_impl}}
```

Kita membuat trait `WalletMixTrait<T1, U1>` dengan metode `mixup<T2, U2>` yang diberikan sebuah instance `Wallet<T1, U1>` dan `Wallet<T2, U2>` membuat sebuah `Wallet<T1, U2>` baru. Seperti yang dijelaskan pada tanda tangan `mixup`, baik `self` maupun `other` akan di-drop pada akhir fungsi, itulah alasan mengapa kode ini tidak akan dikompilasi. Jika Anda telah mengikuti dari awal hingga sekarang, Anda akan tahu bahwa kita harus menambahkan persyaratan untuk semua jenis generik tersebut, menentukan bahwa mereka akan mengimplementasikan trait `Drop` agar kompiler tahu cara menjatuhkan instance `Wallet<T, U>`. Implementasi yang diperbarui adalah sebagai berikut:

```rust
{{#include ../listings/ch08-generic-types-and-traits/no_listing_14_compiling/src/lib.cairo:trait_impl}}
```

Kita menambahkan persyaratan untuk `T1` dan `U1` agar dapat di-drop pada deklarasi `WalletMixImpl`. Kemudian kita lakukan hal yang sama untuk `T2` dan `U2`, kali ini sebagai bagian dari tanda tangan `mixup`. Sekarang kita dapat mencoba fungsi `mixup`:

```rust,noplayground
{{#include ../listings/ch08-generic-types-and-traits/no_listing_14_compiling/src/lib.cairo:main}}
```

Pertama, kita membuat dua instance: satu dari `Wallet<bool, u128>` dan yang lainnya dari `Wallet<felt252, u8>`. Kemudian, kita memanggil `mixup` dan membuat instance baru `Wallet<bool, u8>`.
