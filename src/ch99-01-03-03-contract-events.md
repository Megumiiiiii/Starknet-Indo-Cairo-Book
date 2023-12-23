## Event

Event adalah struktur data kustom yang dipancarkan oleh kontrak pintar selama eksekusi. Mereka memberikan cara bagi kontrak pintar untuk berkomunikasi dengan dunia luar dengan mencatat informasi tentang kejadian-kejadian tertentu dalam sebuah kontrak.

Event memainkan peran penting dalam pembuatan kontrak pintar. Ambil contoh Token Non-Fungible (NFT) yang dibuat di Starknet. Semua ini diindeks dan disimpan dalam sebuah database, kemudian ditampilkan kepada pengguna melalui penggunaan Event-Event ini. Mengabaikan untuk menyertakan sebuah Event dalam kontrak NFT Anda dapat menyebabkan pengalaman pengguna yang buruk. Hal ini karena pengguna mungkin tidak melihat NFT mereka muncul di Wallet mereka (Wallet menggunakan indeks ini untuk menampilkan NFT pengguna).

### Mendefinisikan Event

Semua Event yang berbeda dalam kontrak didefinisikan di bawah `Event` enum, yang mengimplementasikan trait `starknet::Event`, sebagai variant enum. Trait ini didefinisikan dalam pustaka inti seperti berikut:

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/no_listing_event_trait/src/lib.cairo}}
```

Atribut `#[derive(starknet::Event)]` menyebabkan kompiler untuk menghasilkan implementasi untuk trait di atas, diinstansiasi dengan tipe Event kita, yang dalam contoh kami adalah enum berikut:

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_03_example_contract/src/lib.cairo:event}}
```

Setiap variant Event harus menjadi sebuah struct dengan nama yang sama dengan variant tersebut, dan setiap variant harus mengimplementasikan trait `starknet::Event` itu sendiri. Lebih lanjut, anggota-anggota dari variant-variant ini harus mengimplementasikan trait `Serde` (lihat [Lampiran C: Serializing with Serde](./appendix-03-derivable-traits.md)), karena kunci/data ditambahkan ke Event menggunakan proses serialisasi.

Implementasi otomatis dari trait `starknet::Event` akan mengimplementasikan fungsi `append_keys_and_data` untuk setiap variant dari enum `Event` kita. Implementasi yang dihasilkan akan menambahkan sebuah kunci tunggal berdasarkan nama variant (`StoredName`), dan kemudian secara rekursif memanggil `append_keys_and_data` dalam impl dari trait Event untuk tipe variant tersebut.

Dalam kontrak kami, kami mendefinisikan sebuah Event bernama `StoredName` yang memancarkan Address kontrak pemanggil dan nama yang disimpan dalam kontrak, di mana bidang `user` diserialisasikan sebagai kunci dan bidang `name` diserialisasikan sebagai data.
Untuk mengindeks kunci dari sebuah Event, cukup beri anotasi dengan `#[key]` seperti yang ditunjukkan dalam contoh untuk kunci `user`.

Saat memancarkan Event dengan `self.emit(StoredName { user: user, name: name })`, sebuah kunci yang sesuai dengan nama `StoredName`, khususnya `sn_keccak(StoredName)`, ditambahkan ke daftar kunci. `user` diserialisasikan sebagai kunci, berkat atribut `#[key]`, sementara Address diserialisasikan sebagai data. Setelah semuanya diproses, kita mendapatkan kunci dan data berikut: `keys = [sn_keccak("StoredName"), user]` dan `data = [address]`.

### Memancarkan Event

Setelah mendefinisikan Event, kita dapat memancarkannya menggunakan `self.emit`, dengan sintaks berikut:

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_03_example_contract/src/lib.cairo:emit_event}}
```
