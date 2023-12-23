# Penyebaran dan Interaksi dengan Kontrak Voting

Kontrak **`Vote`** dalam Starknet dimulai dengan mendaftarkan pemilih melalui konstruktor kontrak. Tiga pemilih diinisialisasi pada tahap ini, dan Address mereka diteruskan ke dalam fungsi internal **`_register_voters`**. Fungsi ini menambahkan pemilih ke dalam status kontrak, menandai mereka sebagai terdaftar dan memenuhi syarat untuk memberikan suara.

Dalam kontrak ini, konstanta **`YES`** dan **`NO`** didefinisikan untuk mewakili opsi-opsi Voting (1 dan 0, secara berturut-turut). Konstanta-konstanta ini memfasilitasi proses Voting dengan standarisasi nilai-nilai input.

Setelah terdaftar, seorang pemilih dapat memberikan suara menggunakan fungsi **`vote`**, memilih antara 1 (YES) atau 0 (NO) sebagai suara mereka. Ketika memberikan suara, status kontrak diperbarui, mencatat suara dan menandai pemilih telah memberikan suara. Ini memastikan bahwa pemilih tidak dapat memberikan suara lagi dalam proposal yang sama. Pemberian suara memicu peristiwa **`VoteCast`**, mencatat tindakan tersebut.

Kontrak juga memantau upaya Voting yang tidak sah. Jika tindakan yang tidak sah terdeteksi, seperti pengguna yang tidak terdaftar mencoba memberikan suara atau pengguna mencoba memberikan suara lagi, peristiwa **`UnauthorizedAttempt`** akan dipancarkan.

Secara keseluruhan, fungsi-fungsi, status, konstanta-konstanta, dan peristiwa-peristiwa ini menciptakan sistem Voting terstruktur, mengelola siklus hidup suara dari pendaftaran hingga pemungutan, pencatatan peristiwa, dan pengambilan hasil dalam lingkungan Starknet. Konstanta-konstanta seperti **`YES`** dan **`NO`** membantu memperlancar proses Voting, sementara peristiwa-peristiwa memainkan peran penting dalam memastikan transparansi dan penelusuran.

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_12_vote_contract/src/lib.cairo}}
```

<span class="caption">Kontrak pintar Voting</span>

## Penyebaran, Pemanggilan, dan Memanggil Kontrak Voting

Bagian dari pengalaman Starknet adalah penyebaran dan interaksi dengan kontrak pintar.

Setelah kontrak dideploy, kita dapat berinteraksi dengannya dengan memanggil dan memicu fungsi-fungsi kontrak:

- Memanggil kontrak: Berinteraksi dengan fungsi eksternal yang hanya membaca dari status. Fungsi-fungsi ini tidak mengubah status jaringan, sehingga tidak memerlukan biaya atau tanda tangan.
- Memicu kontrak: Berinteraksi dengan fungsi-fungsi eksternal yang dapat menulis ke status. Fungsi-fungsi ini mengubah status jaringan dan memerlukan biaya serta tanda tangan.

Kita akan menyiapkan sebuah node pengembangan lokal menggunakan `katana` untuk melakukan penyebaran kontrak Voting. Selanjutnya, kita akan berinteraksi dengan kontrak tersebut dengan memanggil dan memicu fungsi-fungsinya. Anda juga dapat menggunakan Goerli Testnet sebagai alternatif dari `katana`. Namun, kami merekomendasikan penggunaan `katana` untuk pengembangan dan pengujian lokal. Anda dapat menemukan tutorial lengkap untuk `katana` di [Pengembangan Lokal dengan Katana](https://book.starknet.io/chapter_3/katana.html) dalam bab Buku Starknet.

### Node lokal Starknet `katana`

`katana` dirancang untuk mendukung pengembangan lokal oleh [tim Dojo](https://github.com/dojoengine/dojo/blob/main/crates/katana/README.md). Ini akan memungkinkan Anda melakukan semua hal yang diperlukan dengan Starknet, tetapi secara lokal. Ini adalah alat yang bagus untuk pengembangan dan pengujian.

Untuk menginstal `katana` dari kode sumber, silakan lihat [Pengembangan Lokal dengan Katana](https://book.starknet.io/chapter_3/katana.html) dalam bab Buku Starknet.

Setelah Anda menginstal `katana`, Anda dapat memulai node lokal Starknet dengan:

```bash
katana --accounts 3 --seed 0 --gas-price 250
```

Perintah ini akan memulai node lokal Starknet dengan 3 akun yang telah dideploy. Kami akan menggunakan akun-akun ini untuk menyebar dan berinteraksi dengan kontrak Voting:

```bash
...
PREFUNDED ACCOUNTS
==================

