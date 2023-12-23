# Lampiran F: Memasang Biner Cairo

Jika Anda ingin mengakses biner Cairo, untuk hal-hal yang tidak dapat Anda capai hanya dengan menggunakan Scarb, Anda dapat menginstalnya dengan mengikuti petunjuk di bawah ini.

Langkah pertama adalah menginstal Cairo. Kami akan mengunduh Cairo secara manual, menggunakan repositori cairo atau dengan skrip instalasi. Anda akan memerlukan koneksi internet untuk mengunduhnya.

### Persyaratan

Pertama-tama, Anda harus memiliki Rust dan Git terinstal.

```bash
# Instal Rust stabil
rustup override set stable && rustup update
```

Instal [Git](https://git-scm.com/).

## Memasang Cairo dengan Skrip ([Installer](https://github.com/franalgaba/cairo-installer) oleh [Fran](https://github.com/franalgaba))

### Instal

Jika Anda ingin menginstal rilis tertentu dari Cairo daripada yang terbaru, atur variabel lingkungan `CAIRO_GIT_TAG` (misalnya `export CAIRO_GIT_TAG=v2.2.0`).

```bash
curl -L https://github.com/franalgaba/cairo-installer/raw/main/bin/cairo-installer | bash
```

Setelah menginstal, ikuti [instruksi ini](#set-up-your-shell-environment-for-cairo) untuk menyiapkan lingkungan shell Anda.

### Perbarui

```
rm -fr ~/.cairo
curl -L https://github.com/franalgaba/cairo-installer/raw/main/bin/cairo-installer | bash
```

### Hapus Instalasi

Cairo terinstal di dalam `$CAIRO_ROOT` (default: ~/.cairo). Untuk menghapusnya, cukup hapus:

```bash
rm -fr ~/.cairo
```

kemudian hapus tiga baris ini dari .bashrc:

```bash
export PATH="$HOME/.cairo/target/release:$PATH"
```

dan terakhir, restart shell Anda:

```bash
exec $SHELL
```

### Menyiapkan Lingkungan Shell Anda untuk Cairo

- Tetapkan variabel lingkungan `CAIRO_ROOT` untuk menunjukkan path tempat
  Cairo akan menyimpan datanya. `$HOME/.cairo` adalah defaultnya.
  Jika Anda menginstal Cairo melalui Git checkout, kami sarankan
  untuk menetapkannya ke lokasi yang sama dengan tempat Anda mengclonenya.
- Tambahkan `cairo-*` eksekutor ke `PATH` Anda jika belum ada di sana

Pengaturan di bawah ini seharusnya berfungsi untuk sebagian besar pengguna untuk penggunaan umum.

- Untuk **bash**:

  File startup Bash bervariasi secara luas antara distribusi dalam hal yang mana di antara mereka mengambil sumber daya,
  dalam keadaan apa, dalam urutan apa, dan konfigurasi tambahan apa yang mereka lakukan.
  Oleh karena itu, cara yang paling dapat diandalkan untuk mendapatkan Cairo di semua lingkungan adalah dengan menambahkan perintah konfigurasi Cairo
  ke kedua `.bashrc` (untuk shell interaktif)
  dan file profil yang akan digunakan Bash (untuk shell login).

  Pertama, tambahkan perintah ke `~/.bashrc` dengan menjalankan perintah berikut di terminal Anda:

  ```bash
  echo 'export CAIRO_ROOT="$HOME/.cairo"' >> ~/.bashrc
  echo 'command -v cairo-compile >/dev/null || export PATH="$CAIRO_ROOT/target/release:$PATH"' >> ~/.bashrc
  ```

  Kemudian, jika Anda memiliki `~/.profile`, `~/.bash_profile`, atau `~/.bash_login`, tambahkan perintah di sana juga.
  Jika Anda tidak memiliki yang ini, tambahkan ke `~/.profile`.

  - untuk menambahkan ke `~/.profile`:

    ```bash
    echo 'export CAIRO_ROOT="$HOME/.cairo"' >> ~/.profile
    echo 'command -v cairo-compile >/dev/null || export PATH="$CAIRO_ROOT/target/release:$PATH"' >> ~/.profile
    ```

  - untuk menambahkan ke `~/.bash_profile`:
    ```bash
    echo 'export CAIRO_ROOT="$HOME/.cairo"' >> ~/.bash_profile
    echo 'command -v cairo-compile >/dev/null || export PATH="$CAIRO_ROOT/target/release:$PATH"' >> ~/.bash_profile
    ```

- Untuk **Zsh**:

  ```zsh
  echo 'export CAIRO_ROOT="$HOME/.cairo"' >> ~/.zshrc
  echo 'command -v cairo-compile >/dev/null || export PATH="$CAIRO_ROOT/target/release:$PATH"' >> ~/.zshrc
  ```

  Jika Anda ingin mendapatkan Cairo di shell login non-interaktif juga, tambahkan perintah ke `~/.zprofile` atau `~/.zlogin`.

- Untuk **Fish shell**:

  Jika Anda memiliki Fish 3.2.0 atau yang lebih baru, jalankan ini secara interaktif:

  ```fish
  set -Ux CAIRO_ROOT $HOME/.cairo
  fish_add_path $CAIRO_ROOT/target/release
  ```

  Atau, jalankan potongan kode di bawah ini:

  ```fish
  set -Ux CAIRO_ROOT $HOME/.cairo
  set -U fish_user_paths $CAIRO_ROOT/target/release $fish_user_paths
  ```

Di MacOS, Anda mungkin juga ingin menginstal [Fig](https://fig.io/) yang
menyediakan penyelesaian shell alternatif untuk banyak alat baris perintah dengan
antarmuka popup mirip IDE di jendela terminal.
(Catat bahwa penyelesaian mereka independen dari basis kode Cairo
jadi mereka mungkin sedikit tidak sinkron untuk perubahan antarmuka yang sangat baru.)

### Restart shell Anda

agar perubahan pada `PATH` berlaku.

```sh
exec "$SHELL"
```

## Menginstal Cairo Secara Manual ([Panduan](https://github.com/auditless/cairo-template) oleh [Abdel](https://github.com/abdelhamidbakhta))

### Langkah 1: Instalasi Cairo 1.0

Jika Anda menggunakan sistem Linux x86 dan dapat menggunakan biner rilis, unduh Cairo di sini: <https://github.com/starkware-libs/cairo/releases>.

Untuk yang lain, kami sarankan untuk mengompilasi Cairo dari sumber sebagai berikut:

```bash
# Mulai dengan menetapkan variabel lingkungan CAIRO_ROOT
export CAIRO_ROOT="${HOME}/.cairo"

# Buat folder .cairo jika belum ada
mkdir $CAIRO_ROOT

# clone compiler Cairo ke $CAIRO_ROOT (root default)
cd $CAIRO_ROOT && git clone git@github.com:starkware-libs/cairo.git .

# OPSIONAL/DIANJURKAN: Jika Anda ingin menginstal versi tertentu dari compiler
# Dapatkan semua tag (versi)
git fetch --all --tags
# Lihat tag (Anda juga bisa melakukan ini di repositori compiler cairo)
git describe --tags `git rev-list --tags`
# Pilih versi yang Anda inginkan
git checkout tags/v2.2.0

# Hasilkan biner rilis
cargo build --all --release
```

.

**CATATAN: Menjaga Cairo tetap terbaru**

Sekarang bahwa compiler Cairo Anda berada di repositori yang di-clone, yang perlu Anda lakukan
hanya menarik perubahan terbaru dan membangun kembali sebagai berikut:

```bash
cd $CAIRO_ROOT && git fetch && git pull && cargo build --all --release
```

### Langkah 2: Tambahkan eksekutor Cairo 1.0 ke path Anda

```bash
export PATH="$CAIRO_ROOT/target/release:$PATH"
```

**CATATAN: Jika menginstal dari biner Linux, sesuaikan path tujuan.**

### Langkah 3: Menyiapkan Server Bahasa

#### Ekstensi VS Code

- Jika Anda memiliki ekstensi Cairo 0 sebelumnya terinstal, Anda dapat menonaktifkannya/menghapusnya.
- Instal ekstensi Cairo 1 untuk penyorotan sintaks yang tepat dan navigasi kode. Anda dapat menemukan tautan ke ekstensi [di sini](https://marketplace.visualstudio.com/items?itemName=starkware.cairo1&ssr=false), atau cukup cari "Cairo 1.0" di pasar VS Code.
- Ekstensi akan langsung berfungsi setelah Anda menginstal [Scarb](./ch01-03-hello-scarb.md).

#### Cairo Language Server tanpa Scarb

Jika Anda tidak ingin bergantung pada Scarb, Anda masih dapat menggunakan Cairo Language Server dengan biner compiler.
Dari [Langkah 1](#installing-cairo-with-a-script-installer-by-fran), biner `cairo-language-server` harus dibangun dan menjalankan perintah ini akan menyalin jalurnya ke clipboard Anda.

```bash
which cairo-language-server | pbcopy
```

Perbarui `cairo1.languageServerPath` dari ekstensi Cairo 1.0 dengan menempelkan path tersebut.