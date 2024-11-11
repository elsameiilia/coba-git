# Responsi 1 Praktikum Pemrograman Web I : Paket Risdat
Elsa Meilia Pusparani
H1D023092
SHIFT_B (lama)
SHIFT_D (baru)

## 1. 
a. Membuat Database dan tabel user
```sql
CREATE TABLE user (
username varchar(100) NOT NULL,
password varchar(255) NOT NULL
);
INSERT INTO user (username, password) VALUES ('tes', MD5('tes123'));
```
Disini kita membuat sebuah database bernama coba-ionic lalu membuat tabel yang bernama user
untuk tabel user itu terdiri dari dua data
- username = untuk data username akun
- password = untuk password akun

b. koneksi.php
```php
<?php
header('Access-Control-Allow-Origin: *');
header('Access-Control-Allow-Credentials: true');
header('Access-Control-Allow-Methods: PUT, GET, HEAD, POST, DELETE, OPTIONS');
header('Content-Type: application/json; charset=utf-8');
header('Access-Control-Allow-Headers: X-Requested-With, Content-Type, Origin, Authorization, Accept, Client-Security-Token, Accept-Encoding');
$con = mysqli_connect('localhost', 'root', '', 'coba-ionic') or die("koneksi gagal");
```
Disini dibuat koneksi.php untuk menyambungkan aplikasi kita ke database

c. login.php
```php
<?php
require 'koneksi.php';
$input = file_get_contents('php://input');
$data = json_decode($input, true);
$pesan = [];
$username = trim($data['username']);
$password = md5(trim($data['password']));
$query = mysqli_query($con, "select * from user where username='$username' and
password='$password'");
$jumlah = mysqli_num_rows($query);
if ($jumlah != 0) {
    $value = mysqli_fetch_object($query);
    $pesan['username'] = $value->username;
    $pesan['token'] = time() . '_' . $value->password;
    $pesan['status_login'] = 'berhasil';
} else {
    $pesan['status_login'] = 'gagal';
}
echo json_encode($pesan);
echo mysqli_error($con);
```
login.php ini berisi kode untuk:
- file_get_contents('php://input') mengambil data JSON dari permintaan HTTP, yang kemudian di-decode menjadi array PHP dengan json_decode.
- $username dan $password diperoleh dari data yang diambil, dengan password yang di-hash lagi menggunakan MD5.
- Pengecekan melalui Query memeriksa apakah ada pengguna dengan username dan password yang cocok.
- Jika ada hasil ($jumlah != 0), artinya pengguna berhasil login.
- Jika login berhasil, API akan mengirimkan respons JSON yang berisi username, token, dan status_login sebagai 'berhasil'.
- Jika gagal, hanya status_login akan dikembalikan dengan nilai 'gagal'dan kesalahan pada query akan ditampilkan

## 2. Pembuatan Fitur Login di IONIC
- Deklarasikan provideHttpClient agar aplikasi dapat menggunakan API pada app/app.module.ts
```typescript
@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, IonicModule.forRoot(), AppRoutingModule],
  providers: [{ provide: RouteReuseStrategy, useClass: IonicRouteStrategy }, provideHttpClient()],
  bootstrap: [AppComponent],
})
```
- Pembuatan service untuk authentication untuk menangani proses autentikasi yang didalamnya ada beberapa kegunaan:
  1. Injeksi Dependensi: HttpClient untuk permintaan API dan AlertController untuk notifikasi.
  2. Status Autentikasi: Menyimpan status login dengan BehaviorSubject dan Observable.
  3. Simpan Data (saveData): Menyimpan token dan nama pengguna ke penyimpanan lokal (Preferences) dan mengatur status autentikasi ke true.
  4. Load Data (loadData): Memuat data login dari penyimpanan lokal untuk memeriksa status login.
  5. Clear Data (clearData): Menghapus token dan nama pengguna saat logout.
  6. Post Method (postMethod): Mengirim data ke API dengan permintaan HTTP POST.
  7. Notifikasi (notifikasi): Menampilkan pesan notifikasi ke pengguna.
  8. API URL (apiURL): Mengembalikan URL dasar untuk API.
  9. Logout (logout): Mengatur status autentikasi ke false dan menghapus data login

