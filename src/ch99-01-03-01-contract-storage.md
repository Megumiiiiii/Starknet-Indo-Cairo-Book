# Storage Kontrak

Cara paling umum untuk berinteraksi dengan Storage kontrak adalah melalui variabel Storage. Seperti yang disebutkan sebelumnya, variabel Storage memungkinkan Anda menyimpan data yang akan disimpan di Storage kontrak yang sendiri disimpan di blockchain. Data ini persisten dan dapat diakses serta dimodifikasi kapan saja setelah kontrak diimplementasikan.

Variabel Storage dalam kontrak Starknet disimpan dalam struktur khusus yang disebut `Storage`:

```rust, noplayground
{{#rustdoc_include ../listings/ch99-starknet-smart-contracts/listing_99_03_example_contract/src/lib.cairo:storage}}
```

<span class="caption">Sebuah Struktur Storage</span>

Struktur Storage adalah [struktur](./ch05-00-using-structs-to-structure-related-data.md) seperti struktur lainnya,
kecuali bahwa ini **harus** diberi anotasi `#[storage]`. Anotasi ini memberi tahu kompiler untuk menghasilkan kode yang diperlukan untuk berinteraksi dengan status blockchain, dan memungkinkan Anda membaca dan menulis data dari dan ke Storage. Selain itu, ini memungkinkan Anda mendefinisikan pemetaan Storage menggunakan tipe `LegacyMap`.

