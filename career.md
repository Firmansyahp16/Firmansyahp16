# Catatan Pengalaman Karier: Perjalanan dari Remote Developer menuju Senior Developer

---

## Abstrak

Dokumen ini memaparkan perjalanan karier seorang pengembang perangkat lunak pada satu perusahaan yang sama, dari posisi Remote Developer selama kurang lebih satu tahun hingga posisi Senior Developer pada saat ini. Sistem yang ditangani melayani lebih dari 5.000 pengguna harian dengan volume transaksi ratusan hingga jutaan ringgit per hari pada periode live funding. Pada fase Remote Developer, penulis berfokus pada pembangunan fondasi sistem, migrasi dari platform legacy, serta perancangan arsitektur yang skalabel. Pada fase Senior Developer, penulis memegang kepemilikan penuh atas sistem produksi, yang mencakup penanganan masalah (troubleshooting), pemeliharaan business logic, integritas data, implementasi flow baru (Checker Flow), dan penulisan ulang (rewrite) backend berskala besar. Di luar pekerjaan utama, penulis mengembangkan Loom, yaitu framework pribadi untuk membangun REST API secara lebih cepat dan terstandarisasi.

**Kata kunci:** backend engineering, migrasi sistem, arsitektur queue dan worker, production ownership, Checker Flow, compatibility engineering, REST API

## Ringkasan Data

| Aspek | Keterangan |
|---|---|
| Perusahaan | Sama untuk kedua posisi |
| Durasi sebagai Remote Developer | ± 1 tahun |
| Posisi saat ini | Senior Developer |
| Skala sistem | 5.000+ pengguna harian; transaksi ratusan hingga jutaan ringgit per hari ketika live funding |

---

# Bagian I: Fase Remote Developer (± 1 Tahun)

Fokus utama pada fase ini adalah pengembangan backend/API, migrasi sistem, pemrosesan asynchronous, performance engineering, pengembangan dashboard, pengujian, monitoring, dan deployment. Secara umum, fase ini diarahkan pada pembangunan dan penguatan fondasi sistem, mulai dari migrasi legacy code, pengembangan dashboard, peningkatan performa, hingga perancangan arsitektur untuk pemrosesan asynchronous dan skalabilitas jangka panjang.

## 1. Migrasi Arsitektur Backend

Sistem legacy dimigrasikan dari LoopBack 3 ke LoopBack 4 dengan menggunakan TypeScript. Pekerjaan ini mencakup:

- migrasi menyeluruh codebase dari LoopBack 3 ke LoopBack 4;
- refactoring model, repository, controller, dan service;
- penyesuaian ulang relasi antar-model;
- perbaikan penerapan *dependency injection*;
- penegakan prinsip *separation of concerns*;
- restrukturisasi endpoint agar lebih modular;
- pengurangan technical debt pada codebase lama;
- pemeliharaan kompatibilitas dengan sistem yang sudah berjalan; dan
- pengembangan serta pemeliharaan REST API untuk kebutuhan internal maupun client-facing.

Migrasi ini dipandang tidak hanya sebagai perpindahan framework, tetapi juga sebagai kesempatan untuk merombak struktur backend agar lebih mudah dipelihara (maintainable) dan dikembangkan (scalable) dalam jangka panjang.

## 2. Autentikasi dan Keamanan API

Lapisan autentikasi pada API diselesaikan dan dipelihara oleh penulis.

**JWT Authentication.** JWT diimplementasikan sebagai mekanisme autentikasi utama. Alur autentikasi lintas layanan API dioptimalkan, autentikasi untuk berbagai dashboard ditangani, dan otorisasi diterapkan sesuai kebutuhan masing-masing aplikasi.

**HMAC Authentication.** Mekanisme ini dikembangkan khusus untuk endpoint dan integrasi tertentu yang tidak menggunakan alur JWT standar.

**Cakupan tambahan** meliputi validasi request, otorisasi, penanganan edge case, error handling, dan konsistensi response ketika autentikasi atau otorisasi gagal.

## 3. Performance Engineering pada Skala Besar

Sistem melayani lebih dari 5.000 pengguna harian dengan volume transaksi ratusan hingga jutaan ringgit per hari ketika live funding, sehingga performa menjadi prioritas utama.

**Redis sebagai caching layer.** Penggunaan Redis bertujuan mengurangi query database yang berulang, menekan beban database, dan mempercepat response time API. Cache expiration dan invalidation diatur untuk menjaga keseimbangan antara performa dan konsistensi data.

