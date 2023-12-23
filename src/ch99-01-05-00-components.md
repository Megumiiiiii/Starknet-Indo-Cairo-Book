# Komponen: Blok Bangunan Mirip Lego untuk Kontrak Pintar

Mengembangkan kontrak yang berbagi logika dan penyimpanan umum dapat menyulitkan dan rentan terhadap bug, karena logika tersebut sulit untuk digunakan kembali dan perlu diimplementasikan ulang dalam setiap kontrak. Tetapi bagaimana jika ada cara untuk menyematkan hanya fungsionalitas tambahan yang Anda butuhkan di dalam kontrak Anda, memisahkan logika inti kontrak Anda dari bagian lainnya?

Komponen menyediakan tepat itu. Mereka adalah add-on modular yang mengemas logika, penyimpanan, dan acara yang dapat digunakan kembali yang dapat disertakan ke dalam beberapa kontrak. Mereka dapat digunakan untuk memperluas fungsionalitas kontrak tanpa harus mengimplementasikan ulang logika yang sama berulang kali.

Pikirkan tentang komponen sebagai blok Lego. Mereka memungkinkan Anda untuk memperkaya kontrak Anda dengan menyematkan modul yang Anda tulis sendiri atau oleh orang lain. Modul ini bisa sederhana, seperti komponen kepemilikan, atau lebih kompleks seperti token ERC20 yang lengkap.

Sebuah komponen adalah modul terpisah yang dapat berisi penyimpanan, acara, dan fungsi. Berbeda dengan kontrak, komponen tidak dapat dideklarasikan atau didaftarkan. Logikanya pada akhirnya akan menjadi bagian dari bytecode kontrak tempat ia disematkan.

## Apa Saja yang Ada di dalam Sebuah Komponen?

Sebuah komponen sangat mirip dengan kontrak. Ia dapat berisi:

- Variabel penyimpanan
- Acara (events)
- Fungsi eksternal dan internal

Berbeda dengan kontrak, komponen tidak dapat dideploy secara mandiri. Kode komponen akan menjadi bagian dari kontrak tempat komponen tersebut disematkan.

## Membuat Komponen

Untuk membuat sebuah komponen, pertama-tama tentukan dalam modulnya sendiri yang diberi atribut `#[starknet::component]`. Dalam modul ini, Anda dapat mendeklarasikan struktur `Storage` dan enumerasi `Event`, seperti biasanya dilakukan dalam [Kontrak](./ch99-01-02-a-simple-contract.md).

Langkah selanjutnya adalah mendefinisikan antarmuka komponen, yang berisi tanda tangan fungsi yang akan memungkinkan akses eksternal ke logika komponen. Anda dapat mendefinisikan antarmuka komponen dengan mendeklarasikan sebuah trait dengan atribut `#[starknet::interface]`, sama seperti yang dilakukan dengan kontrak. Antarmuka ini akan digunakan untuk memungkinkan akses eksternal ke fungsi-fungsi komponen menggunakan pola [Dispatcher](./ch99-02-02-contract-dispatcher-library-dispatcher-and-system-calls.md).

Implementasi sebenarnya dari logika eksternal komponen dilakukan dalam blok `impl` yang ditandai sebagai `#[embeddable_as(nama)]`. Biasanya, blok `impl` ini akan menjadi implementasi dari trait yang mendefinisikan antarmuka komponen.

> Catatan: `nama` adalah nama yang akan kita gunakan dalam kontrak untuk merujuk ke komponen tersebut. Ini berbeda dengan nama impl Anda.

Anda juga dapat mendefinisikan fungsi internal yang tidak akan dapat diakses secara eksternal, dengan hanya mengabaikan atribut `#[embeddable_as(nama)]` di atas blok `impl` internal. Anda akan dapat menggunakan fungsi internal ini di dalam kontrak di mana Anda menyematkan komponen tersebut, tetapi tidak dapat berinteraksi dengannya dari luar, karena mereka bukan bagian dari abi kontrak.

Fungsi-fungsi dalam blok `impl` ini mengharapkan argumen seperti `ref self: ComponentState<TContractState>` (untuk fungsi yang mengubah status) atau `self: @ComponentState<TContractState>` (untuk fungsi tampilan). Hal ini membuat impl menjadi generik terhadap `TContractState`, memungkinkan kita untuk menggunakan komponen ini dalam berbagai kontrak.

### Contoh: sebuah Komponen Ownable

> ⚠️ Contoh yang ditunjukkan di bawah belum diaudit dan tidak dimaksudkan untuk digunakan dalam produksi. Para penulis tidak bertanggung jawab atas kerusakan yang disebabkan oleh penggunaan kode ini.

Antarmuka dari komponen Ownable, yang mendefinisikan metode-metode yang tersedia secara eksternal untuk mengelola kepemilikan suatu kontrak, akan terlihat seperti ini:

