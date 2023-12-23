# Kontrak Sederhana

Bab ini akan memperkenalkan dasar-dasar kontrak Starknet dengan contoh kontrak dasar. Anda akan belajar bagaimana menulis kontrak sederhana yang menyimpan sebuah angka tunggal di blockchain.

## Anatomi dari Kontrak Starknet Sederhana

Mari pertimbangkan kontrak berikut untuk menjelaskan dasar-dasar kontrak Starknet. Mungkin tidak mudah untuk memahaminya sekaligus, tetapi kita akan menjelaskannya langkah demi langkah:

```rust
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_01/src/lib.cairo:all}}
```

<span class="caption">Listing 99-1: Kontrak penyimpanan sederhana</span>

> Catatan: Kontrak Starknet didefinisikan dalam [modul](./ch07-02-defining-modules-to-control-scope.md).

### Apa maksudnya kontrak ini?

Pada contoh ini, `Storage` struct mendeklarasikan variabel penyimpanan yang disebut `stored_data` dengan tipe `u128` (bilangan bulat tak bertanda 128 bit).
Anda dapat memandangnya sebagai sebuah slot tunggal dalam sebuah database yang dapat Anda panggil dan ubah dengan memanggil fungsi dari kode yang mengelola database tersebut.
Kontrak ini mendefinisikan dan mengekspos secara publik fungsi-fungsi `set` dan `get` yang dapat digunakan untuk memodifikasi atau mengambil nilai dari variabel tersebut.

### Antarmuka: desain kontrak

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_01/src/lib.cairo:interface}}
```

Antarmuka kontrak mewakili fungsi-fungsi yang kontrak ini tawarkan kepada dunia luar. Di sini, antarmuka mengekspos dua fungsi: `set` dan `get`. Dengan memanfaatkan mekanisme [traits & impls](./ch08-02-traits-in-cairo.md) dari Cairo, kita dapat memastikan bahwa implementasi sebenarnya dari kontrak sesuai dengan antarmukanya. Bahkan, Anda akan mendapatkan kesalahan kompilasi jika kontrak Anda tidak sesuai dengan antarmuka yang dinyatakan.

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_01_bis_wrong_impl/src/lib.cairo:impl}}
```

<span class="caption">Listing 99-1-bis: Implementasi yang salah dari antarmuka kontrak. Ini tidak dapat dikompilasi.</span>

Pada antarmuka, perhatikan tipe generik `TContractState` dari argumen `self` yang dilewatkan secara referensi ke fungsi `set`. Parameter `self` mewakili status kontrak. Melihat argumen `self` yang dilewatkan ke `set` memberi tahu kita bahwa fungsi ini mungkin mengakses status kontrak, karena itulah yang memberi kita akses ke penyimpanan kontrak. Modifikator `ref` mengimplikasikan bahwa `self` dapat dimodifikasi, yang berarti variabel penyimpanan kontrak dapat dimodifikasi di dalam fungsi `set`.

Di sisi lain, `get` mengambil _snapshot_ dari `TContractState`, yang segera memberi tahu kita bahwa ini tidak mengubah status (dan memang, kompiler akan mengeluh jika kita mencoba mengubah penyimpanan di dalam fungsi `get`).

### Fungsi publik didefinisikan dalam blok implementasi

Sebelum kita menjelajahi lebih jauh, mari tentukan beberapa terminologi.

- Dalam konteks Starknet, _fungsi publik_ adalah fungsi yang diakses oleh dunia luar. Dalam contoh di atas, `set` dan `get` adalah fungsi publik. Fungsi publik dapat dipanggil oleh siapa pun, dan dapat dipanggil dari luar kontrak atau dari dalam kontrak. Dalam contoh di atas, `set` dan `get` adalah fungsi publik.

- Apa yang kita sebut sebagai _fungsi eksternal_ adalah fungsi publik yang dipanggil melalui transaksi dan dapat mengubah status kontrak. `set` adalah fungsi eksternal.

- Fungsi _view_ adalah fungsi publik yang dapat dipanggil dari luar kontrak, tetapi tidak dapat mengubah status kontrak. `get` adalah fungsi view.

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_01/src/lib.cairo:impl}}
```

Karena antarmuka kontrak didefinisikan sebagai trait `ISimpleStorage`, untuk sesuai dengan antarmuka, fungsi eksternal kontrak
harus didefinisikan dalam implementasi dari trait ini — yang memungkinkan kita memastikan bahwa implementasi kontrak sesuai dengan antarmukanya.

Namun, hanya mendefinisikan fungsi-fungsi dalam implementasi belum cukup. Blok implementasi harus dianotasi dengan atribut `#[external(v0)]`. Atribut ini mengekspos fungsi-fungsi yang didefinisikan dalam implementasi ini ke dunia luar — lupakan untuk menambahkannya dan fungsi-fungsi Anda tidak dapat dipanggil dari luar. Semua fungsi yang didefinisikan dalam blok yang ditandai dengan `#[external(v0)]` secara konsekuensial adalah _fungsi publik_.

Saat menulis implementasi antarmuka, parameter generik yang sesuai dengan argumen `self` dalam trait harus `ContractState`. Tipe `ContractState` dihasilkan oleh kompiler, dan memberikan akses ke variabel penyimpanan yang didefinisikan dalam struktur `Storage`.
Selain itu, `ContractState` memberikan kita kemampuan untuk menghasilkan acara (events). Nama `ContractState` tidak mengherankan, karena ini adalah representasi dari status kontrak, yang merupakan apa yang kita pikirkan sebagai `self` dalam trait antarmuka kontrak.

### Memodifikasi Status Kontrak

Seperti yang dapat Anda perhatikan, semua fungsi yang perlu mengakses status kontrak didefinisikan dalam implementasi dari suatu trait yang memiliki parameter generik `TContractState`, dan mengambil parameter `self: ContractState`.
Ini memungkinkan kita secara eksplisit melewatkan parameter `self: ContractState` ke fungsi, memungkinkan akses ke variabel penyimpanan kontrak.
Untuk mengakses variabel penyimpanan dari kontrak saat ini, Anda menambahkan awalan `self` ke nama variabel penyimpanan, yang memungkinkan Anda menggunakan metode `read` dan `write` untuk membaca atau menulis nilai variabel penyimpanan.

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_01/src/lib.cairo:write_state}}
```

<span class="caption">Menggunakan `self` dan metode `write` untuk memodifikasi nilai variabel penyimpanan</span>

> Catatan: jika status kontrak dilewatkan sebagai snapshot daripada `ref`, upaya untuk memodifikasinya akan menghasilkan kesalahan kompilasi.

Kontrak ini belum melakukan banyak hal selain memungkinkan siapa pun menyimpan satu angka yang dapat diakses oleh siapa pun di dunia. Siapa pun bisa memanggil `set` lagi dengan nilai yang berbeda dan menimpa angka Anda, tetapi angka tersebut masih tersimpan dalam riwayat blockchain. Nanti, Anda akan melihat bagaimana Anda dapat memberlakukan pembatasan akses agar hanya Anda yang dapat mengubah angka tersebut.