**Pemrosesan asynchronous.** Proses yang memerlukan waktu lama dialihkan dari alur synchronous ke arsitektur berikut:

```
API → Queue → Worker
```

Dengan demikian, API utama tetap responsif terhadap pengguna.

## 4. Arsitektur Queue dan Worker

Sistem Queue dan Worker dibangun dengan Redis dan BullMQ. Satu codebase digunakan dengan dua entry point berbeda, sebagaimana ditunjukkan pada Gambar 1.

```
                    ┌─────────────────────┐
                    │      API / Queue     │
                    │    application.ts    │
                    └──────────┬───────────┘
                               │
                               ▼
                        ┌─────────────┐
                        │    Redis    │
                        │    Queue    │
                        └──────┬──────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │        Worker        │
                    │      worker.ts       │
                    └─────────────────────┘
```

*Gambar 1. Arsitektur Queue dan Worker dengan dua entry point.*

- `application.ts` berfungsi sebagai entry point untuk API utama, penambahan job ke queue, dan penanganan request synchronous.
- `worker.ts` berfungsi sebagai entry point khusus untuk mengambil job dari Redis serta menjalankan proses asynchronous, background process, dan scheduled/cron task.

Pada tahap deployment, API dan worker dijalankan pada container terpisah, meskipun tetap berada dalam satu codebase. Pemisahan ini memungkinkan beban kerja API dan background processing dipisahkan secara jelas.

## 5. Background Processing dan Cron Jobs

Worker dimanfaatkan untuk menjalankan proses yang tidak perlu dieksekusi secara synchronous oleh API, antara lain asynchronous jobs, background tasks, cron jobs, batch processing, notification processing, scheduled jobs, dan automated system jobs. Tujuannya adalah memindahkan beban kerja berat dari API sehingga request utama tidak perlu menunggu proses tersebut selesai.

## 6. Arsitektur Multi-Dashboard

Pembaruan backend diselesaikan untuk mendukung beberapa dashboard dengan kebutuhan berbeda, yaitu satu dashboard operasional/internal dan sekitar enam dashboard client untuk kebutuhan investor/issuer. Cakupan pekerjaannya meliputi:

- endpoint sesuai kebutuhan masing-masing dashboard;
- role dan permission;
- autentikasi dan otorisasi;
- segmentasi data berdasarkan channel;
- business logic yang reusable; dan
- integrasi API dan frontend.

Fokus utamanya adalah menjaga agar seluruh dashboard tetap menggunakan pattern backend yang konsisten.

## 7. Pengembangan Frontend

Meskipun fokus utama berada pada backend, penulis terlibat langsung dalam pengembangan frontend dengan React, TanStack, DaisyUI, dan Tailwind CSS. Pekerjaan tersebut mencakup dashboard internal, integrasi REST API, data fetching, state management, form handling, presentasi data tabular, dan komponen reusable untuk mendukung alur kerja operasional.

## 8. Migrasi ± 200 Form

Migrasi sekitar 200 form dari react-hook-form ke TanStack Form dipimpin oleh penulis. Pekerjaan ini tidak terbatas pada penggantian library, tetapi juga mencakup refactoring menyeluruh terhadap arsitektur form, yaitu:

- migrasi dan refactoring logic form;
- validasi dan error handling;
- standardisasi komponen form yang reusable; dan
- pengurangan duplikasi logic secara signifikan.

## 9. Penerapan Atomic Design

Refactoring UI dilakukan dengan pendekatan Atomic Design untuk menghasilkan komponen yang lebih reusable, mengurangi duplikasi UI, menstandarkan design pattern, dan mempermudah pengembangan serta pemeliharaan fitur ke depan.

## 10. Migrasi Svelte ke React

Penulis terlibat dalam proses porting aplikasi dari Svelte ke React, termasuk integrasi dengan Directus CMS. Pekerjaan ini mencakup migrasi komponen, penyesuaian alur data dan state, integrasi API, serta penyesuaian arsitektur komponen, dengan tetap memastikan bahwa behavior aplikasi konsisten setelah migrasi.

## 11. MongoDB dan Data Layer

MongoDB merupakan database utama pada sistem yang ditangani. Pekerjaan pada lapisan ini mencakup desain schema/model, relasi antar-model, dan integrasi dengan LoopBack 4 serta Redis, dengan tujuan menjaga efisiensi akses data seiring pertumbuhan volume pengguna dan transaksi.