| Account address |  0x03ee9e18edc71a6df30ac3aca2e0b02a198fbce19b7480a63a0d71cbd76652e0
| Private key     |  0x0300001800000000300000180000000000030000000000003006001800006600
| Public key      |  0x01b7b37a580d91bc3ad4f9933ed61f3a395e0e51c9dd5553323b8ca3942bb44e

| Account address |  0x033c627a3e5213790e246a917770ce23d7e562baa5b4d2917c23b1be6d91961c
| Private key     |  0x0333803103001800039980190300d206608b0070db0012135bd1fb5f6282170b
| Public key      |  0x04486e2308ef3513531042acb8ead377b887af16bd4cdd8149812dfef1ba924d

| Account address |  0x01d98d835e43b032254ffbef0f150c5606fa9c5c9310b1fae370ab956a7919f5
| Private key     |  0x07ca856005bee0329def368d34a6711b2d95b09ef9740ebf2c7c7e3b16c1ca9c
| Public key      |  0x07006c42b1cfc8bd45710646a0bb3534b182e83c313c7bc88ecf33b53ba4bcbc
...
```

Sebelum kita dapat berinteraksi dengan kontrak Voting, kita perlu menyiapkan akun pemilih dan admin di Starknet. Setiap akun pemilih harus didaftarkan dan memiliki dana yang cukup untuk memberikan suara. Untuk pemahaman yang lebih mendetail tentang bagaimana akun beroperasi dengan Abstraksi Akun, lihat bab [Abstraksi Akun](https://book.starknet.io/chapter_4/index.html) dalam Buku Starknet.

### Smart Wallet untuk Voting

Selain Scarb, Anda juga perlu menginstal Starkli. Starkli adalah alat baris perintah yang memungkinkan Anda berinteraksi dengan Starknet. Anda dapat menemukan petunjuk instalasinya dalam bab [Penyiapan Lingkungan](https://book.starknet.io/chapter_1/environment_setup.html) dalam Buku Starknet.

Untuk setiap Smart Wallet yang akan kita gunakan, kita harus membuat Signer di dalam keystore terenkripsi dan Deskriptor Akun. Proses ini juga dijelaskan dalam bab [Penyiapan Lingkungan](https://book.starknet.io/chapter_1/environment_setup.html) dalam Buku Starknet.

Kita dapat membuat Signer dan Deskriptor Akun untuk akun-akun yang ingin kita gunakan untuk memberikan suara. Mari buat Smart Wallet untuk memberikan suara dalam kontrak pintar kita.

Pertama, kita buat seorang signer dari sebuah kunci privat:

```bash
starkli signer keystore from-key ~/.starkli-wallets/deployer/account0_keystore.json
```

Kemudian, kita buat Deskriptor Akun dengan mengambil akun katana yang ingin kita gunakan:

```bash
starkli account fetch <Address AKUN KATANA> --rpc http://0.0.0.0:5050 --output ~/.starkli-wallets/deployer/account0_account.json
```

Perintah ini akan membuat file `account0_account.json` yang berisi detail-detail berikut:

```bash
{
  "version": 1,
  "variant": {
        "type": "open_zeppelin",
        "version": 1,
        "public_key": "<KUNCI_PUBLIK_Wallet_CERDAS>"
  },
    "deployment": {
        "status": "deployed",
        "class_hash": "<HASH_KELAS_Wallet_CERDAS>",
        "address": "<Address_Wallet_CERDAS>"
  }
}
```

Anda dapat mengambil hash dari Smart Wallet (akan sama untuk semua Smart Wallet Anda) dengan perintah berikut. Perhatikan penggunaan flag `--rpc` dan ujung titik RPC yang disediakan oleh `katana`:

```
starkli class-hash-at <Address_Wallet_CERDAS> --rpc http://0.0.0.0:5050
```

Untuk kunci publik, Anda dapat menggunakan perintah `starkli signer keystore inspect` dengan direktori file JSON keystore:

```bash
starkli signer keystore inspect ~/.starkli-wallets/deployer/account0_keystore.json
```

Proses ini identik untuk `account_1` dan `account_2` jika Anda ingin memiliki pemilih kedua dan ketiga.

### Penyebaran Kontrak

Sebelum melakukan penyebaran, kita perlu mendeklarasikan kontrak. Kita bisa melakukannya dengan perintah `starkli declare`:

```bash
starkli declare target/dev/starknetbook_chapter_2_Vote.sierra.json --rpc http://0.0.0.0:5050 --account ~/.starkli-wallets/deployer/account0_account.json --keystore ~/.starkli-wallets/deployer/account0_keystore.json
```

Jika versi kompiler yang Anda gunakan lebih lama dari yang digunakan oleh Starkli dan Anda mengalami kesalahan `compiler-version` saat menggunakan perintah di atas, Anda dapat menentukan versi kompiler yang akan digunakan dalam perintah dengan menambahkan flag `--compiler-version x.y.z`.

Jika Anda masih mengalami masalah dengan versi kompiler, cobalah tingkatkan Starkli menggunakan perintah: `starkliup` untuk memastikan Anda menggunakan versi terbaru dari starkli.

Hash kelas dari kontrak adalah: `0x06974677a079b7edfadcd70aa4d12aac0263a4cda379009fca125e0ab1a9ba52`. Anda dapat menemukannya [di eksplorasi blok apa pun](https://goerli.voyager.online/class/0x06974677a079b7edfadcd70aa4d12aac0263a4cda379009fca125e0ab1a9ba52).

Flag `--rpc` menentukan ujung titik RPC yang akan digunakan (yang disediakan oleh `katana`). Flag `--account` menentukan akun yang akan digunakan untuk menandatangani transaksi. Akun yang kita gunakan di sini adalah yang kita buat pada langkah sebelumnya. Flag `--keystore` menentukan file keystore yang akan digunakan untuk menandatangani transaksi.

Karena kita menggunakan node lokal, transaksi akan segera mencapai finalitas. Jika Anda menggunakan Goerli Testnet, Anda perlu menunggu transaksi menjadi final, yang biasanya memerlukan beberapa detik.

Perintah berikut ini akan menyebar kontrak Voting dan mendaftarkan voter_0, voter_1, dan voter_2 sebagai pemilih yang memenuhi syarat. Ini adalah argumen konstruktor, jadi tambahkan akun pemilih yang nantinya bisa memberikan suara.

```bash
starkli deploy <class_hash_of_the_contract_to_be_deployed> <voter_0_address> <voter_1_address> <voter_2_address> --rpc http://0.0.0.0:5050 --account ~/.starkli-wallets/deployer/account0_account.json --keystore ~/.starkli-wallets/deployer/account0_keystore.json
```

Contoh command:

```bash
starkli deploy 0x06974677a079b7edfadcd70aa4d12aac0263a4cda379009fca125e0ab1a9ba52 0x03ee9e18edc71a6df30ac3aca2e0b02a198fbce19b7480a63a0d71cbd76652e0 0x033c627a3e5213790e246a917770ce23d7e562baa5b4d2917c23b1be6d91961c 0x01d98d835e43b032254ffbef0f150c5606fa9c5c9310b1fae370ab956a7919f5 --rpc http://0.0.0.0:5050 --account ~/.starkli-wallets/deployer/account0_account.json --keystore ~/.starkli-wallets/deployer/account0_keystore.json
```

Pada kasus ini, kontrak telah dideploy di Address tertentu: `0x05ea3a690be71c7fcd83945517f82e8861a97d42fca8ec9a2c46831d11f33349`. Address ini akan berbeda untuk Anda. Kita akan menggunakan Address ini untuk berinteraksi dengan kontrak.

### Verifikasi Kelayakan Pemilih

Dalam kontrak Voting kita, ada dua fungsi untuk memvalidasi kelayakan pemilih, yaitu `voter_can_vote` dan `is_voter_registered`. Fungsi-fungsi ini adalah fungsi baca eksternal, yang berarti mereka tidak mengubah status kontrak tetapi hanya membaca status saat ini.

Fungsi `is_voter_registered` memeriksa apakah Address tertentu terdaftar sebagai pemilih yang memenuhi syarat dalam kontrak. Sementara itu, fungsi `voter_can_vote` memeriksa apakah pemilih di Address tertentu saat ini berhak memberikan suara, yaitu mereka terdaftar dan belum memberikan suara.

Anda dapat memanggil fungsi-fungsi ini menggunakan perintah `starkli call`. Perhatikan bahwa perintah `call` digunakan untuk fungsi baca, sementara perintah `invoke` digunakan untuk fungsi yang juga dapat menulis ke penyimpanan. Perintah `call` tidak memerlukan tanda tangan, sedangkan perintah `invoke` memerlukannya.

```bash+
starkli call 0x05ea3a690be71c7fcd83945517f82e8861a97d42fca8ec9a2c46831d11f33349 voter_can_vote 0x03ee9e18edc71a6df30ac3aca2e0b02a198fbce19b7480a63a0d71cbd76652e0 --rpc http://0.0.0.0:5050
```

Pertama, kita tambahkan Address kontrak, kemudian fungsi yang ingin kita panggil, dan terakhir masukan untuk fungsi tersebut. Dalam kasus ini, kita sedang memeriksa apakah pemilih pada Address `0x03ee9e18edc71a6df30ac3aca2e0b02a198fbce19b7480a63a0d71cbd76652e0` bisa memberikan suara.

Karena kami memberikan Address pemilih yang terdaftar sebagai masukan, hasilnya adalah 1 (benar), yang menunjukkan bahwa pemilih berhak memberikan suara.

Selanjutnya, mari panggil fungsi `is_voter_registered` menggunakan Address akun yang tidak terdaftar untuk melihat keluarannya:

```bash
starkli call 0x05ea3a690be71c7fcd83945517f82e8861a97d42fca8ec9a2c46831d11f33349 is_voter_registered 0x44444444444444444 --rpc http://0.0.0.0:5050
```

Dengan Address akun yang tidak terdaftar, output terminalnya adalah 0 (yaitu, salah), yang mengkonfirmasi bahwa akun tersebut tidak memenuhi syarat untuk memberikan suara.

### Memberikan Suara

Sekarang setelah kita memahami cara memverifikasi kelayakan pemilih, kita bisa memberikan suara! Untuk memberikan suara, kita berinteraksi dengan fungsi `vote`, yang ditandai sebagai eksternal, yang memerlukan penggunaan perintah `starknet invoke`.

Syntax perintah `invoke` mirip dengan perintah `call`, tetapi untuk memberikan suara, kita mengirimkan entah `1` (untuk Ya) atau `0` (untuk Tidak) sebagai masukan. Ketika kita memanggil fungsi `vote`, kita akan dikenai biaya, dan transaksi harus ditandatangani oleh pemilih; kita sedang menulis ke penyimpanan kontrak.

```bash
//Memberikan suara Ya
starkli invoke 0x05ea3a690be71c7fcd83945517f82e8861a97d42fca8ec9a2c46831d11f33349 vote 1 --rpc http://0.0.0.0:5050 --account ~/.starkli-wallets/deployer/account0_account.json --keystore ~/.starkli-wallets/deployer/account0_keystore.json

