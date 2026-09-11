# Catatan Pengalaman Karier
### Perjalanan dari Remote Developer menuju Senior Developer

---

## Ringkasan Singkat

| | |
|---|---|
| **Perusahaan** | Sama untuk kedua posisi |
| **Durasi sebagai Remote Developer** | ± 1 tahun |
| **Posisi saat ini** | Senior Developer |
| **Skala sistem** | 5.000+ pengguna harian, transaksi ratusan hingga jutaan ringgit per hari ketika live funding |

Secara garis besar, perjalanan kariernya dapat diringkas sebagai berikut:

> Sebagai **Remote Developer**, ia membangun fondasi sistem, melakukan migrasi dari platform legacy, dan merancang arsitektur untuk skalabilitas.
> Sebagai **Senior Developer**, ia memegang kepemilikan penuh atas sistem produksi: troubleshooting, business logic, integritas data, dan rewrite backend berskala besar.

Di luar pekerjaan utama, ia juga sedang mengembangkan **Loom**, sebuah framework pribadi untuk membangun REST API secara lebih cepat dan terstandarisasi (dijelaskan pada Bagian IV).

---

# Bagian I — Remote Developer (± 1 Tahun)

**Fokus utama:** pengembangan backend/API, migrasi sistem, pemrosesan asynchronous, performance engineering, pengembangan dashboard, testing, monitoring, dan deployment.

Pada fase ini, fokusnya adalah **membangun dan memperkuat fondasi sistem** — mulai dari migrasi legacy code, pengembangan dashboard, peningkatan performa, hingga membangun arsitektur untuk pemrosesan asynchronous dan skalabilitas jangka panjang.

## 1. Migrasi Arsitektur Backend

Ia memigrasi sistem legacy dari **LoopBack 3 ke LoopBack 4** menggunakan **TypeScript**, mencakup:

- Migrasi menyeluruh codebase dari LoopBack 3 ke LoopBack 4
- Refactoring model, repository, controller, dan service
- Penyesuaian ulang relasi antar-model
- Perbaikan penerapan *dependency injection*
- Penegakan prinsip *separation of concerns*
- Restrukturisasi endpoint agar lebih modular
- Pengurangan technical debt dari codebase lama
- Menjaga kompatibilitas dengan sistem yang sudah berjalan
- Pengembangan dan pemeliharaan REST API untuk kebutuhan internal maupun client-facing

Baginya, migrasi ini bukan sekadar perpindahan framework, melainkan kesempatan untuk merombak struktur backend agar lebih maintainable dan scalable dalam jangka panjang.

## 2. Autentikasi dan Keamanan API

Ia menyelesaikan dan memelihara lapisan autentikasi pada API.

**JWT Authentication**
- Mengimplementasikan JWT sebagai mekanisme autentikasi utama
- Mengoptimalkan alur autentikasi lintas layanan API
- Menangani autentikasi untuk berbagai dashboard
- Menerapkan otorisasi sesuai kebutuhan masing-masing aplikasi

**HMAC Authentication**
- Dikembangkan khusus untuk endpoint dan integrasi tertentu yang tidak menggunakan alur JWT standar

**Cakupan tambahan yang ditangani:**
- Validasi request
- Otorisasi
- Penanganan edge case
- Error handling
- Konsistensi response saat autentikasi/otorisasi gagal

## 3. Performance Engineering pada Skala Besar

Sistem yang ditanganinya melayani **5.000+ pengguna harian** dengan volume transaksi **ratusan hingga jutaan ringgit per hari** ketika live funding, sehingga performa menjadi prioritas utama.

**Redis sebagai caching layer**
- Mengurangi query database yang berulang
- Menekan beban database
- Mempercepat response time API
- Mengatur cache expiration dan invalidation
- Menjaga keseimbangan antara performa dan konsistensi data

**Pemrosesan asynchronous**
Proses yang membutuhkan waktu lama dialihkan dari alur synchronous menuju arsitektur:

```
API → Queue → Worker
```

sehingga API utama tetap responsif terhadap pengguna.

## 4. Arsitektur Queue & Worker

Ia mengerjakan sistem **Queue & Worker** menggunakan **Redis** dan **BullMQ**, dengan satu codebase yang memiliki dua entry point berbeda:

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

