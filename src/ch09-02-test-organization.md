# Pengujian Organisasi

Kita akan memikirkan tentang pengujian dalam dua kategori utama: pengujian unit dan pengujian integrasi. Pengujian unit bersifat kecil dan lebih terfokus, menguji satu modul pada satu waktu secara terisolasi, dan dapat menguji fungsi-fungsi pribadi. Meskipun Cairo belum mengimplementasikan konsep fungsi/lapangan publik/privat, tetapi adalah praktik yang baik untuk mulai mengorganisir kode Anda seolah-olah itu demikian. Pengujian integrasi menggunakan kode Anda dengan cara yang sama seperti kode eksternal lainnya, menggunakan hanya antarmuka publik dan mungkin melibatkan beberapa modul per pengujian.

Menulis kedua jenis pengujian ini penting untuk memastikan bahwa bagian-bagian dari perpustakaan Anda melakukan apa yang Anda harapkan, baik secara terpisah maupun bersama-sama.

## Pengujian Unit

Tujuan dari pengujian unit adalah untuk menguji setiap unit kode secara terisolasi dari sisa kode untuk dengan cepat menentukan di mana kode bekerja atau tidak sesuai harapan. Anda akan menempatkan pengujian unit di direktori `src` dalam setiap file dengan kode yang sedang diuji.

Konvensinya adalah membuat modul bernama `tests` di setiap file untuk menyimpan fungsi-fungsi pengujian dan memberikan anotasi modul dengan `cfg(test)`.

### Modul Pengujian dan `#[cfg(test)]`

Anotasi `#[cfg(test)]` pada modul pengujian memberi tahu Cairo untuk mengompilasi dan menjalankan kode pengujian hanya ketika Anda menjalankan `scarb cairo-test`, bukan ketika Anda menjalankan `cairo-run`. Ini menghemat waktu kompilasi saat Anda hanya ingin membangun perpustakaan dan menghemat ruang dalam artefak yang dikompilasi karena pengujian tidak disertakan. Anda akan melihat bahwa karena pengujian integrasi berada di direktori yang berbeda, mereka tidak memerlukan anotasi `#[cfg(test)]`. Namun, karena pengujian unit berada di file yang sama dengan kode, Anda akan menggunakan `#[cfg(test)]` untuk menentukan bahwa mereka tidak boleh disertakan dalam hasil yang dikompilasi.

Ingat bahwa ketika kita membuat proyek `adder` baru di bagian pertama bab ini, kita menulis pengujian pertama ini:

```rust
{{#include ../listings/ch09-testing-cairo-programs/no_listing_06_cfg_attr/src/lib.cairo}}
```

<span class="caption">Nama file: src/lib.cairo</span>

Atribut `cfg` singkatan dari konfigurasi dan memberi tahu Cairo bahwa item berikutnya hanya harus disertakan dengan opsi konfigurasi tertentu. Dalam hal ini, opsi konfigurasi adalah `test`, yang disediakan oleh Cairo untuk mengompilasi dan menjalankan pengujian. Dengan menggunakan atribut `cfg`, Cairo mengompilasi kode pengujian kita hanya jika kita secara aktif menjalankan pengujian dengan `scarb cairo-test`. Ini mencakup fungsi bantu apa pun yang mungkin ada dalam modul ini, selain fungsi-fungsi yang dianotasi dengan `#[test]`.

## Pengujian Integrasi

Pengujian integrasi menggunakan perpustakaan Anda dengan cara yang sama seperti kode lainnya. Tujuannya adalah untuk menguji apakah banyak bagian dari perpustakaan Anda bekerja bersama dengan benar. Unit kode yang bekerja dengan benar secara independen bisa memiliki masalah saat diintegrasikan, jadi cakupan pengujian dari kode yang terintegrasi juga penting. Untuk membuat pengujian integrasi, pertama-tama Anda memerlukan direktori `tests`.

### Direktori `tests`

```shell
adder
├── Scarb.toml
├── src
│   ├── lib.cairo
│   ├── tests
│   │   └── integration_test.cairo
│   └── tests.cairo
```

```rust
{{#include ../listings/ch09-testing-cairo-programs/no_listing_07_integration_test/src/lib.cairo}}
```

<span class="caption">Nama file: src/lib.cairo</span>

```rust
#[cfg(tests)]
mod integration_tests;
```

<span class="caption">Nama file: src/tests.cairo</span>

Masukkan kode pada Listing 9-11 ke dalam file _src/tests/integration_test.cairo_:

```rust
{{#include ../listings/ch09-testing-cairo-programs/no_listing_07_integration_test/src/tests/integration_tests.cairo}}
```

<span class="caption">Nama file: src/tests/integration_test.cairo</span>

Kita perlu membawa fungsi-fungsi yang diuji ke dalam ruang lingkup setiap file pengujian. Untuk alasan itu, kami menambahkan `use adder::it_adds_two` di bagian atas kode, yang tidak perlu kami lakukan dalam pengujian unit.

Kemudian, untuk menjalankan semua pengujian integrasi kita, kita hanya perlu menambahkan filter untuk hanya menjalankan pengujian yang path-nya mengandung "integration_tests".

```shell
$ scarb test -f integration_tests
Running cairo-test adder
testing adder ...
running 1 tests
test adder::tests::integration_tests::internal ... ok (gas usage est.: 3770)
test result: ok. 1 passed; 0 failed; 0 ignored; 0 filtered out;
```

Hasil dari pengujian sama dengan yang telah kita lihat sebelumnya: satu baris untuk setiap pengujian.