## 12. Pengujian

End-to-End Testing dikembangkan menggunakan Supertest. Pengujian mencakup alur kritikal seperti autentikasi, otorisasi, endpoint API, alur transaksi, background job, dan integrasi antar-komponen, untuk memastikan bahwa perubahan pada backend tidak merusak fungsionalitas inti bisnis.

## 13. Monitoring dan Observability

Loki dan Grafana digunakan untuk centralized logging dan monitoring. Kedua perangkat tersebut membantu pemantauan perilaku API dan background processing, investigasi failed jobs, serta percepatan proses debugging dan troubleshooting berdasarkan log operasional.

## 14. Docker dan Containerization

Docker dan Docker Compose digunakan dengan container yang dipisahkan berdasarkan workload:

```
Production / Staging
├── API Container
├── Worker Container
├── UI Container (6 dashboard client + 1 dashboard operasional)
└── Redis Container
```

Pemisahan ini membuat setiap komponen memiliki lifecycle dan workload yang lebih independen.

## 15. CI/CD dan Deployment

CI/CD dikelola secara manual untuk lingkungan staging dan production. Cakupannya meliputi build aplikasi, build Docker image, configuration management, deployment, pembaruan container, dan troubleshooting deployment.

## 16. Production Troubleshooting

Pada akhir fase ini, penulis mulai terlibat langsung dalam investigasi error API, debugging background job, troubleshooting worker, analisis log, investigasi masalah performa, dan perbaikan bug yang ditemukan melalui monitoring production.

---

# Bagian II: Fase Senior Developer (Posisi Saat Ini)

Setelah melanjutkan karier di perusahaan yang sama, peran penulis berkembang menjadi Senior Developer, dan fokus pekerjaan bergeser secara signifikan. Fase Remote Developer lebih banyak berkaitan dengan:

> *building, migration, architecture, optimization,* dan *feature development*,

sedangkan fokus utama pada fase Senior Developer adalah:

> *production ownership, troubleshooting, business logic, data integrity, operational support, custom reporting,* dan *system maintenance*.

## 17. Production Bug Fixing

Bug pada sistem production ditangani secara menyeluruh, mulai dari investigasi, penentuan root cause, perbaikan implementasi, pemastian bahwa perbaikan tidak menimbulkan regresi, pengujian terhadap alur terkait, hingga deployment dan monitoring pascaperubahan. Sumber bug tidak selalu berasal dari code, tetapi dapat pula berasal dari business logic, data, perilaku API/dashboard, background processing, integrasi, maupun alur operasional.

## 18. Perbaikan Business Logic

Perubahan dan perbaikan business logic ditangani ketika:

- behavior aplikasi tidak sesuai dengan kebutuhan bisnis;
- terdapat kondisi yang belum ditangani oleh implementasi;
- logic yang ada memerlukan penyesuaian;
- business rule mengalami perubahan; atau
- ditemukan edge case yang sebelumnya belum diperhitungkan.

Pekerjaan ini menuntut lebih dari sekadar perbaikan syntax, karena alasan di balik perilaku sistem tertentu perlu dipahami terlebih dahulu sebelum diterjemahkan ke dalam implementasi yang tepat.

## 19. Production Troubleshooting Menyeluruh

Pada fase ini, troubleshooting menjadi bagian yang jauh lebih dominan. Investigasi umumnya menelusuri seluruh lapisan sistem, sebagaimana ditunjukkan pada Gambar 2.

```
Dashboard
   ↓
API
   ↓
Business Logic
   ↓
Database
   ↓
Background Process
   ↓
Data Eksternal/Operasional
```

*Gambar 2. Alur investigasi troubleshooting end-to-end.*

Penelusuran dilakukan secara end-to-end untuk menemukan akar penyebab sebenarnya. Akar penyebab tidak selalu berada pada level code, tetapi dapat berasal dari data, business rule, state yang tidak tepat, proses operasional, integrasi, atau environment deployment.

## 20. Koreksi Data Manual

Koreksi data secara manual dilakukan ketika suatu kondisi tidak dapat diselesaikan melalui workflow normal dashboard. Prosesnya terdiri atas:

1. investigasi kondisi data;
2. identifikasi penyebab ketidaksesuaian data;
3. penentuan data yang perlu dikoreksi;
4. pelaksanaan koreksi;
5. pemastian konsistensi dengan business logic; dan
6. validasi hasil setelah koreksi.

