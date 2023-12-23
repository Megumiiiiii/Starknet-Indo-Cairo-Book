# Komponen di dalam mesin

Komponen memberikan modularitas yang kuat pada kontrak Starknet. Tetapi bagaimana sebenarnya sihir ini terjadi di balik layar?

Bab ini akan menjelajahi secara mendalam tentang internal kompilator untuk menjelaskan mekanisme yang memungkinkan komposabilitas komponen.

## Pengantar tentang Implementasi yang Dapat Ditanam

Sebelum mempelajari lebih dalam tentang komponen, kita perlu memahami _implementasi yang dapat ditanam_.

Sebuah implementasi dari trait antarmuka Starknet (ditandai dengan `#[starknet::interface]`) dapat dibuat dapat ditanam. Implementasi yang dapat ditanam dapat disisipkan ke dalam kontrak apa pun, menambahkan titik masuk baru dan memodifikasi ABI dari kontrak tersebut.

Mari kita lihat contoh untuk melihat hal ini dalam aksi:

```rust
{{#include ../listings/ch99-starknet-smart-contracts/components/no_listing_01_embeddable/src/lib.cairo}}
```

Dengan menyisipkan `SimpleImpl`, kita secara eksternal mengekspos `ret4` dalam ABI kontrak.

Sekarang setelah kita lebih familiar dengan mekanisme penyisipan, kita dapat melihat bagaimana komponen membangun dari hal ini.

## Di Dalam Komponen: Implementasi Generik

Ingatlah sintaks blok impl yang digunakan dalam komponen:

```rust
{{#include ../listings/ch99-starknet-smart-contracts/components/listing_01_ownable/src/component.cairo:impl_signature}}
```

Poin-poin kunci:

- `OwnableImpl` membutuhkan implementasi dari trait `HasComponent<TContractState>` oleh kontrak yang mendasarinya, yang secara otomatis dihasilkan dengan macro `component!()` saat menggunakan sebuah komponen di dalam sebuah kontrak.

  Kompilator akan menghasilkan sebuah impl yang membungkus setiap fungsi di dalam `OwnableImpl`, mengganti argumen `self: ComponentState<TContractState>` dengan `self: TContractState`, di mana akses ke status komponen dilakukan melalui fungsi `get_component` dalam trait `HasComponent<TContractState>`.

  Untuk setiap komponen, kompilator akan menghasilkan trait `HasComponent`. Trait ini mendefinisikan antarmuka untuk menjembatani antara `TContractState` sebenarnya dari kontrak generik, dan `ComponentState<TContractState>`.

  ```rust
  // dihasilkan per komponen
  trait HasComponent<TContractState> {
      fn get_component(self: @TContractState) -> @ComponentState<TContractState>;
      fn get_component_mut(ref self: TContractState) -> ComponentState<TContractState>;
      fn get_contract(self: @ComponentState<TContractState>) -> @TContractState;
      fn get_contract_mut(ref self: ComponentState<TContractState>) -> TContractState;
      fn emit<S, impl IntoImp: traits::Into<S, Event>>(ref self: ComponentState<TContractState>, event: S);
  }
  ```

  Dalam konteks kita, `ComponentState<TContractState>` adalah tipe yang khusus untuk komponen ownable, yaitu memiliki anggota berdasarkan variabel penyimpanan yang didefinisikan dalam `ownable_component::Storage`. Berpindah dari `TContractState` generik ke `ComponentState<TContractState>` akan memungkinkan kita untuk menyisipkan `Ownable` ke dalam kontrak apa pun yang ingin menggunakannya. Arah sebaliknya (`ComponentState<TContractState>` ke `ContractState`) berguna untuk dependensi (lihat contoh implementasi `Upgradeable` komponen yang bergantung pada implementasi `IOwnable` di bagian [Ketergantungan Komponen](./ch99-01-05-02-component-dependencies.md)).

  Singkatnya, seseorang harus memikirkan implementasi dari `HasComponent<T>` di atas sebagai mengatakan: **"Kontrak yang memiliki status T memiliki komponen yang dapat diupgrade".**

