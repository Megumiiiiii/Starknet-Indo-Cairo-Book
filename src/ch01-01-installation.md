# Instalasi

Cairo dapat diinstal dengan cara mengunduh [Scarb](https://docs.swmansion.com/scarb/docs). Scarb mengemas kompiler Cairo dan server bahasa Cairo ke dalam paket yang mudah diinstal sehingga Anda dapat mulai menulis kode Cairo segera.

Scarb juga merupakan pengelola paket Cairo dan sangat terinspirasi oleh [Cargo](https://doc.rust-lang.org/cargo/), sistem build dan pengelola paket Rust.

Scarb menangani banyak tugas untuk Anda, seperti membangun kode Anda (baik itu murni Cairo atau kontrak Starknet), mengunduh perpustakaan yang dibutuhkan oleh kode Anda, membangun perpustakaan-perpustakaan tersebut, dan menyediakan dukungan LSP untuk ekstensi VSCode Cairo 1.

Ketika Anda menulis program Cairo yang lebih kompleks, Anda mungkin akan menambahkan dependensi, dan jika Anda memulai proyek menggunakan Scarb, mengelola kode eksternal dan dependensi akan menjadi jauh lebih mudah dilakukan.

Mari mulai dengan menginstal Scarb.

## Menginstal Scarb

### Persyaratan

Scarb memerlukan Git yang dapat dijalankan dalam variabel lingkungan `PATH`.

### Instalasi

Untuk menginstal Scarb, silakan merujuk ke [petunjuk instalasi](https://docs.swmansion.com/scarb/download). Kami sangat merekomendasikan Anda untuk menginstal Scarb [melalui asdf](https://docs.swmansion.com/scarb/download.html#install-via-asdf), sebuah alat CLI yang dapat mengelola berbagai versi runtime bahasa pada basis proyek-per-proyek. Hal ini akan memastikan bahwa versi Scarb yang Anda gunakan untuk bekerja pada suatu proyek selalu sesuai dengan yang ditentukan dalam pengaturan proyek, menghindari masalah karena ketidakcocokan versi.
Atau, Anda dapat menjalankan perintah berikut di terminal Anda, dan ikuti petunjuk yang muncul di layar. Ini akan menginstal rilis stabil terbaru dari Scarb.

```bash
curl --proto '=https' --tlsv1.2 -sSf https://docs.swmansion.com/scarb/install.sh | sh
```

- Verifikasi instalasi dengan menjalankan perintah berikut di sesi terminal baru, seharusnya mencetak versi Scarb dan Cairo, misalnya:

<!-- TODO UPDATE VERSION -->

```bash
$ scarb --version
scarb 2.4.0 (cba988e68 2023-12-06)
cairo: 2.4.0 (https://crates.io/crates/cairo-lang-compiler/2.4.0)
sierra: 1.4.0
```

## Menginstal ekstensi VSCode

Cairo memiliki ekstensi VSCode yang menyediakan penyorotan sintaks, penyelesaian kode, dan fitur-fitur berguna lainnya. Anda dapat menginstalnya dari [Marketplace VSCode](https://marketplace.visualstudio.com/items?itemName=starkware.cairo1).
Setelah terinstal, masuklah ke pengaturan ekstensi, dan pastikan untuk mencentang opsi `Enable Language Server` dan `Enable Scarb`.