Pekerjaan ini memerlukan pemahaman mendalam tidak hanya terhadap skema database, tetapi juga terhadap business rules dan siklus hidup data.

## 21. Custom Reporting

Laporan khusus dibuat untuk kebutuhan bisnis yang tidak tersedia secara default pada dashboard. Tahapannya mencakup pemahaman requirement, penentuan sumber data, pengambilan dan pengolahan data, agregasi atau filtering sesuai kebutuhan, penyesuaian format laporan, dan validasi hasil terhadap data aktual. Pekerjaan ini bersifat lebih ad-hoc dan berorientasi bisnis dibandingkan pengembangan fitur pada umumnya. Laporan dihasilkan dalam dua format, yaitu CSV dan XLSX, dengan code yang dibangun setelah data diakses langsung dari database.

## 22. Full API Rewrite: Go dan Echo

Salah satu pekerjaan terbesar pada fase ini adalah penulisan ulang (rewrite) API secara menyeluruh, dari stack LoopBack 4 dan TypeScript menjadi Go dan Echo, dengan melanjutkan repository yang sudah ada. Alasan utamanya adalah penanganan transaction yang lebih mudah pada Go, sehingga mendukung berbagai business logic pada sistem.

Pendekatan yang digunakan tidak membangun API baru dengan contract baru, melainkan:

> mengganti implementasi backend tanpa mengharuskan dashboard memahami perbedaan apa pun di baliknya.

### 22.1 Pemertahanan Pola API yang Sudah Ada

Meskipun backend ditulis ulang, seluruh pattern yang telah digunakan dashboard dipertahankan, termasuk pola endpoint, request, response, struktur data, behavior, dan pola integrasi (Gambar 3). Dengan demikian, dashboard tidak memerlukan logic berbeda akibat perubahan implementasi backend.

```
                    Dashboard
                       │
              Existing API Pattern
                       │
             ┌─────────┴─────────┐
             │                   │
             ▼                   ▼
        LoopBack 4            Go + Echo
             │                   │
             └─────────┬─────────┘
                       │
                Consistent UI
```

*Gambar 3. Pemertahanan pola API pada implementasi lama dan baru.*

### 22.2 Landasan Pendekatan Rewrite

Rewrite ini mencerminkan pendekatan *compatibility-first*. Pendekatan konvensional adalah:

> backend baru → contract API baru → dashboard harus diubah.

Pendekatan yang diterapkan adalah:

> pola dashboard yang sudah ada dipertahankan → implementasi backend diganti.

Dengan demikian, perubahan besar terjadi pada lapisan backend, sedangkan dashboard tetap berjalan dengan pola yang telah dikenalnya.

## 23. Go dan Echo: Hasil Akhir

Implementasi API dengan Go dan Echo pada akhirnya tidak dilanjutkan penggunaannya. Meskipun demikian, pengalaman ini tetap bernilai karena mencakup:

- pemahaman menyeluruh terhadap seluruh existing API;
- pemahaman dependency antar-endpoint;
- pemahaman posisi dashboard sebagai konsumen API;
- pemahaman mendalam terhadap business logic yang berjalan;
- proses rewrite backend secara menyeluruh;
- penjagaan kompatibilitas sistem;
- penyamaan behavior antara implementasi lama dan baru;
- troubleshooting sepanjang proses migrasi; dan
- pengambilan keputusan arsitektur berdasarkan kondisi sistem nyata, bukan sekadar preferensi teknologi.

Oleh karena itu, pengalaman ini lebih tepat dipandang sebagai pengalaman *system rewrite* dan *compatibility engineering*, bukan semata-mata pengalaman menggunakan bahasa Go.

## 24. CI/CD dan Deployment pada Level Senior

CI/CD masih dijalankan secara manual, mencakup deployment staging dan production, build, Docker image, configuration, pembaruan service, dan troubleshooting deployment. Perbedaan utamanya adalah bahwa pada level Senior Developer, deployment tidak lagi terbatas pada feature development, tetapi juga mencakup bug fix, koreksi business logic, koreksi data, perubahan API, hotfix production, dan maintenance rutin.

## 25. Checker Flow (Maker, Checker, Viewer)

Analisis dan implementasi dilakukan terhadap sebuah flow baru di luar flow normal sistem, yaitu Checker Flow. Flow ini melibatkan tiga role, dan masing-masing role terikat pada tepat satu InvestorApplication (IA). Pada flow ini, pemilik IA tidak diizinkan membuat investment atau withdrawal secara langsung. Hak akses setiap role dirangkum pada Tabel 1.