//Memberikan suara Tidak
starkli invoke 0x05ea3a690be71c7fcd83945517f82e8861a97d42fca8ec9a2c46831d11f33349 vote 0 --rpc http://0.0.0.0:5050 --account ~/.starkli-wallets/deployer/account0_account.json --keystore ~/.starkli-wallets/deployer/account0_keystore.json
```

Anda akan diminta untuk memasukkan kata sandi untuk signer. Setelah Anda memasukkan kata sandi, transaksi akan ditandatangani dan dikirimkan ke jaringan Starknet. Anda akan menerima hash transaksi sebagai keluaran. Dengan perintah starkli transaction, Anda dapat mendapatkan lebih banyak detail tentang transaksi tersebut:

```bash
starkli transaction <TRANSACTION_HASH> --rpc http://0.0.0.0:5050
```

Ini akan menampilkan:

```bash
{
  "transaction_hash": "0x5604a97922b6811060e70ed0b40959ea9e20c726220b526ec690de8923907fd",
  "max_fee": "0x430e81",
  "version": "0x1",
  "signature": [
    "0x75e5e4880d7a8301b35ff4a1ed1e3d72fffefa64bb6c306c314496e6e402d57",
    "0xbb6c459b395a535dcd00d8ab13d7ed71273da4a8e9c1f4afe9b9f4254a6f51"
  ],
  "nonce": "0x3",
  "type": "INVOKE",
  "sender_address": "0x3ee9e18edc71a6df30ac3aca2e0b02a198fbce19b7480a63a0d71cbd76652e0",
  "calldata": [
    "0x1",
    "0x5ea3a690be71c7fcd83945517f82e8861a97d42fca8ec9a2c46831d11f33349",
    "0x132bdf85fc8aa10ac3c22f02317f8f53d4b4f52235ed1eabb3a4cbbe08b5c41",
    "0x0",
    "0x1",
    "0x1",
    "0x1"
  ]
}
```

Jika Anda mencoba memberikan suara dua kali dengan signer yang sama, Anda akan mendapatkan kesalahan:

```bash
Error: code=ContractError, message="Contract error"
```

Kesalahan tersebut tidak memberikan informasi yang sangat informatif, tetapi Anda dapat memperoleh lebih banyak detail ketika melihat keluaran di terminal tempat Anda memulai `katana` (node Starknet lokal kita):

```bash
...
Transaction execution error: "Error in the called contract (0x03ee9e18edc71a6df30ac3aca2e0b02a198fbce19b7480a63a0d71cbd76652e0):
    Error at pc=0:81:
    Got an exception while executing a hint: Custom Hint Error: Execution failed. Failure reason: \"USER_ALREADY_VOTED\".
    ...
