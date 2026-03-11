Halo teman-teman!

Pernahkah kamu bertanya:  
**“Bagaimana Node.js bisa menangani ribuan request sekaligus—padahal cuma pakai satu thread?”**

Jawabannya ada pada **Event Loop**—mekanisme ajaib di balik Node.js yang membuatnya **non-blocking**, cepat, dan sangat efisien. Tapi jangan khawatir: ini bukan sihir, bukan sulap, dan bukan sesuatu yang hanya dimengerti ilmuwan. Ini adalah sistem logis yang bisa kamu pahami—dan bahkan **ujicoba sendiri dalam 5 menit**.

Mari kita kupas **apa itu Event Loop**, bagaimana ia bekerja, dan **kita akan lihat buktinya lewat kode nyata yang bisa kamu jalankan sekarang juga!**

---

### 🔍 **Apa Itu Event Loop? (Penjelasan Sederhana)**

Bayangkan kamu di restoran:
- Kamu pesan makanan → pelayan catat → kirim ke dapur.
- Tapi pelayan **tidak nunggu** sampai masakan matang.  
- Ia langsung layani pelanggan berikutnya.
- Saat makanan siap, koki teriak → pelayan ambil → antar ke meja.

Itulah **Event Loop**:  
Node.js tidak menunggu operasi lama (seperti baca file, koneksi database, atau API eksternal) selesai. Ia **menyimpan callback-nya**, lanjut ke tugas berikutnya, lalu **menjalankan callback itu saat operasi selesai**.

Dan semua ini terjadi dalam **satu thread utama**, tapi tetap bisa menangani banyak operasi bersamaan—karena operasi I/O (input/output) di-handle oleh sistem operasi di latar belakang.

---

### 🔧 **Langkah 1: Buat File Demo Event Loop**

Buka terminal, buat folder baru:

```bash
mkdir event-loop-demo
cd event-loop-demo
```

Buat file:

```bash
# macOS/Linux:
touch demo.js

# Windows:
type nul > demo.js
```

---

### 📄 **Langkah 2: Tulis Kode untuk Mengamati Event Loop**

Buka `demo.js`, lalu salin kode berikut:

```js
console.log('✅ [1] Mulai eksekusi');

// Operasi sinkron (langsung jalan)
console.log('✅ [2] Ini kode sinkron');

// setTimeout = operasi async (masuk ke "callback queue")
setTimeout(() => {
  console.log('⏰ [5] Timeout 0 detik selesai');
}, 0);

// Promise = microtask (prioritas lebih tinggi dari setTimeout)
Promise.resolve().then(() => {
  console.log('⚡ [4] Promise selesai');
});

// Operasi sinkron lagi
console.log('✅ [3] Akhir kode sinkron');

// Simulasi beban CPU (blocking!)
// for (let i = 0; i < 1e9; i++) {} // Hati-hati: ini bikin lag!
```

---

### ▶️ **Langkah 3: Jalankan Kode & Amati Urutannya**

Di terminal, jalankan:

```bash
node demo.js
```

**Output yang muncul:**
```
✅ [1] Mulai eksekusi
✅ [2] Ini kode sinkron
✅ [3] Akhir kode sinkron
⚡ [4] Promise selesai
⏰ [5] Timeout 0 detik selesai
```

> ❗ Perhatikan:  
> - Meski `setTimeout` diatur `0` detik, ia **tidak jalan duluan**!  
> - `Promise.then()` jalan **sebelum** `setTimeout`—karena microtask diproses dulu oleh Event Loop.

---

### 🔄 **Bagaimana Event Loop Bekerja? (Siklus Sederhana)**

Event Loop punya beberapa tahap (fase), tapi yang paling penting untuk pemula:

1. **Eksekusi kode sinkron** → jalan langsung, blokir thread.
2. **Microtasks** → seperti `Promise.then()`, `queueMicrotask()`.  
   → Diproses **setelah setiap operasi sinkron**, sebelum lanjut ke fase berikutnya.
3. **Callback Queue** → seperti `setTimeout`, `fs.readFile`, `http request`.  
   → Diproses **setelah microtasks selesai**.

Jadi urutan prioritas:  
**Sinkron → Microtasks → Callbacks**

---

### 🧪 **Langkah 4: Uji dengan Operasi I/O Asli (File System)**

Ubah `demo.js` jadi seperti ini:

```js
const fs = require('fs');

console.log('[1] Mulai');

// Async: baca file (non-blocking)
fs.readFile('package.json', 'utf8', (err, data) => {
  if (err) return console.log('❌ File tidak ditemukan');
  console.log('[4] File selesai dibaca');
});

// Sinkron: jalan langsung
console.log('[2] Melanjutkan eksekusi...');

// Microtask
Promise.resolve().then(() => {
  console.log('[3] Promise selesai');
});

console.log('[2.5] Selesai bagian sinkron');
```

> 💡 Pastikan ada file `package.json` di folder yang sama. Jika belum, buat dulu:
> ```bash
> npm init -y
> ```

Jalankan:
```bash
node demo.js
```

**Output kira-kira:**
```
[1] Mulai
[2] Melanjutkan eksekusi...
[2.5] Selesai bagian sinkron
[3] Promise selesai
[4] File selesai dibaca
```

> 🔥 Ini bukti nyata:  
> - `fs.readFile` **tidak menghentikan** eksekusi!  
> - Kode terus jalan → baru callback-nya dipanggil saat file selesai dibaca.

---

### ⚠️ **Peringatan Penting: Jangan Blokir Event Loop!**

Event Loop **sangat cepat**—tapi **bisa macet** jika kamu pakai operasi sinkron berat, seperti:

```js
// ❌ JANGAN LAKUKAN INI DI SERVER PRODUKSI!
for (let i = 0; i < 1e10; i++) { /* loop berat */ }
```

Karena Node.js single-threaded, **semua request akan nunggu** sampai loop ini selesai. Itu disebut **blocking the event loop**—dan musuh utama performa!

Solusinya:  
- Gunakan operasi **async** (seperti `fs.promises`),  
- Atau pindahkan beban berat ke **worker threads**.

---

### **Kesimpulannya**

Event Loop bukan konsep abstrak—ia adalah **mesin nyata** yang membuat Node.js unggul dalam menangani I/O intensif. Dengan memahami bahwa:
- Kode sinkron jalan dulu,
- Lalu microtasks (`Promise`),
- Lalu callbacks (`setTimeout`, I/O),

…kamu bisa menulis kode yang **efisien, responsif, dan mudah diprediksi**.

Dan yang paling keren?  
Kamu **baru saja melihat Event Loop bekerja dengan matamu sendiri**—hanya dengan beberapa baris kode dan perintah `node`.

Karena di dunia Node.js, **kecepatan bukan soal banyak thread… tapi soal tidak pernah menunggu**. Dan itulah keajaiban Event Loop. 🌀