*Tabel 1. Hak akses pada Checker Flow*

| Role | Hak akses |
|---|---|
| Maker | Membuat request withdrawal atau investment, serta membatalkan (cancel) request yang belum di-confirm atau di-reject oleh Checker |
| Checker | Melakukan confirm atau reject terhadap request; confirm menghasilkan investment atau withdrawal sesuai request |
| Viewer | Hanya melihat, tanpa melakukan action apa pun |

### 25.1 Status Request

Status request pada Checker Flow dijelaskan pada Tabel 2.

*Tabel 2. Status request*

| Status | Keterangan |
|---|---|
| Pending Confirmation | Menunggu keputusan Checker |
| Confirmed | Disetujui oleh Checker |
| Rejected | Ditolak oleh Checker |
| Cancelled | Dibatalkan oleh Maker sebelum diputuskan |
| Processed | Status sementara: request telah di-confirm, tetapi proses pembuatan investment atau withdrawal masih berjalan |

### 25.2 Audit Trail

Setiap event terhubung dengan satu request, dan seluruh jenis action disimpan.

### 25.3 Cakupan Implementasi

Implementasi mencakup penambahan properti baru, pengecekan authorization per role, perubahan business logic, dan penambahan transactional secara menyeluruh pada Checker Flow.

### 25.4 Alasan Penggunaan Transaction

Transaction diperlukan untuk mencegah inkonsistensi perubahan data apabila terjadi error atau kesalahan yang menyebabkan proses berhenti di tengah jalan. Hal ini sangat penting karena sistem berkaitan dengan transaksi finansial.

---

# Bagian III: Analisis Komparatif dan Ringkasan

## 26. Peta Perkembangan Tanggung Jawab

```
REMOTE DEVELOPER
       │
       ├── Build Backend
       ├── LoopBack 3 → LoopBack 4
       ├── REST API
       ├── JWT / HMAC
       ├── Redis
       ├── BullMQ
       ├── Queue / Worker
       ├── Cron Jobs
       ├── MongoDB
       ├── React Dashboard
       ├── Svelte → React
       ├── Migrasi ± 200 Form
       ├── Atomic Design
       ├── E2E Testing
       ├── Loki / Grafana
       ├── Docker
       └── Manual CI/CD
                    │
                    ▼
             SENIOR DEVELOPER
                    │
       ├── Production Ownership
       ├── Bug Fixing
       ├── Business Logic
       ├── Troubleshooting
       ├── Root Cause Analysis
       ├── Koreksi Data Manual
       ├── Custom Reporting
       ├── API Maintenance
       ├── Full API Rewrite
       ├── Go + Echo
       ├── API Compatibility
       ├── Dashboard Compatibility
       ├── Checker Flow (Maker / Checker / Viewer)
       ├── Transactional Flow
       └── Manual CI/CD
```

*Gambar 4. Peta perkembangan tanggung jawab.*

## 27. Perbedaan Utama antara Fase Remote Developer dan Senior Developer

*Tabel 3. Perbandingan fase Remote Developer dan Senior Developer*

| Area | Remote Developer | Senior Developer |
|---|---|---|
| Backend | Build dan migrasi | Maintenance dan rewrite |
| Arsitektur | Perancangan arsitektur | Pemeliharaan dan perubahan arsitektur tanpa merusak sistem yang ada |
| API | Development | Maintenance, troubleshooting, dan rewrite |
| Framework | LoopBack 3 → 4 | LoopBack 4 → Go/Echo |
| Performa | Redis dan async processing | Troubleshooting performa production |
| Queue | Perancangan Queue/Worker | Maintenance dan troubleshooting |
| Database | Optimasi query/model | Investigasi dan koreksi data manual |
| Dashboard | Pengembangan dan migrasi | Pemeliharaan kompatibilitas |
| Business Logic | Implementasi fitur | Koreksi dan pemeliharaan business rule; flow baru (Checker Flow) |
| Bug | Perbaikan isu development | Ownership bug production |
| Data | Alur aplikasi normal | Koreksi data manual |
| Report | — | Custom/ad-hoc reporting |
| Testing | Pengembangan E2E | Validasi/regresi untuk perbaikan |
| Monitoring | Setup Loki/Grafana | Pemanfaatan monitoring untuk troubleshooting |
| Deployment | Manual CI/CD | Manual CI/CD dan perubahan production |
| Migrasi API | LB3 → LB4 | LB4 → Go/Echo |
| Konsistensi data | Caching dan invalidation | Transactional flow pada proses finansial |
| Cakupan | Pengembangan fitur/sistem | Kepemilikan sistem end-to-end |

