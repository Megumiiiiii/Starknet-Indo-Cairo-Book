## Lampiran E - Jenis dan Sifat Umum serta Preludium Cairo

### Preludium

Preludium Cairo adalah kumpulan modul, fungsi, jenis data, dan sifat yang sering digunakan yang secara otomatis dibawa ke dalam lingkup setiap modul dalam suatu keranjang Cairo tanpa perlu pernyataan impor eksplisit. Preludium Cairo menyediakan blok bangunan dasar yang dibutuhkan pengembang untuk memulai program Cairo dan menulis kontrak pintar.

Preludium pustaka inti didefinisikan dalam berkas [lib.cairo](https://github.com/starkware-libs/cairo/blob/v2.4.0/corelib/src/lib.cairo) dari keranjang corelib dan berisi jenis data primitif, sifat, operator, dan fungsi utilitas Cairo. Ini mencakup: Jenis data - felts, bools, array, dicts, dll. Sifat - perilaku untuk aritmatika, perbandingan, serialisasi. Operator - aritmatika, logis, bitwise. Fungsi utilitas - pembantu untuk array, peta, boxing, dll. Preludium pustaka inti menyampaikan konstruksi pemrograman dan operasi dasar yang diperlukan untuk program Cairo dasar, tanpa memerlukan impor eksplisit dari elemen-elemen tersebut. Karena preludium pustaka inti diimpor secara otomatis, isinya tersedia untuk digunakan dalam keranjang Cairo mana pun tanpa impor eksplisit. Ini mencegah pengulangan dan memberikan pengalaman pengembangan yang lebih baik. Inilah yang memungkinkan Anda menggunakan `ArrayTrait::append()` atau sifat `Default` tanpa membawanya secara eksplisit ke dalam lingkup.

Anda dapat memilih preludium yang akan digunakan. Misalnya, dengan menambahkan `edition = "2023_10"` dalam berkas konfigurasi `Scarb.toml` akan memuat preludium dari Oktober 2023, yang lebih terbatas daripada yang dari Januari 2023.
Compiler saat ini mengekspos 2 versi preludium yang berbeda:
- Versi umum, dengan banyak sifat yang tersedia, sesuai dengan `edition = "2023_01"`.
- Versi yang lebih terbatas, termasuk sifat-sifat yang paling penting diperlukan untuk pemrograman cairo umum, sesuai dengan `edition = 2023_10`.

### Daftar jenis dan sifat umum

Bagian berikut memberikan gambaran singkat tentang jenis dan sifat yang sering digunakan saat mengembangkan program Cairo. Sebagian besar termasuk dalam preludium dan tidak perlu diimpor secara eksplisit - tetapi tidak semua.

| Impor                     | Path                                                  | Penggunaan                                                                                                                                                                            |
| ------------------------- | ----------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `OptionTrait`             | `core::option::OptionTrait`                           | `OptionTrait<T>` mendefinisikan seperangkat metode yang diperlukan untuk memanipulasi nilai opsional.                                                                                   |
| `ResultTrait`             | `core::result::ResultTrait`                           | `ResultTrait<T, E>` Jenis untuk Address kontrak Starknet, nilai dalam rentang [0, 2 \*\* 251).                                                                                           |
| `ContractAddress`         | `starknet::ContractAddress`                           | `ContractAddress` adalah jenis untuk mewakili Address kontrak pintar.                                                                                                                   |
| `ContractAddressZeroable` | `starknet::contract_address::ContractAddressZeroable` | `ContractAddressZeroable` adalah implementasi dari sifat `Zeroable` untuk jenis `ContractAddress`. Diperlukan untuk memeriksa apakah nilai `t:ContractAddress` adalah nol atau tidak.     |
| `contract_address_const`  | `starknet::contract_address_const`                    | `contract_address_const!` adalah fungsi yang memungkinkan instansiasi nilai Address kontrak konstan.                                                                                  |
| `Into`                    | `traits::Into;`                                       | `Into<T>` adalah sifat yang digunakan untuk konversi antar jenis. Jika ada implementasi Into<T,S> untuk jenis T dan S, Anda dapat mengonversi T menjadi S.                               |
| `TryInto`                 | `traits::TryInto;`                                    | `TryInto<T>` adalah sifat yang digunakan untuk konversi antar jenis. Jika ada implementasi TryInto<T,S> untuk jenis T dan S, Anda dapat mengonversi T menjadi S.                         |
| `get_caller_address`      | `starknet::get_caller_address`                        | `get_caller_address()` adalah fungsi yang mengembalikan Address pemanggil kontrak. Ini dapat digunakan untuk mengidentifikasi pemanggil fungsi kontrak.                                |
| `get_contract_address`    | `starknet::info::get_contract_address`                | `get_contract_address()` adalah fungsi yang mengembalikan Address kontrak saat ini. Ini dapat digunakan untuk mendapatkan Address kontrak yang sedang dieksekusi.                         |

Ini bukan daftar yang lengkap, tetapi mencakup beberapa jenis dan sifat yang sering digunakan dalam pengembangan kontrak. Untuk informasi lebih lanjut, lihat dokumentasi resmi dan jelajahi pustaka dan kerangka kerja yang tersedia.