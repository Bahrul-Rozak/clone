Halo teman-teman!

Sebelum kita bicara soal *asynchronous* atau *Event Loop*, ada satu fondasi yang **harus kamu pahami dulu**: **Call Stack**.  

Bayangkan Call Stack ini seperti **tumpukan piring di dapur**. Setiap kali kamu memanggil fungsi, itu seperti meletakkan piring baru di atas tumpukan. Saat fungsi selesai, piring itu diambil dari atas. Dan JavaScript—termasuk Node.js—hanya punya **satu tangan** untuk mengambil/menaruh piring. Artinya: **ia hanya bisa mengerjakan satu hal pada satu waktu**.

Mari kita lihat **cara kerja Call Stack secara langsung**, dengan kode nyata yang bisa kamu jalankan sendiri—step by step!

---

### 🔧 **Langkah 1: Buat File Demo Call Stack**

Buka terminal, lalu buat folder dan file:

```bash
mkdir call-stack-demo
cd call-stack-demo
```

Buat file:

```bash
# macOS/Linux:
touch demo.js

# Windows (Command Prompt):
type nul > demo.js
```

---

### 📄 **Langkah 2: Tulis Kode untuk Mengamati Call Stack**

Buka `demo.js` di editor, lalu salin kode berikut:

```js
function fungsiC() {
  console.log('Masuk fungsiC');
  console.log('Keluar fungsiC');
}

function fungsiB() {
  console.log('Masuk fungsiB');
  fungsiC(); // panggil fungsiC
  console.log('Keluar fungsiB');
}

function fungsiA() {
  console.log('Masuk fungsiA');
  fungsiB(); // panggil fungsiB
  console.log('Keluar fungsiA');
}

console.log('✅ Mulai program');
fungsiA(); // mulai dari sini
console.log('✅ Program selesai');
```

---

### ▶️ **Langkah 3: Jalankan dan Amati Urutan Eksekusi**

Di terminal, jalankan:

```bash
node demo.js
```

**Output yang muncul:**
```
✅ Mulai program
Masuk fungsiA
Masuk fungsiB
Masuk fungsiC
Keluar fungsiC
Keluar fungsiB
Keluar fungsiA
✅ Program selesai
```

> 🔍 **Apa yang terjadi di balik layar?**  
> Saat `fungsiA()` dipanggil, ia masuk ke **Call Stack**.  
> Lalu `fungsiA` panggil `fungsiB` → `fungsiB` masuk ke atas stack.  
> Lalu `fungsiB` panggil `fungsiC` → `fungsiC` masuk ke puncak stack.  
> Setelah `fungsiC` selesai, ia di-*pop* (dikeluarkan). Begitu juga seterusnya.

Visualisasinya seperti ini:

```
[Global]                → ✅ Mulai program
[Global, fungsiA]       → Masuk fungsiA
[Global, fungsiA, fungsiB] → Masuk fungsiB
[Global, fungsiA, fungsiB, fungsiC] → Masuk fungsiC
[Global, fungsiA, fungsiB] → Keluar fungsiC
[Global, fungsiA]       → Keluar fungsiB
[Global]                → Keluar fungsiA
[Global]                → ✅ Program selesai
```

---

### ⚠️ **Apa yang Terjadi Jika Call Stack Terlalu Dalam?**

JavaScript punya batas kedalaman Call Stack. Jika kamu membuat fungsi yang memanggil dirinya sendiri tanpa henti (*infinite recursion*), kamu akan dapat error:

Ubah `demo.js` jadi:

```js
function recursive() {
  console.log('Masih jalan...');
  recursive(); // panggil diri sendiri!
}

recursive();
```

Jalankan:

```bash
node demo.js
```

Setelah ratusan/ribuan pemanggilan, kamu akan lihat:

```
RangeError: Maximum call stack size exceeded
```

> 💥 Ini disebut **stack overflow**—Call Stack kelebihan beban!

---

### 🔗 **Hubungan Call Stack dengan Asynchronous & Event Loop**

Inilah bagian krusial:

- **Semua kode sinkron** masuk ke Call Stack.
- Tapi **kode asynchronous** (seperti `setTimeout`, `fs.readFile`) **tidak langsung masuk ke Call Stack**.
- Mereka dikirim ke **Web APIs** (di browser) atau **C++ thread pool** (di Node.js).
- Saat selesai, callback-nya dikirim ke **Callback Queue**.
- **Event Loop** menunggu Call Stack **kosong**, lalu ambil callback dari queue → masukkan ke Call Stack.

Jadi, Call Stack adalah **tempat eksekusi sebenarnya**—dan Event Loop hanya boleh mengisi ulang saat stack-nya kosong.

---

### 🧪 **Langkah 4: Gabungkan Call Stack + Async (Opsional Tapi Sangat Direkomendasikan)**

Buat file baru: `async-callstack.js`

```js
console.log('[1] Mulai');

setTimeout(() => {
  console.log('[4] Timeout selesai');
}, 0);

function proses() {
  console.log('[2] Di dalam fungsi proses');
}
proses();

console.log('[3] Selesai eksekusi sinkron');
```

Jalankan:

```bash
node async-callstack.js
```

**Output:**
```
[1] Mulai
[2] Di dalam fungsi proses
[3] Selesai eksekusi sinkron
[4] Timeout selesai
```

> ❗ Perhatikan:  
> - `[4]` muncul **terakhir**, meski timeout-nya 0 detik!  
> - Karena callback `setTimeout` **harus menunggu Call Stack benar-benar kosong** sebelum dijalankan.

---

### **Kesimpulannya**

Call Stack adalah **mekanisme inti** yang menjelaskan **mengapa JavaScript single-threaded tapi tetap bisa menangani banyak tugas**. Ia adalah tempat di mana semua fungsi benar-benar dieksekusi—satu per satu, dari bawah ke atas, lalu kembali turun.

Tanpa memahami Call Stack, kamu tidak akan pernah benar-benar paham:
- Kenapa `setTimeout` tidak jalan tepat waktu,
- Kenapa infinite recursion crash,
- Atau bagaimana Event Loop benar-benar bekerja.

Dan yang terpenting: **kamu baru saja melihatnya bekerja dengan matamu sendiri**—hanya dengan `console.log` dan perintah `node`.

Karena di dunia JavaScript, **semua dimulai dari satu tumpukan… dan satu langkah pada satu waktu**. 🧱
