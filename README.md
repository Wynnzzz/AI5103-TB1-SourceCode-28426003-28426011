# Perbandingan Genetic Algorithm dan Particle Swarm Optimization pada Optimal Power Flow Sistem IEEE 30-Bus

Tugas Proyek **AI5103 Fundamental AI dan Sains Data**

| Nama | NIM |
|---|---|
| Berwyn Izzut Taghyir | 28426003 |
| Varraz Hazzandra A | 28426011 |

## Deskripsi

Repositori ini memuat kode sumber untuk membandingkan *Genetic Algorithm* (GA) dan *Particle Swarm Optimization* (PSO) pada masalah *Optimal Power Flow* (OPF) sistem IEEE 30-bus. Tujuan optimasi adalah meminimalkan biaya bahan bakar pembangkit dengan tetap memenuhi batas operasi sistem. *Hill Climbing* dengan *random restart* (HC) digunakan sebagai pembanding dasar (*baseline*) pada anggaran evaluasi yang sama dengan GA dan PSO.

Seluruh implementasi ditulis dari awal menggunakan NumPy, termasuk pembentukan matriks admitansi bus (Ybus) dan penyelesaian aliran daya dengan metode Newton-Raphson.

## Studi Kasus

- **Sistem**: IEEE 30-bus (30 bus, 41 saluran, 6 generator), basis daya 100 MVA.
- **Total beban**: 283,4 MW dan 126,2 MVAr.
- **Sumber data jaringan**: [ITI/models, ieee-30-bus.m](https://github.com/ITI/models/blob/master/electric-grid/physical/reference/ieee-30bus/models/ieee-30-bus.m).
- **Fungsi objektif**: biaya bahan bakar kuadratik ditambah penalti pelanggaran batas,

  $$\min \sum_{i} \left(a_i + b_i P_{g,i} + c_i P_{g,i}^2\right) + PI$$

  Rugi daya dilaporkan terpisah dan tidak masuk ke fungsi objektif.

### Variabel keputusan (17 variabel kontrol)

| Variabel | Jumlah | Batas |
|---|---|---|
| Daya aktif generator PV (bus 2, 5, 8, 11, 13) | 5 | Pmin sampai Pmax |
| Tegangan generator (bus 1, 2, 5, 8, 11, 13) | 6 | 0,95 sampai 1,10 p.u. |
| Tap transformator (6-9, 6-10, 4-12, 28-27) | 4 | 0,90 sampai 1,10 |
| Kapasitor shunt (bus 10, 24) | 2 | 0 sampai 5 MVAr |

Bus 1 adalah *slack*, sehingga daya aktifnya dihitung dari hasil aliran daya dan bukan variabel keputusan.

### Penanganan batas (*constraint*)

Persamaan aliran daya dipenuhi melalui Newton-Raphson. Batas pertidaksamaan ditangani dengan metode penalti untuk:

- daya aktif generator slack,
- daya reaktif seluruh generator,
- tegangan bus beban (0,95 sampai 1,10 p.u.).

Solusi yang aliran dayanya tidak konvergen diberi nilai objektif 10⁶. Batas termal saluran **tidak** diperiksa.

## Metode

| Metode | Konfigurasi |
|---|---|
| GA | Populasi 100, 200 generasi, seleksi turnamen (ukuran 3), crossover aritmatika (pc = 0,9), mutasi Gaussian (pm = 0,1), elitisme 2 individu |
| PSO | 20 partikel, 200 iterasi, bobot inersia turun linear 0,9 ke 0,4, c1 = c2 = 2,0, kecepatan dibatasi 20% rentang variabel |
| HC (*random restart*) | Tetangga dibangkitkan dengan mutasi Gaussian pada sebagian variabel, langkah diperkecil saat stagnan, dan *restart* acak bila tetap macet |

### Rancangan eksperimen

- Lima *seed* independen: 42, 123, 456, 789, 2024.
- Empat konfigurasi, masing-masing dijalankan lima kali (total 20 run):

  | Konfigurasi | Anggaran evaluasi objektif |
  |---|---|
  | GA | 20.000 |
  | PSO | 4.020 |
  | HC setara PSO | 4.020 |
  | HC setara GA | 20.000 |

- Setiap solusi akhir diaudit ulang dengan toleransi kelayakan 0,001 MW/MVAr untuk daya dan 0,0001 p.u. untuk tegangan.
- *Gap* dihitung terhadap biaya layak terendah dari seluruh run. Nilai ini adalah acuan empiris dari eksperimen ini, bukan optimum global maupun *best-known* dari literatur. Variabel `REFERENCE_VALUE` dan `REFERENCE_SOURCE` pada notebook dapat diisi bila acuan literatur tersedia.

## Hasil

Seluruh 20 run menghasilkan solusi layak. Biaya bahan bakar ($/jam) pada lima *seed*:

| Metode | Terbaik | Rata-rata | Simpangan baku | Terburuk | Gap rata-rata (%) |
|---|---|---|---|---|---|
| GA | 799,432346 | 799,444015 | 0,011340 | 799,462653 | 0,002609 |
| PSO | **799,423159** | 799,498757 | 0,114901 | 799,698998 | 0,009457 |
| HC setara PSO | 799,440075 | 799,451912 | 0,018414 | 799,484204 | 0,003597 |
| HC setara GA | 799,430241 | **799,437748** | **0,004802** | 799,441513 | **0,001825** |

Waktu komputasi per run sekitar 35 sampai 42 detik untuk GA dan 7 sampai 8 detik untuk PSO (bergantung perangkat keras).

Catatan pembacaan hasil:

- PSO memperoleh biaya terbaik tunggal, tetapi variasi antar-*seed* paling besar.
- GA lebih stabil daripada PSO, tetapi memakai anggaran evaluasi sekitar 5 kali lipat dan waktu komputasi sekitar 5 kali lipat.
- Selisih biaya antar metode sangat kecil (di bawah 0,03% terhadap acuan), dan HC dengan anggaran setara tetap kompetitif. Perbedaan ini belum diuji signifikansi statistiknya.
- Hasil berlaku untuk satu instans (IEEE 30-bus) dan lima *seed*.

## Struktur Repositori

```
.
├── AI5103-TB1-SourceCode-28426003-28426011.ipynb   # kode sumber dan eksperimen
├── README.md
└── (opsional) AI5103-TB1-Laporan-28426003-28426011.pdf
```

Isi notebook secara berurutan:

1. Data jaringan IEEE 30-bus (bus, saluran, generator).
2. Pembentukan Ybus dan aliran daya Newton-Raphson.
3. Variabel keputusan, fungsi objektif, dan penalti.
4. Implementasi HC, GA, dan PSO.
5. Eksperimen lima *seed*, audit kelayakan, statistik, dan *gap*.
6. Visualisasi: kurva konvergensi, sebaran biaya per *seed*, profil tegangan, dan diagram jaringan dengan pengaturan hasil optimasi.

## Cara Menjalankan

Prasyarat: Python 3.9 atau lebih baru.

```bash
pip install numpy pandas matplotlib jupyter
jupyter notebook AI5103-TB1-SourceCode-28426003-28426011.ipynb
```

Jalankan seluruh sel secara berurutan (*Run All*). Dataset sudah tertanam di dalam notebook, sehingga tidak ada berkas data eksternal. Seluruh eksperimen memerlukan beberapa menit karena 20 run optimasi dijalankan, dan hasil dapat direproduksi karena *seed* ditetapkan secara eksplisit.

## Keterbatasan

- Batas termal saluran tidak dimodelkan.
- Penalti memakai bobot tetap dan tidak di-*tuning*.
- Hiperparameter GA, PSO, dan HC tidak dioptimalkan lebih lanjut.
- Acuan *gap* bersifat empiris dan berasal dari run pada eksperimen ini.
