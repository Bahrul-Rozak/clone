Halo teman-teman!

Di dunia backend, **logging bukan sekadar “cetak pesan ke konsol”**—ia adalah mata dan telinga sistemmu saat kamu tidak sedang mengawasi. Bayangkan aplikasimu berjalan di server 24/7: siapa yang akan tahu kalau ada error? Siapa yang akan melacak aktivitas pengguna? Siapa yang akan membantumu debug saat sesuatu tiba-tiba rusak? Jawabannya: **sistem logging**.

Dan kabar baiknya? Kamu bisa membuat sistem logging sederhana tapi sangat efektif hanya dengan **Node.js dan modul `fs`**, tanpa dependensi eksternal. Sistem ini akan mencatat setiap kejadian penting—seperti error, permintaan masuk, atau aktivitas kritis—ke dalam file log yang terstruktur, aman, dan mudah dianalisis.

Berikut prinsip dasarnya:
- Setiap entri log harus punya **timestamp**, **level** (info/warning/error), **pesan**, dan **konteks** (misalnya ID pengguna).
- Log disimpan dalam file teks (misalnya `app.log`), dengan format konsisten.
- Operasi penulisan menggunakan **append**, agar data lama tidak hilang.
- Semua operasi **tidak boleh mengganggu alur utama aplikasi**—jadi harus async dan aman dari error.

Mari kita bangun sistem logging sederhana:

```js
const fs = require('fs').promises;
const path = require('path');

// Path file log
const LOG_FILE = path.join(__dirname, 'logs', 'app.log');

// Pastikan folder logs ada
async function pastikanFolderLogs() {
  try {
    await fs.access(path.dirname(LOG_FILE));
  } catch {
    await fs.mkdir(path.dirname(LOG_FILE), { recursive: true });
  }
}

// Fungsi utama: tulis log
async function log(level, message, context = {}) {
  await pastikanFolderLogs();

  const timestamp = new Date().toISOString();
  const contextStr = Object.keys(context).length > 0 
    ? ` | ${JSON.stringify(context)}` 
    : '';
  
  const logEntry = `[${timestamp}] ${level.toUpperCase()} - ${message}${contextStr}\n`;

  try {
    // Gunakan appendFile agar tidak overwrite
    await fs.appendFile(LOG_FILE, logEntry, 'utf8');
  } catch (err) {
    // Jangan biarkan error logging menghentikan aplikasi utama
    console.error('Gagal menulis log:', err.message);
  }
}

// Shortcut untuk level umum
const logger = {
  info: (msg, ctx) => log('info', msg, ctx),
  warn: (msg, ctx) => log('warn', msg, ctx),
  error: (msg, ctx) => log('error', msg, ctx)
};

module.exports = logger;
```

Sekarang, kamu bisa menggunakannya di seluruh aplikasimu:

```js
const logger = require('./logger');

// Contoh penggunaan
logger.info('Server berjalan di port 3000');
logger.warn('API eksternal lambat', { url: 'https://api.coin.com', durasi: '2500ms' });
logger.error('Gagal menyimpan ke database', { userId: '123', error: 'Koneksi timeout' });
```

Hasil di file `logs/app.log`:
```
[2026-03-02T08:15:30.123Z] INFO - Server berjalan di port 3000
[2026-03-02T08:16:05.456Z] WARN - API eksternal lambat | {"url":"https://api.coin.com","durasi":"2500ms"}
[2026-03-02T08:17:22.789Z] ERROR - Gagal menyimpan ke database | {"userId":"123","error":"Koneksi timeout"}
```

Beberapa hal penting yang membuat sistem ini andal:
1. **Folder otomatis dibuat**: Jika folder `logs` belum ada, sistem akan membuatnya (`recursive: true`).
2. **Aman dari crash**: Error saat menulis log tidak menghentikan aplikasi utama—karena ditangani di dalam `catch`.
3. **Konteks fleksibel**: Kamu bisa tambahkan data tambahan (seperti ID pengguna, URL, dll) dalam bentuk objek—otomatis diubah ke JSON.
4. **Format standar**: Timestamp ISO + level huruf besar memudahkan parsing otomatis nanti (misalnya pakai tools seperti grep atau ELK stack).

Untuk pengembangan lebih lanjut, kamu bisa:
- Tambahkan **rotasi log harian** (buat file baru tiap hari),
- Batasi ukuran file,
- Atau integrasikan dengan layanan cloud.

**Kesimpulannya**, sistem logging sederhana ini adalah fondasi tak terlihat yang menjaga aplikasimu tetap transparan dan dapat diaudit. Ia tidak mewah, tapi **menyelamatkanmu dari kebutaan operasional**. Karena di dunia backend, **apa yang tidak tercatat… sama saja tidak pernah terjadi**. Dan dengan beberapa baris kode, kamu sudah memberi aplikasimu kemampuan untuk “berbicara” saat ada yang salah—bahkan tengah malam sekalipun.
