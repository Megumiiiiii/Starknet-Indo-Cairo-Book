## Lampiran D - Alat Pengembangan yang Berguna

Pada lampiran ini, kita akan membahas beberapa alat pengembangan yang berguna yang disediakan oleh proyek Cairo. Kita akan melihat tentang pemformatan otomatis, cara cepat untuk menerapkan perbaikan peringatan, sebuah linter, dan integrasi dengan IDE.

### Pemformatan Otomatis dengan `scarb fmt`

Proyek-proyek Scarb dapat diformat menggunakan perintah `scarb fmt`. Jika Anda menggunakan biner Cairo secara langsung, Anda dapat menjalankan `cairo-format` sebagai gantinya. Banyak proyek kolaboratif menggunakan `scarb fmt` untuk mencegah adanya argumen tentang gaya mana yang harus digunakan saat menulis kode Cairo: semua orang memformat kode mereka menggunakan alat ini.

Untuk memformat proyek Cairo apa pun, masukkan perintah berikut:

### Integrasi IDE Menggunakan `cairo-language-server`

Untuk membantu integrasi IDE, komunitas Cairo merekomendasikan penggunaan [`cairo-language-server`][cairo-language-server]<!-- ignore -->. Alat ini adalah serangkaian utilitas yang berpusat pada kompiler yang berbicara dalam [Language Server Protocol][lsp]<!-- ignore -->, yang merupakan spesifikasi untuk IDE dan bahasa pemrograman berkomunikasi satu sama lain. Berbagai klien dapat menggunakan `cairo-language-server`, seperti [ekstensi Cairo untuk Visual Studio Code][vscode-cairo].

[lsp]: http://langserver.org/
[vscode-cairo]: https://marketplace.visualstudio.com/items?itemName=starkware.cairo1

Kunjungi [halaman][vscode-cairo]<!-- ignore --> `vscode-cairo` untuk menginstalnya di VSCode. Anda akan mendapatkan kemampuan seperti autocompletion, lompat ke definisi, dan kesalahan inline.

[cairo-language-server]: https://github.com/starkware-libs/cairo/tree/main/crates/cairo-lang-language-server

> Catatan: Jika Anda telah menginstal Scarb, seharusnya berfungsi dengan baik secara otomatis dengan ekstensi Cairo VSCode, tanpa perlu instalasi manual dari server bahasa.