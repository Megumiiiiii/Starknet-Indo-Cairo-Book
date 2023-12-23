# Hash

Pada intinya, penghashan adalah proses mengubah data masukan (sering disebut sebagai pesan) dengan panjang apa pun menjadi nilai berukuran tetap, biasanya disebut "hash." Transformasi ini bersifat deterministik, artinya masukan yang sama akan selalu menghasilkan nilai hash yang sama. Fungsi hash adalah komponen mendasar dalam berbagai bidang, termasuk penyimpanan data, kriptografi, dan verifikasi integritas data - dan sering digunakan dalam pengembangan kontrak pintar, terutama saat bekerja dengan pohon Merkle.

Dalam bab ini, kami akan mempresentasikan dua fungsi hash yang diimplementasikan secara asli dalam perpustakaan Cairo: `Poseidon` dan `Pedersen`. Kami akan membahas kapan dan bagaimana menggunakannya, serta melihat contoh dengan program cairo.

### Fungsi hash dalam Cairo

Perpustakaan inti Cairo menyediakan dua fungsi hash: Pedersen dan Poseidon.

Fungsi hash Pedersen adalah algoritma kriptografi yang bergantung pada kriptografi kurva eliptika. Fungsi-fungsi ini melakukan operasi pada titik-titik sepanjang kurva eliptika — pada dasarnya, melakukan perhitungan dengan lokasi-lokasi titik ini — yang mudah dilakukan ke satu arah dan sulit untuk dibatalkan. Kesulitan satu arah ini didasarkan pada Masalah Logaritma Diskrit Kurva Eliptika (ECDLP), yang merupakan masalah yang sangat sulit untuk dipecahkan dan memastikan keamanan fungsi hash. Kesulitan membalikkan operasi-operasi ini adalah yang membuat fungsi hash Pedersen aman dan dapat diandalkan untuk tujuan kriptografi.

Poseidon adalah keluarga fungsi hash yang dirancang untuk menjadi sangat efisien sebagai sirkuit aljabar. Desainnya sangat efisien untuk sistem bukti Tanpa Pengetahuan, termasuk STARKs (sehingga, Cairo). Poseidon menggunakan metode yang disebut 'konstruksi spons,' yang menyerap data dan mengubahnya dengan aman menggunakan proses yang dikenal sebagai permutasi Hades. Versi Cairo dari Poseidon didasarkan pada permutasi state dengan tiga elemen dengan [parameter tertentu](https://github.com/starkware-industries/poseidon/blob/main/poseidon3.txt).

#### Kapan Menggunakan Mereka?

Pedersen adalah fungsi hash pertama yang digunakan di Starknet, dan masih digunakan untuk menghitung Address variabel dalam penyimpanan (misalnya, `LegacyMap` menggunakan Pedersen untuk menghash kunci dari pemetaan penyimpanan di Starknet). Namun, karena Poseidon lebih murah dan lebih cepat daripada Pedersen saat bekerja dengan sistem bukti STARK, sekarang ini adalah fungsi hash yang direkomendasikan untuk digunakan dalam program Cairo.

### Bekerja dengan Hash

Perpustakaan inti memudahkan penggunaan hash. Trait `Hash` diimplementasikan untuk semua jenis yang dapat dikonversi menjadi `felt252`, termasuk `felt252` itu sendiri. Untuk jenis yang lebih kompleks seperti struct, menurunkan `Hash` memungkinkan mereka di-hash dengan mudah menggunakan fungsi hash pilihan Anda - asalkan semua bidang struct itu sendiri dapat di-hash. Anda tidak dapat menurunkan trait `Hash` pada struct yang berisi nilai yang tidak dapat di-hash, seperti `Array<T>` atau `Felt252Dict<T>`, bahkan jika `T` itu sendiri dapat di-hash.

Trait `Hash` disertai dengan `HashStateTrait` yang mendefinisikan metode dasar untuk bekerja dengan hash. Mereka memungkinkan Anda untuk menginisialisasi status hash yang akan berisi nilai-nilai sementara hash setelah setiap aplikasi fungsi hash; memperbarui status hash, dan menyelesaikannya ketika perhitungan selesai. `HashStateTrait` didefinisikan sebagai berikut:

```rust
{{#include ../listings/ch11-advanced-features/no_listing_03_hash_trait/src/lib.cairo:hashtrait}}
```

Untuk menggunakan hash dalam kode Anda, Anda harus mengimpor trait dan fungsi yang relevan. Pada contoh berikut, kami akan menunjukkan bagaimana menghash sebuah struct menggunakan fungsi hash Pedersen dan Poseidon.

Langkah pertama adalah menginisialisasi hash dengan `PoseidonTrait::new() -> HashState` atau `PedersenTrait::new(base: felt252) -> HashState` tergantung pada fungsi hash yang ingin kita gunakan. Kemudian status hash dapat diperbarui dengan fungsi `update(self: HashState, value: felt252) -> HashState` atau `update_with(self: S, value: T) -> S` sebanyak yang diperlukan. Kemudian fungsi `finalize(self: HashState) -> felt252` dipanggil pada status hash dan mengembalikan nilai hash sebagai `felt252`.

```rust
{{#include ../listings/ch11-advanced-features/no_listing_04_hash_pedersen/src/lib.cairo:import}}
```

```rust
{{#include ../listings/ch11-advanced-features/no_listing_04_hash_pedersen/src/lib.cairo:structure}}
```

Karena struct kami mendapatkan trait HashTrait, kita dapat memanggil fungsi sebagai berikut untuk hashing Poseidon:

```rust
{{#rustdoc_include ../listings/ch11-advanced-features/no_listing_04_hash_poseidon/src/lib.cairo:main}}
```

Dan sebagai berikut untuk hashing Pedersen:

```rust
{{#rustdoc_include ../listings/ch11-advanced-features/no_listing_04_hash_pedersen/src/lib.cairo:main}}
```

### Hashing Lanjutan: Menghash Array dengan Poseidon

Mari kita lihat contoh penghashan fungsi yang berisi `Span<felt252>`. Untuk menghash `Span<felt252>` atau sebuah struct yang berisi `Span<felt252>`, Anda dapat menggunakan fungsi bawaan dalam Poseidon `poseidon_hash_span(mut span: Span<felt252>) -> felt252`. Demikian pula, Anda dapat menghash `Array<felt252>` dengan memanggil `poseidon_hash_span` pada span-nya.

Pertama, mari kita impor trait dan fungsi berikut:

```rust
{{#include ../listings/ch11-advanced-features/no_listing_05_advanced_hash/src/lib.cairo:import}}
```

Sekarang kita tentukan strukturnya, seperti yang mungkin Anda perhatikan, kita tidak mendapatkan trait Hash. Jika Anda mencoba mendapatkan trait Hash pada struktur ini, itu akan menyebabkan error karena struktur tersebut berisi suatu bidang yang tidak dapat di-hash.

```rust, noplayground
{{#include ../listings/ch11-advanced-features/no_listing_05_advanced_hash/src/lib.cairo:structure}}
```

Dalam contoh ini, kita menginisialisasi HashState (`hash`) dan memperbarui statusnya, kemudian memanggil fungsi `finalize()` pada HashState untuk mendapatkan hash yang dihitung `hash_felt252`. Kami menggunakan `poseidon_hash_span` pada `Span` dari `Array<felt252>` untuk menghitung hash-nya.

```rust
{{#rustdoc_include ../listings/ch11-advanced-features/no_listing_05_advanced_hash/src/lib.cairo:main}}
```