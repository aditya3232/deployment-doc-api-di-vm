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

Tanggung jawabnya adalah menjalankan aplikasi dan meneruskan proses bootstrap kepada layer aplikasi.

Prinsipnya:

> `main.go` tidak seharusnya berisi business logic.

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
go run main.go start
```

Dengan pemisahan ini, aplikasi dapat memiliki command lain apabila dibutuhkan, seperti:

```text
go run main.go start
go run main.go worker
go run main.go migrate
```

tanpa menempatkan seluruh logic command tersebut di `main.go`.

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

Package `config` bertanggung jawab terhadap **application configuration dan initialization infrastructure**.

### `config.go`

Mengelola konfigurasi umum aplikasi.

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
└── seeds/
```

Package ini menangani kebutuhan database yang berkaitan dengan **schema lifecycle dan initial data**.

## Migrations

```text
000001_create_users_table.up.sql
000001_create_users_table.down.sql
```

Digunakan untuk membuat dan melakukan rollback schema database.

## Seeds

```text
user_seed.go
```

Digunakan untuk memasukkan data awal atau dummy data.

---

# 7. `internal/`

Package `internal` digunakan untuk menyimpan implementation detail aplikasi.

Dalam Go, package di dalam directory `internal` tidak dapat digunakan secara bebas oleh package dari luar parent tree yang diperbolehkan.

Dengan demikian, `internal` cocok digunakan untuk application-specific implementation.

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

`core` merupakan bagian utama yang berisi **business/application logic**.

Terdiri dari:

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

Domain merepresentasikan object dan konsep yang digunakan dalam business domain aplikasi.

## `entity/`

```text
entity/
├── health_check_entity.go
├── jwt_entity.go
└── user_entity.go
```

Berisi entity seperti:

```text
User
JWT
Health Check
```

Entity merepresentasikan object yang memiliki makna dalam domain aplikasi.

## `model/`

```text
model/
└── user_model.go
```

Model merepresentasikan bentuk data yang diperlukan oleh aplikasi.

Pemisahan ini memungkinkan aplikasi membedakan:

```text
Business Entity
      vs
Data/Application Model
```

---

# 10. `internal/core/service/`

```text
core/service/
├── health_check_service.go
├── jwt_service.go
└── user_service.go
```

Service merupakan bagian yang menangani **business/application logic**.

Contohnya:

```text
user_service.go
```

menangani business logic yang berkaitan dengan User.

```text
jwt_service.go
```

menangani logic yang berkaitan dengan JWT.

```text
health_check_service.go
```

menangani logic health check.

Secara umum:

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
Repository
```

Business logic utama sebaiknya berada di service, bukan di handler.

---

# 11. `internal/adapter/`

```text
internal/adapter/
├── handler/
├── repository/
└── storage/
```

Adapter merupakan penghubung antara application core dengan external system.

Secara konseptual:

```text
                  Application Core
                         │
                  ┌──────┴──────┐
                  │   Adapter   │
                  └──────┬──────┘
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
             HTTP      Database    MinIO
```

Adapter menangani detail teknis dari external system.

---

# 12. `internal/adapter/handler/`

```text
adapter/handler/
├── health_check_handler.go
├── request/
├── response/
├── upload_image_handler.go
└── user_handler.go
```

Handler merupakan **delivery layer**.

Tanggung jawabnya:

* menerima HTTP request
* parsing request
* melakukan input validation
* memanggil service
* mengubah hasil service menjadi HTTP response
* menentukan HTTP status code

Alurnya:

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

# 13. `handler/request/`

```text
handler/request/
└── user_request.go
```

Berisi object yang digunakan untuk menerima HTTP request.

Contohnya:

```go
type CreateUserRequest struct {
    Name  string `json:"name"`
    Email string `json:"email"`
}
```

Dengan demikian HTTP request tidak harus langsung menggunakan domain entity.

```text
HTTP Contract
      │
      ▼
Request Object
      │
      ▼
Application / Domain
```

---

# 14. `handler/response/`

```text
handler/response/
├── default_response.go
└── user_response.go
```

Digunakan untuk menentukan bentuk response yang dikirim kepada client.

Alurnya:

```text
User Entity
     │
     ▼
