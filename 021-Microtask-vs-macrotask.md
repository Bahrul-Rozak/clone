Halo teman-teman!

Di dunia JavaScript yang asynchronous, tidak semua callback diciptakan sama. Ada **dua kategori utama** yang menentukan **kapan** callback-mu akan dieksekusi: **Microtask** dan **Macrotask** (sering juga disebut Task). Memahami perbedaan keduanya adalah kunci untuk menulis kode async yang dapat diprediksi dan menghindari bug yang sulit dilacak.

Mari kita kupas tuntas **apa itu Microtask vs Macrotask**, bagaimana perbedaannya, dan **kita akan buktikan lewat kode nyata yang bisa kamu jalankan sendiri!**

---

### 🔍 **Apa Itu Microtask dan Macrotask?**

#### **Microtask** (Prioritas Tinggi)
Microtask adalah callback yang diproses **segera setelah Call Stack kosong**, bahkan **sebelum** Event Loop melanjutkan ke fase berikutnya. Mereka memiliki prioritas tertinggi dalam antrian async.

Contoh Microtask:
- `Promise.then()`, `Promise.catch()`, `Promise.finally()`
- `queueMicrotask()`
- `process.nextTick()` (Node.js-specific, bahkan lebih prioritas dari Promise!)
- `MutationObserver` callbacks (di browser)

#### **Macrotask** (Prioritas Rendah)
Macrotask adalah callback yang masuk ke **Callback Queue** (Task Queue) dan diproses **setelah semua Microtask selesai**. Ini adalah antrian "biasa" untuk operasi async.

Contoh Macrotask:
- `setTimeout()`, `setInterval()`
- `setImmediate()` (Node.js)
- I/O callbacks (seperti `fs.readFile`)
- Event listeners (click, load, dll di browser)
- `requestAnimationFrame()` (di browser)

---

### 🔄 **Perbedaan Utama: Urutan Eksekusi**

Ini adalah aturan emas Event Loop:

```
1. Eksekusi semua kode sinkron di Call Stack
2. Proses SEMUA Microtask yang ada (sampai queue kosong)
3. Render UI (di browser) atau lanjut ke fase berikutnya (di Node.js)
4. Proses SATU Macrotask dari queue
5. Kembali ke langkah 2 (proses Microtask lagi)
6. Ulangi...
```

Artinya: **Microtask selalu didahulukan sebelum Macrotask!**

---

### 🔧 **Langkah 1: Buat File Demo Microtask vs Macrotask**

Buka terminal, lalu buat folder dan file:

```bash
mkdir microtask-macrotask-demo
cd microtask-macrotask-demo
```

Buat file:

```bash
# macOS/Linux:
touch demo.js

# Windows:
type nul > demo.js
```

---

### 📄 **Langkah 2: Tulis Kode Perbandingan Lengkap**

Buka `demo.js`, lalu salin kode berikut:

```js
console.log('═══════════════════════════════════════════════');
console.log('🚀 [1] KODE SINKRON DIMULAI');

// Macrotask - setTimeout (0ms)
setTimeout(() => {
  console.log('⏰ [7] MACROTASK - setTimeout (0ms)');
}, 0);

// Microtask - Promise
Promise.resolve().then(() => {
  console.log('⚡ [4] MICROTASK - Promise.then() #1');
});

// Microtask - Promise kedua
Promise.resolve().then(() => {
  console.log('⚡ [5] MICROTASK - Promise.then() #2');
});

// Macrotask - setTimeout (10ms)
setTimeout(() => {
  console.log('⏰ [8] MACROTASK - setTimeout (10ms)');
}, 10);

// Kode sinkron lanjutan
console.log('✅ [2] Kode sinkron lanjutan');

// Microtask - queueMicrotask
queueMicrotask(() => {
  console.log('⚡ [6] MICROTASK - queueMicrotask()');
});

// Macrotask - setImmediate (Node.js specific)
setImmediate(() => {
  console.log('⏰ [9] MACROTASK - setImmediate()');
});

// Kode sinkron terakhir
console.log('✅ [3] SEMUA KODE SINKRON SELESAI');
console.log('═══════════════════════════════════════════════');
```

---

### ▶️ **Langkah 3: Jalankan dan Amati Urutan Eksekusi**

Di terminal, jalankan:

```bash
node demo.js
```

**Output yang muncul:**
```
═══════════════════════════════════════════════
🚀 [1] KODE SINKRON DIMULAI
✅ [2] Kode sinkron lanjutan
✅ [3] SEMUA KODE SINKRON SELESAI
═══════════════════════════════════════════════
⚡ [4] MICROTASK - Promise.then() #1
⚡ [5] MICROTASK - Promise.then() #2
⚡ [6] MICROTASK - queueMicrotask()
⏰ [7] MACROTASK - setTimeout (0ms)
⏰ [8] MACROTASK - setTimeout (10ms)
⏰ [9] MACROTASK - setImmediate()
```