**`application.ts`** — entry point untuk API utama, penambahan job ke queue, dan penanganan request synchronous.

**`worker.ts`** — entry point khusus untuk mengambil job dari Redis, menjalankan proses asynchronous, background process, serta scheduled/cron task.

**Deployment:** API dan worker dijalankan pada container terpisah, tetap dalam satu codebase namun dengan entry point dan peran deployment yang berbeda — sehingga beban kerja API dan background processing dapat dipisah secara jelas.

## 5. Background Processing & Cron Jobs

Worker dimanfaatkan untuk menjalankan berbagai proses yang tidak perlu dijalankan secara synchronous oleh API, antara lain:

- Asynchronous jobs
- Background tasks
- Cron jobs
- Batch processing
- Notification processing
- Scheduled jobs
- Automated system jobs

Tujuannya: memindahkan beban kerja berat dari API agar request utama tidak perlu menunggu proses tersebut selesai.

## 6. Arsitektur Multi-Dashboard

Ia menyelesaikan pembaruan backend untuk mendukung beberapa dashboard dengan kebutuhan berbeda:

- 1 dashboard operasional/internal
- ± 6 dashboard client untuk kebutuhan investor/issuer

**Cakupan pekerjaannya:**
- Endpoint sesuai kebutuhan masing-masing dashboard
- Role dan permission
- Autentikasi dan otorisasi
- Segmentasi data berdasarkan channel
- Business logic yang reusable
- Integrasi API dan frontend

Fokus utamanya adalah menjaga agar seluruh dashboard tetap menggunakan pattern backend yang konsisten.

## 7. Pengembangan Frontend

Meski fokus utamanya adalah backend, ia juga terlibat langsung dalam pengembangan frontend menggunakan **React, TanStack, DaisyUI,** dan **Tailwind CSS** — mencakup dashboard internal, integrasi REST API, data fetching, state management, form handling, presentasi data tabular, serta komponen reusable untuk mendukung alur kerja operasional.

## 8. Migrasi ± 200 Form

Ia memimpin migrasi sekitar **200 form** dari **react-hook-form ke TanStack Form**. Pekerjaan ini tidak sebatas mengganti library, tetapi juga melakukan refactoring menyeluruh terhadap arsitektur form:

- Migrasi dan refactoring logic form
- Validasi dan error handling
- Standardisasi komponen form reusable
- Pengurangan duplikasi logic secara signifikan

## 9. Penerapan Atomic Design

Ia melakukan refactoring UI dengan pendekatan **Atomic Design** untuk menghasilkan komponen yang lebih reusable, mengurangi duplikasi UI, menstandarkan design pattern, dan mempermudah pengembangan serta maintenance fitur ke depannya.

## 10. Migrasi Svelte → React

Ia terlibat dalam proses porting aplikasi dari **Svelte ke React**, termasuk integrasi dengan **Directus CMS** — mencakup migrasi komponen, penyesuaian alur data/state, integrasi API, dan penyesuaian arsitektur komponen, sambil memastikan behavior aplikasi tetap konsisten pasca-migrasi.

## 11. MongoDB & Data Layer

**MongoDB** menjadi database utama sistem yang ditanganinya. Pekerjaannya mencakup desain schema/model, relasi antar-model, serta integrasi dengan LoopBack 4 dan Redis — dengan tujuan menjaga efisiensi akses data seiring pertumbuhan volume pengguna dan transaksi.

## 12. Testing

Ia mengembangkan **End-to-End Testing menggunakan Supertest**, mencakup alur kritikal seperti autentikasi, otorisasi, endpoint API, alur transaksi, background job, dan integrasi antar-komponen — untuk memastikan perubahan backend tidak merusak fungsionalitas inti bisnis.

## 13. Monitoring & Observability

Ia menggunakan **Loki** dan **Grafana** untuk centralized logging dan monitoring, yang membantunya memantau perilaku API dan background processing, menginvestigasi failed jobs, serta mempercepat debugging dan troubleshooting berdasarkan log operasional.

## 14. Docker & Containerization

Ia menggunakan **Docker** dan **Docker Compose**, dengan container yang dipisah berdasarkan workload:

```
Production / Staging
├── API Container
├── Worker Container
├── UI Container (6 dashboard client + 1 dashboard operasional)
└── Redis Container
```

