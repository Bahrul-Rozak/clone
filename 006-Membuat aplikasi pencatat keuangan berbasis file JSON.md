Halo teman-teman!

Bayangkan kamu ingin membuat aplikasi sederhana untuk mencatat pemasukan dan pengeluaran—tanpa database, tanpa server eksternal, hanya bermodalkan file JSON dan Node.js. Ini bukan cuma latihan pemula, tapi contoh nyata bagaimana **backend ringan bisa sangat powerful** untuk kebutuhan pribadi, UMKM, atau prototipe cepat. Dengan memanfaatkan modul `fs` (File System) yang sudah ada di Node.js, kita bisa membuat sistem pencatatan keuangan yang menyimpan semua data ke file `keuangan.json`, lengkap dengan fitur tambah transaksi, lihat riwayat, dan hitung saldo—semua dalam satu file JavaScript.

Langkah pertama: kita butuh struktur data. Setiap transaksi akan berisi:
- `id` (unik),
- `jenis` ("pemasukan" atau "pengeluaran"),
- `jumlah` (angka),
- `deskripsi` (teks),
- `tanggal` (timestamp).

Kita simpan semua transaksi dalam array di file `keuangan.json`. Jika file belum ada, aplikasi akan membuatnya otomatis.

Berikut kode lengkap aplikasinya:

```js
const fs = require('fs').promises;
const path = require('path');

// Path file data
const DATA_FILE = path.join(__dirname, 'keuangan.json');

// Fungsi bantu: baca data (buat file jika belum ada)
async function bacaData() {
  try {
    const data = await fs.readFile(DATA_FILE, 'utf8');
    return JSON.parse(data);
  } catch (err) {
    if (err.code === 'ENOENT') {
      // File belum ada → buat baru
      await fs.writeFile(DATA_FILE, '[]', 'utf8');
      return [];
    }
    throw err;
  }
}

// Fungsi: tambah transaksi
async function tambahTransaksi(jenis, jumlah, deskripsi) {
  if (!['pemasukan', 'pengeluaran'].includes(jenis)) {
    throw new Error('Jenis harus "pemasukan" atau "pengeluaran"');
  }
  if (jumlah <= 0) {
    throw new Error('Jumlah harus lebih dari 0');
  }

  const transaksi = {
    id: Date.now().toString(), // ID sederhana berbasis timestamp
    jenis,
    jumlah,
    deskripsi,
    tanggal: new Date().toISOString()
  };

  const data = await bacaData();
  data.push(transaksi);
  await fs.writeFile(DATA_FILE, JSON.stringify(data, null, 2), 'utf8');
  console.log('✅ Transaksi berhasil ditambahkan!');
}

// Fungsi: tampilkan semua transaksi & hitung saldo
async function tampilkanLaporan() {
  const data = await bacaData();
  if (data.length === 0) {
    console.log('Belum ada transaksi.');
    return;
  }

  let totalPemasukan = 0;
  let totalPengeluaran = 0;

  console.log('\n=== RIWAYAT TRANSAKSI ===');
  data.forEach((trx) => {
    const tanda = trx.jenis === 'pemasukan' ? '+' : '-';
    console.log(`${tanda} ${trx.deskripsi} | Rp${trx.jumlah.toLocaleString()} | ${trx.tanggal.split('T')[0]}`);
    
    if (trx.jenis === 'pemasukan') totalPemasukan += trx.jumlah;
    else totalPengeluaran += trx.jumlah;
  });

  const saldo = totalPemasukan - totalPengeluaran;
  console.log('\n=== RINGKASAN ===');
  console.log(`Pemasukan: Rp${totalPemasukan.toLocaleString()}`);
  console.log(`Pengeluaran: Rp${totalPengeluaran.toLocaleString()}`);
  console.log(`Saldo: Rp${saldo.toLocaleString()}`);
}

// Contoh penggunaan
(async () => {
  try {
    // Tambah transaksi
    await tambahTransaksi('pemasukan', 1500000, 'Gaji bulanan');
    await tambahTransaksi('pengeluaran', 75000, 'Makan siang');
    await tambahTransaksi('pengeluaran', 200000, 'Bayar listrik');

    // Tampilkan laporan
    await tampilkanLaporan();
  } catch (err) {
    console.error('❌ Error:', err.message);
  }
})();
```

Mari kita bahas bagian pentingnya:

1. **Penanganan file fleksibel**: Fungsi `bacaData()` otomatis membuat file `keuangan.json` jika belum ada, sehingga aplikasi siap jalan sejak pertama kali dijalankan.
2. **Validasi input**: Sebelum menyimpan, kita pastikan jenis transaksi valid dan jumlahnya positif—ini mencegah data rusak.
3. **Format JSON rapi**: Saat menulis, kita gunakan `JSON.stringify(data, null, 2)` agar file-nya mudah dibaca manusia (ada indentasi).
4. **ID unik sederhana**: Kita pakai `Date.now()` sebagai ID—cukup unik untuk aplikasi personal (untuk skala besar, gunakan UUID).
5. **Error handling menyeluruh**: Semua operasi async dibungkus try-catch, dan error sistem file (seperti file tidak ditemukan) ditangani secara spesifik.

Aplikasi ini bisa kamu kembangkan lebih jauh:  
- Tambahkan perintah CLI dengan `process.argv` agar bisa dipanggil via terminal (`node app.js tambah pemasukan 100000 "Bonus"`),  
- Tambahkan fitur hapus transaksi berdasarkan ID,  
- Atau ekspor ke CSV dengan stream.

**Kesimpulannya**, membuat aplikasi pencatat keuangan berbasis file JSON bukan hanya latihan teknis—tapi demonstrasi nyata bagaimana **Node.js bisa menjadi alat produktivitas pribadi yang andal**. Tanpa dependensi eksternal, tanpa setup rumit, kamu punya sistem yang:  
- Menyimpan data secara persisten,  
- Aman dari crash (karena setiap operasi menulis ulang file utuh),  
- Dan bisa dikembangkan sesuai kebutuhan.  

Ini adalah fondasi yang sama digunakan oleh banyak tools developer modern—dari konfigurasi hingga cache lokal. Karena kadang, **solusi terbaik bukan yang paling canggih… tapi yang paling sederhana dan langsung jalan**.