- Penambahan authGuard dan autoLoginGuard yang berfungsi untuk mengontrol akses pengguna ke halaman tertentu dalam aplikasi berdasarkan status autentikasi 
  1. authGuard:
     - Fungsi: Memastikan bahwa hanya pengguna yang telah terautentikasi yang dapat mengakses halaman tertentu.
     - Cara Kerja:
        Guard ini memeriksa status autentikasi dari AuthenticationService. Jika pengguna sudah login (isAuthenticated adalah true), guard akan mengizinkan akses ke halaman. Jika pengguna belum login (isAuthenticated adalah false), guard akan mengarahkan pengguna ke halaman login (/login) untuk memastikan keamanan.
  2. autoLoginGuard:
     - Fungsi: Mencegah pengguna yang sudah login mengakses halaman seperti halaman login atau halaman pendaftaran.
     - Cara Kerja:
        Jika pengguna sudah terautentikasi (isAuthenticated adalah true), guard ini akan mengarahkan mereka langsung ke halaman beranda (/home), mencegah mereka mengakses halaman login atau halaman yang hanya diperlukan untuk pengguna yang belum login.
        Jika pengguna belum login, mereka tetap dapat mengakses halaman yang dilindungi guard ini.

- Pembuatan Login Page dimana login page ini digunakan untuk tampilan login pagenya seperti form untuk mengambil data inputan
  1. Mengecek apakah username dan password tidak kosong.
  2. Mengirim data autentikasi ke server menggunakan postMethod dari AuthenticationService.
  3. Jika login berhasil (status_login == "berhasil"), token dan username disimpan menggunakan saveData, dan pengguna diarahkan ke halaman utama (/home).
  4. Jika gagal, pesan kesalahan ditampilkan menggunakan notifikasi.

- Pembuatan Home Page dimana home page ini digunakan untuk tampilan yang akan ditampilkan setelah halaman login
  1. Property nama: Menyimpan nama pengguna yang diambil dari AuthenticationService.
  2. Fungsi logout(): Menghapus status autentikasi pengguna dengan memanggil metode logout pada AuthenticationService, lalu mengarahkan pengguna kembali ke halaman login (/login).

## Alur Login

1. **Pengguna Mengakses Halaman Login**
   - Pengguna memasukkan **username** dan **password**, kemudian menekan tombol **Login**.
   - Aplikasi akan memvalidasi apakah kedua input ini telah diisi.

2. **Proses Autentikasi**
   - Fungsi `login()` pada komponen `LoginPage` mengirimkan data `username` dan `password` ke server API menggunakan `postMethod` dari `AuthenticationService`.
   - Server memverifikasi data login dan, jika berhasil, mengembalikan token autentikasi beserta nama pengguna.
   - Jika login berhasil:
     - Token dan nama pengguna disimpan di penyimpanan lokal melalui fungsi `saveData` pada `AuthenticationService`.
     - Status autentikasi diperbarui ke `true`, dan pengguna diarahkan ke halaman utama (`/home`).
   - Jika login gagal:
     - Aplikasi menampilkan notifikasi pesan kesalahan, seperti "Username atau Password Salah".

3. **Halaman Utama (Home Page)**
   - Setelah berhasil login, pengguna diarahkan ke halaman **Home**.
   - Halaman ini menyambut pengguna dengan pesan "Selamat datang, [nama pengguna]" yang diambil dari `AuthenticationService`.
   - Pengguna dapat logout dengan menekan tombol **Logout** di halaman ini.

4. **Logout**
   - Saat pengguna menekan tombol **Logout**:
     - Fungsi `logout()` akan memanggil metode `clearData` pada `AuthenticationService` untuk menghapus token dan nama pengguna dari penyimpanan.
     - Status autentikasi diperbarui ke `false`, dan pengguna diarahkan kembali ke halaman login (`/login`).

## Screenshoot
<img src="1.png" width="300px">
<img src="2.png" width="300px">
<img src="3.png" width="300px">
<img src="4.png" width="300px">