Pemisahan ini membuat setiap komponen memiliki lifecycle dan workload yang lebih independen.

## 15. CI/CD & Deployment

Ia mengelola **CI/CD secara manual** untuk staging dan production — mencakup build aplikasi, build Docker image, configuration management, deployment, update container, serta troubleshooting deployment.

## 16. Production Troubleshooting

Ia mulai terlibat langsung dalam investigasi error API, debugging background job, troubleshooting worker, analisis log, investigasi performance issue, hingga perbaikan bug yang ditemukan melalui monitoring production.

---

# Bagian II — Senior Developer (Posisi Saat Ini)

Setelah melanjutkan di perusahaan yang sama, perannya berkembang menjadi **Senior Developer**. Fokus pekerjaannya pun bergeser secara signifikan.

Jika fase Remote Developer lebih banyak berkaitan dengan:

> **building, migration, architecture, optimization,** dan **feature development**

maka sebagai Senior Developer, fokus utamanya menjadi:

> **production ownership, troubleshooting, business logic, data integrity, operational support, custom reporting,** dan **system maintenance**

## 17. Production Bug Fixing

Ia menangani penyelesaian bug pada sistem production secara menyeluruh — mulai dari investigasi, penentuan root cause, perbaikan implementasi, memastikan fix tidak menimbulkan regresi, testing terhadap alur terkait, hingga deployment dan monitoring pasca-perubahan.

Sumber bug tidak selalu berasal dari code, tetapi juga bisa berasal dari business logic, data, perilaku API/dashboard, background processing, integrasi, maupun alur operasional.

## 18. Perbaikan Business Logic

Ia menangani berbagai kebutuhan perubahan dan perbaikan pada business logic, misalnya ketika:

- Behavior aplikasi tidak sesuai dengan kebutuhan bisnis
- Ada kondisi yang belum ditangani implementasi
- Logic yang ada memerlukan penyesuaian
- Business rule mengalami perubahan
- Ditemukan edge case yang sebelumnya belum diperhitungkan

Baginya, pekerjaan ini menuntut lebih dari sekadar memperbaiki syntax — ia perlu memahami **alasan di balik perilaku sistem tertentu**, lalu menerjemahkannya ke dalam implementasi yang tepat.

## 19. Production Troubleshooting Menyeluruh

Sebagai Senior Developer, troubleshooting menjadi bagian yang jauh lebih dominan dalam pekerjaannya. Alur investigasi yang dilakukannya biasanya menyusuri seluruh lapisan sistem:

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

Ia menelusuri masalah secara end-to-end untuk menemukan akar penyebab sebenarnya — yang tidak selalu berada di level code, melainkan bisa berasal dari data, business rule, incorrect state, proses operasional, integrasi, atau bahkan environment deployment.

## 20. Koreksi Data Manual

Ia menangani perbaikan data secara manual ketika suatu kondisi tidak dapat diselesaikan melalui workflow normal dashboard, dengan proses:

1. Investigasi kondisi data
2. Memahami penyebab ketidaksesuaian data
3. Menentukan data yang perlu dikoreksi
4. Melakukan koreksi
5. Memastikan konsistensi dengan business logic
6. Validasi hasil setelah koreksi

Pekerjaan ini menuntut pemahaman mendalam bukan hanya terhadap skema database, tetapi juga business rules dan siklus hidup data.

## 21. Custom Reporting

Ia membuat laporan khusus untuk kebutuhan bisnis yang tidak tersedia secara default di dashboard — mencakup pemahaman requirement, penentuan sumber data, pengambilan dan pengolahan data, agregasi/filtering sesuai kebutuhan, penyesuaian format laporan, hingga validasi hasil terhadap data aktual. Pekerjaan ini bersifat lebih ad-hoc dan berorientasi bisnis dibanding pengembangan fitur pada umumnya. Laporan dibuat dalam dua format, CSV dan XLSX, dibangun menggunakan code setelah mengakses data langsung dari database.

## 22. Full API Rewrite — Go + Echo