Setiap variabel yang disimpan dalam struktur Storage disimpan di lokasi yang berbeda dalam Storage kontrak. Address Storage variabel ditentukan oleh nama variabel, dan kunci eventual dari variabel jika itu adalah [pemetaan](#storing-mappings).

## Address Storage

Address variabel Storage dihitung sebagai berikut:

- Jika variabel adalah nilai tunggal (bukan pemetaan), Addressnya adalah hash `sn_keccak` dari pengkodean ASCII nama variabel. `sn_keccak` adalah versi Starknet dari fungsi hash Keccak256, yang keluarannya dipotong menjadi 250 bit.
- Jika variabel adalah [pemetaan](#storing-mappings), Address nilai pada kunci `k_1,...,k_n` adalah `h(...h(h(sn_keccak(nama_variabel),k_1),k_2),...,k_n)` di mana ℎ adalah hash Pedersen dan nilai akhir diambil `mod (2^251) − 256`.
- Jika ini adalah pemetaan ke nilai-nilai kompleks (misalnya, tuple atau struktur), maka nilai kompleks ini terletak dalam segmen yang berkelanjutan dimulai dari Address yang dihitung pada poin sebelumnya. Perlu dicatat bahwa 256 elemen lapangan adalah batasan saat ini pada ukuran maksimal nilai Storage kompleks.

Anda dapat mengakses Address variabel Storage dengan memanggil fungsi `address` pada variabel, yang mengembalikan nilai `StorageBaseAddress`.

```rust, noplayground
{{#rustdoc_include ../listings/ch99-starknet-smart-contracts/listing_99_03_example_contract/src/lib.cairo:owner_address}}
```

## Berinteraksi dengan Variabel Storage

Variabel yang disimpan dalam struktur Storage dapat diakses dan dimodifikasi menggunakan fungsi `read` dan `write`, dan Anda dapat mendapatkan Address mereka di Storage menggunakan fungsi `addr`. Fungsi-fungsi ini secara otomatis dihasilkan oleh kompiler untuk setiap variabel Storage.

Untuk membaca nilai dari variabel Storage `owner`, yang merupakan nilai tunggal, kita memanggil fungsi `read` pada variabel `owner`, tanpa parameter.

```rust, noplayground
{{#rustdoc_include ../listings/ch99-starknet-smart-contracts/listing_99_03_example_contract/src/lib.cairo:read_owner}}
```

<span class="caption">Memanggil fungsi `read` pada variabel `owner`</span>

Untuk membaca nilai dari variabel Storage `names`, yang merupakan pemetaan dari `ContractAddress` ke `felt252`, kita memanggil fungsi `read` pada variabel `names`, dengan memberikan kunci `address` sebagai parameter. Jika pemetaan memiliki lebih dari satu kunci, kita akan memberikan kunci-kunci lain sebagai parameter juga.

```rust, noplayground
{{#rustdoc_include ../listings/ch99-starknet-smart-contracts/listing_99_03_example_contract/src/lib.cairo:read}}
```

<span class="caption">Memanggil fungsi `read` pada variabel `names`</span>

Untuk menulis nilai ke variabel Storage, kita memanggil fungsi `write` dengan memberikan kunci-kunci akhir nilai sebagai argumen. Seperti halnya dengan fungsi `read`, jumlah argumen tergantung pada jumlah kunci - di sini, kita hanya memberikan nilai untuk ditulis ke variabel `owner` karena itu adalah variabel sederhana.

```rust, noplayground
{{#rustdoc_include ../listings/ch99-starknet-smart-contracts/listing_99_03_example_contract/src/lib.cairo:write_owner}}
```

<span class="caption">Memanggil fungsi `write` pada variabel `owner`</span>

```rust, noplayground
{{#rustdoc_include ../listings/ch99-starknet-smart-contracts/listing_99_03_example_contract/src/lib.cairo:write}}
```

<span class="caption">Memanggil fungsi `write` pada variabel `names`</span>

## Menyimpan Tipe Kustom

Trait `Store`, yang didefinisikan dalam modul `starknet::storage_access`, digunakan untuk menentukan bagaimana suatu tipe harus disimpan dalam Storage. Agar suatu tipe dapat disimpan dalam Storage, ia harus mengimplementasikan trait `Store`. Sebagian besar tipe dari pustaka inti, seperti bilangan bulat tidak bertanda (`u8`, `u128`, `u256`, ...), `felt252`, `bool`, `ContractAddress`, dll. mengimplementasikan trait `Store` dan dapat disimpan tanpa tindakan lebih lanjut.

Tetapi bagaimana jika Anda ingin menyimpan tipe yang Anda tentukan sendiri, seperti enum atau struct? Dalam hal ini, Anda harus secara eksplisit memberi tahu kompiler cara menyimpan tipe ini.

Dalam contoh kita, kita ingin menyimpan struct `Person` dalam Storage, yang dapat dilakukan dengan mengimplementasikan trait `Store` untuk tipe `Person`. Ini dapat dicapai dengan menambahkan atribut `#[derive(starknet::Store)]` di atas definisi struct kita.

```rust, noplayground
{{#rustdoc_include ../listings/ch99-starknet-smart-contracts/listing_99_03_example_contract/src/lib.cairo:person}}
```

Demikian pula, Enums dapat ditulis ke Storage jika mereka mengimplementasikan trait `Store`, yang dapat dengan mudah di-derive selama semua tipe terkait mengimplementasikan trait `Store`.

```rust, noplayground
{{#rustdoc_include ../listings/ch99-starknet-smart-contracts/listing_99_03_example_contract/src/lib.cairo:enum_store}}
```

### Tata Letak Storage Structs

Pada Starknet, structs disimpan dalam Storage sebagai urutan tipe primitif. Elemen-elemen struct disimpan dalam urutan yang sama seperti yang didefinisikan dalam definisi struct. Elemen pertama dari struct disimpan pada Address dasar struct, yang dihitung seperti yang dijelaskan dalam [Address Storage](#storage-addresses) dan dapat diperoleh dengan memanggil `var.address()`, dan elemen-elemen berikutnya disimpan pada Address yang berdekatan dengan elemen pertama. Sebagai contoh, tata letak Storage untuk variabel `owner` bertipe `Person` akan menghasilkan tata letak berikut:

| Kolom   | Address             |
| ------- | ------------------ |
| nama    | owner.address()    |
| Address  | owner.address() +1 |

### Tata Letak Storage Enums

Ketika Anda menyimpan suatu varian enum, pada dasarnya yang Anda simpan adalah indeks varian dan nilai terkait yang mungkin. Indeks ini dimulai dari 0 untuk varian pertama enum Anda dan bertambah 1 untuk setiap varian berikutnya. Jika varian Anda memiliki nilai terkait, itu disimpan mulai dari Address yang segera mengikuti Address dasar. Sebagai contoh, misalkan kita memiliki enum `RegistrationType` dengan varian `finite`, yang membawa tanggal batas terkait. Tata letak Storagenya akan terlihat seperti ini:

| Elemen                            | Address                          |
| --------------------------------- | ------------------------------- |
| Indeks varian (misalnya 1 untuk finite) | registration_type.address()     |
| Tanggal batas terkait             | registration_type.address() + 1 |

## Pemetaan Storage

Pemetaan Storage mirip dengan tabel hash karena memungkinkan pemetaan kunci ke nilai. Namun, tidak seperti tabel hash biasa, data kunci itu sendiri tidak disimpan - hanya hash-nya yang digunakan untuk mencari nilai terkait dalam Storage kontrak.
Pemetaan tidak memiliki konsep panjang atau apakah sepasang kunci/nilai diatur. Satu-satunya cara untuk menghapus pemetaan adalah dengan mengatur nilainya ke nilai default nol.

Pemetaan hanya digunakan untuk menghitung lokasi data dalam Storage suatu
kontrak dengan memberikan kunci tertentu. Oleh karena itu, mereka **hanya diperbolehkan sebagai variabel Storage**.
Mereka tidak dapat digunakan sebagai parameter atau parameter pengembalian fungsi kontrak,
dan tidak dapat digunakan sebagai tipe dalam struct.

<div align="center">
    <img src="mappings.png" alt="pemetaan" width="500px"/>
<div align="center">
    </div>
    <span class="caption">Memetakan kunci ke nilai Storage</span>
</div>

Untuk mendeklarasikan pemetaan, gunakan tipe `LegacyMap` yang diapit dalam tanda kurung sudut `<>`,
menentukan tipe kunci dan nilai.

Anda juga dapat membuat pemetaan yang lebih kompleks dengan beberapa kunci. Anda dapat menemukannya dalam Listing 99-2bis seperti variabel Storage `allowances` yang populer dalam Standar ERC20 yang memetakan `owner` dan `spender` yang diizinkan ke jumlah `allowance` menggunakan beberapa kunci yang dilewatkan dalam tupel:

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/no_listing_01_storage_mapping/src/lib.cairo:here}}
```

<span class="caption">Listing 99-2bis: Menyimpan pemetaan</span>

Address dalam Storage dari suatu variabel yang disimpan dalam pemetaan dihitung sesuai dengan deskripsi dalam bagian [Address Storage](#storage-addresses).
Jika kunci pemetaan adalah struct, setiap elemen struct menjadi kunci. Selain itu, struct harus mengimplementasikan trait `Hash`, yang dapat diperoleh dengan atribut `#[derive(Hash)]`. Sebagai contoh, jika Anda memiliki struct dengan dua bidang, Addressnya akan menjadi `h(h(sn_keccak(nama_variabel),k_1),k_2)` - di mana `k_1` dan `k_2` adalah nilai dari dua bidang struct.

Demikian pula, dalam kasus pemetaan bertingkat seperti `LegacyMap((ContractAddress, ContractAddress), u8)`, Addressnya akan dihitung dengan cara yang sama: `h(h(sn_keccak(nama_variabel),k_1),k_2)`.

Untuk lebih jelasnya tentang tata letak Storage kontrak, kunjungi [Dokumentasi Starknet](https://docs.starknet.io/documentation/architecture_and_concepts/Smart_Contracts/contract-storage/#storage_variables)