```

Kunci untuk kesalahan tersebut adalah `USER_ALREADY_VOTED`.

```bash
assert(can_vote == true, 'USER_ALREADY_VOTED');
```

Kita dapat mengulangi proses untuk membuat Signer dan Account Descriptor untuk akun yang ingin kita gunakan untuk memberikan suara. Ingatlah bahwa setiap Signer harus dibuat dari sebuah private key, dan setiap Account Descriptor harus dibuat dari public key, Address smart wallet, dan smart wallet class hash (yang sama untuk setiap pemilih).

```bash
starkli invoke 0x05ea3a690be71c7fcd83945517f82e8861a97d42fca8ec9a2c46831d11f33349 vote 0 --rpc http://0.0.0.0:5050 --account ~/.starkli-wallets/deployer/account1_account.json --keystore ~/.starkli-wallets/deployer/account1_keystore.json

starkli invoke 0x05ea3a690be71c7fcd83945517f82e8861a97d42fca8ec9a2c46831d11f33349 vote 1 --rpc http://0.0.0.0:5050 --account ~/.starkli-wallets/deployer/account2_account.json --keystore ~/.starkli-wallets/deployer/account2_keystore.json
```

### Visualisasi Hasil Voting

Untuk memeriksa hasil Voting, kita memanggil fungsi `get_vote_status`, fungsi tampilan lainnya, melalui perintah `starknet call`.

```bash
starkli call 0x05ea3a690be71c7fcd83945517f82e8861a97d42fca8ec9a2c46831d11f33349 get_vote_status --rpc http://0.0.0.0:5050
```

Keluarannya menampilkan perolehan suara "Ya" dan "Tidak" beserta persentase relatifnya.
