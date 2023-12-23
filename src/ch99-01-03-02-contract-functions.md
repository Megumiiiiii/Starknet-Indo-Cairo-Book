## Fungsi Kontrak

Pada bagian ini, kita akan melihat berbagai jenis fungsi yang dapat Anda temui dalam kontrak:

### 1. Konstruktor

Konstruktor adalah jenis fungsi khusus yang hanya dijalankan sekali saat mendeploy kontrak, dan dapat digunakan untuk menginisialisasi status sebuah kontrak.

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_03_example_contract/src/lib.cairo:constructor}}
```

Beberapa aturan penting yang perlu diperhatikan:

1. Kontrak Anda tidak boleh memiliki lebih dari satu konstruktor.
2. Fungsi konstruktor Anda harus diberi nama `constructor`.
3. Harus diberi anotasi dengan atribut `#[constructor]`.

### 2. Fungsi Publik

Seperti yang disebutkan sebelumnya, fungsi publik dapat diakses dari luar kontrak. Mereka harus didefinisikan di dalam blok implementasi yang dianotasi dengan atribut `#[external(v0)]`. Atribut ini hanya memengaruhi keterlihatan (publik vs privat/internal), namun tidak memberi informasi kepada kita tentang kemampuan fungsi-fungsi ini untuk memodifikasi status kontrak.

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_03_example_contract/src/lib.cairo:impl_public}}
```

### Fungsi Eksternal

Fungsi eksternal adalah fungsi yang dapat memodifikasi status sebuah kontrak. Mereka bersifat publik dan dapat dipanggil oleh kontrak lain atau dari luar.
Fungsi eksternal adalah fungsi _publik_ di mana `self: ContractState` dilewatkan sebagai referensi dengan kata kunci `ref`, memungkinkan Anda untuk memodifikasi status kontrak.

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_03_example_contract/src/lib.cairo:external}}
```

### Fungsi View

Fungsi view adalah fungsi hanya baca yang memungkinkan Anda mengakses data dari kontrak sambil memastikan bahwa status kontrak tidak dimodifikasi. Mereka dapat dipanggil oleh kontrak lain atau dari luar.
Fungsi view adalah fungsi _publik_ di mana `self: ContractState` dilewatkan sebagai snapshot, mencegah Anda untuk memodifikasi status kontrak.

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_03_example_contract/src/lib.cairo:view}}
```

> **Catatan:** Penting untuk dicatat bahwa baik fungsi eksternal maupun fungsi view bersifat publik. Untuk membuat fungsi internal dalam sebuah kontrak, Anda perlu mendefinisikannya di luar blok implementasi yang dianotasi dengan atribut `#[external(v0)]`.

### 3. Fungsi Privat

Fungsi yang tidak didefinisikan dalam blok yang dianotasi dengan atribut `#[external(v0)]` adalah fungsi privat (juga disebut fungsi internal). Mereka hanya dapat dipanggil dari dalam kontrak.

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_03_example_contract/src/lib.cairo:state_internal}}
```

> Tunggu, apa itu atribut `#[generate_trait]`? Di mana definisi trait untuk blok implementasi ini? Nah, atribut `#[generate_trait]` adalah atribut khusus yang memberi tahu kompiler untuk menghasilkan definisi trait untuk blok implementasi tersebut. Ini memungkinkan Anda untuk menghilangkan kode boilerplate dari definisi trait dan implementasinya untuk blok implementasi. Kita akan melihat lebih lanjut tentang ini di [bagian selanjutnya](./ch99-01-03-04-reducing-boilerplate.md).

Pada titik ini, Anda mungkin masih bertanya-tanya apakah semua ini benar-benar diperlukan jika Anda tidak perlu mengakses status kontrak dalam fungsi Anda (misalnya, fungsi bantuan/pustaka). Sebenarnya, Anda juga dapat mendefinisikan fungsi internal di luar blok implementasi. Satu-satunya alasan mengapa kita _perlu_ mendefinisikan fungsi di dalam blok impl adalah jika kita ingin mengakses status kontrak.

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_03_example_contract/src/lib.cairo:stateless_internal}}
```
