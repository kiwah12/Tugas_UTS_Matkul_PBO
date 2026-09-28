# Sistem Manajemen Toko Madura

Tugas UTS Mata Kuliah **Pemrograman Berorientasi Objek**

| **Nama** Muhammad Risky Alpianur | **NIM** 2509116101 |

---

## 2. Deskripsi Proyek

Toko Madura adalah toko kelontong yang buka 24 jam dan menjual bermacam barang
kebutuhan harian. Pencatatan barang yang masih manual membuat pemilik toko sulit
mengetahui stok yang menipis, harga jual yang seharusnya, serta rekap penjualan.

Program ini dibuat untuk membantu pemilik toko mengelola data barang dagangan.Barang di toko dikelompokkan menjadi empat jenis yang
punya karakteristik dan aturan harga berbeda:

| Jenis | Atribut khusus | Aturan harga jual |
|---|---|---|
| Makanan | tanggal kedaluwarsa, kemasan/curah | harga beli + margin 15% |
| Minuman | volume (ml), dingin/suhu ruang | harga beli + margin 20% + Rp1.000 bila dingin |
| Minuman Bersoda | (turunan Minuman) kadar gula (gram) | harga Minuman + cukai berpemanis Rp500 |
| Rokok | merek, isi per bungkus | harga beli + margin 10% + cukai Rp250/batang, minimal usia pembeli 18 tahun, tidak boleh diskon |

Semua harga jual dibulatkan ke atas ke kelipatan Rp100.

### Fitur Program

1. **Create**: tambah produk baru (Makanan / Minuman / Rokok / Minuman Bersoda).
2. **Read**: tampilkan seluruh produk dalam tabel, lihat detail satu produk, dan cari produk
   berdasarkan **nama/kode** atau berdasarkan **rentang harga**.
3. **Update**: ubah nama, harga beli, stok, serta atribut khusus tiap kategori.
4. **Delete**: hapus produk dengan konfirmasi.
5. **Transaksi penjualan**: keranjang multi-barang, diskon, validasi usia pembeli rokok,
   stok berkurang setelah pembayaran berhasil, dan cetak struk beserta kembalian.
6. **Restock**: tambah stok produk (dengan catatan opsional).
7. **Laporan toko**: jumlah produk per kategori, nilai persediaan, total pendapatan,
   dan daftar produk yang perlu di-restock.

---

## 3. Diagram Kelas & Hierarki Class

```mermaid
classDiagram
    class Produk {
        <<abstract>>
        -String kode
        -String nama
        -double hargaBeli
        -int stok
        +getKategori()* String
        +hitungHargaJual()* double
        +hitungHargaJual(double diskon) double
        +getInfoTambahan()* String
        +tambahStok(int) void
        +tambahStok(int, String) void
        +kurangiStok(int) boolean
        +tampilkanDetail() void
    }
    class Makanan {
        -String tanggalKadaluarsa
        -boolean kemasan
        +hitungHargaJual() double
    }
    class Minuman {
        -int volumeMl
        -boolean dingin
        +hitungHargaJual() double
    }
    class MinumanBersoda {
        -int kadarGulaGram
        +hitungHargaJual() double
    }
    class Rokok {
        -String merek
        -int jumlahBatang
        +hitungHargaJual() double
        +hitungHargaJual(double) double
        +bolehDibeli(int) boolean
    }
    class DataToko {
        -List~Produk~ daftarProduk
        +tambahProduk(Produk) boolean
        +cariByKode(String) Produk
        +cariProduk(String) List
        +cariProduk(double, double) List
        +ubahProduk(...) boolean
        +hapusProduk(String) boolean
        +jualProduk(String, int) double
        +jualProduk(String, int, double) double
    }
    Produk <|-- Makanan
    Produk <|-- Minuman
    Produk <|-- Rokok
    Minuman <|-- MinumanBersoda
    DataToko o-- Produk
    Main ..> DataToko
```
---

## 4. Alur Program

### Cara kerja sistem

