# Mengurangi kode yang berulang

Pada bagian sebelumnya, kita melihat contoh blok implementasi dalam kontrak yang tidak memiliki trait yang sesuai.

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_03_example_contract/src/lib.cairo:state_internal}}
```

Ini bukan kali pertama kita menemui atribut ini, kita sudah membicarakannya dalam [Traits in Cairo](./ch08-02-traits-in-cairo.md). Pada bagian ini, kita akan melihat lebih dalam tentang atribut tersebut dan melihat bagaimana penggunaannya dalam kontrak-kontrak.

Ingatlah bahwa untuk mengakses ContractState dalam sebuah fungsi, fungsi tersebut harus didefinisikan dalam blok implementasi yang parameter generiknya adalah `ContractState`. Hal ini menyiratkan bahwa kita pertama-tama perlu mendefinisikan trait generik yang mengambil `TContractState`, dan kemudian mengimplementasikan trait ini untuk tipe `ContractState`.
Namun dengan menggunakan atribut `#[generate_trait]`, seluruh proses ini dapat dilewati dan kita bisa langsung mendefinisikan blok implementasi, tanpa parameter generik apapun, dan menggunakan `self: ContractState` dalam fungsi-fungsi kita.

Jika kita harus secara manual mendefinisikan trait untuk implementasi `InternalFunctions`, akan terlihat seperti ini:

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/no_listing_99_02_explicit_internal_fn/src/lib.cairo:state_internal}}
```
