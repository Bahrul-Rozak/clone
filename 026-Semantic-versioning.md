Halo teman-teman!

Pernahkah kamu melihat versi package seperti `1.2.3`, `2.0.0`, atau `0.5.1` dan bertanya-tanya **apa arti angka-angka itu**? Atau kenapa ada package yang update dari `1.9.0` ke `1.10.0`, tapi ada yang langsung loncat ke `2.0.0`?

Jawabannya ada pada **Semantic Versioning (SemVer)**—sistem penomoran versi yang **standar dan bermakna** di dunia software. Ini bukan sekadar angka acak, tapi **bahasa komunikasi** antara pembuat package dan penggunanya tentang **apa yang berubah** di setiap rilis.

Mari kita pelajari **apa itu Semantic Versioning**, bagaimana cara kerjanya, dan bagaimana ini mempengaruhi proyek Node.js-mu—lengkap dengan contoh praktis yang bisa kamu jalankan sendiri!

---

## 🔍 **Apa Itu Semantic Versioning?**

**Semantic Versioning** (atau **SemVer**) adalah sistem penomoran versi dengan format:

```
MAJOR.MINOR.PATCH
```

Contoh:
- `1.0.0` → Versi pertama yang stabil
- `1.2.3` → Versi 1, fitur ke-2, perbaikan bug ke-3
- `2.0.0` → Versi 2 (ada breaking changes!)

Setiap angka punya **makna spesifik**:

| **Bagian** | **Kapan Naik?** | **Artinya** | **Contoh Perubahan** |
|------------|-----------------|-------------|---------------------|
| **MAJOR** | Breaking changes | API berubah, tidak kompatibel dengan versi sebelumnya | Hapus fungsi lama, ubah signature fungsi, ubah behavior |
| **MINOR** | Fitur baru | Fitur baru ditambahkan, tapi tetap kompatibel | Tambah fungsi baru, tambah parameter opsional |
| **PATCH** | Bug fix | Perbaikan bug, tanpa perubahan API | Fix bug, improve performance, update dokumentasi |

---

## 📐 **Format Lengkap SemVer**

Format lengkap SemVer adalah:

```
MAJOR.MINOR.PATCH-prerelease+build
```

Contoh:
- `1.2.3` → Versi stabil
- `1.2.3-beta.1` → Pre-release (beta)
- `1.2.3+build.123` → Build metadata
- `2.0.0-alpha.2` → Pre-release alpha versi 2

---

## 🔧 **Langkah 1: Buat Proyek Node.js Baru**

Buka terminal, lalu buat folder baru:

```bash
mkdir semantic-versioning-demo
cd semantic-versioning-demo
```

Inisialisasi package.json:

```bash
npm init -y
```

---

## 📦 **Langkah 2: Install Package dengan Berbagai Range Version**

Sekarang, mari kita lihat bagaimana NPM menangani berbagai format version range.

### **1. Exact Version (Versi Tepat)**

```bash
npm install lodash@4.17.21
```

Cek `package.json`:

```json
"dependencies": {
  "lodash": "4.17.21"
}
```

Artinya: **Hanya versi 4.17.21** yang diizinkan. Tidak boleh 4.17.22 atau 4.17.20.

---

### **2. Caret Range (^) - Default NPM**

```bash
npm install express@^4.18.2
```

Cek `package.json`:

```json
"dependencies": {
  "express": "^4.18.2"
}
```

Artinya: **Versi 4.x.x**, selama `x` tidak breaking changes.
- ✅ Boleh: `4.18.3`, `4.19.0`, `4.20.0`
- ❌ Tidak boleh: `5.0.0` (major version naik = breaking changes)

**Caret (`^`)** adalah **default** saat kamu install package tanpa spesifik versi:

```bash
npm install express
# Sama dengan: npm install express@^4.18.2
```

---

### **3. Tilde Range (~) - Hanya Patch Updates**

```bash
npm install axios@~1.1.3
```

Cek `package.json`:

```json
"dependencies": {
  "axios": "~1.1.3"
}
```