## 28. Keseluruhan Technology Stack

- **Backend:** TypeScript, LoopBack 3, LoopBack 4, Go, Echo, REST API, JWT, HMAC
- **Database dan caching:** MongoDB, Redis
- **Queue dan background processing:** BullMQ, Redis Queue, Worker, Cron Jobs, Background Jobs, Batch Processing, Asynchronous Processing
- **Frontend:** React, Svelte, TanStack, TanStack Form, DaisyUI, Tailwind CSS, Directus CMS, Atomic Design
- **Pengujian:** Supertest, E2E Testing
- **Monitoring:** Loki, Grafana
- **Infrastruktur:** Docker, Docker Compose, Containerized Deployment, Manual CI/CD, Staging dan Production Deployment

---

# Bagian IV: Proyek Pribadi: Loom

Di luar pekerjaan utama, penulis mengembangkan Loom, yaitu framework pribadi untuk membangun REST API secara lebih cepat, konsisten, dan minim boilerplate.

## 29. Latar Belakang dan Konsep Inti

Gagasan Loom bersumber dari pengalaman membangun endpoint CRUD yang berpola hampir sama secara berulang, yang tetap harus ditulis manual pada setiap proyek. Loom dirancang dengan pendekatan:

> terpusat pada interface MODEL dan rules per property, kemudian CRUD dasar terbentuk secara otomatis.

Sebagai pengganti penulisan controller, validasi, dan endpoint secara manual untuk setiap resource, hanya dua hal yang perlu didefinisikan:

- **Interface MODEL**, yaitu struktur data/entity yang merepresentasikan resource; dan
- **Rules per property**, yaitu aturan pada level masing-masing field (misalnya tipe data, wajib/opsional, validasi, format, relasi, dan batasan lainnya).

Berdasarkan kedua definisi tersebut, Loom menghasilkan operasi CRUD dasar (Create, Read, Update, Delete) secara otomatis, tanpa penulisan ulang logic yang berulang untuk setiap resource baru.

## 30. Tujuan Pengembangan

Tujuan pengembangan Loom adalah:

- mempercepat proses scaffolding API baru;
- mengurangi duplikasi logic CRUD antar-resource;
- menstandarkan cara mendefinisikan validasi dan rules pada level property;
- memisahkan definisi "apa data ini" dari "bagaimana data diakses"; dan
- menjadikan penambahan resource baru semudah mendefinisikan interface model.

## 31. Technology Stack

**Stack utama:** TypeScript; Hono sebagai framework utama; Zod sebagai schema dan validator; dan MongoDB sebagai database.

**Stack alternatif yang dieksplorasi:** Go dan MongoDB.

Eksplorasi kedua stack ini memungkinkan perbandingan trade-off antara kecepatan pengembangan (TypeScript dan Bun) dan performa atau efisiensi runtime (Go), dengan tetap mempertahankan MongoDB sebagai database pada kedua pendekatan.

## 32. Relevansi dengan Pengalaman Profesional

Loom berkaitan erat dengan pengalaman profesional pada pekerjaan utama, terutama pada aspek berikut:

- pengalaman merancang arsitektur backend (LoopBack 3/4, Go dan Echo);
- pemahaman mendalam terhadap pola desain REST API dan konsistensi contract;
- pengalaman menjaga compatibility API pada berbagai layer konsumen (dashboard); dan
- kebiasaan menstandarkan pattern agar mudah dipelihara pada skala besar.

Pada dasarnya, Loom merupakan upaya menerapkan pelajaran dari pengalaman profesional, khususnya mengenai pentingnya konsistensi pattern dan pengurangan duplikasi, ke dalam sebuah perangkat yang dapat digunakan ulang pada proyek apa pun.

---

# Kesimpulan

Perjalanan karier yang dipaparkan tidak terbatas pada perpindahan dari Remote Developer menjadi Senior Developer. Perjalanan tersebut lebih tepat dipahami sebagai perkembangan dari pengembang yang membangun sistem, menjadi engineer yang memahami keseluruhan sistem, hingga menjadi engineer yang memegang kepemilikan penuh atas sistem production. Perkembangan ini disertai dengan pengasahan pemikiran arsitektural secara berkelanjutan melalui proyek pribadi seperti Loom.
