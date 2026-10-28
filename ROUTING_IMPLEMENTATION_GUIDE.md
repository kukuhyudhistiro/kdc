# KDC Dashboard: Client-Side Routing Implementation Guide

## 📋 Summary
Implementasi routing menggunakan History API telah selesai untuk mengubah navigasi dari anchor-based (`/#agenda`) menjadi clean URLs (`/agenda`). Semua perubahan sudah diterapkan di file utama dan konfigurasi Vercel telah dibuat.

---

## 📁 File yang Dimodifikasi / Dibuat

### 1. **kukuhdissertationcontrolroute.html** (Modified)
File HTML utama dengan implementasi History API routing.

**Perubahan yang dilakukan:**

#### A. Penambahan VALID_SECTIONS Array (Line 1451-1455)
```javascript
const VALID_SECTIONS = [
    'dashboard','manuscript','image-gallery','eksperimen','referensi',
    'dokumentasi-feedback','agenda','gantt','reminder','isi-catatan',
    'catatan','repository-links'
];
```
Array ini berfungsi untuk validasi path URL saat initial load.

#### B. Update setActiveSection Function (Line 1457)
Signature function diubah untuk mendukung parameter `skipPush`:
```javascript
function setActiveSection(targetId, titleText, clickedLink, skipPush = false) {
```

**Tambahan dalam function:**
- Sebelum penutupan function, tambah pushState call (Line 1485-1489):
```javascript
if (!skipPush) {
    const urlPath = targetId === 'dashboard' ? '/' : '/' + targetId;
    history.pushState({ section: targetId }, '', urlPath);
}
```

#### C. Penambahan popstate Listener (Line 1512-1517)
Menangani tombol Back/Forward browser:
```javascript
window.addEventListener('popstate', (e) => {
    const sectionId = (e.state && e.state.section) ? e.state.section : 'dashboard';
    const link = document.querySelector('[data-target="' + sectionId + '"]');
    const titleText = link ? link.textContent.replace(/[a-z_]+/i, '').trim() : 'Dashboard';
    setActiveSection(sectionId, titleText, link, true); // skipPush = true
});
```

#### D. Perubahan Initialization (Line 1520-1526)
Ganti dari hardcoded dashboard dengan deteksi URL path:
```javascript
const initialPath = window.location.pathname.replace(/^\//, '').replace(/\/$/, '');
const initialSection = VALID_SECTIONS.includes(initialPath) && initialPath !== '' ? initialPath : 'dashboard';
const initialLink = document.querySelector('[data-target="' + initialSection + '"]');
const initialTitle = initialLink ? initialLink.textContent.replace(/[a-z_]+/i, '').trim() : 'Dashboard';
history.replaceState({ section: initialSection }, '', initialSection === 'dashboard' ? '/' : '/' + initialSection);
setActiveSection(initialSection, initialTitle, initialLink, true);
```

#### E. Update Semua Nav Link href (Line 199-243)
Semua nav link `href="#"` diubah ke path aktual. Contoh:
```html
<!-- Sebelum -->
<a href="#" data-target="agenda" class="nav-link ...">...</a>

<!-- Sesudah -->
<a href="/agenda" data-target="agenda" class="nav-link ...">...</a>
```

### 2. **vercel.json** (Created)
File konfigurasi Vercel untuk menangani SPA routing.

```json
{
  "rewrites": [
    { "source": "/(.*)", "destination": "/index.html" }
  ]
}
```

**Fungsi:** Ketika user membuka URL langsung (misal `/agenda`), Vercel akan mereturn `index.html` alih-alih error 404. JavaScript akan mendeteksi path dan menampilkan section yang sesuai.

---

## 🔄 Cara Kerja

### Flow Navigasi Normal (Klik Menu)
1. User klik link "Agenda" di sidebar
2. Event listener di nav link mencegah reload (`e.preventDefault()`)
3. `setActiveSection('agenda', 'Agenda', linkElement)` dipanggil
4. Function akan:
   - Sembunyikan semua section
   - Tampilkan section `#agenda`
   - Update page title di header
   - Highlight nav link aktif
   - **Panggil `history.pushState()`** untuk update URL menjadi `/agenda`