```rust
{{#include ../listings/ch99-starknet-smart-contracts/components/listing_01_ownable/src/component.cairo:interface}}
```

Komponen itu sendiri didefinisikan sebagai:

<!-- TODO -->

```rust
{{#include ../listings/ch99-starknet-smart-contracts/components/listing_01_ownable/src/component.cairo:component}}
```

Sintaks ini sebenarnya cukup mirip dengan sintaks yang digunakan untuk kontrak. Satu-satunya perbedaan terkait atribut `#[embeddable_as]` di atas impl dan genericity blok impl yang akan kita kupas secara detail.

Seperti yang dapat Anda lihat, komponen kami memiliki dua blok `impl`: satu yang sesuai dengan implementasi trait antarmuka, dan satu berisi metode-metode yang seharusnya tidak diekspos secara eksternal dan hanya dimaksudkan untuk penggunaan internal. Mengekspos `assert_only_owner` sebagai bagian dari antarmuka tidak akan masuk akal, karena hanya dimaksudkan untuk digunakan secara internal oleh kontrak yang menyematkan komponen.

## Melihat Lebih Dekat pada Blok `impl`

<!-- TODO kutipan sintaks blok impl saja -->

```rust
{{#include ../listings/ch99-starknet-smart-contracts/components/listing_01_ownable/src/component.cairo:impl_signature}}
```

Atribut `#[embeddable_as]` digunakan untuk menandai impl sebagai yang dapat disematkan di dalam kontrak. Ini memungkinkan kita untuk menentukan nama impl yang akan digunakan dalam kontrak untuk merujuk ke komponen ini. Dalam kasus ini, komponen akan disebut sebagai `Ownable` dalam kontrak yang menyematkannya.

Implementasinya sendiri generik terhadap `ComponentState<TContractState>`, dengan batasan tambahan bahwa `TContractState` harus mengimplementasikan trait `HasComponent<T>`. Ini memungkinkan kita untuk menggunakan komponen dalam berbagai kontrak, selama kontrak mengimplementasikan trait `HasComponent`. Memahami mekanisme ini secara detail tidak diperlukan untuk menggunakan komponen, tetapi jika Anda penasaran tentang cara kerjanya, Anda dapat membaca lebih lanjut di bagian [Komponen di balik layar](./ch99-01-05-01-components-under-the-hood.md).

Salah satu perbedaan utama dari kontrak pintar biasa adalah bahwa akses ke penyimpanan dan acara dilakukan melalui tipe generik `ComponentState<TContractState>` dan bukan `ContractState`. Perhatikan bahwa meskipun tipe berbeda, akses penyimpanan atau pemicuan acara dilakukan dengan cara yang serupa melalui `self.storage_var_name.read()` atau `self.emit(...)`.

> Catatan: Untuk menghindari kebingungan antara nama yang dapat disematkan dan nama impl, kami merekomendasikan untuk tetap menggunakan sufiks `Impl` dalam nama impl.

## Memigrasi Kontrak ke Sebuah Komponen

Karena baik kontrak maupun komponen memiliki banyak kesamaan, sebenarnya sangat mudah untuk bermigrasi dari kontrak ke komponen. Perubahan yang diperlukan hanya:

- Menambahkan atribut `#[starknet::component]` ke dalam modul.
- Menambahkan atribut `#[embeddable_as(nama)]` ke blok `impl` yang akan disematkan dalam kontrak lain.
- Menambahkan parameter generic ke blok `impl`:
  - Menambahkan `TContractState` sebagai parameter generic.
  - Menambahkan `+HasComponent<TContractState>` sebagai batasan impl.
- Mengubah tipe argumen `self` dalam fungsi-fungsi di dalam blok `impl` menjadi `ComponentState<TContractState>` bukan `ContractState`.

Untuk trait yang tidak memiliki definisi eksplisit dan dihasilkan menggunakan `#[generate_trait]`, logikanya sama - namun trait tersebut generik terhadap `TContractState` bukan `ComponentState<TContractState>`, seperti yang ditunjukkan dalam contoh dengan `InternalTrait`.

## Menggunakan komponen di dalam sebuah kontrak

Kekuatan utama dari komponen adalah bagaimana hal itu memungkinkan penggunaan kembali primitif yang telah dibangun di dalam kontrak Anda dengan sejumlah kecil boilerplate yang terbatas. Untuk mengintegrasikan sebuah komponen ke dalam kontrak Anda, Anda perlu:

1. Mendeklarasikannya dengan macro `component!()`, dengan menyebutkan

   1. Path ke komponen `path::to::component`.
   2. Nama variabel penyimpanan dalam penyimpanan kontrak Anda yang merujuk ke penyimpanan komponen ini (misalnya `ownable`).
   3. Nama variabel dalam enum acara kontrak Anda yang merujuk ke acara komponen ini (misalnya `OwnableEvent`).

