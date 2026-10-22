<div align="center">

# 🛠️ Service Marketplace Platform

**An enterprise-grade, decoupled service marketplace bridging local service requests with verified professionals.**

![Laravel](https://img.shields.io/badge/Laravel_11-FF2D20?style=for-the-badge&logo=laravel&logoColor=white)
![Next.js](https://img.shields.io/badge/Next.js_14-000000?style=for-the-badge&logo=nextdotjs&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![Flutter](https://img.shields.io/badge/Flutter-02569B?style=for-the-badge&logo=flutter&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge)

[System Architecture](#-system-architecture) •
[Database Schema](#-database-schema) •
[Getting Started](#-getting-started) •
[API Reference](#-api-reference)

---

</div>

## 📌 Executive Summary

The **Service Marketplace Platform** is a multi-client ecosystem designed to solve local service discovery across domains such as mechanical repair, plumbing, home cleaning, electrical, tutoring, and tech diagnostics. 

### Key Features & Value Proposition
* **Dual-Discovery Marketplace Engine:**
  * **Direct Search:** Clients search, filter, and review verified provider profiles based on skills, radius, and historical performance.
  * **On-Demand Bidding System:** Clients post target requests, automatically alerting matching regional service providers who bid with customized estimates and timelines.
* **Decoupled API Architecture:** A single high-performance REST backend serving both a web dashboard and cross-platform native mobile clients.
* **Real-time Event Handling:** Asynchronous job queues for notifications, updates, and proposal triggers.

---

## 🏗️ System Architecture

The platform follows a clean, decoupled architecture utilizing a unified **Laravel 13 REST API** backed by **MySQL** and **Redis**, consumed concurrently by web and mobile clients.

```
                         ┌─────────────────────────────────┐
                         │   Laravel 13 REST API Engine    │
                         │  (Sanctum Auth + MySQL + Redis) │
                         └────────────────┬────────────────┘
                                          │
                 ┌────────────────────────┴────────────────────────┐
                 ▼                                                 ▼
   ┌───────────────────────────┐                     ┌───────────────────────────┐
   │    Next.js 14 Web Portal   │                     │    Flutter Mobile App     │
   │ (React, TypeScript, Dark) │                     │     (iOS & Android)       │
   └───────────────────────────┘                     └───────────────────────────┘
```

### Technology Breakdown

| Component | Stack / Technologies |
| :--- | :--- |
| **Backend API** | Laravel 13, PHP 8.3, Sanctum Token Auth, Predis |
| **Database & Cache** | MySQL 8.0 / PostgreSQL, Redis (Queues, Session, Caching) |
| **Web Frontend** | Next.js 14 (App Router), TypeScript, Tailwind CSS, Shadcn UI, TanStack Query v5 |
| **Mobile App** | Flutter 3.x, Dart, Riverpod State Management, Dio HTTP Client |
| **Infrastructure** | Nginx Reverse Proxy, Docker & Docker Compose, Let's Encrypt SSL |

---

## 🗄️ Database Schema & Data Model

```
 ┌──────────────┐         ┌───────────────────┐         ┌────────────────────┐
 │    users     │         │ provider_profiles │         │ service_categories │
 ├──────────────┤         ├───────────────────┤         ├────────────────────┤
 │ id (PK)      │<───────1│ id (PK)           │    ┌───>│ id (PK)            │
 │ name         │         │ user_id (FK)      │    │    │ name, slug, icon   │
 │ email        │         │ business_name     │    │    └────────────────────┘
 │ role (enum)  │         │ location, radius  │    │               ▲
 └──────┬───────┘         └─────────┬─────────┘    │               │
        │                           ▲              │               │
        │ 1                         │ 1            │               │
        ▼                           │              │               │
 ┌──────────────┐                   │              │      ┌────────┴───────────┐
 │service_demand│                   │              │      │ provider_services  │
 ├──────────────┤                   │              │      ├────────────────────┤
 │ id (PK)      │                   │              └─────>│ provider_id (FK)   │
 │ client_id(FK)│                   │                     │ service_id (FK)    │
 │ service_id   ├───────────────────┘                     └────────────────────┘
 │ budget       │
 └──────┬───────┘
        │ 1
        ▼ N
 ┌───────────────────┐
 │  demand_proposals │
 ├───────────────────┤
 │ id (PK)           │
 │ demand_id (FK)    │
 │ provider_id (FK)  │
 │ price, pitch      │
 └───────────────────┘
```

---

## 🚀 Getting Started

### Prerequisites
* **PHP:** `>= 8.3`
* **Composer:** `>= 2.6`
* **Node.js:** `>= 20.x` & **pnpm** / **npm**
* **Docker:** Installed & running *(for containerized infrastructure)*
* **Services:** Local or containerized MySQL & Redis

---

### Local Backend Setup (Laravel API)

1. **Navigate to the API working directory:**
   ```bash
   cd backend
   ```

2. **Install PHP Dependencies:**
   ```bash
   composer install
   ```

3. **Configure Environment File:**
   ```bash
   cp .env.example .env
   php artisan key:generate
   ```

4. **Set Local Database & Redis Credentials (`backend/.env`):**
   ```ini
   DB_CONNECTION=mysql
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_DATABASE=service_marketplace
   DB_USERNAME=root
   DB_PASSWORD=root

   CACHE_STORE=redis
   QUEUE_CONNECTION=redis
   REDIS_CLIENT=predis
   REDIS_HOST=127.0.0.1
   ```

5. **Run Database Migrations & Seeders:**
   ```bash
   php artisan migrate --seed
   ```

6. **Boot Local Development Server:**
   ```bash
   php artisan serve
   ```
   > 🌐 **API Base Endpoint:** `http://localhost:8000/api/v1`

---

## 📡 API Reference Overview

All API endpoints are strictly versioned under `/api/v1/` and accept standard JSON payloads with `Accept: application/json` headers.

| Method | Endpoint | Description | Auth Guard |
| :---: | :--- | :--- | :---: |
| `POST` | `/api/v1/auth/register` | Register new user account (Client or Provider) | Public |
| `POST` | `/api/v1/auth/login` | Authenticate and issue Sanctum Bearer Token | Public |
| `GET` | `/api/v1/services` | Retrieve hierarchical service taxonomy | Public |
| `GET` | `/api/v1/providers` | Query & filter provider profiles by service/location | Public |
| `POST` | `/api/v1/demands` | Create a new service demand request | Client |
| `GET` | `/api/v1/demands/feed` | List open demands matching provider's subscribed tags | Provider |
| `POST` | `/api/v1/demands/{id}/proposals` | Submit a pitch and cost quote for a client demand | Provider |
| `PATCH` | `/api/v1/proposals/{id}/accept` | Accept a target provider proposal | Client |

---

## 📂 Project Structure

```
service-marketplace/
├── backend/               # Laravel 13 REST API Application
│   ├── app/
│   │   ├── Http/Controllers/Api/v1/
│   │   ├── Models/
│   │   └── Services/
│   ├── database/migrations/
│   └── routes/api.php
├── frontend/              # Next.js 14 Web Client (React/TS/Tailwind)
├── mobile/                # Flutter Cross-Platform Application
├── docker-compose.yml     # Production VPS Orchestration
└── README.md
```

---

## 📄 License

This repository is open-source software licensed under the [MIT License](LICENSE).