> 🔥 **Perhatikan polanya:**
> - Semua Microtask (`⚡`) dieksekusi **berturut-turut** sebelum Macrotask pertama!
> - Meski `setTimeout(0ms)` diatur 0 detik, ia tetap **menunggu semua Microtask selesai**.
> - `setImmediate()` selalu paling akhir karena masuk ke fase terpisah di Event Loop Node.js.

---

### 🧪 **Langkah 4: Contoh Nyata - process.nextTick() vs Promise**

Di Node.js, ada satu microtask yang **bahkan lebih prioritas** dari Promise: `process.nextTick()`.

Buat file baru: `nexttick-demo.js`

```js
console.log('═══════════════════════════════════════════════');
console.log('🚀 Mulai');

// process.nextTick - Microtask dengan prioritas TERTINGGI
process.nextTick(() => {
  console.log('⚡ [2] process.nextTick() #1');
});

process.nextTick(() => {
  console.log('⚡ [3] process.nextTick() #2');
});

// Promise - Microtask biasa
Promise.resolve().then(() => {
  console.log('⚡ [4] Promise.then() #1');
});

Promise.resolve().then(() => {
  console.log('⚡ [5] Promise.then() #2');
});

// Macrotask
setTimeout(() => {
  console.log('⏰ [6] setTimeout()');
}, 0);

console.log('✅ [1] Kode sinkron selesai');
console.log('═══════════════════════════════════════════════');
```

Jalankan:

```bash
node nexttick-demo.js
```

**Output:**
```
═══════════════════════════════════════════════
🚀 Mulai
✅ [1] Kode sinkron selesai
═══════════════════════════════════════════════
⚡ [2] process.nextTick() #1
⚡ [3] process.nextTick() #2
⚡ [4] Promise.then() #1
⚡ [5] Promise.then() #2
⏰ [6] setTimeout()
```

> 💡 **Kesimpulan:**
> - `process.nextTick()` diproses **sebelum** Promise!
> - Urutan prioritas di Node.js: `process.nextTick()` > `Promise` > `setTimeout/setImmediate`

---

### 🧪 **Langkah 5: Kasus Nyata - Mengapa Ini Penting?**

Buat file: `real-case.js`

```js
const users = [];

// Simulasi async operation
function addUserAsync(user) {
  return new Promise((resolve) => {
    setTimeout(() => {
      users.push(user);
      resolve(user);
    }, 100);
  });
}

console.log('═══════════════════════════════════════════════');
console.log('🚀 Mulai proses');

// Tambah user async
addUserAsync({ id: 1, name: 'Andi' }).then(() => {
  console.log('✅ User Andi ditambahkan');
});

// Microtask yang berjalan SEBELUM Promise di atas selesai
Promise.resolve().then(() => {
  console.log('⚡ Microtask jalan');
  console.log('📊 Jumlah user saat ini:', users.length);
});

// Kode sinkron
console.log('✅ Kode sinkron selesai');
console.log('═══════════════════════════════════════════════');
```

Jalankan:

```bash
node real-case.js
```

**Output:**
```
═══════════════════════════════════════════════
🚀 Mulai proses
✅ Kode sinkron selesai
═══════════════════════════════════════════════
⚡ Microtask jalan
📊 Jumlah user saat ini: 0
✅ User Andi ditambahkan
```

> ⚠️ **Pelajaran penting:**
> - Microtask jalan **sebelum** Promise dari `addUserAsync` selesai!
> - Ini bisa menyebabkan bug jika kamu mengharapkan data sudah ada.
> - Solusi: Gunakan `.then()` berantai atau async/await untuk memastikan urutan.

---

### 📊 **Tabel Perbandingan Lengkap**

| **Karakteristik** | **Microtask** | **Macrotask** |
|-------------------|---------------|---------------|
| **Prioritas** | Sangat tinggi | Rendah |
| **Queue** | Microtask Queue | Callback Queue (Task Queue) |
| **Diproses kapan?** | Setelah Call Stack kosong, sebelum render | Setelah Microtask selesai |
| **Contoh** | `Promise.then()`, `queueMicrotask()`, `process.nextTick()` | `setTimeout()`, `setInterval()`, `setImmediate()`, I/O |
| **Urutan eksekusi** | Semua microtask diproses sekaligus | Satu per satu, bergantian dengan microtask |
| **Use case** | Update state yang konsisten, batch DOM updates | Delayed execution, polling, scheduled tasks |

---

### 🧪 **Langkah 6: Visualisasi Lengkap Siklus Event Loop**

Buat file: `event-loop-cycle.js`

