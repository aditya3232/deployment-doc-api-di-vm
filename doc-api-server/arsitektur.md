# Layered Architecture

## Overview

Project ini menggunakan pendekatan **Layered Architecture** dengan beberapa prinsip dari **Clean Architecture / Hexagonal Architecture**.

Tujuan utama dari struktur ini adalah memisahkan aplikasi berdasarkan tanggung jawabnya sehingga setiap bagian memiliki fungsi yang jelas.

Secara umum, arsitektur aplikasi dapat digambarkan sebagai berikut:

```text
┌──────────────────────────────────────────────┐
│                  Delivery                    │
│          HTTP Handler / Request / Response   │
├──────────────────────────────────────────────┤
│               Application                    │
│                  Service                     │
├──────────────────────────────────────────────┤
│                  Domain                      │
│            Entity / Business Model           │
├──────────────────────────────────────────────┤
│               Infrastructure                 │
│       Repository / Storage / External        │
└──────────────────────────────────────────────┘
```

Pada project ini, layer tersebut direpresentasikan melalui package:

```text
internal/
├── adapter/
├── app/
├── core/
└── middleware/
```

---

# 1. Project Structure

Struktur project secara keseluruhan:

```text
.
├── arsitektur.md
├── cmd
│   ├── root.go
│   └── start.go
│
├── config
│   ├── config.go
│   ├── database.go
│   ├── fiber.go
│   ├── minio.go
│   ├── redis.go
│   └── zerolog.go
│
├── database
│   ├── migrations
│   │   ├── 000001_create_users_table.down.sql
│   │   └── 000001_create_users_table.up.sql
│   └── seeds
│       └── user_seed.go
│
├── deployment
│   └── docker-compose.yml
│
├── Dockerfile
├── go.mod
├── go.sum
│
├── internal
│   ├── adapter
│   │   ├── handler
│   │   │   ├── health_check_handler.go
│   │   │   ├── request
│   │   │   │   └── user_request.go
│   │   │   ├── response
│   │   │   │   ├── default_response.go
│   │   │   │   └── user_response.go
│   │   │   ├── upload_image_handler.go
│   │   │   └── user_handler.go
│   │   │
│   │   ├── repository
│   │   │   └── user_repository.go
│   │   │
│   │   └── storage
│   │       └── minio.go
│   │
│   ├── app
│   │   └── app.go
│   │
│   ├── core
│   │   ├── domain
│   │   │   ├── entity
│   │   │   │   ├── health_check_entity.go
│   │   │   │   ├── jwt_entity.go
│   │   │   │   └── user_entity.go
│   │   │   │
│   │   │   └── model
│   │   │       └── user_model.go
│   │   │
│   │   └── service
│   │       ├── health_check_service.go
│   │       ├── jwt_service.go
│   │       └── user_service.go
│   │
│   └── middleware
│       └── middleware_adapter.go
│
├── main.go
└── utils
    └── conv.go
```

---

# 2. Layer Architecture

Secara logical, struktur tersebut dapat digambarkan menjadi:

```text
                    HTTP Request
                         │
                         ▼
              ┌─────────────────────┐
              │      Handler        │
              │ adapter/handler     │
              └──────────┬──────────┘
                         │
                         ▼
              ┌─────────────────────┐
              │      Service        │
              │   core/service      │
              └──────────┬──────────┘
                         │
                         ▼
              ┌─────────────────────┐
              │       Domain        │
              │ core/domain/entity  │
              │ core/domain/model   │
              └──────────┬──────────┘
                         │
                         ▼
              ┌─────────────────────┐
              │    Repository       │
              │ adapter/repository  │
              └──────────┬──────────┘
                         │
                         ▼
                    PostgreSQL
```

Untuk kebutuhan storage:

```text
Service
   │
   ▼
Storage Adapter
   │
   ▼
MinIO
```

---

# 3. `main.go`

```text
main.go
```

`main.go` merupakan entry point utama aplikasi.

Tanggung jawabnya biasanya adalah menjalankan aplikasi:

```text
main.go
   │
   ▼
cmd
   │
   ▼
app
```

`main.go` sebaiknya tidak berisi business logic.

Prinsipnya:

> `main.go` hanya menjadi titik awal aplikasi, sedangkan proses konfigurasi dan bootstrap aplikasi dilakukan oleh layer lain.

---

# 4. `cmd/`

```text
cmd/
├── root.go
└── start.go
```

Package `cmd` bertanggung jawab terhadap **command-line interface dan application command**.

Contohnya:

```text
cmd/start.go
```

digunakan untuk menjalankan aplikasi.

Sedangkan:

```text
cmd/root.go
```

digunakan sebagai root command.

Dengan pemisahan ini, aplikasi dapat memiliki beberapa command apabila dibutuhkan.

Contohnya:

```text
go run main.go start
go run main.go worker
go run main.go migrate
```

tanpa harus menempatkan seluruh logic command tersebut di `main.go`.

---

# 5. `config/`

```text
config/
├── config.go
├── database.go
├── fiber.go
├── minio.go
├── redis.go
└── zerolog.go
```

Layer ini bertanggung jawab terhadap **application configuration dan initialization terhadap infrastructure**.

### `config.go`

Mengelola konfigurasi umum aplikasi.

Contohnya:

```text
environment
application name
port
database configuration
redis configuration
```

### `database.go`

Mengatur koneksi database.

```text
Application
     │
     ▼
config/database.go
     │
     ▼
PostgreSQL
```

### `fiber.go`

Mengatur konfigurasi HTTP framework Fiber.

### `minio.go`

Mengatur koneksi dan konfigurasi MinIO.

### `redis.go`

Mengatur koneksi Redis.

### `zerolog.go`

Mengatur konfigurasi logging menggunakan Zerolog.

---

# 6. `database/`

```text
database/
├── migrations/
│   ├── 000001_create_users_table.down.sql
│   └── 000001_create_users_table.up.sql
│
└── seeds/
    └── user_seed.go
```

Package ini menangani kebutuhan database yang bersifat **database lifecycle**.

## Migrations

Migration digunakan untuk mengelola perubahan schema database.

Contohnya:

```text
000001_create_users_table.up.sql
```

digunakan untuk membuat tabel.

Sedangkan:

```text
000001_create_users_table.down.sql
```

digunakan untuk rollback migration.

## Seeds

Seeds digunakan untuk memasukkan initial atau dummy data.

Contohnya:

```text
user_seed.go
```

digunakan untuk memasukkan data awal user.

---

# 7. `internal/`

Package `internal` merupakan bagian penting dari aplikasi.

Dalam Go, package di bawah directory `internal` hanya dapat di-import oleh package yang berada di parent tree yang sesuai.

Dengan demikian, `internal` dapat digunakan untuk menyimpan **implementation detail aplikasi yang tidak dimaksudkan untuk digunakan oleh external package**.

Strukturnya:

```text
internal/
├── adapter/
├── app/
├── core/
└── middleware/
```

---

# 8. `internal/core/`

```text
internal/core/
├── domain/
└── service/
```

`core` merupakan bagian utama dari business/application logic.

Di dalamnya terdapat:

```text
domain
service
```

---

# 9. `internal/core/domain/`

```text
core/domain/
├── entity/
└── model/
```

Layer domain merepresentasikan object dan konsep yang berkaitan dengan aplikasi.

## `entity/`

```text
entity/
├── health_check_entity.go
├── jwt_entity.go
└── user_entity.go
```

Entity merepresentasikan objek utama yang digunakan oleh aplikasi.

Contohnya:

```text
User
JWT
Health Check
```

Entity biasanya memiliki lifecycle atau identitas tertentu di dalam business domain.

Contoh:

```go
type User struct {
    ID    string
    Name  string
    Email string
}
```

---

# 10. `core/domain/model/`

```text
model/
└── user_model.go
```

Model dapat digunakan untuk merepresentasikan bentuk data tertentu yang dibutuhkan oleh application atau domain.

Pemisahan `entity` dan `model` memungkinkan aplikasi membedakan:

```text
Business Entity
      vs
Data Model
```

Walaupun implementasinya dapat berbeda tergantung kebutuhan aplikasi.

---

# 11. `internal/core/service/`

```text
core/service/
├── health_check_service.go
├── jwt_service.go
└── user_service.go
```

Service merupakan bagian yang menangani **business/application logic**.

Ini adalah salah satu bagian paling penting dalam arsitektur.

Contohnya:

```text
user_service.go
```

dapat menangani:

```text
Create User
Get User
Update User
Delete User
```

Sedangkan:

```text
jwt_service.go
```

menangani logic yang berkaitan dengan JWT.

Dan:

```text
health_check_service.go
```

menangani logic health check.

Prinsip penting:

> Handler sebaiknya tidak mengandung business logic yang kompleks. Business logic ditempatkan di service.

Contoh alur:

```text
HTTP Request
     │
     ▼
User Handler
     │
     ▼
User Service
     │
     ▼
User Repository
```

---

# 12. `internal/adapter/`

```text
internal/adapter/
├── handler/
├── repository/
└── storage/
```

Adapter berfungsi sebagai penghubung antara application/core dengan dunia luar.

Secara konsep:

```text
                  Application Core
                         │
                         │
                  ┌──────┴──────┐
                  │   Adapter   │
                  └──────┬──────┘
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
             HTTP      Database    MinIO
```

Dengan demikian core tidak perlu mengetahui detail implementasi external system secara langsung.

---

# 13. `internal/adapter/handler/`

