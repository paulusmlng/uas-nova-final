# Analisis Sentimen Ulasan Produk E-Commerce Menggunakan TF-IDF dan Support Vector Machine (SVM)

## Kelompok Nova
1. Felix Maicleine - 221111389
2. Paulus Silvester Simalango - 221113232
3. Marvin Octavius - 221112156

## Masalah & Tujuan
Proyek ini mengangkat permasalahan Analisis Sentimen pada ulasan teks produk e-commerce dalam Bahasa Indonesia. Tujuannya adalah untuk mengklasifikasikan sebuah ulasan ke dalam kategori sentimen positif atau negatif secara otomatis menggunakan model machine learning (SVM).

Aplikasi ini telah di-deploy ke Railway dan dapat diakses secara publik melalui URL berikut:

**[https://nova-analisis-sentimen.up.railway.app](https://nova-analisis-sentimen.up.railway.app)**

---

## Teknologi yang Digunakan

* **Backend:** Python, Flask, Gunicorn
* **Machine Learning:** Scikit-learn (SVC, TfidfVectorizer), Pandas
* **Preprocessing NLP:** Sastrawi (Stemming, Stopword Removal), NLTK (Tokenization), Kamus Normalisasi (Excel)
* **Frontend:** HTML5, CSS3 (Desain Responsif)
* **Deployment:** Railway, Git, GitHub

---

## Struktur Proyek

```
uts-sentiment-web/
├── app.py                 # Server Flask (backend), logika ML, dan preprocessing
├── svm_tfidf_model3.pkl   # Model SVM & TF-IDF yang sudah dilatih
├── kamuskatabaku.xlsx     # Kamus normalisasi kata baku (dibaca saat startup)
├── templates/
│   └── index.html         # Halaman frontend (UI)
├── Procfile               # Perintah start untuk Railway (menjalankan gunicorn)
├── requirements.txt       # Daftar dependensi Python untuk di-install server
└── .gitignore             # File untuk mengabaikan folder venv dan file cache
```

---

## Cara Menjalankan di Lingkungan Lokal

1.  **Clone Repository**
    ```bash
    git clone [https://github.com/FelixMaicleine/uts-sentiment-web.git](https://github.com/FelixMaicleine/uts-sentiment-web.git)
    cd uts-sentiment
    ```

2.  **Buat Virtual Environment**
    ```bash
    python -m venv venv
    ```

3.  **Aktifkan Virtual Environment**
    * Di Windows (Command Prompt): `venv\Scripts\activate`
    * Di macOS/Linux: `source venv/bin/activate`

4.  **Install Semua Dependensi (requirements.txt)**
    ```bash
    pip install -r requirements.txt
    ```

5.  **Jalankan Aplikasi**
    ```bash
    python app.py
    ```

6.  **Buka di Browser**
    Buka browser Anda dan akses alamat `http://127.0.0.1:5000` untuk melihat aplikasi berjalan.

---

## Proses Deployment ke Railway

1.  **Persiapan File:**
    * **`Procfile`**: Dibuat untuk memberi tahu Railway perintah apa yang harus dijalankan untuk memulai server (gunicorn).
    * **`Prequirements.txt`**: Dibuat untuk memberi tahu Railway dependensi yang diinstal/dibutuhkan.
    * **`.gitignore`**: Dibuat untuk memastikan folder `venv` yang berisi ratusan MB library lokal tidak ikut ter-upload ke GitHub.

2.  **Inisialisasi Git & GitHub:**
    * Proyek diinisialisasi sebagai repository Git (`git init`).
    * Sebuah repository baru dibuat di GitHub.
    * Proyek lokal di-push ke repository GitHub (`git push origin main`).

3.  **Koneksi ke Railway:**
    * Sebuah proyek baru dibuat di Railway.
    * Proyek Railway dihubungkan ke repository GitHub.
    * Railway secara otomatis mendeteksi `requirements.txt` dan `Procfile`.

4.  **Deploy Otomatis:**
    * Railway meng-install semua dependensi dan membangun aplikasi.
    * Setelah *build* selesai, Railway menjalankan server `gunicorn` sesuai perintah `Procfile`.
    * Sebuah URL publik otomatis digenerate dan aplikasi menjadi live.
    * Setiap `git push` baru ke branch `main` akan otomatis memicu proses *re-deploy* (CI/CD).