Artinya: **Hanya patch updates** di versi 1.1.x.
- ✅ Boleh: `1.1.4`, `1.1.5`
- ❌ Tidak boleh: `1.2.0` (minor version naik), `2.0.0` (major version naik)

---

### **4. Greater Than / Less Than**

```bash
npm install moment@">=2.29.0 <3.0.0"
```

Cek `package.json`:

```json
"dependencies": {
  "moment": ">=2.29.0 <3.0.0"
}
```

Artinya: **Versi 2.29.0 atau lebih tinggi, tapi kurang dari 3.0.0**.

---

### **5. Wildcard (*) - Semua Versi**

```bash
npm install debug@*
```

Cek `package.json`:

```json
"dependencies": {
  "debug": "*"
}
```

Artinya: **Versi apapun** (tidak direkomendasikan untuk production!).

---

### **6. Specific Range dengan AND/OR**

```bash
npm install react@">=16.8.0 <17.0.0 || >=18.0.0"
```

Artinya: **React 16.8.0 - 16.x.x ATAU React 18.x.x** (tapi tidak 17.x.x).

---

## 📊 **Tabel Perbandingan Version Range**

| **Format** | **Contoh** | **Boleh Update Ke** | **Tidak Boleh** | **Kegunaan** |
|------------|------------|---------------------|-----------------|--------------|
| Exact | `1.2.3` | ❌ Tidak boleh | Semua versi lain | Production critical |
| Caret | `^1.2.3` | `1.2.4`, `1.3.0` | `2.0.0` | Default, aman |
| Tilde | `~1.2.3` | `1.2.4` | `1.3.0`, `2.0.0` | Hanya bug fixes |
| Greater | `>=1.2.3` | Semua versi ≥ 1.2.3 | Semua versi < 1.2.3 | Minimum requirement |
| Wildcard | `*` | Semua versi | ❌ Tidak ada | Development only |
| Hyphen | `1.2.3 - 2.0.0` | `1.2.3` - `2.0.0` | < 1.2.3 atau > 2.0.0 | Range spesifik |

---

## 🧪 **Langkah 3: Praktik Lengkap - Buat Package Sendiri**

Sekarang, mari kita buat **package sendiri** dan lihat bagaimana SemVer bekerja.

### **3.1 Buat Package Sederhana**

Buat folder baru:

```bash
mkdir my-math-package
cd my-math-package
npm init -y
```

Buat file `index.js`:

```js
// my-math-package/index.js
function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

module.exports = { add, subtract };
```

Edit `package.json`:

```json
{
  "name": "my-math-package",
  "version": "1.0.0",
  "description": "Simple math utilities",
  "main": "index.js",
  "keywords": ["math", "utilities"],
  "author": "Your Name",
  "license": "MIT"
}
```

---

### **3.2 Test Package Versi 1.0.0**

Buat folder test:

```bash
cd ..
mkdir test-consumer
cd test-consumer
npm init -y
```

Install package local:

```bash
npm install ../my-math-package
```

Buat file `test.js`:

```js
const math = require('my-math-package');

console.log('Add:', math.add(5, 3));       // 8
console.log('Subtract:', math.subtract(10, 4)); // 6
```

Jalankan:

```bash
node test.js
```

Output:

```
Add: 8
Subtract: 6
```

---

### **3.3 Update ke Versi 1.1.0 (MINOR - Fitur Baru)**

Kembali ke package:

```bash
cd ../my-math-package
```

Update `index.js` - tambah fungsi baru:

```js
function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

// ✅ FITUR BARU - MINOR VERSION NAIK
function multiply(a, b) {
  return a * b;
}

module.exports = { add, subtract, multiply };
```

Update `package.json`:

```json
{
  "name": "my-math-package",
  "version": "1.1.0",  // Naik dari 1.0.0 → 1.1.0
  "description": "Simple math utilities",
  "main": "index.js",
  "keywords": ["math", "utilities"],
  "author": "Your Name",
  "license": "MIT"
}
```

Test di consumer:

