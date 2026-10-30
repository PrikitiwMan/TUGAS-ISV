# Laporan Tugas Praktikum Mandiri — Kalibrasi Kamera (Papan Catur)

## Setup
- Kamera: smartphone, citra 960×1280 px (portrait; foto dari WhatsApp, sudah terkompres)
- Papan catur lipat 8×8 kotak → **7×7 sudut dalam** (49 titik/citra). Ukuran kotak diasumsikan **25.0 mm** (hanya memengaruhi tvecs, tidak memengaruhi K, D, maupun error)
- Foto: 29 → terdeteksi 26 → dipakai 24 (outlier dibuang: papan_27.jpg, papan_06.jpg)
- Deteksi (`detector.py`): `findChessboardCorners/SB` pada citra diperkecil (hanya untuk pencarian awal); bila gagal (garis lipatan), cari sebagian kisi lalu tentukan posisinya di papan 8×8 dengan pencocokan pola terang/gelap + validasi homografi. Semua sudut diperhalus `cornerSubPix` pada **resolusi penuh**.
- Kalibrasi: `cv2.calibrateCamera` (Zhang), model D = [k1,k2,p1,p2,k3]

## Hasil
Matriks intrinsik K:

```
[[   956.20  0.00    470.12]
 [  0.00    956.16    657.47]
 [  0.00     0.00     1.00]]
```
fx=956.20, fy=956.16, u0=470.12, v0=657.47 (piksel)

Koefisien distorsi D = [k1, k2, p1, p2, k3]:
`[0.3118, -1.8877, 0.0098, -0.0070, 3.0422]`

| Metrik | Nilai |
|---|---|
| **Mean Reprojection Error** (rata-rata RMSE per citra) | **0.8368 px** |
| RMS global (`ret` dari calibrateCamera) | 0.8624 px |
| Error relatif (MRE ÷ sisi terpanjang citra) | 0.065 % |
| Validasi silang (kalibrasi 12 citra, uji 12 citra lain) | 1.6044 px |

Variasi model dengan k3 dikunci 0 (`--fix-k3`): fx=959.1, fy=958.7, u0=470.2, v0=654.7, D=[0.2387, -0.973, 0.0093, -0.0068, 0.0], MRE=0.8407 px.

## Plot dan bukti
- `reports/figures/koefisien_distorsi.png`, `error_reproyeksi.png`, `distorsi_radial.png`, `cakupan_sudut.png`, `sebelum_sesudah_undistort.jpg`
- `reports/scan/` — hasil scan papan (garis kuning tepi area 8×8); `reports/detected/` — sudut terdeteksi

## Catatan / keterbatasan
- **Error dalam piksel bergantung pada resolusi.** Foto yang dipakai hanya 960×1280 px; foto asli kamera (≈3000 px) akan menghasilkan error piksel ≈3× lebih besar untuk ketelitian relatif yang sama, dan K (fx, fy, u0, v0) ikut membesar sebanding. Bandingkan antar-kalibrasi memakai error relatif (MRE ÷ ukuran citra), bukan piksel mentah.
- Sudut papan hanya menempati bagian tengah bingkai, **tidak menjangkau tepi/pojok**. Suku orde tinggi (k2, k3) tidak terkendala di tepi; nilainya besar dan koreksi di pojok citra tidak dapat dipercaya. Validasi silang (1.60 px) jauh di atas MRE.
- Foto yang tidak terdeteksi / dibuang: papan terlalu miring atau kecil, atau hasil ekstrapolasi tidak lolos validasi.
- Untuk hasil lebih kuat: pakai foto asli beresolusi penuh (bukan kiriman WhatsApp), papan rata, isi seluruh bingkai (termasuk pojok), 15–20 pose.

## Menjalankan ulang
```
pip install -r requirements.txt
python calibrate.py                 # butuh calibrate.py DAN detector.py di folder yang sama
python calibrate.py --square-mm 30  # jika ukuran kotak berbeda
python calibrate.py --fix-k3        # k3 dikunci 0
```
Foto diletakkan di `data/raw/` (jpg/jpeg/png, resolusi berapa pun).