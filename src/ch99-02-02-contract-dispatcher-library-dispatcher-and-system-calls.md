# Interaksi dengan kontrak dan kelas lain menggunakan Dispatchers dan syscalls

Setiap kali sebuah antarmuka kontrak didefinisikan, dua dispatcher secara otomatis dibuat dan diekspor oleh kompiler. Mari pertimbangkan sebuah antarmuka yang kami sebut IERC20, ini adalah:

1. Contract Dispatcher `IERC20Dispatcher`
2. Library Dispatcher `IERC20LibraryDispatcher`

Kompiler juga menghasilkan sebuah trait `IERC20DispatcherTrait`, yang memungkinkan kita untuk memanggil fungsi-fungsi yang didefinisikan dalam antarmuka pada struktur dispatcher.

Pada bab ini, kita akan membahas apa itu dispatcher tersebut, bagaimana cara kerjanya, dan bagaimana menggunakannya.

Untuk memahami konsep-konsep dalam bab ini secara efektif, kita akan menggunakan antarmuka IERC20 dari bab sebelumnya (lihat Listing 99-4):

## Contract Dispatcher

Seperti yang telah disebutkan sebelumnya, trait yang dianotasi dengan atribut `#[starknet::interface]` secara otomatis menghasilkan sebuah dispatcher dan sebuah trait saat dikompilasi.
Antarmuka `IERC20` kita diperluas menjadi sesuatu seperti ini:

**Catatan:** Kode yang diperluas untuk antarmuka IERC20 kita jauh lebih panjang, tetapi untuk menjaga bab ini ringkas dan langsung pada intinya, kita fokus pada satu fungsi view `name`, dan satu fungsi eksternal `transfer`.

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_05_dispatcher_trait/src/lib.cairo}}
```

<span class="caption">Listing 99-5: Bentuk yang diperluas dari trait IERC20</span>

Seperti yang bisa Anda lihat, dispatcher "klasik" hanyalah sebuah struktur yang membungkus Address kontrak dan mengimplementasikan `DispatcherTrait` yang dihasilkan oleh kompiler, memungkinkan kita untuk memanggil fungsi-fungsi dari kontrak lain. Ini berarti kita dapat membuat sebuah struktur dengan Address kontrak yang ingin kita panggil, dan kemudian dengan mudah memanggil fungsi-fungsi yang didefinisikan dalam antarmuka pada struktur dispatcher seolah-olah mereka adalah metode dari tipe tersebut.

Perlu diperhatikan juga bahwa semua ini diabstraksikan di belakang layar berkat kekuatan dari plugin Cairo.

### Memanggil Kontrak menggunakan Contract Dispatcher

Berikut adalah contoh sebuah kontrak yang bernama `TokenWrapper` menggunakan dispatcher untuk memanggil fungsi yang didefinisikan pada token ERC-20. Memanggil `transfer_token` akan mengubah status kontrak yang di-deploy pada `contract_address`.

```rust,noplayground
{{#rustdoc_include ../listings/ch99-starknet-smart-contracts/listing_99_06_sample_contract/src/lib.cairo:here}}
```

<span class="caption">Listing 99-6: Sebuah kontrak contoh yang menggunakan Contract Dispatcher</span>

Seperti yang bisa Anda lihat, kita harus mengimpor terlebih dahulu `IERC20DispatcherTrait` dan `IERC20Dispatcher` yang dihasilkan oleh kompiler, yang memungkinkan kita untuk melakukan panggilan ke metode-metode yang diimplementasikan untuk struktur `IERC20Dispatcher` (`name`, `transfer`, dll.), dengan memberikan `contract_address` dari kontrak yang ingin kita panggil pada struktur `IERC20Dispatcher`.

## Library Dispatcher

Perbedaan kunci antara dispatcher kontrak dan dispatcher library terletak pada konteks eksekusi logika yang didefinisikan dalam kelas tersebut. Sementara dispatcher reguler digunakan untuk memanggil fungsi dari **kontrak** (dengan status terkait), dispatcher library digunakan untuk memanggil **kelas** (tanpa status).

Mari kita pertimbangkan dua kontrak, A dan B.

Ketika A menggunakan `IBDispatcher` untuk memanggil fungsi dari **kontrak** B, konteks eksekusi logika yang didefinisikan di B adalah milik B. Ini berarti nilai yang dikembalikan oleh `get_caller_address()` di B akan mengembalikan Address A, dan memperbarui variabel penyimpanan di B akan memperbarui penyimpanan B.

Ketika A menggunakan `IBLibraryDispatcher` untuk memanggil fungsi dari **kelas** B, konteks eksekusi logika yang didefinisikan di kelas B adalah milik A. Ini berarti nilai yang dikembalikan oleh variabel `get_caller_address()` di kelas B akan mengembalikan Address pemanggil A, dan memperbarui variabel penyimpanan di kelas B akan memperbarui penyimpanan A (ingatlah bahwa **kelas** B tidak memiliki status; tidak ada status yang dapat diperbarui!)

Bentuk yang diperluas dari struct dan trait yang dihasilkan oleh kompiler terlihat seperti ini:

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_07_library_dispatcher/src/lib.cairo}}
```

Perhatikan bahwa perbedaan utama antara dispatcher kontrak reguler dan dispatcher library adalah bahwa yang pertama menggunakan `call_contract_syscall` sedangkan yang terakhir menggunakan `library_call_syscall`.

<span class="caption">Listing 99-7: Bentuk yang diperluas dari trait IERC20</span>

### Memanggil Kontrak menggunakan Library Dispatcher

Berikut adalah contoh kode untuk memanggil kontrak menggunakan Library Dispatcher.

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_08_using_library_dispatcher/src/lib.cairo:here}}
```

<span class="caption">Listing 99-8: Sebuah kontrak contoh yang menggunakan Library Dispatcher</span>

Seperti yang dapat Anda lihat, kita harus mengimpor terlebih dahulu `IContractBDispatcherTrait` dan `IContractBLibraryDispatcher` ke dalam kontrak kita yang dihasilkan dari antarmuka kami oleh kompiler. Kemudian, kita dapat membuat sebuah instance dari `IContractBLibraryDispatcher` dengan memberikan `class_hash` dari kelas yang ingin kita panggil dengan library. Dari sana, kita dapat memanggil fungsi-fungsi yang didefinisikan dalam kelas tersebut, menjalankan logiknya dalam konteks kontrak kita. Ketika kita memanggil `set_value` pada KontrakA, itu akan membuat panggilan library ke fungsi `set_value` di KontrakB, memperbarui nilai variabel penyimpanan `value` di KontrakA.

## Menggunakan syscalls tingkat rendah

Cara lain untuk memanggil kontrak dan kelas lain adalah dengan menggunakan `starknet::call_contract_syscall` dan `starknet::library_call_syscall`. Dispatcher yang kita jelaskan di bagian sebelumnya adalah sintaksis tingkat tinggi untuk sistem panggilan tingkat rendah ini.

Menggunakan syscalls ini bisa berguna untuk penanganan kesalahan yang disesuaikan atau untuk mendapatkan lebih banyak kontrol atas serialisasi/deserialisasi data panggilan dan data yang dikembalikan. Berikut adalah contoh yang menunjukkan bagaimana menggunakan `call_contract_syscall` untuk memanggil fungsi `transfer` dari sebuah kontrak ERC20:

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_09_call_contract_syscall/src/lib.cairo}}
```

<span class="caption">Listing 99-9: Sebuah kontrak contoh yang menggunakan syscalls</span>

Untuk menggunakan syscall ini, kita memberikan Address kontrak, selector dari fungsi yang ingin kita panggil, dan argumen panggilan.

Argumen panggilan harus disediakan sebagai sebuah array dari `felt252`. Untuk membangun array ini, kita melakukan serialisasi parameter fungsi yang diharapkan menjadi `Array<felt252>` menggunakan trait `Serde`, dan kemudian melewatkan array ini sebagai calldata. Pada akhirnya, kita akan mendapatkan kembali sebuah nilai yang telah diserialisasi yang perlu kita deserialisasi sendiri!
