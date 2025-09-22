# 🏛️ Архитектура проекта — TrendBazaar

## 📐 Общая структура
Система построена по классической многослойной архитектуре:

- **Фронтенд:** веб-клиент на React + TypeScript, обеспечивает взаимодействие пользователя с приложением через браузер.  
- **Бэкенд:** Java Spring Boot с REST API, обрабатывает запросы от фронтенда, реализует бизнес-логику и взаимодействие с базой данных.  
- **Слой сервисов:** содержит бизнес-логику приложения — обработку заказов, управление продуктами, корзиной и пользователями.  
- **Репозитории:** взаимодействуют с базой данных PostgreSQL через JPA/Hibernate для сохранения и получения данных.  
- **Внешние ресурсы:** кэш (Redis), сервис уведомлений, файловое хранилище для хранения изображений товаров.  
- **Развёртывание:** Docker-контейнеры для фронтенда и бэкенда, Nginx в качестве балансировщика и CDN для статики.

## 🖼️ Схема архитектуры
```mermaid
graph TB
    %% ---------- Пользователь ----------
    subgraph "👤 Пользователь"
        U[Покупатель / Администратор]
    end

    %% ---------- Frontend ----------
    subgraph "🌐 Frontend"
        U --> |HTTPS| RA[React App<br/>TypeScript]
        RA --> |REST API| GLB
    end

    %% ---------- Gateway / LB ----------
    GLB[Nginx<br/>Load Balancer]

    %% ---------- Backend ----------
    subgraph "⚙️ Spring Boot Backend"
        GLB --> |/api/*| SB[Tomcat]
        SB --> CC[Category<br/>Controller]
        SB --> PC[Product<br/>Controller]
        SB --> OC[Order<br/>Controller]
        SB --> UC[User<br/>Controller]
        SB --> CARTC[Cart<br/>Controller]
        SB --> LC[Log<br/>Controller]
    end

    %% ---------- Services ----------
    subgraph "🔧 Service Layer"
        CC --> CS[Category<br/>Service]
        PC --> PS[Product<br/>Service]
        OC --> OS[Order<br/>Service]
        UC --> US[User<br/>Service]
        CARTC --> CARTS[Cart<br/>Service]
        LC --> LS[Log<br/>Service]
    end

    %% ---------- Repositories ----------
    subgraph "🗃️ Repositories"
        CS --> CR[(Category<br/>Repository)]
        PS --> PR[(Product<br/>Repository)]
        OS --> OR[(Order<br/>Repository)]
        US --> UR[(User<br/>Repository)]
        CARTS --> CARTR[(Cart<br/>Repository)]
    end

    %% ---------- Database ----------
    subgraph "🐘 PostgreSQL"
        CR --> DB[(PostgreSQL)]
        PR --> DB
        OR --> DB
        UR --> DB
        CARTR --> DB
    end

    %% ---------- External Resources ----------
    subgraph "📦 External Resources"
        NOTIF[Notification<br/>Service]
        CACHE[Redis<br/>Cache]
        FILES[File<br/>Storage]
        OS --> NOTIF
        LS --> NOTIF
        CARTS --> CACHE
        PS --> CACHE
        OS --> FILES
    end

    %% ---------- Deployment ----------
    subgraph "🐳 Deployment"
        RA -.-> STATIC[Static<br/>Files CDN]
        SB -.-> DOCK[Docker<br/>Image]
        DB -.-> VOL[(Persistent<br/>Volume)]
        CACHE -.-> DOCKRED[Redis<br/>Container]
        FILES -.-> VOL
    end

    %% ---------- Цветовые классы ----------
    classDef frontend fill:#61dafb,stroke:#282c34,color:#000
    classDef backend fill:#6db33f,stroke:#fff,color:#000
    classDef service fill:#ffd966,stroke:#000,color:#000
    classDef repo fill:#9fc5e8,stroke:#000,color:#000
    classDef db fill:#336791,stroke:#fff,color:#fff
    classDef deployment fill:#239aef,stroke:#fff,color:#fff
    classDef external fill:#f9d71c,stroke:#000,color:#000

    class RA,GLB frontend
    class SB,CC,PC,OC,UC,CARTC,LC backend
    class CS,PS,OS,US,CARTS,LS service
    class CR,PR,OR,UR,CARTR repo
    class DB db
    class STATIC,DOCK,VOL,DOCKRED deployment
    class NOTIF,CACHE,FILES external
```