User Response
     │
     ▼
JSON Response
```

Dengan demikian domain entity tidak harus diekspos secara langsung kepada API client.

---

# 15. `internal/adapter/repository/`

```text
adapter/repository/
└── user_repository.go
```

Repository bertanggung jawab terhadap komunikasi dengan database.

Contohnya:

```text
INSERT
SELECT
UPDATE
DELETE
```

Alurnya:

```text
User Service
     │
     ▼
User Repository
     │
     ▼
PostgreSQL
```

Service tidak perlu mengetahui detail query SQL atau implementasi ORM.

---

# 16. `internal/adapter/storage/`

```text
adapter/storage/
└── minio.go
```

Digunakan sebagai adapter untuk object storage seperti MinIO.

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

Contohnya dapat menangani:

```text
Upload Image
Download Object
Delete Object
Presigned URL
```

---

# 17. `internal/middleware/`

```text
internal/middleware/
└── middleware_adapter.go
```

Middleware merupakan **cross-cutting concern** yang dapat digunakan oleh banyak endpoint atau module.

Contohnya:

```text
Authentication
Authorization
Logging
Recovery
CORS
Rate Limiting
Request ID
```

Alur request:

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

# 18. `internal/app/`

```text
internal/app/
└── app.go
```

`app` berfungsi sebagai **composition root / application bootstrap**.

Di sini dependency antar-component dirangkai.

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

Dengan demikian `app.go` dapat menjadi tempat untuk melakukan dependency injection.

---

# 19. `deployment/`

```text
deployment/
└── docker-compose.yml
```

Berisi konfigurasi deployment dan infrastructure untuk menjalankan aplikasi beserta dependency-nya.

Contohnya:

```text
Application
PostgreSQL
Redis
MinIO
```

---

# 20. `Dockerfile`

```text
Dockerfile
```

Digunakan untuk membuat container image aplikasi.

Jika menggunakan multi-stage build:

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

Runtime image tidak perlu membawa seluruh environment development dan compiler Go.

---

# 21. `utils/`

```text
utils/
└── conv.go
```

Berisi helper yang bersifat generic.

Contohnya conversion function.

Utility sebaiknya tidak berisi business logic.

Jika sebuah helper hanya digunakan oleh satu domain, lebih baik helper tersebut ditempatkan pada domain terkait.

---

# 22. Module Separation

Selain pemisahan berdasarkan layer, aplikasi juga telah melakukan **logical separation berdasarkan module/feature**.

Contohnya module User terdiri dari:

```text
User Module
│
├── Entity
│   └── user_entity.go
│
├── Model
│   └── user_model.go
│
├── Service
│   └── user_service.go
│
├── Repository
│   └── user_repository.go
│
├── Handler
│   └── user_handler.go
│
├── Request
│   └── user_request.go
│
└── Response
    └── user_response.go
```

Sedangkan JWT:

```text
JWT Module
│
├── Entity
│   └── jwt_entity.go
│
└── Service
    └── jwt_service.go
```

Health Check:

```text
Health Check Module
│
├── Entity
│   └── health_check_entity.go
│
├── Service
│   └── health_check_service.go
│
└── Handler
    └── health_check_handler.go
```

Jadi walaupun secara filesystem setiap component masih berada pada layer masing-masing:

```text
core/
adapter/
```

secara **logical architecture**, fitur atau module sudah dapat dibedakan.

---

# 23. Module Separation melalui Interface

Pemisahan module tidak hanya dilakukan melalui struktur folder.

Salah satu bagian yang lebih penting adalah **bagaimana module berkomunikasi satu sama lain**.

Contohnya terdapat `UserService`:

```go
type userService struct {
    repo       repository.UserRepositoryInterface
    cfg        *config.Config
    jwtService JwtServiceInterface
    redis      *redis.Client
}
```

Perhatikan dependency berikut:

```go
jwtService JwtServiceInterface
```

`UserService` tidak bergantung langsung kepada implementasi `JWT Service`.

User module hanya mengetahui contract:

```go
type JwtServiceInterface interface {
    // capability yang dibutuhkan User
}
```

Dengan demikian User tidak perlu mengetahui:

* bagaimana JWT dibuat
* library JWT yang digunakan
* bagaimana token di-sign
* bagaimana secret key dikelola
* bagaimana JWT di-parse

User hanya mengetahui:

> "Saya membutuhkan kemampuan dari JWT service."

---

# 24. Dependency melalui Interface

Secara konseptual:

```text
┌──────────────────────┐
│     User Module      │
│                      │
│    UserService       │
└──────────┬───────────┘
           │
           │ depends on
           ▼