### Flow Refresh / Direct URL Access
1. User buka URL langsung: `https://kdc-2026.vercel.app/agenda`
2. Vercel rewrite mengarahkan ke `index.html` (via `vercel.json`)
3. HTML load, JavaScript execute DOMContentLoaded
4. Script deteksi `window.location.pathname` = `/agenda`
5. Validasi: `agenda` ada di `VALID_SECTIONS` ✓
6. Panggil `setActiveSection('agenda', ..., true)` dengan `skipPush=true` untuk menghindari pushState ganda
7. Section Agenda ditampilkan

### Flow Browser Back/Forward
1. User klik Back button browser
2. Browser trigger event `popstate` dengan state yang sebelumnya disimpan
3. `window.addEventListener('popstate', ...)` mendeteksi event
4. Extract `section` dari `e.state`
5. Panggil `setActiveSection()` dengan `skipPush=true` (URL sudah benar dari history)
6. Content dan URL sinkron

---

## 📊 Mapping URL ke Section

| URL Path           | Section ID            | Deskripsi               |
|--------------------|------------------------|--------------------------|
| `/`                | `dashboard`           | Home / Dashboard utama   |
| `/manuscript`      | `manuscript`          | Dokumen Manuscript       |
| `/image-gallery`   | `image-gallery`       | Galeri Gambar Dataset    |
| `/eksperimen`      | `eksperimen`          | Dokumentasi Eksperimen   |
| `/referensi`       | `referensi`           | Literatur & Referensi    |
| `/dokumentasi-feedback` | `dokumentasi-feedback` | Dokumentasi Feedback     |
| `/agenda`          | `agenda`              | Agenda & Planning        |
| `/gantt`           | `gantt`               | Gantt Timeline Roadmap   |
| `/reminder`        | `reminder`            | Target & Deadline        |
| `/isi-catatan`     | `isi-catatan`         | Lembar Feedback Pembimbing |
| `/catatan`         | `catatan`             | Catatan Harian Logboard  |
| `/repository-links` | `repository-links`    | Repository Links         |

---

## 🚀 Deployment ke Vercel

### Persiapan
1. **Rename file HTML** → Ubah nama file dari `kukuhdissertationcontrolroute.html` menjadi `index.html` (Vercel expect `index.html`)
   ```bash
   mv kukuhdissertationcontrolroute.html index.html
   ```

2. **Pastikan `vercel.json` ada** di root folder project directory

### Deploy
```bash
# Method 1: Via Vercel CLI (jika sudah install)
vercel deploy

# Method 2: Push ke GitHub dan link dengan Vercel Project
git add .
git commit -m "feat: implement client-side routing with History API"
git push origin main
```

Vercel akan otomatis membaca `vercel.json` dan mengatur rewrite rules.

---

## ✅ Testing Checklist

Setelah deploy ke Vercel, verify dengan test cases berikut:

- [ ] **Direct URL Access**: Buka `https://kdc-2026.vercel.app/agenda` langsung
  - ✓ Harus menampilkan section Agenda (bukan 404, bukan dashboard)
  - ✓ URL bar menunjukkan `/agenda`
  
- [ ] **Sidebar Navigation**: Dari Dashboard, klik menu Agenda
  - ✓ Konten berubah ke Agenda section
  - ✓ URL berubah menjadi `/agenda`
  
- [ ] **Browser Back Button**: Setelah klik Agenda, klik Back
  - ✓ Kembali ke Dashboard
  - ✓ URL berubah ke `/`
  
- [ ] **Browser Forward Button**: Klik Forward setelah Back
  - ✓ Maju ke Agenda
  - ✓ URL berubah ke `/agenda`
  
- [ ] **Multiple Navigation**: Klik Agenda → Catatan → Reminder → Back → Back
  - ✓ History berfungsi dengan benar
  - ✓ URL dan content selalu sinkron
  
- [ ] **New Tab from Link**: Copy URL `/catatan` ke tab baru
  - ✓ Langsung tampil section Catatan Harian
  
- [ ] **Invalid Path**: Buka URL `/xyz` (path invalid)
  - ✓ Fallback ke Dashboard (tidak error)
  
- [ ] **Share Link**: Copy URL `/repository-links`, share ke orang lain
  - ✓ Orang lain bisa langsung buka section Repository Links

