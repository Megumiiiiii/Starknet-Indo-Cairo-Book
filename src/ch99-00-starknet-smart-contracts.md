# Kontrak Pintar Starknet

Sepanjang bagian-bagian sebelumnya, Anda sebagian besar telah menulis program-program dengan `main` sebagai titik masuk. Pada bagian-bagian mendatang, Anda akan belajar menulis dan mendeploy kontrak-kontrak Starknet.

Salah satu aplikasi dari bahasa Cairo adalah menulis kontrak pintar untuk jaringan Starknet. Starknet adalah jaringan tanpa izin yang memanfaatkan teknologi zk-STARKs untuk skalabilitas. Sebagai solusi skalabilitas Layer-2 untuk Ethereum, tujuan Starknet adalah menawarkan transaksi yang cepat, aman, dan berbiaya rendah. Ini berfungsi sebagai Validity Rollup (umumnya dikenal sebagai zero-knowledge Rollup) dan dibangun di atas bahasa Cairo dan Starknet VM.

Kontrak-kontrak Starknet, dengan kata-kata sederhana, adalah program-program yang dapat berjalan pada Starknet VM. Karena mereka berjalan di VM, mereka memiliki akses ke status persisten Starknet, dapat mengubah atau memodifikasi variabel-variabel dalam status Starknet, berkomunikasi dengan kontrak-kontrak lain, dan berinteraksi dengan mudah dengan L1 yang mendasarinya.

Kontrak-kontrak Starknet ditandai dengan atribut `#[contract]`. Kami akan lebih mendalam tentang ini pada bagian-bagian selanjutnya.
Jika Anda ingin mempelajari lebih lanjut tentang jaringan Starknet itu sendiri, arsitekturnya, dan perangkat yang tersedia, Anda sebaiknya membaca [Buku Starknet](https://book.starknet.io/). Bagian ini akan fokus pada penulisan kontrak pintar di Cairo.

#### Scarb

Scarb mendukung pengembangan kontrak pintar untuk Starknet. Untuk mengaktifkan fungsionalitas ini, Anda perlu melakukan beberapa konfigurasi dalam file `Scarb.toml` Anda (lihat [Instalasi](./ch01-01-installation.md) untuk cara menginstal Scarb).
Anda harus menambahkan dependensi `starknet` dan menambahkan bagian `[[target.starknet-contract]]` untuk mengaktifkan kompilasi kontrak.

Berikut adalah file Scarb.toml minimal yang diperlukan untuk mengompilasi crate yang berisi kontrak-kontrak Starknet:

```toml
[package]
name = "package_name"
version = "0.1.0"

[dependencies]
starknet = ">=2.4.0"

[[target.starknet-contract]]
```

Untuk konfigurasi tambahan, seperti dependensi kontrak eksternal, silakan merujuk ke [Dokumentasi Scarb](https://docs.swmansion.com/scarb/docs/extensions/starknet/contract-target.html#compiling-external-contracts).

Setiap contoh dalam bab ini dapat digunakan dengan Scarb.