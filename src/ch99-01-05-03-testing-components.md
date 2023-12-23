# Pengujian Komponen

Pengujian komponen sedikit berbeda dibandingkan dengan pengujian kontrak.
Kontrak perlu diuji terhadap keadaan tertentu, yang dapat dicapai dengan cara mendeploy kontrak dalam sebuah uji coba, atau dengan hanya mendapatkan objek `ContractState` dan memodifikasinya dalam konteks uji coba Anda.

Komponen adalah konstruk umum, yang dimaksudkan untuk diintegrasikan dalam kontrak, yang tidak dapat didedikasikan sendiri dan tidak memiliki objek `ContractState` yang dapat kita gunakan. Jadi, bagaimana kita mengujinya?

Mari kita pertimbangkan bahwa kita ingin menguji komponen yang sangat sederhana bernama "Counter", yang akan memungkinkan setiap kontrak memiliki penghitung yang dapat diinkrementasi. Komponen ini didefinisikan sebagai berikut:

```rust, noplayground
{{#include ../listings/ch99-starknet-smart-contracts/components/listing_03_test_component/src/counter.cairo:component}}
```

## Pengujian komponen dengan mendeploy kontrak tiruan

Cara termudah untuk menguji komponen adalah dengan mengintegrasikannya dalam kontrak tiruan. Kontrak tiruan ini hanya digunakan untuk tujuan pengujian, dan hanya mengintegrasikan komponen yang ingin Anda uji. Ini memungkinkan Anda untuk menguji komponen dalam konteks sebuah kontrak, dan menggunakan Dispatcher untuk memanggil titik masuk komponen.

Kita dapat mendefinisikan kontrak tiruan tersebut sebagai berikut:

```rust, noplayground
{{#include ../listings/ch99-starknet-smart-contracts/components/listing_03_test_component/src/lib.cairo:mock_contract}}
```

Kontrak ini sepenuhnya didedikasikan untuk pengujian komponen `Counter`. Ini menyematkan komponen dengan macro `component!`, mengekspos titik masuk komponen dengan memberikan anotasi alias impl dengan `#[abi(embed_v0)]`.

Kita juga perlu mendefinisikan sebuah antarmuka yang akan diperlukan untuk berinteraksi secara eksternal dengan kontrak tiruan ini.

```rust, noplayground
{{#include ../listings/ch99-starknet-smart-contracts/components/listing_03_test_component/src/counter.cairo:interface}}
```

Sekarang kita dapat menulis pengujian untuk komponen dengan mendeploy kontrak tiruan ini dan memanggil titik masuknya, seperti yang kita lakukan dengan kontrak biasa.

```rust, noplayground
{{#include ../listings/ch99-starknet-smart-contracts/components/listing_03_test_component/src/tests_deployed.cairo}}
```

## Pengujian Komponen tanpa Mendeploy Kontrak

Pada [Komponen di bawah Tampilan](./ch99-01-05-01-components-under-the-hood.md), kita melihat bahwa komponen memanfaatkan genericity untuk mendefinisikan penyimpanan dan logika yang dapat disematkan dalam beberapa kontrak. Jika sebuah kontrak menyematkan sebuah komponen, sebuah `HasComponent` trait dibuat dalam kontrak ini, dan metode-metode komponen dibuat tersedia.

Hal ini memberi informasi kepada kita bahwa jika kita dapat menyediakan sebuah `TContractState` konkret yang mengimplementasikan trait `HasComponent` ke dalam struktur `ComponentState`, seharusnya kita dapat langsung memanggil metode-metode dari komponen menggunakan objek `ComponentState` konkret ini, tanpa harus mendeploy mock.

Mari kita lihat bagaimana kita dapat melakukannya dengan menggunakan type alias. Kita masih perlu mendefinisikan sebuah kontrak tiruan - mari gunakan yang sama seperti di atas - tetapi kali ini, kita tidak perlu mendeploynya.

Pertama, kita perlu mendefinisikan sebuah implementasi konkret dari tipe generic `ComponentState` menggunakan sebuah type alias. Kita akan menggunakan tipe `MockContract::ContractState` untuk melakukannya.

```rust, noplayground
{{#rustdoc_include ../listings/ch99-starknet-smart-contracts/components/listing_03_test_component/src/tests_direct.cairo:type_alias}}
```

Kami mendefinisikan tipe `TestingState` sebagai alias dari tipe `CounterComponent::ComponentState<MockContract::ContractState>`. Dengan melewati tipe `MockContract::ContractState` sebagai tipe konkret untuk `ComponentState`, kami memberikan alias implementasi konkret dari struktur `ComponentState` menjadi `TestingState`.

Karena `MockContract` menyematkan `CounterComponent`, metode-metode dari `CounterComponent` yang didefinisikan dalam blok `CounterImpl` sekarang dapat digunakan pada objek `TestingState`.

Sekarang setelah kita telah membuat metode-metode ini tersedia, kita perlu menginisialisasi sebuah objek tipe `TestingState` yang akan kita gunakan untuk menguji komponen. Kita dapat melakukannya dengan memanggil fungsi `component_state_for_testing`, yang secara otomatis menyimpulkan bahwa itu harus mengembalikan objek tipe `TestingState`.

Kita bahkan dapat mengimplementasikannya sebagai bagian dari trait `Default`, yang memungkinkan kita untuk mengembalikan `TestingState` kosong dengan sintaks `Default::default()`.

Mari kita ringkas apa yang telah kita lakukan sejauh ini:

- Kita mendefinisikan sebuah kontrak tiruan yang menyematkan komponen yang ingin kita uji.
- Kita mendefinisikan sebuah implementasi konkret dari `ComponentState<TContractState>` menggunakan type alias dengan `MockContract::ContractState`, yang kita beri nama `TestingState`.
- Kita mendefinisikan sebuah fungsi yang menggunakan `component_state_for_testing` untuk mengembalikan objek `TestingState`.

Sekarang kita dapat menulis pengujian untuk komponen dengan memanggil fungsi-fungsi nya secara langsung, tanpa harus mendeploy kontrak tiruan. Pendekatan ini lebih ringan dibandingkan sebelumnya, dan memungkinkan pengujian fungsi internal dari komponen yang tidak mudah diekspos ke dunia luar.

```rust, noplayground
{{#rustdoc_include ../listings/ch99-starknet-smart-contracts/components/listing_03_test_component/src/tests_direct.cairo:test}}
```