```bash
cd ../test-consumer
npm update my-math-package
```

Update `test.js`:

```js
const math = require('my-math-package');

console.log('Add:', math.add(5, 3));
console.log('Subtract:', math.subtract(10, 4));
console.log('Multiply:', math.multiply(6, 7)); // ✅ Fitur baru!
```

Jalankan:

```bash
node test.js
```

Output:

```
Add: 8
Subtract: 6
Multiply: 42
```

✅ **Sukses!** Fitur baru ditambahkan **tanpa breaking changes**.

---

### **3.4 Update ke Versi 1.1.1 (PATCH - Bug Fix)**

Kembali ke package:

```bash
cd ../my-math-package
```

Fix bug di fungsi `subtract`:

```js
function add(a, b) {
  return a + b;
}

// ✅ BUG FIX - PATCH VERSION NAIK
function subtract(a, b) {
  // Sebelumnya: return a - b; (sudah benar)
  // Tapi misal ada bug, kita fix di sini
  return a - b;
}

function multiply(a, b) {
  return a * b;
}

module.exports = { add, subtract, multiply };
```

Update `package.json`:

```json
{
  "version": "1.1.1"  // Naik dari 1.1.0 → 1.1.1
}
```

Test di consumer:

```bash
cd ../test-consumer
npm update my-math-package
node test.js
```

✅ **Sukses!** Bug diperbaiki, API tetap sama.

---

### **3.5 Breaking Change - Versi 2.0.0 (MAJOR)**

Kembali ke package:

```bash
cd ../my-math-package
```

Ubah API secara breaking:

```js
// ❌ BREAKING CHANGE - MAJOR VERSION NAIK

// Sebelumnya: add(a, b)
// Sekarang: add(numbers) - terima array
function add(numbers) {
  return numbers.reduce((sum, num) => sum + num, 0);
}

function subtract(a, b) {
  return a - b;
}

function multiply(a, b) {
  return a * b;
}

module.exports = { add, subtract, multiply };
```

Update `package.json`:

```json
{
  "version": "2.0.0"  // Naik dari 1.1.1 → 2.0.0 (BREAKING!)
}
```

Test di consumer:

```bash
cd ../test-consumer
npm update my-math-package
node test.js
```

Output:

```
TypeError: math.add is not a function or its return value is not iterable
```

❌ **Error!** Karena API berubah breaking.

**Solusi**: Update kode consumer:

```js
const math = require('my-math-package');

// ✅ Update sesuai API baru
console.log('Add:', math.add([5, 3]));  // Sekarang terima array
console.log('Subtract:', math.subtract(10, 4));
console.log('Multiply:', math.multiply(6, 7));
```

Jalankan lagi:

```bash
node test.js
```

Output:

```
Add: 8
Subtract: 6
Multiply: 42
```

✅ **Sukses!** Tapi butuh **perubahan kode** di consumer.

---

## 📋 **Cheat Sheet: Kapan Naikkan Versi?**

### **PATCH (x.y.Z+1)**
Naikkan **PATCH** jika:
- ✅ Fix bug
- ✅ Improve performance (tanpa ubah API)
- ✅ Update dokumentasi
- ✅ Refactor internal (tanpa ubah public API)

Contoh:
- `1.2.3` → `1.2.4` (fix bug di fungsi `calculate`)
- `2.0.0` → `2.0.1` (improve error handling)

---

### **MINOR (x.Y+1.z)**
Naikkan **MINOR** jika:
- ✅ Tambah fitur baru
- ✅ Tambah fungsi/method baru
- ✅ Tambah parameter opsional
- ✅ Tambah konstanta/enumerasi baru

Contoh:
- `1.2.3` → `1.3.0` (tambah fungsi `divide`)
- `2.0.1` → `2.1.0` (tambah opsi konfigurasi)

---

### **MAJOR (X+1.y.z)**
Naikkan **MAJOR** jika:
- ❌ Hapus fungsi/method
- ❌ Ubah signature fungsi (parameter wajib berubah)
- ❌ Ubah return type
- ❌ Ubah behavior yang breaking
- ❌ Rename/remove public API
- ❌ Drop support untuk versi Node.js lama

