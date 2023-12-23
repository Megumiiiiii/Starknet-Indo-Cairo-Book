# ABIs dan Antarmuka Kontrak

Interaksi lintas kontrak antara kontrak pintar pada blockchain adalah praktik umum yang memungkinkan kita membangun kontrak yang fleksibel sehingga dapat berkomunikasi satu sama lain.

Untuk mencapai hal ini di Starknet, diperlukan sesuatu yang disebut antarmuka.

## ABI - Antarmuka Biner Aplikasi

Di Starknet, ABI dari suatu kontrak adalah representasi JSON dari fungsi-fungsi dan struktur kontrak, memberikan kemampuan kepada siapa pun (atau kontrak lainnya) untuk membentuk panggilan yang terenkripsi kepadanya. Ini adalah sebuah cetak biru yang memberi petunjuk bagaimana fungsi-fungsi seharusnya dipanggil, parameter masukan yang diharapkan, dan dalam format apa.

Meskipun kita menulis logika kontrak pintar kita dalam bahasa Cairo tingkat tinggi, mereka disimpan pada VM sebagai bytecode yang dapat dieksekusi dalam format biner. Karena bytecode ini tidak dapat dibaca oleh manusia, diperlukan interpretasi untuk dipahami. Di sinilah ABIs berperan, mendefinisikan metode-metode khusus yang dapat dipanggil ke suatu kontrak pintar untuk dieksekusi. Tanpa ABI, menjadi praktis tidak mungkin bagi aktor eksternal untuk memahami bagaimana berinteraksi dengan sebuah kontrak.

ABIs biasanya digunakan dalam antarmuka pengguna aplikasi terdesentralisasi (dApps), memungkinkan untuk memformat data dengan benar, sehingga dapat dimengerti oleh kontrak pintar dan sebaliknya. Ketika Anda berinteraksi dengan kontrak pintar melalui penjelajah blok seperti [Voyager](https://voyager.online/) atau [Starkscan](https://starkscan.co/), mereka menggunakan ABI kontrak untuk memformat data yang Anda kirim ke kontrak dan data yang dikembalikannya.

## Interface

Antarmuka dari sebuah kontrak adalah daftar fungsi yang diekspos secara publik oleh kontrak tersebut.
Antarmuka ini menentukan tanda tangan fungsi (nama, parameter, visibilitas, dan nilai kembalian) yang terdapat dalam kontrak pintar tanpa menyertakan badan fungsi.

Antarmuka kontrak dalam Cairo adalah traits yang diberi anotasi dengan atribut `#[starknet::interface]`. Jika Anda baru mengenal traits, lihat bab khusus tentang [traits](./ch08-02-traits-in-cairo.md).

Spesifikasi penting adalah bahwa trait ini harus generik terhadap tipe `TContractState`. Hal ini diperlukan agar fungsi-fungsi dapat mengakses penyimpanan kontrak, sehingga mereka dapat membaca dan menulis ke dalamnya.

> Catatan: Konstruktor kontrak tidak termasuk dalam antarmuka. Begitu pula fungsi internal tidak termasuk dalam antarmuka.

Berikut adalah contoh antarmuka untuk kontrak token ERC20. Seperti yang dapat Anda lihat, ini adalah trait generik terhadap tipe `TContractState`. Fungsi-fungsi `view` memiliki parameter self dengan tipe `@TContractState`, sedangkan fungsi-fungsi `external` memiliki parameter self dengan tipe yang dilewatkan dengan referensi `ref self: TContractState`.

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_04_interface/src/lib.cairo}}
```

<span class="caption">Daftar 99-4: Antarmuka ERC20 sederhana</span>

Pada bab selanjutnya, kita akan melihat bagaimana kita dapat memanggil kontrak dari kontrak pintar lain menggunakan _dispatchers_ dan _syscalls_.
