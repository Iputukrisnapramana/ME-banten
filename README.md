# Me-banten

Aplikasi web untuk **deteksi otomatis jenis banten Bali** menggunakan kamera dan model AI berbasis Teachable Machine.

---

## Fitur

- Deteksi lima jenis banten Bali: **Canang Sari, Pejati, Gebogan, Daksina, Lamak**.
- Menggunakan kamera perangkat (HP/laptop) secara langsung.
- Menampilkan penjelasan, makna, cara pembuatan, dan link YouTube (jika ada) untuk setiap banten yang terdeteksi.
- Tampilan modern, responsif, dan mudah digunakan.

---

## Struktur Proyek

```
Me-banten_prototipe/
│
├── index.html                // Halaman utama aplikasi web
├── style.css                 // Gaya/tampilan aplikasi
├── logo.png                  // Logo aplikasi
├── teachablemachine-image.min.js // Library untuk model Teachable Machine
│
├── model/                    // Model AI hasil training Teachable Machine
│   ├── model.json
│   ├── metadata.json
│   └── weights.bin
│
├── data/
│   └── sampelbanten.csv      // Data sampel banten (jika diperlukan)
│
└── banten_bali/              // Kumpulan gambar referensi banten
    ├── canang_sari/
    ├── pejati_pejati/
    ├── gebogan_gebogan/
    ├── daksina_daksina/
    └── lamak_lamak/
```

---

## Cara Menjalankan

1. **Download/clone** seluruh isi folder ini ke komputer Anda.
2. **Buka file `index.html`** di browser modern (Chrome, Edge, Firefox, dsb).
3. **Izinkan akses kamera** saat diminta.
4. Arahkan kamera ke banten yang ingin dideteksi, lalu klik tombol **"Deteksi Banten"**.
5. Hasil deteksi, penjelasan, dan info terkait akan muncul di bawah tombol.

> **Catatan:**  
> Tidak perlu instalasi server atau library tambahan. Semua berjalan di sisi client (browser).

---

## Penjelasan Teknis & Contoh Kode

### 1. **index.html** (Kode utama)

```html
<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Deteksi Banten Bali</title>
  <link rel="icon" href="logo.png">
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <header>
    <img src="logo.png" alt="Logo">
    <h1>Me-banten</h1>
  </header>
  <main>
    <video id="video" autoplay playsinline></video>
    <button id="detectBtn">Deteksi Banten</button>
    <div id="result">Arahkan kamera ke banten, lalu klik tombol di atas untuk deteksi.</div>
  </main>
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.7.4/dist/tf.min.js"></script>
  <script src="teachablemachine-image.min.js"></script>
  <script>
    // Data entitas dari CSV
    const bantenData = {
      "canang_sari": {
        nama: "Canang Sari",
        kategori: "Harian",
        makna: "Canang Sari merupakan simbol rasa syukur sehari-hari kepada Tuhan atas kehidupan yang telah diberikan. Biasanya dipersembahkan di pagi atau sore hari.",
        cara: "Dibuat dari janur yang dianyam menjadi wadah kecil, diisi dengan bunga warna-warni (cempaka, sandat, mawar), kepeng uang sebagai simbol rezeki, serta diberi dupa untuk doa.",
        youtube: "YouTube"
      },
      // ... (data banten lain serupa)
    };

    const URL = "model/";
    let model, webcam, maxPredictions;
    let video = document.getElementById('video');
    let detectBtn = document.getElementById('detectBtn');
    let resultDiv = document.getElementById('result');

    // Normalisasi label model ke key bantenData
    function normalizeLabel(label) {
      if (label === 'Canang Sari') return 'canang_sari';
      if (label === 'Pejati') return 'pejati_pejati';
      if (label === 'Gebogan') return 'gebogan_gebogan';
      if (label === 'Daksina') return 'daksina_daksina';
      if (label === 'Lamak') return 'lamak_lamak';
      return label;
    }

    // Minta akses kamera saat halaman dibuka
    async function setupCamera() {
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: "environment" }, audio: false });
        video.srcObject = stream;
      } catch (err) {
        resultDiv.innerHTML = 'Tidak dapat mengakses kamera: ' + err.message;
        detectBtn.disabled = true;
      }
    }

    // Load model teachable machine
    async function loadModel() {
      model = await tmImage.load(URL + "model.json", URL + "metadata.json");
      maxPredictions = model.getTotalClasses();
    }

    // Deteksi banten dari frame video
    async function predict() {
      if (!model) {
        resultDiv.innerHTML = 'Model belum siap.';
        return;
      }
      // Tampilkan loading ala AI
      resultDiv.innerHTML = `
        <div class="ai-loading">
          Mendeteksi banten <span class="ai-dots"><span class="ai-dot">.</span><span class="ai-dot">.</span><span class="ai-dot">.</span></span>
        </div>
      `;
      // Ambil prediksi dari frame video
      const prediction = await model.predict(video);
      // Cari prediksi dengan probabilitas tertinggi
      let best = prediction[0];
      for (let i = 1; i < prediction.length; i++) {
        if (prediction[i].probability > best.probability) best = prediction[i];
      }
      // Tampilkan hasil
      if (best.probability > 0.7) {
        const key = normalizeLabel(best.className);
        const data = bantenData[key];
        if (data) {
          resultDiv.innerHTML = `
            <div class="banten-card">
              <div class="banten-title">
                <svg width="28" height="28" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><circle cx="12" cy="12" r="12" fill="gold"/><text x="12" y="17" text-anchor="middle" font-size="15" font-family="Arial" fill="#1e1e1e" font-weight="bold">🪔</text></svg>
                ${data.nama}
              </div>
              <div class="banten-kategori">${data.kategori}</div>
              <div class="banten-section"><span class="banten-label">Makna & Fungsi:</span><span class="desc">${data.makna}</span></div>
              <div class="banten-section"><span class="banten-label">Cara Membuat:</span><span class="desc">${data.cara}</span></div>
              <div class="banten-section">
                <span class="banten-label">Link YouTube:</span>
                <a class="youtube-link" href="${data.youtube}" target="_blank">
                  <svg class="youtube-icon" viewBox="0 0 24 24"><path fill="#fff" d="M23.498 6.186a2.994 2.994 0 0 0-2.108-2.116C19.228 3.5 12 3.5 12 3.5s-7.228 0-9.39.57A2.994 2.994 0 0 0 .502 6.186C0 8.353 0 12 0 12s0 3.647.502 5.814a2.994 2.994 0 0 0 2.108 2.116C4.772 20.5 12 20.5 12 20.5s7.228 0 9.39-.57a2.994 2.994 0 0 0 2.108-2.116C24 15.647 24 12 24 12s0-3.647-.502-5.814zM9.545 15.568V8.432L15.818 12l-6.273 3.568z"/></svg>
                  Tonton di YouTube
                </a>
              </div>
            </div>
          `;
        } else {
          resultDiv.innerHTML = `<span class="label">${best.className}</span><br>Data detail tidak ditemukan.`;
        }
      } else {
        resultDiv.innerHTML = 'Banten tidak terdeteksi dengan yakin. Coba ulangi.';
      }
    }

    // Inisialisasi
    window.addEventListener('DOMContentLoaded', async () => {
      await setupCamera();
      await loadModel();
      detectBtn.disabled = false;
    });
    detectBtn.onclick = predict;
  </script>
</body>
</html>
```

