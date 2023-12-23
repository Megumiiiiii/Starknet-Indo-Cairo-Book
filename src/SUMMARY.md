# Bahasa Pemrograman Cairo

[Bahasa Pemrograman Cairo](title-page.md)
[Pendahuluan](ch00-00-introduction.md)

## Memulai

- [Memulai](ch01-00-getting-started.md)

  - [Instalasi](ch01-01-installation.md)
  - [Hello, World!](ch01-02-hello-world.md)

## Konsep Pemrograman Umum

- [Konsep Pemrograman Umum](ch02-00-common-programming-concepts.md)
  - [Variabel dan Mutabilitas](ch02-01-variables-and-mutability.md)
  - [Tipe Data](ch02-02-data-types.md)
  - [Fungsi](ch02-03-functions.md)
  - [Komentar](ch02-04-comments.md)
  - [Alur Kontrol](ch02-05-control-flow.md)

## Koleksi Umum

- [Koleksi Umum](ch03-00-common-collections.md)
  - [Array](ch03-01-arrays.md)
  - [Kamus](ch03-02-dictionaries.md)
  - [Struktur Data Kustom](ch03-03-custom-data-structures.md)

## Memahami Kepemilikan

- [Memahami Kepemilikan](ch04-00-understanding-ownership.md)
  - [Apa itu Kepemilikan?](ch04-01-what-is-ownership.md)
  - [Referensi dan Snapshot](ch04-02-references-and-snapshots.md)

## Menggunakan Struct untuk Menyusun Data Terkait

- [Menggunakan Struct untuk Menyusun Data Terkait](ch05-00-using-structs-to-structure-related-data.md)
  - [Mendefinisikan dan Menginstansiasi Struct](ch05-01-defining-and-instantiating-structs.md)
  - [Contoh Program Menggunakan Struct](ch05-02-an-example-program-using-structs.md)
  - [Sintaks Metode](ch05-03-method-syntax.md)

## Enum dan Pattern Matching

- [Enum dan Pattern Matching](ch06-00-enums-and-pattern-matching.md)
  - [Enum](ch06-01-enums.md)
  - [Konstruksi Alur Kontrol Match](ch06-02-the-match-control-flow-construct.md)

## Mengelola Proyek Cairo dengan Paket, Crate, dan Modul

- [Mengelola Proyek Cairo dengan Paket, Crate, dan Modul](ch07-00-managing-cairo-projects-with-packages-crates-and-modules.md)

  - [Paket dan Crate](ch07-01-packages-and-crates.md)
  - [Mendefinisikan Modul untuk Mengontrol Ruang Lingkup](ch07-02-defining-modules-to-control-scope.md)
  - [Path untuk Merujuk Item dalam Pohon Modul](ch07-03-paths-for-referring-to-an-item-in-the-module-tree.md)
  - [Membawa Path ke Ruang Lingkup dengan Kata Kunci 'use'](ch07-04-bringing-paths-into-scope-with-the-use-keyword.md)
  - [Memisahkan Modul ke Berbagai File](ch07-05-separating-modules-into-different-files.md)

## Tipe Data Generik

- [Tipe Generik](ch08-00-generic-types-and-traits.md)

  - [Fungsi Generik](ch08-01-generic-data-types.md)
  - [Traits di Cairo](ch08-02-traits-in-cairo.md)

## Pengujian Program Cairo

- [Pengujian Program Cairo](ch09-00-testing-cairo-programs.md)

  - [Cara Menulis Uji Coba](ch09-01-how-to-write-tests.md)
  - [Organisasi Pengujian](ch09-02-test-organization.md)

## Penanganan Kesalahan

- [Penanganan Kesalahan](ch10-00-error-handling.md)

  - [Kesalahan Tak Dapat Dipulihkan dengan panic](ch10-01-unrecoverable-errors-with-panic.md)
  - [Kesalahan Dapat Dipulihkan dengan Result](ch10-02-recoverable-errors.md)

## Fitur Lanjutan

- [Fitur Lanjutan](ch11-00-advanced-features.md)

  - [Overloading Operator](ch11-01-operator-overloading.md)
  - [Macros](ch11-02-macros.md)
  - [Bekerja dengan Hash](ch11-03-hash.md)

## Kontrak Pintar Starknet

- [Kontrak Pintar Starknet](./ch99-00-starknet-smart-contracts.md)

  - [Pengantar untuk kontrak pintar](./ch99-01-01-introduction-to-smart-contracts.md)
  - [Kontrak sederhana](./ch99-01-02-a-simple-contract.md)
  - [Penjelajahan lebih dalam ke dalam kontrak](./ch99-01-03-00-a-deeper-dive-into-contracts.md)

    - [Penyimpanan Kontrak](./ch99-01-03-01-contract-storage.md)
    - [Fungsi Kontrak](./ch99-01-03-02-contract-functions.md)
    - [Acara Kontrak](./ch99-01-03-03-contract-events.md)
    - [Mengurangi ketidaklengkapan](./ch99-01-03-04-reducing-boilerplate.md)
    - [Optimasi biaya penyimpanan](./ch99-01-03-05-optimizing-storage.md)

  - [Komponen](./ch99-01-05-00-components.md)

    - [Di balik layar](./ch99-01-05-01-components-under-the-hood.md)
    - [Ketergantungan Komponen](./ch99-01-05-02-component-dependencies.md)
    - [Menguji komponen](./ch99-01-05-03-testing-components.md)

  - [ABIs dan Interaksi Antar Kontrak](./ch99-02-00-abis-and-cross-contract-interactions.md)

    - [ABIs dan Antarmuka](./ch99-02-01-abis-and-interfaces.md)
    - [Dispatcher Kontrak, Dispatcher Perpustakaan, dan panggilan sistem](./ch99-02-02-contract-dispatcher-library-dispatcher-and-system-calls.md)

  - [Contoh lain](./ch99-01-04-00-other-examples.md)

    - [Mengimplementasikan dan Berinteraksi dengan kontrak Voting](./ch99-01-04-01-voting-contract.md)

  - [Pesan L1 <> L2](./ch99-04-00-L1-L2-messaging.md)
  - [Pertimbangan Keamanan](./ch99-03-security-considerations.md)

- [Lampiran](appendix-00.md)

  - [A - Kata Kunci](appendix-01-keywords.md)
  - [B - Operator dan Simbol](appendix-02-operators-and-symbols.md)
  - [C - Traits yang Dapat Diperoleh](appendix-03-derivable-traits.md)
  - [D - Alat Pengembangan yang Berguna](appendix-04-useful-development-tools.md)
  - [E - Tipe & Traits Umum dan Pralang Cairo](appendix-05-common-types-and-traits-and-cairo-prelude.md)
  - [F - Menginstal Biner Cairo](appendix-06-cairo-binaries.md)