2. Menambahkan path ke penyimpanan dan acara komponen ke `Storage` dan `Event` kontrak. Mereka harus cocok dengan nama yang diberikan pada langkah 1 (misalnya `ownable: ownable_component::Storage` dan `OwnableEvent: ownable_component::Event`).

   Variabel penyimpanan **HARUS** diberi atribut `#[substorage(v0)]`.

3. Menyematkan logika komponen yang didefinisikan di dalam kontrak Anda, dengan menginstansiasi impl generik komponen dengan `ContractState` konkret menggunakan sebuah alias impl. Alias ini harus diberi anotasi `#[abi(embed_v0)]` untuk secara eksternal mengekspos fungsi-fungsi komponen.

   Seperti yang dapat Anda lihat, InternalImpl tidak ditandai dengan `#[abi(embed_v0)]`.
   Memang, kami tidak ingin mengekspos secara eksternal fungsi-fungsi yang didefinisikan dalam impl ini. Namun, mungkin kita masih ingin mengaksesnya secara internal.

<!-- TODO: Add content on impl aliases -->

Contohnya, untuk menyematkan komponen `Ownable` yang didefinisikan di atas, kita akan melakukan hal berikut:

```rust
{{#include ../listings/ch99-starknet-smart-contracts/components/listing_01_ownable/src/contract.cairo:all}}
```

Logika komponen sekarang menjadi bagian dari kontrak tanpa hambatan! Kita dapat berinteraksi dengan fungsi-fungsi komponen secara eksternal dengan memanggilnya menggunakan `IOwnableDispatcher` yang diinisialisasi dengan Address kontrak.

```rust
{{#include ../listings/ch99-starknet-smart-contracts/components/listing_01_ownable/src/component.cairo:interface}}
```

## Menggabungkan Komponen untuk Komposabilitas Maksimal

Komposabilitas dari komponen benar-benar terlihat ketika menggabungkan beberapa komponen bersama-sama. Setiap komponen menambahkan fiturnya ke kontrak. Anda akan dapat bergantung pada implementasi komponen-komponen [Openzeppelin's](https://github.com/OpenZeppelin/cairo-contracts) di masa mendatang untuk dengan cepat menyematkan semua fungsionalitas umum yang Anda butuhkan dalam sebuah kontrak.

Para pengembang dapat fokus pada logika inti kontrak mereka sambil mengandalkan komponen-komponen yang telah diuji dan diaudit untuk segala sesuatu yang lain.

Komponen bahkan dapat [bergantung](./ch99-01-05-02-component-dependencies.md) pada komponen lain dengan membatasi
`TContractstate` yang mereka generikkan untuk mengimplementasikan trait dari komponen lain. Sebelum kita masuk ke dalam mekanisme ini, mari kita pertama-tama melihat [bagaimana komponen bekerja di bawah
kap](./ch99-01-05-01-components-under-the-hood).

## Troubleshooting

Anda mungkin mengalami beberapa kesalahan saat mencoba mengimplementasikan komponen. Sayangnya, beberapa dari mereka tidak memiliki pesan kesalahan yang bermakna untuk membantu dalam debugging. Bagian ini bertujuan untuk memberikan beberapa petunjuk untuk membantu Anda memperbaiki kode Anda.

- `Trait not found. Not a trait.`

  Kesalahan ini dapat terjadi ketika Anda tidak mengimpor blok impl komponen dengan benar ke dalam kontrak Anda. Pastikan untuk mematuhi sintaks berikut:

  ```rust
  #[abi(embed_v0)]
  impl NAMA_IMPL = komponen::NAMA_TERSIMPAN<ContractState>
  ```

  Merujuk ke contoh sebelumnya kita, ini akan menjadi:

  ```rust
  #[abi(embed_v0)]
  impl OwnableImpl = upgradeable::Ownable<ContractState>
  ```

- `Plugin diagnostic: name is not a substorage member in the contract's Storage.
Consider adding to Storage: (...)`

  Compiler membantu Anda banyak dalam debugging ini dengan memberikan rekomendasi tindakan yang harus diambil. Pada dasarnya, Anda lupa menambahkan penyimpanan komponen ke penyimpanan kontrak Anda. Pastikan untuk menambahkan path ke penyimpanan komponen yang diberi atribut `#[substorage(v0)]` ke penyimpanan kontrak Anda.

- `Plugin diagnostic: name is not a nested event in the contract's Event enum.
Consider adding to the Event enum:`

  Serupa dengan kesalahan sebelumnya, compiler memberitahu Anda bahwa Anda lupa menambahkan acara komponen ke acara kontrak Anda. Pastikan untuk menambahkan path ke acara komponen ke acara kontrak Anda.

- Fungsi-fungsi komponen tidak dapat diakses secara eksternal

  Hal ini dapat terjadi jika Anda lupa memberi blok impl komponen dengan anotasi `#[abi(embed_v0)]`. Pastikan untuk menambahkan anotasi ini saat menyematkan blok impl komponen dalam kontrak Anda.