┌──────────────────────┐
│ JwtServiceInterface  │
└──────────┬───────────┘
           ▲
           │ implements
           │
┌──────────┴───────────┐
│     JWT Module       │
│                      │
│    jwtService       │
└──────────────────────┘
```

Jadi dependency tidak diarahkan langsung ke concrete implementation.

```text
User
  │
  ▼
JwtServiceInterface
  ▲
  │
JWT Service
```

Bukan:

```text
User
  │
  ▼
Concrete JWT Service
```

---

# 25. Dependency Inversion

Pola tersebut merupakan penerapan prinsip **Dependency Inversion Principle (DIP)**.

High-level module:

```text
UserService
```

tidak bergantung langsung kepada low-level implementation:

```text
JWT implementation
```

Keduanya berkomunikasi melalui abstraction:

```text
UserService
      │
      ▼
JwtServiceInterface
      ▲
      │
JwtService
```

Dengan demikian coupling menjadi lebih rendah.

---

# 26. Kenapa Interface Diletakkan di Sisi Consumer?

Pada contoh:

```go
type userService struct {
    jwtService JwtServiceInterface
}
```

`UserService` mendefinisikan capability yang dia butuhkan.

Ini merupakan pendekatan **consumer-driven interface**.

Misalnya User hanya membutuhkan:

```go
type JwtServiceInterface interface {
    GenerateToken(userID int64) (string, error)
}
```

Maka User tidak perlu bergantung pada interface besar seperti:

```go
type JwtServiceInterface interface {
    GenerateToken(...)
    ValidateToken(...)
    RefreshToken(...)
    RevokeToken(...)
    ParseClaims(...)
    ...
}
```

User hanya meminta capability yang memang dia perlukan.

Prinsipnya:

> **Consumer mendefinisikan interface berdasarkan kebutuhan consumer.**

Hal ini juga membantu menjaga boundary antar-module.

---

# 27. Repository Interface

Konsep yang sama juga terlihat pada repository.

`UserService` menggunakan:

```go
repository.UserRepositoryInterface
```

Sehingga service tidak perlu mengetahui apakah repository menggunakan:

```text
GORM
SQL
PostgreSQL driver
MySQL
Mock repository
```

Secara konseptual:

```text
User Service
      │
      ▼
UserRepositoryInterface
      ▲
      │
      ├── PostgreSQL Repository
      ├── Mock Repository
      └── Test Repository
```

Business/application logic tetap bergantung kepada abstraction.

---

# 28. Constructor Injection

Dependency tersebut kemudian diberikan melalui constructor:

```go
func NewUserService(
    repo repository.UserRepositoryInterface,
    cfg *config.Config,
    jwtService JwtServiceInterface,
    redis *redis.Client,
) UserServiceInterface {
    return &userService{
        repo:       repo,
        cfg:        cfg,
        jwtService: jwtService,
        redis:      redis,
    }
}
```

Ini disebut **Dependency Injection melalui constructor**.

`UserService` tidak membuat dependency-nya sendiri.

Contoh yang dihindari:

```go
func NewUserService() UserServiceInterface {
    jwtService := NewJwtService()
    repo := NewUserRepository()

    // ...
}
```

Karena UserService menjadi terlalu mengetahui bagaimana dependency dibuat.

Lebih baik:

```text
Application Composition
        │
        ├── Create Repository
        ├── Create JWT Service
        ├── Create Redis
        │
        ▼
    UserService
```

---

# 29. Module Boundary melalui Interface

Dengan pendekatan ini, module separation tidak hanya berbentuk:

```text
Folder Separation
```

tetapi juga:

```text
             Module Boundary
                   │
                   ▼
        ┌─────────────────────┐
        │      Interface      │
        └─────────────────────┘
                   │
         ┌─────────┴─────────┐
         ▼                   ▼
     Consumer            Provider
      Module               Module