```
Mulai -> isi data awal (9 produk contoh)
   |
   v
+-> Tampilkan menu -> user memilih angka 0-9
|        |
|        |- 1 Tambah produk  -> pilih kategori -> isi data -> harga jual otomatis
|        |- 2 Daftar produk  -> tabel semua produk
|        |- 3 Detail produk  -> rincian 1 produk (berdasarkan kode)
|        |- 4 Cari produk    -> nama/kode ATAU rentang harga
|        |- 5 Ubah produk    -> ubah nama/harga beli/stok + atribut khusus
|        |- 6 Hapus produk   -> konfirmasi (y/t)
|        |- 7 Transaksi      -> isi keranjang -> hitung total -> bayar -> cetak struk
|        |- 8 Laporan toko   -> ringkasan toko & stok menipis
|        |- 9 Restock        -> tambah stok (catatan opsional)
|        +- 0 Keluar ------------------------------> Selesai
+--------+  (kembali ke menu setelah menekan ENTER)
```

## 5. Penjelasan Bagian Kode

### a. Inheritance (hierarchical): Produk diturunkan ke Makanan, Minuman, Rokok

```java
public abstract class Produk {
    private String kode;
    private String nama;
    private double hargaBeli;
    private int stok;

    public Produk(String kode, String nama, double hargaBeli, int stok) 
    public abstract String getKategori();
    public abstract double hitungHargaJual();
    public abstract String getInfoTambahan();
}
```

Kelas dibuat abstract karena "produk" hanyalah konsep umum; yang benar-benar dijual
selalu berupa makanan, minuman, atau rokok. Sub-class memanggil constructor induk dengan super:

```java
public class Minuman extends Produk {
    public Minuman(String kode, String nama, double hargaBeli, int stok,
                   int volumeMl, boolean dingin) {
        super(kode, nama, hargaBeli, stok); 
        this.volumeMl = volumeMl;      
        this.dingin = dingin;
    }
}
```

### b. Inheritance (multilevel): `Produk` → `Minuman` → `MinumanBersoda`

```java
public class MinumanBersoda extends Minuman {
    public static final double CUKAI_BERPEMANIS = 500;

    @Override
    public double hitungHargaJual() {
        return super.hitungHargaJual() + CUKAI_BERPEMANIS; 
    }
}
```

MinumanBersoda mewarisi semua sifat Minuman (yang sendirinya mewarisi Produk),
lalu menambah atribut kadarGulaGram dan aturan cukai.

### c. Polymorphism: Method Overriding (aturan harga tiap sub-class berbeda)

```java
// Makanan.java
@Override
public double hitungHargaJual() {
    double harga = getHargaBeli() * (1 + MARGIN);       
    return Math.ceil(harga / 100.0) * 100;
}

// Minuman.java
@Override
public double hitungHargaJual() {
    double harga = getHargaBeli() * (1 + MARGIN);       
    if (dingin) {
        harga += BIAYA_PENDINGINAN;               
    }
    return Math.ceil(harga / 100.0) * 100;
}

// Rokok.java
@Override
public double hitungHargaJual() {
    double harga = getHargaBeli() * (1 + MARGIN) + (CUKAI_PER_BATANG * jumlahBatang);
    return Math.ceil(harga / 100.0) * 100;       
}

@Override
public void tampilkanDetail() {
    super.tampilkanDetail();
    System.out.println("  Peringatan      : Dilarang menjual kepada anak di bawah umur!");
}
```

Objek Makanan, Minuman, MinumanBersoda, dan Rokok disimpan dalam satu `List<Produk>`.
Saat `hitungHargaJual()` dipanggil, Java otomatis menjalankan versi milik class aslinya
(*dynamic method dispatch*):

```java
for (Produk p : data) {
    System.out.println(p.toBarisTabel());  
}
```

### d. Polymorphism: Method Overloading (nama sama, parameter berbeda)

```java
// Produk.java
public double hitungHargaJual()                      
public double hitungHargaJual(double diskonPersen)  

public void tambahStok(int jumlah)                 
public void tambahStok(int jumlah, String catatan)  

// DataToko.java
public List<Produk> cariProduk(String kataKunci)               
public List<Produk> cariProduk(double hargaMin, double hargaMax)

public double jualProduk(String kode, int jumlah)                  
public double jualProduk(String kode, int jumlah, double diskonPersen)
```

### e. Condition (if-else)

```java
// Produk.java: status stok
public String getStatusStok() {
    if (stok == 0) return "HABIS";
    if (stok <= 5) return "MENIPIS";
    return "AMAN";
}

// Main.java: validasi usia hanya berlaku untuk Rokok
if (p instanceof Rokok r) {
    if (usia < 0) usia = bacaInt("Usia pembeli   : ");
    if (!r.bolehDibeli(usia)) {
        System.out.println("[!] Ditolak. Pembeli di bawah 18 tahun.");
        lolos = false;
    }
}
```