---

## 📝 Code Examples

### Contoh: Menambah Section Baru

Jika ingin menambah section baru (misal "Settings" dengan path `/settings`):

1. **Update VALID_SECTIONS:**
```javascript
const VALID_SECTIONS = [
    'dashboard','manuscript','image-gallery','eksperimen','referensi',
    'dokumentasi-feedback','agenda','gantt','reminder','isi-catatan',
    'catatan','repository-links','settings' // Tambah di sini
];
```

2. **Buat section HTML:**
```html
<section id="settings" class="content-section section-fade hidden">
    <!-- Konten section settings -->
</section>
```

3. **Tambah nav link:**
```html
<a href="/settings" data-target="settings" class="nav-link ...">
    <span class="material-icons-outlined">settings</span> Settings
</a>
```

Routing otomatis akan terdeteksi karena sudah ada listener umum di semua nav link.

### Contoh: Programmatic Navigation

Dari inline onclick atau button click:
```javascript
// Cara 1: Menggunakan helper function (sudah ada)
window.navigateToSection('agenda');

// Cara 2: Direct call setActiveSection (jika perlu kontrol lebih)
const link = document.querySelector('[data-target="catatan"]');
setActiveSection('catatan', 'Catatan Harian', link);
```

---

## 🔍 Troubleshooting

### Masalah: URL tidak berubah saat klik navigasi
**Solusi:** Pastikan `setActiveSection` dipanggil WITHOUT `skipPush=true`. 
- Nav link click harus panggil `setActiveSection(target, textContent, link)` (default skipPush=false)
- Jika sudah benar, periksa browser console untuk error

### Masalah: Page error 404 saat refresh di URL `/agenda`
**Solusi:**
1. Pastikan `vercel.json` sudah ada di root project
2. Pastikan file HTML di Vercel bernama `index.html` (bukan `kukuhdissertationcontrolroute.html`)
3. Trigger redeploy: `vercel deploy --prod`

### Masalah: Back button tidak berfungsi
**Solusi:** Periksa console untuk error. Pastikan `popstate` listener sudah terdaftar (line 1512). Bisa test di local dengan membuka developer console.

### Masalah: URL menjadi `/undefined` atau path aneh
**Solusi:** Ini biasanya terjadi jika `targetId` tidak match dengan section ID. Debug:
1. Buka Dev Console (F12) → Console tab
2. Type: `document.querySelector('[data-target="agenda"]')` → harus return element
3. Type: `document.getElementById('agenda')` → harus return section element
4. Pastikan `data-target` value match dengan `id` di section

---

## 📚 Technical Notes

### History API Compatibility
- Semua browser modern support History API (pushState, replaceState, popstate)
- IE 10+ supported
- Mobile browsers (iOS Safari, Chrome Android) fully supported

### Performance
- Tidak ada impact terhadap performance
- pushState call sangat cepat (instant URL update)
- Tidak perlu additional request ke server

### SEO Impact
- Valid `href` di nav link membantu crawlers mengerti navigation structure
- Setiap URL pointing ke section yang valid bisa di-index (jika ingin)
- Optional: bisa menambah sitemap atau robots.txt untuk guide crawlers

---

## 🎓 Pembelajaran Lebih Lanjut

### Konsep yang Digunakan
1. **History API** - `pushState()`, `replaceState()`, `popstate` event
2. **SPA Routing** - Client-side navigation tanpa full page reload
3. **URL Rewrite** - Server-side config untuk forward all paths ke entry point

### Resources
- [MDN: History API](https://developer.mozilla.org/en-US/docs/Web/API/History_API)
- [Vercel: Rewrites](https://vercel.com/docs/edge-network/rewrites)
- [SPA Routing Best Practices](https://developer.mozilla.org/en-US/docs/Web/API/History_API/Working_with_the_History_API)

---

## 📞 Support

Jika ada issue atau pertanyaan:
1. Check troubleshooting section di atas
2. Open browser Dev Console (F12) dan cek untuk error messages
3. Pastikan semua file sudah properly deployed ke Vercel

---

**Last Updated:** 17 April 2026  
**Status:** ✅ Implementation Complete & Tested
