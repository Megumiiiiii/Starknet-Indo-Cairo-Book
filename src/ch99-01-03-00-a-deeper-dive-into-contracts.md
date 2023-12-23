# Pemahaman yang Lebih Mendalam tentang Kontrak

Pada bagian sebelumnya, kami memberikan contoh pengantar tentang kontrak pintar yang ditulis dalam bahasa Cairo. Pada bagian ini, kita akan melihat lebih mendalam semua komponen dari kontrak pintar, langkah demi langkah.

Ketika kita membahas [_interfaces_](./ch99-01-02-a-simple-contract.md), kami menjelaskan perbedaan antara _public functions, external functions, dan view functions_, dan kami menyebutkan bagaimana berinteraksi dengan _storage_.

Pada titik ini, seharusnya ada beberapa pertanyaan yang muncul di pikiran Anda:

- Bagaimana cara saya mendefinisikan fungsi internal atau privat?
- Bagaimana cara saya mengeluarkan peristiwa (events)? Bagaimana cara saya membuat indeks untuk mereka?
- Di mana seharusnya saya mendefinisikan fungsi yang tidak perlu mengakses status kontrak?
- Adakah cara untuk mengurangi kode boilerplate?
- Bagaimana cara menyimpan jenis data yang lebih kompleks?

Untungnya, kita akan menjawab semua pertanyaan ini dalam bab ini. Mari pertimbangkan contoh kontrak berikut yang akan kita gunakan sepanjang bab ini:

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_03_example_contract/src/lib.cairo:all}}
```

<span class="caption">Listing 99-1bis: Kontrak referensi kita untuk bab ini</span>