Salah satu pekerjaan terbesarnya pada fase ini adalah melakukan **rewrite menyeluruh terhadap API**, dari stack lama **LoopBack 4 + TypeScript** menjadi **Go + Echo**, dengan melanjutkan repository yang sudah ada. Alasan utamanya adalah penanganan transaction yang lebih mudah pada Go, sehingga dapat membantu dalam berbagai macam business logic pada sistem.

Pendekatan yang digunakan bukan membangun API baru dengan contract baru, melainkan:

> **Mengganti implementasi backend tanpa mengharuskan dashboard memahami perbedaan apa pun di baliknya.**

### 22.1 Mempertahankan Pola API yang Sudah Ada

Meski backend ditulis ulang menggunakan Go + Echo, seluruh pattern yang sudah digunakan dashboard tetap dipertahankan — termasuk pola endpoint, request, response, struktur data, behavior, dan pola integrasi.

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

Dashboard tidak perlu memiliki logic berbeda hanya karena implementasi backend berubah.

### 22.2 Filosofi di Balik Rewrite

Rewrite ini mencerminkan pendekatan **compatibility-first** yang dipegangnya. Alih-alih:

> Backend baru → contract API baru → dashboard harus diubah

pendekatan yang digunakan adalah:

> Pola dashboard yang sudah ada dipertahankan → implementasi backend yang diganti.

Dengan begitu, perubahan besar terjadi di lapisan backend, sementara dashboard tetap berjalan dengan pola yang sudah dikenalnya.

## 23. Go + Echo — Hasil Akhir

Implementasi API menggunakan Go + Echo yang dibangunnya pada akhirnya **tidak dilanjutkan penggunaannya**. Meski demikian, pengalaman ini tetap berharga karena mencakup:

- Pemahaman menyeluruh terhadap seluruh existing API
- Pemahaman dependency antar-endpoint
- Pemahaman posisi dashboard sebagai konsumen API
- Pemahaman mendalam terhadap business logic yang berjalan
- Proses rewrite backend secara menyeluruh
- Penjagaan kompatibilitas sistem
- Penyamaan behavior antara implementasi lama dan baru
- Troubleshooting sepanjang proses migrasi
- Pengambilan keputusan arsitektur berbasis kondisi sistem nyata, bukan sekadar preferensi teknologi

Dengan demikian, pengalaman ini lebih tepat dipandang sebagai pengalaman **system rewrite dan compatibility engineering**, bukan semata pengalaman menggunakan bahasa Go.

## 24. CI/CD & Deployment — Level Senior

CI/CD masih dijalankan secara manual, mencakup deployment staging dan production, build, Docker image, configuration, update service, hingga troubleshooting deployment.

Perbedaan utamanya: pada level Senior Developer, deployment yang dilakukan tidak lagi hanya terkait feature development, tetapi juga mencakup bug fix, koreksi business logic, koreksi data, perubahan API, hotfix production, dan maintenance rutin.

---

# Bagian III — Perbandingan dan Ringkasan

## 25. Peta Perkembangan Tanggung Jawab

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
       └── Manual CI/CD
