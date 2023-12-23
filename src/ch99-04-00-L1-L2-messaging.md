# Pesan L1-L2

Fitur penting dari Layer 2 adalah kemampuannya untuk berinteraksi dengan Layer 1.

Starknet memiliki sistem `L1-L2` Messaging sendiri, yang berbeda dari mekanisme konsensusnya dan pengiriman pembaruan status di L1. Messaging adalah cara bagi smart-contract di L1 untuk berinteraksi dengan smart-contract di L2 (atau sebaliknya), memungkinkan kita melakukan transaksi "cross-chain". Sebagai contoh, kita dapat melakukan beberapa komputasi pada suatu rantai dan menggunakan hasil komputasi ini pada rantai lain.

Semua jembatan di Starknet menggunakan `L1-L2` messaging. Katakanlah Anda ingin menjembatani token dari Ethereum ke Starknet. Anda hanya perlu mendepositkan token Anda di kontrak jembatan L1, yang secara otomatis akan memicu pencetakan token yang sama di L2. Penggunaan kasus lain yang bagus untuk `L1-L2` messaging adalah [pengelolaan DeFi](https://starkware.co/resource/defi-pooling/).

Di Starknet, penting untuk dicatat bahwa sistem messaging ini **asynchronous** dan **asymmetric**.

- **Asynchronous**: ini berarti bahwa dalam kode kontrak Anda (baik itu solidity atau cairo), Anda tidak dapat menunggu hasil dari pesan yang dikirim di rantai lain dalam eksekusi kode kontrak Anda.
- **Asymmetric**: mengirim pesan dari Ethereum ke Starknet (`L1->L2`) sepenuhnya diotomatisasi oleh sequencer Starknet, yang berarti pesan tersebut secara otomatis dikirim ke kontrak target di L2. Namun, saat mengirim pesan dari Starknet ke Ethereum (`L2->L1`), hanya hash dari pesan yang dikirim di L1 oleh sequencer Starknet. Anda kemudian harus mengonsumsi pesan tersebut secara manual melalui transaksi di L1.

Mari kita telusuri detailnya.

## Kontrak StarknetMessaging

Komponen penting dari sistem `L1-L2` Messaging adalah kontrak [`StarknetCore`](https://etherscan.io/address/0xc662c410C0ECf747543f5bA90660f6ABeBD9C8c4). Ini adalah kumpulan kontrak Solidity yang didistribusikan di Ethereum yang memungkinkan Starknet untuk berfungsi dengan baik. Salah satu kontrak dari `StarknetCore` disebut `StarknetMessaging` dan merupakan kontrak yang bertanggung jawab atas pengiriman pesan antara Starknet dan Ethereum. `StarknetMessaging` mengikuti [antarmuka](https://github.com/starkware-libs/cairo-lang/blob/4e233516f52477ad158bc81a86ec2760471c1b65/src/starkware/starknet/eth/IStarknetMessaging.sol#L6) dengan fungsi-fungsi yang memungkinkan pengiriman pesan ke L2, menerima pesan di L1 dari L2, dan membatalkan pesan.

```js
interface IStarknetMessaging is IStarknetMessagingEvents {

    function sendMessageToL2(
        uint256 toAddress,
        uint256 selector,
        uint256[] calldata payload
    ) external returns (bytes32);

    function consumeMessageFromL2(uint256 fromAddress, uint256[] calldata payload)
        external
        returns (bytes32);

    function startL1ToL2MessageCancellation(
        uint256 toAddress,
        uint256 selector,
        uint256[] calldata payload,
        uint256 nonce
    ) external;

    function cancelL1ToL2Message(
        uint256 toAddress,
        uint256 selector,
        uint256[] calldata payload,
        uint256 nonce
    ) external;
}
```

<span class="caption"> Antarmuka kontrak messaging Starknet</span>

Dalam kasus pesan `L1->L2`, sequencer Starknet terus-menerus mendengarkan log yang dihasilkan oleh kontrak `StarknetMessaging` di Ethereum. Begitu sebuah pesan terdeteksi dalam log, sequencer menyiapkan dan mengeksekusi `L1HandlerTransaction` untuk memanggil fungsi di kontrak L2 yang dituju. Proses ini memakan waktu 1-2 menit (beberapa detik untuk blok Ethereum ditambang, dan kemudian sequencer harus membangun dan mengeksekusi transaksi).

Pesan `L2->L1` disiapkan oleh eksekusi kontrak di L2 dan merupakan bagian dari blok yang dihasilkan. Ketika sequencer menghasilkan blok, ia mengirim hash setiap pesan yang disiapkan oleh eksekusi kontrak
ke kontrak `StarknetCore` di L1, di mana pesan tersebut kemudian dapat dikonsumsi setelah blok yang mereka miliki terbukti dan diverifikasi di Ethereum (saat ini sekitar 3-4 jam).

## Mengirim pesan dari Ethereum ke Starknet

Jika Anda ingin mengirim pesan dari Ethereum ke Starknet, kontrak Solidity Anda harus memanggil fungsi `sendMessageToL2` dari kontrak `StarknetMessaging`. Untuk menerima pesan-pesan ini di Starknet, Anda perlu menandai fungsi-fungsi yang dapat dipanggil dari L1 dengan atribut `#[l1_handler]`.

Mari kita lihat sebuah kontrak sederhana yang diambil dari [tutorial ini](https://github.com/glihm/starknet-messaging-dev/blob/main/solidity/src/ContractMsg.sol) di mana kita ingin mengirim pesan ke Starknet.
`_snMessaging` adalah variabel state yang sudah diinisialisasi dengan Address kontrak `StarknetMessaging`. Anda dapat memeriksa Address-Address tersebut [di sini](https://docs.starknet.io/documentation/tools/important_addresses/).

```js
// Mengirim pesan di Starknet dengan satu nilai felt.
function sendMessageFelt(
    uint256 contractAddress,
    uint256 selector,
    uint256 myFelt
)
    external
    payable
{
    // Kami "serialize" nilai felt ke dalam payload, yang merupakan array uint256.
    uint256[] memory payload = new uint256[](1);
    payload[0] = myFelt;

    // msg.value harus selalu >= 20_000 wei.
    _snMessaging.sendMessageToL2{value: msg.value}(
        contractAddress,
        selector,
        payload
    );
}
```

Fungsi ini mengirim pesan dengan satu nilai felt ke kontrak `StarknetMessaging`.
Harap dicatat bahwa jika Anda ingin mengirim data yang lebih kompleks, Anda dapat melakukannya. Hanya perhatikan bahwa kontrak cairo Anda hanya akan memahami tipe data `felt252`. Jadi Anda harus memastikan bahwa serialisasi data Anda ke dalam array `uint256` mengikuti skema serialisasi cairo.

Penting untuk dicatat bahwa kita memiliki `{value: msg.value}`. Faktanya, nilai minimum yang harus kami kirim di sini adalah `20k wei`, karena kontrak `StarknetMessaging` akan mendaftarkan
hash pesan kita dalam penyimpanan Ethereum.

Selain dari `20k wei` itu, karena `L1HandlerTransaction` yang akan dieksekusi oleh sequencer tidak terikat pada akun mana pun (pesan berasal dari L1), Anda juga harus memastikan
bahwa Anda membayar cukup biaya di L1 agar pesan Anda dapat didekripsi dan diproses di L2.

Biaya dari `L1HandlerTransaction` dihitung secara regular seperti halnya untuk transaksi `Invoke`. Untuk ini, Anda dapat memperkirakan konsumsi gas menggunakan `starkli` atau `snforge` untuk mengetahui biaya eksekusi pesan Anda.

Tanda tangan dari `sendMessageToL2` adalah:

```js
function sendMessageToL2(
        uint256 toAddress,
        uint256 selector,
        uint256[] calldata payload
    ) external override returns (bytes32);
```

Parameter-parameter tersebut adalah sebagai berikut:

- `toAddress`: Address kontrak di L2 yang akan dipanggil.
- `selector`: Selektor dari fungsi kontrak ini di `toAddress`. Selektor (fungsi) ini harus memiliki atribut `#[l1_handler]` agar dapat dipanggil.
- `payload`: Payload selalu berupa array `felt252` (yang direpresentasikan oleh `uint256` dalam Solidity). Untuk alasan ini, kami telah memasukkan input `myFelt` ke dalam array.
  Inilah sebabnya mengapa kita perlu memasukkan data input ke dalam sebuah array.

Di sisi Starknet, untuk menerima pesan ini, kita memiliki:

```rust
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_04_L1_L2_messaging/src/lib.cairo:felt_msg_handler}}
```

Kita perlu menambahkan atribut `#[l1_handler]` ke dalam fungsi kita. Handler L1 adalah fungsi khusus yang hanya dapat dieksekusi oleh `L1HandlerTransaction`. Tidak ada yang perlu dilakukan secara khusus untuk menerima transaksi dari L1, karena pesan tersebut secara otomatis diteruskan oleh sequencer. Pada fungsi `#[l1_handler]` Anda, penting untuk memverifikasi pengirim pesan L1 untuk memastikan bahwa kontrak kita hanya dapat menerima pesan dari kontrak L1 yang terpercaya.

## Mengirim pesan dari Starknet ke Ethereum

Ketika mengirim pesan dari Starknet ke Ethereum, Anda harus menggunakan syscall `send_message_to_l1` dalam kontrak Cairo Anda. Syscall ini memungkinkan Anda untuk mengirim pesan ke kontrak `StarknetMessaging` di L1. Berbeda dengan pesan `L1->L2`, pesan `L2->L1` harus dikonsumsi secara manual, yang berarti bahwa Anda perlu kontrak Solidity Anda untuk memanggil fungsi `consumeMessageFromL2` dari kontrak `StarknetMessaging` secara eksplisit untuk mengonsumsi pesan tersebut.

Untuk mengirim pesan dari L2 ke L1, apa yang akan kita lakukan di Starknet adalah:

```rust
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_04_L1_L2_messaging/src/lib.cairo:felt_msg_send}}
```

Kami hanya membangun payload dan meneruskannya, bersama dengan Address kontrak L1, ke fungsi syscall.

Pada L1, bagian penting adalah untuk membangun payload yang sama seperti pada L2. Kemudian Anda memanggil `consumeMessageFromL2` dengan melewati Address kontrak L2 dan payload.
Harap diperhatikan bahwa Address kontrak L2 yang diharapkan oleh `consumeMessageFromL2` adalah Address kontrak dari akun yang mengirim transaksi di L2,
dan bukan Address kontrak yang menjalankan `send_message_to_l1_syscall`.

```js
function consumeMessageFelt(
    uint256 fromAddress,
    uint256[] calldata payload
)
    external
{
    let messageHash = _snMessaging.consumeMessageFromL2(fromAddress, payload);

    // Anda dapat menggunakan hash pesan di sini jika diinginkan.

    // Kami mengharapkan payload hanya berisi nilai felt252 (yang merupakan uint256 dalam Solidity).
    require(payload.length == 1, "Payload tidak valid");

    uint256 my_felt = payload[0];

    // Dari sini, Anda dapat dengan aman menggunakan `my_felt` karena pesan telah diverifikasi oleh StarknetMessaging.
    require(my_felt > 0, "Nilai tidak valid");
}
```

Seperti yang terlihat, dalam konteks ini, kita tidak perlu memverifikasi kontrak mana dari L2 yang mengirim pesan. Namun, kita sebenarnya menggunakan `consumeMessageFromL2` untuk memvalidasi input (Address pengirim di L2 dan payload) untuk memastikan kita hanya mengonsumsi pesan yang valid.

Penting untuk diingat bahwa di L1 kita mengirim payload `uint256`, namun tipe data dasar di Starknet adalah `felt252`; namun, `felt252` sekitar 4 bit lebih kecil dari `uint256`. Jadi kita harus memperhatikan nilai yang terkandung dalam payload dari pesan yang kita kirim. Jika, di L1, kita membangun pesan dengan nilai di atas maksimum `felt252`, pesan tersebut akan terjebak dan tidak pernah dikonsumsi di L2.

## Cairo Serde

Sebelum mengirim pesan antara L1 dan L2, Anda harus ingat bahwa kontrak Starknet, yang ditulis dalam Cairo, hanya dapat memahami data yang telah diserialkan. Dan data yang diserialkan selalu berupa array `felt252`.
Pada Solidity, kita memiliki tipe `uint256`, dan `felt252` sekitar 4 bit lebih kecil dari `uint256`. Jadi kita harus memperhatikan nilai-nilai yang terkandung dalam payload pesan yang kita kirim.
Jika, di L1, kita membangun pesan dengan nilai di atas maksimum `felt252`, pesan tersebut akan terjebak dan tidak pernah dikonsumsi di L2.

Sebagai contoh, nilai `uint256` sebenarnya dalam Cairo direpresentasikan oleh sebuah struktur seperti:

```rust,does_not_compile
struct u256 {
    rendah: u128,
    tinggi: u128,
}
```

yang akan diserialkan sebagai **DUA** felt, satu untuk `rendah` dan satu untuk `tinggi`. Ini berarti untuk mengirim hanya satu `u256` ke Cairo, Anda harus mengirim payload dari L1 dengan **DUA** nilai.

```js
uint256[] memory payload = new uint256[](2);
// Mari kirim nilai 1 sebagai u256 di Cairo: rendah = 1, tinggi = 0.
payload[0] = 1;
payload[1] = 0;
```

Jika Anda ingin mempelajari lebih lanjut tentang mekanisme pengiriman pesan, Anda dapat mengunjungi [dokumentasi Starknet](https://docs.starknet.io/documentation/architecture_and_concepts/Network_Architecture/messaging-mechanism/).

Anda juga dapat menemukan [panduan terperinci di sini](https://github.com/glihm/starknet-messaging-dev) untuk menguji pengiriman pesan secara lokal.