```

Contoh:

```text
User Module
    │
    │ JwtServiceInterface
    ▼
JWT Module
```

Hal ini membuat dependency antar-module menjadi eksplisit.

---

# 30. Complete Dependency Flow

Jika digabungkan, architecture project dapat digambarkan seperti berikut:

```text
                         Client
                           │
                           ▼
                    ┌─────────────┐
                    │   Handler   │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │ UserService │
                    └──────┬──────┘
                           │
             ┌─────────────┼─────────────┐
             │             │             │
             ▼             ▼             ▼
       Repository    JwtService      Redis Client
       Interface     Interface
             │             │
             ▼             ▼
       PostgreSQL       JWT Module
```

Perhatikan bahwa User Service tidak mengetahui detail implementasi JWT.

User hanya mengetahui:

```text
JwtServiceInterface
```

Begitu juga User Service tidak mengetahui detail database.

User hanya mengetahui:

```text
UserRepositoryInterface
```

---

# 31. Layered Architecture + Module Separation

Dengan demikian, terdapat dua bentuk pemisahan yang berjalan bersamaan.

## Layer Separation

Memisahkan **technical responsibility**:

```text
Handler
   ↓
Service
   ↓
Domain
   ↓
Repository / Storage
```

## Module Separation

Memisahkan **business responsibility**:

```text
User
JWT
Health Check
```

## Interface Separation

Memisahkan **dependency antar-module**:

```text
User
 │
 └── JwtServiceInterface
            │
            ▼
          JWT
```

Sehingga architecture dapat digambarkan:

```text
                 Application
                     │
       ┌─────────────┼─────────────┐
       │             │             │
       ▼             ▼             ▼
     User           JWT       Health Check
       │             │             │
       │             │             │
   ┌───┴────┐        │        ┌────┴─────┐
   │        │        │        │          │
Handler  Service   Service  Handler    Service
   │        │        │        │          │
   │        └───┐    │        └────┐     │
   │            │    │             │     │
   │       Interfaces             │     │
   │            │    │             │     │
   └────────────┴────┴─────────────┴─────┘
```

---

# 32. Kesimpulan

Arsitektur project ini sudah menerapkan **Layered Architecture** dengan separation of concerns yang cukup jelas.

Setiap bagian memiliki tanggung jawab masing-masing:

```text
Handler
    → HTTP communication

Service
    → Business/Application Logic

Domain
    → Business Entity / Model

Repository
    → Database Access

Storage
    → Object Storage

Middleware
    → Cross-cutting Concern

Config
    → Configuration & Infrastructure

App
    → Dependency Injection & Composition
```

Selain pemisahan layer, project ini juga sudah melakukan **logical module separation**.

Contohnya:

```text
User
JWT
Health Check
```

dan dependency antar-module tidak harus menggunakan concrete implementation.

Contohnya:

```text
UserService
     │
     ▼
JwtServiceInterface
     ▲
     │
JwtService
```

serta:

```text
UserService
     │
     ▼
UserRepositoryInterface
     ▲
     │
UserRepository
```

Dengan demikian, module tidak terlalu bergantung pada detail implementation module lainnya.

### Kesimpulan utama

> **Project ini menggunakan Layered Architecture dengan logical module separation dan Dependency Inversion melalui interface.**

Layer digunakan untuk memisahkan **technical responsibility**, sedangkan module/feature digunakan untuk memisahkan **business responsibility**.

Interface kemudian digunakan sebagai **boundary antar-component/module**, sehingga sebuah module cukup mengetahui capability yang dibutuhkannya tanpa mengetahui detail implementasi dari module lain.

Dengan fondasi seperti ini, aplikasi masih dapat berjalan sebagai **monolith**, tetapi memiliki struktur dan dependency boundary yang lebih terkontrol.

Apabila di masa depan salah satu module memiliki kebutuhan untuk scaling, deployment, ownership, atau fault isolation secara independen, module tersebut memiliki fondasi yang lebih baik untuk diekstrak menjadi **microservice**.

> **Layer memisahkan tanggung jawab teknis, module memisahkan tanggung jawab bisnis, dan interface membatasi dependency antar-module.**