- `Ownable` dianotasi dengan atribut `embeddable_as(<nama>)`:

  `embeddable_as` mirip dengan `embeddable`; ia hanya berlaku untuk `impls` dari trait `starknet::interface` dan memungkinkan penyisipan implementasi ini di dalam modul kontrak. Dengan demikian, `embeddable_as(<nama>)` memiliki peran lain dalam konteks komponen. Pada akhirnya, ketika menyisipkan `OwnableImpl` ke dalam suatu kontrak, kita berharap mendapatkan sebuah impl dengan fungsi-fungsi berikut:

  ```rust
  {{#include ../listings/ch99-starknet-smart-contracts/components/listing_01_ownable/src/component.cairo:trait_def}}
  ```

  Perhatikan bahwa meskipun dimulai dengan sebuah fungsi yang menerima tipe generik `ComponentState<TContractState>`, kita ingin berakhir dengan sebuah fungsi yang menerima `ContractState`. Di sinilah `embeddable_as(<nama>)` berperan. Untuk melihat gambaran keseluruhan, kita perlu melihat impl yang dihasilkan oleh kompilator karena adanya anotasi `embeddable_as(Ownable)`:

  ```rust
  #[starknet::embeddable]
  impl Ownable<
            TContractState, +HasComponent<TContractState>
  , impl TContractStateDrop: Drop<TContractState>
  > of super::IOwnable<TContractState> {

    fn owner(self: @TContractState) -> ContractAddress {
        let component = HasComponent::get_component(self);
        OwnableImpl::owner(component, )
    }

    fn transfer_ownership(ref self: TContractState, new_owner: ContractAddress
  ) {
        let mut component = HasComponent::get_component_mut(ref self);
        OwnableImpl::transfer_ownership(ref component, new_owner, )
    }

    fn renounce_ownership(ref self: TContractState) {
        let mut component = HasComponent::get_component_mut(ref self);
        OwnableImpl::renounce_ownership(ref component, )
    }
  }
  ```

  Perhatikan bahwa berkat adanya impl dari `HasComponent<TContractState>`, kompilator dapat membungkus fungsi-fungsi kita dalam sebuah impl baru yang tidak secara langsung mengetahui tentang tipe `ComponentState`. `Ownable`, yang nama kita pilih saat menulis `embeddable_as(Ownable)`, adalah impl yang akan kita sisipkan di dalam sebuah kontrak yang menginginkan kepemilikan.

## Integrasi Kontrak

Kita telah melihat bagaimana implementasi generik memungkinkan kegunaan ulang komponen. Selanjutnya, mari kita lihat bagaimana sebuah kontrak mengintegrasikan sebuah komponen.

Kontrak menggunakan **alias impl** untuk menginisialisasi implementasi generik komponen dengan tipe konkret `ContractState` dari kontrak.

```rust
{{#include ../listings/ch99-starknet-smart-contracts/components/listing_01_ownable/src/contract.cairo:embedded_impl}}
```

Baris di atas menggunakan mekanisme penyisipan impl Cairo bersamaan dengan sintaks alias impl. Kita menginisialisasi `OwnableImpl<TContractState>` yang generik dengan tipe konkret `ContractState`. Ingatlah bahwa `OwnableImpl<TContractState>` memiliki parameter impl generik `HasComponent<TContractState>`. Implementasi dari trait ini dihasilkan oleh macro `component!`.

Perhatikan bahwa hanya kontrak yang menggunakan yang dapat mengimplementasikan trait ini karena hanya kontrak yang mengetahui tentang kedua status kontrak dan status komponen.

Ini menyatukan semua hal untuk menyisipkan logika komponen ke dalam kontrak.

## Simpulan Penting

- Impl yang dapat disisipkan memungkinkan penyisipan logika komponen ke dalam kontrak dengan menambahkan titik masuk dan memodifikasi ABI kontrak.
- Kompilator secara otomatis menghasilkan implementasi trait `HasComponent` saat sebuah komponen digunakan dalam sebuah kontrak. Ini menciptakan jembatan antara status kontrak dan status komponen, memungkinkan interaksi di antara keduanya.
- Komponen mengemas logika yang dapat digunakan kembali secara generik, tanpa tergantung pada kontrak tertentu.
  Kontrak mengintegrasikan komponen melalui alias impl dan mengaksesnya melalui trait `HasComponent` yang dihasilkan.
- Komponen membangun dari impl yang dapat disisipkan dengan mendefinisikan logika komponen yang generik yang dapat diintegrasikan ke dalam setiap kontrak yang ingin menggunakan komponen tersebut. Alias impl menginisialisasi impl generik ini dengan tipe penyimpanan konkret kontrak.