### f. Looping

```java
// Menu utama berulang sampai user memilih 0
while (jalan) {
    tampilkanMenu();
    ...
}

// Pembeli boleh menambah beberapa barang
do {
    ...
} while (bacaYaTidak("Tambah barang lain? (y/t) : "));

// Menghitung total belanja
for (ItemBelanja item : keranjang) {
    total += item.subtotal();
}
```

---

## 7. Screenshot Program Berjalan & Penjelasannya

**`Menu Utama`**

<img width="1000" height="328" alt="image" src="https://github.com/user-attachments/assets/a8ba2b35-07b6-459c-b452-48abd64076b4" />

Menu utama berisi pilihan 0-9. Program berulang (looping `while`) sampai user memilih `0`.

**`Daftar Produk` (Sebelum Tambah Produk)**

<img width="912" height="327" alt="image" src="https://github.com/user-attachments/assets/473d3b56-eb9d-4db0-a9c3-1416cf46afc0" />

Tabel semua produk. Kolom **Harga Jual** berbeda tiap jenis karena tiap sub-class meng-override
`hitungHargaJual()`. Kolom **Status** ditentukan dengan if-else: AMAN, MENIPIS (stok ≤ 5), atau HABIS.

**`Tambah Produk`**

<img width="927" height="312" alt="image" src="https://github.com/user-attachments/assets/e30d1224-aa76-4726-8149-79b9d05c7d19" />

Pengguna memilih kategori, mengisi data, lalu harga jual dihitung otomatis oleh sub-class yang sesuai.

**`Daftar Produk` (Sesudah Tambah Produk)**

<img width="1015" height="320" alt="image" src="https://github.com/user-attachments/assets/cf6ebb3f-fd23-406c-ae18-cdbad42a0c47" />

Produk baru muncul di tabel.

**`Ubah Produk`**

<img width="1167" height="617" alt="image" src="https://github.com/user-attachments/assets/1e6a273e-0774-4482-b2c4-465435e42515" />

Data diubah lewat setter (encapsulation). Atribut khusus (kadaluarsa, dingin, isi batang) ditanyakan
sesuai tipe asli objek memakai `instanceof`.

**`Hapus Produk`**

<img width="797" height="167" alt="image" src="https://github.com/user-attachments/assets/36548461-00db-4047-800e-9ed7045e9891" />

Penghapusan meminta konfirmasi (y/t) terlebih dahulu.

**`Daftar Produk` (Sebelum dihapus Produk)**

<img width="1015" height="320" alt="image" src="https://github.com/user-attachments/assets/cf6ebb3f-fd23-406c-ae18-cdbad42a0c47" />

**`Daftar Produk` (Sesudah dihapus Produk)**

<img width="980" height="325" alt="image" src="https://github.com/user-attachments/assets/6d4453e8-a598-4924-ad17-fbe2ca7a42f8" />

Produk yang dihapus sudah tidak ada di tabel.

**`Struk & Transaksi`**

<img width="1212" height="591" alt="image" src="https://github.com/user-attachments/assets/d3197fbb-a151-4537-90f3-77713d47744a" />

Transaksi penjualan: stok berkurang, total dihitung, lalu struk dicetak beserta kembalian.

**`Detail Produk Minuman Bersoda & Rokok`**

<img width="932" height="387" alt="image" src="https://github.com/user-attachments/assets/eb9e4d18-e607-417f-9574-7eabe824a163" />

**`Cari Produk (Overloading)`**

<img width="996" height="302" alt="image" src="https://github.com/user-attachments/assets/63e99513-ac6b-4603-a443-b3d454005d37" />

**`Transaksi Multi-Barang dengan Diskon & Struk`**

<img width="1403" height="882" alt="image" src="https://github.com/user-attachments/assets/ff6ba458-9914-4984-9434-519e306301a3" />


**`Rokok Ditolak untuk Pembeli di Bawah Umur`**

<img width="882" height="473" alt="image" src="https://github.com/user-attachments/assets/adce9185-3412-4f63-95f7-b31ae1c679b6" />


**`Laporan Toko`**

<img width="970" height="300" alt="image" src="https://github.com/user-attachments/assets/526341bf-1de8-4e72-8462-ef19af4234bf" />