```

## 26. Perbedaan Utama: Remote Developer vs Senior Developer

| Area | Remote Developer | Senior Developer |
|---|---|---|
| Backend | Build & migrasi | Maintenance & rewrite |
| Arsitektur | Merancang arsitektur | Menjaga & mengubah arsitektur tanpa breaking existing system |
| API | Development | Maintenance, troubleshooting & rewrite |
| Framework | LoopBack 3 → 4 | LoopBack 4 → Go/Echo |
| Performa | Redis & async processing | Troubleshooting performa production |
| Queue | Merancang Queue/Worker | Maintenance & troubleshooting |
| Database | Optimasi query/model | Investigasi & koreksi data manual |
| Dashboard | Develop & migrasi | Menjaga kompatibilitas |
| Business Logic | Implementasi fitur | Koreksi & pemeliharaan business rule |
| Bug | Fix issue development | Ownership bug production |
| Data | Alur aplikasi normal | Koreksi data manual |
| Report | — | Custom/ad-hoc reporting |
| Testing | Pengembangan E2E | Validasi/regresi untuk fix |
| Monitoring | Setup Loki/Grafana | Penggunaan monitoring untuk troubleshooting |
| Deployment | Manual CI/CD | Manual CI/CD + perubahan production |
| Migrasi API | LB3 → LB4 | LB4 → Go/Echo |
| Cakupan | Pengembangan fitur/sistem | Kepemilikan sistem end-to-end |

## 27. Keseluruhan Technology Stack

**Backend**
TypeScript · LoopBack 3 · LoopBack 4 · Go · Echo · REST API · JWT · HMAC

**Database & Caching**
MongoDB · Redis

**Queue & Background Processing**
BullMQ · Redis Queue · Worker · Cron Jobs · Background Jobs · Batch Processing · Asynchronous Processing

**Frontend**
React · Svelte · TanStack · TanStack Form · DaisyUI · Tailwind CSS · Directus CMS · Atomic Design

**Testing**
Supertest · E2E Testing

**Monitoring**
Loki · Grafana

**Infrastruktur**
Docker · Docker Compose · Containerized Deployment · Manual CI/CD · Staging & Production Deployment

---

# Bagian IV — Project Pribadi: Loom

Di luar pekerjaan utama, ia sedang mengembangkan **Loom**, sebuah framework pribadi untuk membangun REST API secara lebih cepat, konsisten, dan minim boilerplate.

## 28. Latar Belakang & Konsep Inti

Ide utama Loom lahir dari pengalaman berulang kali membangun endpoint CRUD yang polanya hampir selalu sama, namun tetap harus ditulis manual satu per satu di setiap proyek. Loom dirancang untuk mengatasi hal ini dengan pendekatan:

> **Terpusat pada interface MODEL dan rules per property, lalu CRUD dasar terbentuk secara otomatis.**

Alih-alih menulis controller, validasi, dan endpoint secara manual untuk setiap resource, cukup didefinisikan:

- **Interface MODEL** — struktur data/entity yang merepresentasikan resource
- **Rules per property** — aturan pada level masing-masing field (misalnya tipe data, wajib/opsional, validasi, format, relasi, dan batasan lainnya)

Dari dua definisi tersebut, Loom secara otomatis menghasilkan operasi **CRUD dasar** (Create, Read, Update, Delete) tanpa perlu menulis ulang logic yang berulang untuk setiap resource baru.

## 29. Tujuan Pengembangan

Beberapa tujuan yang ingin dicapai lewat Loom:

- Mempercepat proses scaffolding API baru
- Mengurangi duplikasi logic CRUD antar-resource
- Menstandarkan cara mendefinisikan validasi dan rules pada level property
- Memisahkan definisi "apa data ini" dari "bagaimana cara mengaksesnya"
- Menjadikan penambahan resource baru semudah mendefinisikan interface model-nya

## 30. Tech Stack

**Stack utama**
- **TypeScript**
- **Hono** — sebagai framework utama
- **Zod** - sebagai schema dan validator
- **MongoDB** — sebagai database

**Alternatif stack yang juga dieksplorasi**
- **Go**
- **MongoDB**

Eksplorasi dua stack ini memungkinkan perbandingan trade-off antara kecepatan pengembangan (TypeScript + Bun) dan performa/efisiensi runtime (Go), sambil tetap mempertahankan MongoDB sebagai database di kedua pendekatan.

## 31. Relevansi dengan Pengalaman Profesional

Loom cukup erat kaitannya dengan pengalaman profesional di pekerjaan utama — terutama dari sisi:

- Pengalaman merancang backend architecture (LoopBack 3/4, Go + Echo)
- Pemahaman mendalam terhadap pola desain REST API dan konsistensi contract
- Pengalaman menjaga compatibility API di berbagai layer konsumen (dashboard)
- Kebiasaan menstandarkan pattern agar mudah di-maintain dalam skala besar

Loom pada dasarnya adalah upaya menerapkan pelajaran dari pengalaman profesional — terutama soal pentingnya konsistensi pattern dan pengurangan duplikasi — ke dalam sebuah tool yang bisa dipakai ulang di proyek mana pun.

---

# Ringkasan Akhir

Perjalanan kariernya bukan sekadar:

**Remote Developer → Senior Developer**

tetapi lebih tepat digambarkan sebagai:

**Developer yang membangun sistem → Engineer yang memahami keseluruhan sistem → Engineer yang memiliki kepemilikan penuh atas sistem production — sambil terus mengasah pemikiran arsitektural lewat project pribadi seperti Loom.**