```js
console.log('\n╔════════════════════════════════════════════════════════╗');
console.log('║          SIKLUS LENGKAP EVENT LOOP - ITERASI 1         ║');
console.log('╚════════════════════════════════════════════════════════╝\n');

// Fase 1: Eksekusi kode sinkron
console.log('[1] Eksekusi kode sinkron');

// Fase 2: Daftarkan berbagai task
setTimeout(() => {
  console.log('[6] Macrotask: setTimeout');
}, 0);

setImmediate(() => {
  console.log('[7] Macrotask: setImmediate');
});

Promise.resolve().then(() => {
  console.log('[4] Microtask: Promise #1');
});

Promise.resolve().then(() => {
  console.log('[5] Microtask: Promise #2');
});

process.nextTick(() => {
  console.log('[3] Microtask: process.nextTick');
});

console.log('[2] Kode sinkron selesai\n');

console.log('╔════════════════════════════════════════════════════════╗');
console.log('║              URUTAN EKSEKUSI YANG DIHARAPKAN:          ║');
console.log('║  1 → 2 → 3 → 4 → 5 → 6 → 7                            ║');
console.log('╚════════════════════════════════════════════════════════╝\n');
```

Jalankan:

```bash
node event-loop-cycle.js
```

**Output:**
```
╔════════════════════════════════════════════════════════╗
║          SIKLUS LENGKAP EVENT LOOP - ITERASI 1         ║
╚════════════════════════════════════════════════════════╝

[1] Eksekusi kode sinkron
[2] Kode sinkron selesai

╔════════════════════════════════════════════════════════╗
║              URUTAN EKSEKUSI YANG DIHARAPKAN:          ║
║  1 → 2 → 3 → 4 → 5 → 6 → 7                            ║
╚════════════════════════════════════════════════════════╝

[3] Microtask: process.nextTick
[4] Microtask: Promise #1
[5] Microtask: Promise #2
[6] Macrotask: setTimeout
[7] Macrotask: setImmediate
```

---

### ⚠️ **Common Pitfalls (Jebakan Umum)**

#### ❌ **Pitfall 1: Infinite Microtask Loop**

```js
// ❌ JANGAN LAKUKAN INI!
function infiniteLoop() {
  Promise.resolve().then(() => {
    console.log('Microtask terus-menerus...');
    infiniteLoop(); // Panggil diri sendiri di microtask
  });
}

infiniteLoop();
// Hasil: Macrotask tidak pernah jalan, UI freeze!
```

#### ❌ **Pitfall 2: Mengira setTimeout(0) = Langsung Jalan**

```js
// ❌ Salah asumsi
setTimeout(() => {
  console.log('Ini tidak langsung jalan!');
}, 0);

Promise.resolve().then(() => {
  console.log('Ini jalan DULUAN!');
});
```

#### ✅ **Solusi: Gunakan async/await untuk Urutan yang Jelas**

```js
async function prosesBerurutan() {
  await addUserAsync({ id: 1, name: 'Andi' });
  console.log('✅ User Andi sudah pasti ada di sini');
  console.log('📊 Jumlah user:', users.length);
}
```

---

### 🔗 **Visualisasi Urutan Prioritas Lengkap di Node.js**

```
Event Loop Iteration:
├─ 1. Eksekusi kode sinkron
├─ 2. Proses SEMUA process.nextTick()
├─ 3. Proses SEMUA Microtasks lain (Promise, queueMicrotask)
├─ 4. Render (di browser) / Lanjut ke fase I/O (di Node.js)
├─ 5. Proses SATU Macrotask (setTimeout, setImmediate, I/O)
└─ 6. Kembali ke langkah 2
```

---

### **Kesimpulannya**

Memahami perbedaan **Microtask vs Macrotask** adalah fondasi krusial untuk menguasai JavaScript asynchronous. Kuncinya:

1. **Microtask** (Promise, process.nextTick) diproses **segera setelah** Call Stack kosong—bahkan sebelum render atau I/O.
2. **Macrotask** (setTimeout, setImmediate, I/O) diproses **setelah semua Microtask selesai**.
3. Urutan prioritas di Node.js: `process.nextTick()` > `Promise` > `setTimeout`/`setImmediate`.
4. **Semua Microtask dalam queue diproses sekaligus** sebelum Event Loop lanjut ke Macrotask berikutnya.

Dengan memahami ini, kamu bisa:
- Menulis kode async yang urutannya dapat diprediksi,
- Menghindari infinite loop yang tidak disengaja,
- Memanfaatkan `process.nextTick()` untuk update state yang konsisten,
- Dan akhirnya, **menguasai Event Loop** alih-alih dikuasai olehnya.

Karena di dunia JavaScript, **bukan seberapa cepat kode-mu berjalan—tapi seberapa tepat urutannya**. Dan dengan pengetahuan Microtask vs Macrotask ini, kamu sekarang punya kendali penuh atas urutan itu! 🎯