Contoh:
- `1.3.0` → `2.0.0` (hapus fungsi `oldMethod`)
- `2.1.0` → `3.0.0` (ubah parameter dari string ke object)

---

## 🛠️ **Langkah 4: Tools untuk Manage Version**

### **npm version**

NPM punya command built-in untuk update version:

```bash
# Naikkan PATCH
npm version patch
# 1.2.3 → 1.2.4

# Naikkan MINOR
npm version minor
# 1.2.3 → 1.3.0

# Naikkan MAJOR
npm version major
# 1.2.3 → 2.0.0

# Custom version
npm version 2.5.0
```

Command ini akan:
1. Update `package.json`
2. Commit perubahan ke Git
3. Create Git tag (misal: `v1.2.4`)

---

### **npm outdated**

Cek package yang perlu di-update:

```bash
npm outdated
```

Output:

```
Package    Current  Wanted  Latest
express    4.18.2   4.18.3  4.18.3
lodash     4.17.21  4.17.21 4.17.21
```

---

### **npm update**

Update package sesuai range di `package.json`:

```bash
# Update semua
npm update

# Update package spesifik
npm update express
```

---

### **npm-check-updates**

Tool untuk update version range di `package.json`:

Install:

```bash
npm install -g npm-check-updates
```

Cek update:

```bash
ncu
```

Output:

```
express     ^4.18.2  →  ^4.18.3
lodash      ^4.17.21 →  ^4.17.21
new-package ^1.0.0   →  ^2.0.0
```

Update `package.json`:

```bash
ncu -u
npm install
```

---

## 📊 **Contoh Nyata di Dunia Production**

### **React**

```
16.8.0 → 16.9.0 → 16.10.0 → 16.14.0 → 17.0.0 → 18.0.0
   ↑        ↑         ↑          ↑         ↑        ↑
  Minor    Minor     Minor      Minor     Major    Major
  (Hooks)  (Fitur)   (Fitur)    (Stabil)  (Breaking) (Breaking)
```

### **Express**

```
4.17.1 → 4.17.2 → 4.17.3 → 4.18.0 → 4.18.1 → 4.18.2
   ↑        ↑         ↑         ↑        ↑        ↑
  Patch    Patch     Patch     Minor    Patch    Patch
  (Fix)    (Fix)     (Fix)    (Fitur)   (Fix)    (Fix)
```

### **Vue**

```
2.6.14 → 2.7.0 → 3.0.0 → 3.1.0 → 3.2.0 → 3.3.0
   ↑        ↑       ↑       ↑       ↑       ↑
  Patch    Minor   Major   Minor   Minor   Minor
  (Fix)   (Fitur) (Breaking) (Fitur) (Fitur) (Fitur)
```

---

## ⚠️ **Common Mistakes (Kesalahan Umum)**

### ❌ **Mistake 1: Tidak Naikkan Versi Saat Breaking**

```js
// Versi 1.0.0
function greet(name) {
  return `Hello, ${name}!`;
}

// Versi tetap 1.0.0, tapi API berubah! ❌
function greet(firstName, lastName) {
  return `Hello, ${firstName} ${lastName}!`;
}
```

**Benar:**

```js
// Versi 2.0.0 ✅
function greet(firstName, lastName) {
  return `Hello, ${firstName} ${lastName}!`;
}
```

---

### ❌ **Mistake 2: Naikkan Major untuk Bug Fix**

```json
{
  "version": "1.2.3"
}
```

Fix bug → langsung naik ke `2.0.0` ❌

**Benar:**

```json
{
  "version": "1.2.4"  // ✅ PATCH version
}
```

---

### ❌ **Mistake 3: Gunakan Wildcard di Production**

```json
{
  "dependencies": {
    "express": "*"  // ❌ Berbahaya!
  }
}
```

**Benar:**

```json
{
  "dependencies": {
    "express": "^4.18.2"  // ✅ Aman
  }
}
```