```text
adapter/handler/
├── health_check_handler.go
├── request/
├── response/
├── upload_image_handler.go
└── user_handler.go
```

Handler merupakan **delivery layer**.

Tanggung jawab handler:

* menerima HTTP request
* melakukan parsing request
* melakukan validation pada input
* memanggil service
* mengubah hasil service menjadi HTTP response
* menentukan HTTP status code

Handler sebaiknya tidak menjadi tempat business logic utama.

Alur:

```text
Client
  │
  │ HTTP
  ▼
Handler
  │
  ▼
Service
```

---

# 14. `handler/request/`

```text
handler/request/
└── user_request.go
```

Package ini menyimpan object yang digunakan untuk menerima HTTP request.

Contohnya:

```go
type CreateUserRequest struct {
    Name  string `json:"name"`
    Email string `json:"email"`
}
```

Dengan adanya request object, bentuk HTTP request tidak harus langsung menggunakan domain entity.

Ini membantu memisahkan:

```text
HTTP Contract
      vs
Domain Object
```

---

# 15. `handler/response/`

```text
handler/response/
├── default_response.go
└── user_response.go
```

Response digunakan untuk menentukan bentuk data yang dikirim kembali kepada client.

Contohnya:

```text
User Entity
     │
     ▼
User Response
     │
     ▼
JSON Response
```

Dengan demikian domain entity tidak harus selalu diekspos secara langsung kepada API client.

---

# 16. `internal/adapter/repository/`

```text
adapter/repository/
└── user_repository.go
```

Repository merupakan adapter untuk komunikasi dengan database.

Tanggung jawabnya meliputi:

```text
INSERT
SELECT
UPDATE
DELETE
```

Contohnya:

```text
User Service
     │
     ▼
User Repository
     │
     ▼
PostgreSQL
```

Service tidak perlu mengetahui detail SQL atau ORM yang digunakan.

Dengan begitu database implementation dapat diganti tanpa mengubah business logic secara besar-besaran.

---

# 17. `internal/adapter/storage/`

```text
adapter/storage/
└── minio.go
```

Package ini merupakan adapter untuk object storage.

Dalam project ini menggunakan MinIO.

Alurnya:

```text
Handler
   │
   ▼
Service
   │
   ▼
Storage Adapter
   │
   ▼
MinIO
```

Contohnya dapat digunakan untuk:

```text
Upload image
Download object
Delete object
Generate presigned URL
```

---

# 18. `internal/middleware/`

```text
internal/middleware/
└── middleware_adapter.go
```

Middleware merupakan **cross-cutting concern** yang digunakan oleh banyak bagian aplikasi.

Contohnya:

```text
Authentication
Authorization
Logging
Recovery
Request ID
CORS
Rate Limiting
```

Middleware berada di luar business domain karena fungsinya dapat digunakan oleh banyak endpoint atau module.

Contohnya:

```text
Request
   │
   ▼
Middleware
   │
   ▼
Handler
   │
   ▼
Service
```

---

# 19. `internal/app/`

```text
internal/app/
└── app.go
```

Package `app` berfungsi sebagai **application composition / bootstrap layer**.

Di sinilah berbagai dependency dapat dirangkai.

Contohnya:

```text
Config
  │
  ├── Database
  ├── Redis
  ├── MinIO
  └── Logger
        │
        ▼
Repositories
        │
        ▼
Services
        │
        ▼
Handlers
        │
        ▼
Fiber Router
```

Dengan kata lain, `app.go` dapat menjadi tempat untuk melakukan dependency injection.

---

# 20. `deployment/`

```text
deployment/
└── docker-compose.yml
```

Berisi konfigurasi deployment atau infrastructure untuk menjalankan aplikasi dan dependency-nya.

Contohnya:

```text
Application
PostgreSQL
Redis
MinIO
```

Docker Compose membantu menjalankan environment secara konsisten.

---

# 21. `Dockerfile`

```text
Dockerfile
```

Digunakan untuk membuat container image aplikasi.

Biasanya Dockerfile dapat menggunakan multi-stage build:

```text
Build Stage
     │
     ▼
Go Compiler
     │
     ▼
Binary
     │
     ▼
Runtime Stage
```

Keuntungan utamanya adalah runtime image dapat dibuat lebih kecil dan tidak perlu membawa compiler Go.

---

# 22. `utils/`

```text
utils/
└── conv.go
```

Berisi utility atau helper yang bersifat generic.

Contohnya conversion function.

Utility sebaiknya tetap sederhana dan tidak berisi business logic.

Jika sebuah helper hanya digunakan oleh satu domain, lebih baik helper tersebut ditempatkan di domain/module terkait daripada dimasukkan ke `utils`.

---

# 23. Dependency Flow

Secara keseluruhan, request flow aplikasi dapat digambarkan seperti berikut:

```text
                         Client
                           │
                           │ HTTP
                           ▼
                    ┌─────────────┐
                    │   Handler   │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │   Service   │
                    └──────┬──────┘
                           │
                  ┌────────┴────────┐
                  │                 │
                  ▼                 ▼
               Domain          Repository
                                    │
                                    ▼
                               PostgreSQL
```

Untuk upload:

```text
Handler
   │
   ▼
Service
   │
   ▼
Storage Adapter
   │
   ▼
MinIO
```

---

# 24. Separation of Concerns

Arsitektur ini memisahkan beberapa tanggung jawab:

| Layer        | Responsibility                                   |
| ------------ | ------------------------------------------------ |
| `handler`    | HTTP request/response                            |
| `service`    | Business/application logic                       |
| `domain`     | Business entity dan model                        |
| `repository` | Database access                                  |
| `storage`    | Object storage                                   |
| `middleware` | Cross-cutting concern                            |
| `config`     | Configuration dan infrastructure initialization  |
| `app`        | Dependency injection dan application composition |
| `database`   | Migration dan seed                               |
| `deployment` | Deployment/infrastructure                        |
| `cmd`        | Application command                              |
| `utils`      | Generic helper                                   |

Dengan separation tersebut, perubahan pada satu concern tidak harus menyebabkan perubahan pada seluruh aplikasi.

---

# 25. Apakah Struktur Ini Sudah Modular?

**Ya, secara logical aplikasi ini sudah melakukan pemisahan module/fitur.**

Contohnya dapat dilihat dari:

```text
User
├── user_entity.go
├── user_model.go
├── user_service.go
├── user_repository.go
├── user_handler.go
├── user_request.go
└── user_response.go
```

Begitu juga:

```text
JWT
├── jwt_entity.go
└── jwt_service.go
```

dan:

```text
Health Check
├── health_check_entity.go
├── health_check_service.go
└── health_check_handler.go
```

Artinya setiap fitur/domain sudah memiliki bagian masing-masing di dalam layer.

Secara konseptual:

```text
                  Application
                       │
       ┌───────────────┼────────────────┐
       │               │                │
       ▼               ▼                ▼
     User             JWT          Health Check
       │               │                │
       ├── Domain      ├── Domain       ├── Domain
       ├── Service     ├── Service      ├── Service
       └── Adapter     └── Adapter      └── Adapter
```

---

# 26. Layered Architecture vs Modular Monolith

Penting untuk membedakan dua konsep ini.

### Layered Architecture

Menentukan **bagaimana tanggung jawab teknis dipisahkan**:

```text
Handler
   ↓
Service
   ↓
Domain
   ↓
Repository
```

### Modular Architecture

Menentukan **bagaimana business domain dipisahkan**:

```text
User
Product
Order
Payment
```

Keduanya dapat digunakan secara bersamaan.

Pada project ini, struktur saat ini lebih kuat merepresentasikan:

```text
Layered Architecture
        +
Logical Module Separation
```

Daripada strict Modular Monolith.

---

# 27. Kesimpulan

Struktur project saat ini sudah memiliki **separation of concerns yang baik**.

Layer-layer utama telah dipisahkan:

```text
Handler
   ↓
Service
   ↓
Domain
   ↓
Repository / Storage
```

Selain itu, fitur/domain juga sudah dipisahkan secara logical.

Contohnya:

```text
User
├── Entity
├── Model
├── Service
├── Repository
├── Handler
├── Request
└── Response
```

Sehingga dapat disimpulkan bahwa aplikasi ini **sudah memisahkan module/fitur secara logical**, meskipun pemisahan filesystem utamanya masih menggunakan pendekatan **layer-based**.

Dengan kata lain:

> **Project ini menggunakan Layered Architecture dengan pemisahan module/domain secara logical.**

Pendekatan ini memberikan separation of concerns yang baik dan menjadi fondasi yang cukup kuat untuk berkembang menuju **Modular Monolith** apabila jumlah domain dan kompleksitas aplikasi semakin bertambah.

Jika suatu saat ingin membuat boundary module menjadi lebih kuat, struktur dapat dievolusikan dari:

```text
internal/
├── adapter/
├── core/
└── middleware/
```

menjadi:

```text
internal/
├── user/
│   ├── domain/
│   ├── service/
│   ├── repository/
│   └── handler/
│
├── product/
│   ├── domain/
│   ├── service/
│   ├── repository/
│   └── handler/
│
└── order/
    ├── domain/
    ├── service/
    ├── repository/
    └── handler/
```

Namun refactor tersebut **bukan sebuah kewajiban**. Struktur saat ini sudah valid untuk aplikasi monolith yang menerapkan separation of concerns dan logical module separation.

> **Intinya: Layer memisahkan technical responsibility, sedangkan module memisahkan business responsibility.**