### 2. **style.css** (Contoh style utama)

```css
body {
  background: #1e1e1e;
  color: #fff;
  font-family: 'Segoe UI', Arial, sans-serif;
  margin: 0;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
}
#video {
  width: 100%;
  max-width: 350px;
  border-radius: 16px;
  border: 2.5px solid gold;
  background: #222;
  margin-bottom: 18px;
  box-shadow: 0 4px 32px 0 #000a;
}
#detectBtn {
  background: linear-gradient(90deg, #c8b773 0%, #9a8f4e 50%, #fffbe6 100%);
  color: #1e1e1e;
  border: none;
  border-radius: 12px;
  padding: 14px 38px;
  font-size: 1.2rem;
  font-weight: bold;
  cursor: pointer;
  margin-bottom: 28px;
  box-shadow: 0 0 16px 2px gold, 0 2px 16px 0 #0007;
}
#result {
  background: rgba(35,35,35,0.85);
  border-radius: 22px;
  padding: 32px 24px 24px 24px;
  min-height: 90px;
  width: 100%;
  color: #fff;
  font-size: 1.1rem;
  text-align: left;
  border: 2px solid gold;
  margin-bottom: 18px;
}
```

### 3. **model/**
- Berisi file model hasil training Teachable Machine: `model.json`, `metadata.json`, `weights.bin`.
- Model ini digunakan untuk mendeteksi gambar banten dari kamera.

### 4. **banten_bali/**
- Berisi gambar-gambar referensi banten yang bisa digunakan untuk pelatihan model atau dokumentasi.

---

## Kustomisasi

- Untuk menambah jenis banten baru:
  1. Latih ulang model di Teachable Machine.
  2. Ganti file model di folder `model/`.
  3. Tambahkan entri baru pada objek `bantenData` di `index.html`.
  4. (Opsional) Tambahkan gambar referensi di folder yang sesuai.

- Untuk mengubah tampilan, edit file `style.css`.

---

## Lisensi

Aplikasi ini dibuat untuk edukasi dan pelestarian budaya Bali.  
Silakan gunakan, modifikasi, dan distribusikan dengan tetap mencantumkan sumber. 