---

## 🎯 **Best Practices**

### 1. **Selalu Ikuti SemVer**

Jangan asal naikkan versi. Tanya diri:
- Apakah ada breaking change? → Major
- Apakah ada fitur baru? → Minor
- Apakah hanya bug fix? → Patch

---

### 2. **Gunakan Caret (^) untuk Dependencies**

```json
{
  "dependencies": {
    "express": "^4.18.2"  // ✅ Boleh update minor/patch
  }
}
```

---

### 3. **Gunakan Exact Version untuk Critical Packages**

```json
{
  "dependencies": {
    "crypto-lib": "3.2.1"  // ✅ Versi tepat, tidak boleh auto-update
  }
}
```

---

### 4. **Tulis Changelog**

Buat file `CHANGELOG.md`:

```markdown
# Changelog

## [2.0.0] - 2026-03-11
### Breaking Changes
- Ubah signature fungsi `greet()` dari `greet(name)` ke `greet(firstName, lastName)`

## [1.1.0] - 2026-03-10
### Added
- Tambah fungsi `multiply()`

## [1.0.1] - 2026-03-09
### Fixed
- Fix bug di fungsi `subtract()`
```

---

### 5. **Test Sebelum Release**

```bash
# Jalankan test
npm test

# Build project
npm run build

# Publish (jika package publik)
npm publish
```

---

## 📊 **Visualisasi SemVer**

```
Version: MAJOR.MINOR.PATCH
         ↑      ↑     ↑
         │      │     └─ Bug fixes (1.2.3 → 1.2.4)
         │      └─────── New features (1.2.3 → 1.3.0)
         └────────────── Breaking changes (1.2.3 → 2.0.0)

Timeline:
1.0.0 → 1.0.1 → 1.0.2 → 1.1.0 → 1.1.1 → 1.2.0 → 2.0.0
  │       │       │       │       │       │       │
  │       │       │       │       │       │       └─ Breaking!
  │       │       │       │       │       └─ New feature
  │       │       │       │       └─ Bug fix
  │       │       │       └─ New feature
  │       │       └─ Bug fix
  │       └─ Bug fix
  └─ Initial release
```

---

## 🎯 **Kesimpulannya**

**Semantic Versioning** adalah **bahasa universal** di dunia software development. Dengan memahami format `MAJOR.MINOR.PATCH`, kamu bisa:

1. **Komunikasikan perubahan** dengan jelas ke pengguna package-mu
2. **Kelola dependencies** dengan lebih aman di proyekmu
3. **Hindari breaking changes** yang tidak disengaja
4. **Update package** dengan percaya diri
5. **Bekerja di tim** dengan standar yang konsisten

### **Rule of Thumb:**

> **"Jika kamu ragu, tanyakan: Apakah ini breaking change? Jika ya → Major. Jika tidak → Minor atau Patch."**

Dan ingat: **SemVer bukan sekadar angka—tapi janji** kepada pengguna bahwa versi baru tidak akan merusak kode mereka (kecuali major version naik).

Dengan mengikuti SemVer, kamu menunjukkan bahwa kamu adalah developer yang **profesional, bertanggung jawab, dan peduli pada komunitas**! 🚀

---

## 💡 **Quick Reference**

```bash
# Update version
npm version patch    # 1.2.3 → 1.2.4
npm version minor    # 1.2.3 → 1.3.0
npm version major    # 1.2.3 → 2.0.0

# Cek outdated packages
npm outdated

# Update packages
npm update

# Install dengan version range
npm install express@^4.18.2   # Caret (default)
npm install express@~4.18.2   # Tilde
npm install express@4.18.2    # Exact
npm install express@">=4.0.0" # Range

# Tools
npm install -g npm-check-updates
ncu          # Cek update
ncu -u       # Update package.json
```

Sekarang kamu sudah paham **Semantic Versioning** dari A sampai Z! Terapkan ini di setiap proyekmu, dan kamu akan menjadi developer yang lebih terpercaya dan profesional! 🎓
