## Optimisasi Penyimpanan dengan `StorePacking`

Bit-packing adalah konsep sederhana: Gunakan sebanyak mungkin bit untuk menyimpan sebuah data. Ketika dilakukan dengan baik, ini dapat secara signifikan mengurangi ukuran data yang perlu Anda simpan. Hal ini terutama penting dalam kontrak pintar, di mana penyimpanan memiliki biaya yang mahal.

Ketika menulis kontrak pintar Cairo, penting untuk mengoptimalkan penggunaan penyimpanan untuk mengurangi biaya gas. Memang, sebagian besar biaya yang terkait dengan sebuah transaksi berhubungan dengan pembaruan penyimpanan; dan setiap slot penyimpanan membutuhkan biaya gas untuk ditulis.
Ini berarti dengan mem-packing beberapa nilai ke dalam slot yang lebih sedikit, Anda dapat mengurangi biaya gas yang dikeluarkan oleh pengguna kontrak pintar Anda.

Cairo menyediakan trait `StorePacking` untuk memungkinkan pem-packing bidang struct ke dalam slot penyimpanan yang lebih sedikit. Sebagai contoh, pertimbangkan sebuah struct `Sizes` dengan 3 bidang dari tipe yang berbeda. Ukuran totalnya adalah 8 + 32 + 64 = 104 bit. Ini kurang dari 128 bit dari sebuah `u128` tunggal. Artinya kita dapat mem-packing semua 3 bidang ke dalam sebuah variabel `u128` tunggal. Karena sebuah slot penyimpanan dapat menampung hingga 251 bit, nilai yang ter-packing kita akan hanya membutuhkan satu slot penyimpanan daripada 3.

```rust
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_13_storage_packing/src/lib.cairo:here}}
```

<span class="caption">Mengoptimalkan penyimpanan dengan mengimplementasikan trait `StorePacking`</span>

Fungsi `pack` menggabungkan ketiga bidang ke dalam sebuah nilai `u128` tunggal dengan melakukan bitshift dan penambahan. Fungsi `unpack` membalikkan proses ini untuk mengekstrak kembali bidang-bidang asli ke dalam sebuah struct.

Jika Anda tidak familiar dengan operasi bit, berikut penjelasan dari operasi-operasi yang dilakukan dalam contoh ini:
Tujuannya adalah mem-packing bidang `tiny`, `small`, dan `medium` ke dalam sebuah nilai `u128` tunggal.
Pertama, saat mem-packing:

- `tiny` adalah `u8` sehingga kita hanya mengonversinya secara langsung ke `u128` dengan `.into()`. Ini menciptakan sebuah nilai `u128` dengan 8 bit rendah diatur ke nilai dari `tiny`.
- `small` adalah `u32` sehingga pertama-tama kita menggesernya ke kiri sebanyak 8 bit (menambahkan 8 bit dengan nilai 0 ke kiri) untuk membuat ruang untuk 8 bit yang diambil oleh `tiny`. Lalu kita tambahkan `tiny` ke `small` untuk menggabungkannya menjadi sebuah nilai `u128` tunggal. Nilai dari `tiny` sekarang mengambil bit 0-7 dan nilai dari `small` mengambil bit 8-39.
- Demikian pula, `medium` adalah `u64` sehingga kita menggesernya ke kiri sebanyak 40 bit (8 + 32) (`TWO_POW_40`) untuk membuat ruang bagi bidang-bidang sebelumnya. Ini mengambil bit 40-103.

Ketika melakukan pem-unpacking:

- Pertama, kita ekstrak `tiny` dengan melakukan bitwise AND (&) dengan sebuah bitmask dari 8 satu (`& MASK_8`). Ini mengisolasi 8 bit terendah dari nilai yang ter-pack, yang merupakan nilai `tiny`.
- Untuk `small`, kita geser ke kanan sebanyak 8 bit (`/ TWO_POW_8`) untuk menyesuaikannya dengan bitmask, lalu gunakan bitwise AND dengan bitmask 32 satu.
- Untuk `medium`, kita geser ke kanan sebanyak 40 bit. Karena ini adalah nilai terakhir yang di-pack, kita tidak perlu menerapkan bitmask karena bit yang lebih tinggi sudah 0.

Teknik ini dapat digunakan untuk setiap kelompok bidang yang cocok dalam ukuran bit dari tipe penyimpanan yang ter-pack. Sebagai contoh, jika Anda memiliki sebuah struct dengan beberapa bidang yang ukuran bit-nya totalnya adalah 256 bit, Anda dapat mem-pack mereka ke dalam variabel `u256` tunggal. Jika ukuran bit-nya totalnya adalah 512 bit, Anda dapat mem-pack mereka ke dalam variabel `u512` tunggal, dan seterusnya. Anda dapat mendefinisikan struct dan logika Anda sendiri untuk mem-pack dan mem-unpack mereka.

Sisa pekerjaan dilakukan secara ajaib oleh compiler - jika sebuah tipe mengimplementasikan trait `StorePacking`, maka compiler akan tahu bahwa dapat menggunakan implementasi `StoreUsingPacking` dari trait `Store` untuk mem-pack sebelum menulis dan mem-unpack setelah membaca dari penyimpanan.
Satu detail penting, namun, adalah bahwa tipe yang dihasilkan oleh `StorePacking::pack` juga harus mengimplementasikan `Store` agar `StoreUsingPacking` dapat berfungsi. Kebanyakan waktu, kita akan ingin mem-pack ke dalam felt252 atau u256 - namun jika Anda ingin mem-pack ke dalam tipe milik Anda sendiri, pastikan bahwa tipe tersebut mengimplementasikan trait `Store`.
