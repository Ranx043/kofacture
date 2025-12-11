# KOFACTURE - Technical Architecture

## System Overview

```
                                    ┌─────────────────────────────────────┐
                                    │           LOAD BALANCER             │
                                    │         (Cloudflare CDN)            │
                                    └─────────────────┬───────────────────┘
                                                      │
                        ┌─────────────────────────────┴─────────────────────────────┐
                        │                                                           │
                        ▼                                                           ▼
          ┌─────────────────────────┐                             ┌─────────────────────────┐
          │      FRONTEND           │                             │        BACKEND          │
          │    (Vercel Edge)        │                             │      (Railway)          │
          │                         │                             │                         │
          │  ┌───────────────────┐  │                             │  ┌───────────────────┐  │
          │  │   Next.js 14      │  │        REST API             │  │    FastAPI        │  │
          │  │   App Router      │◄─┼─────────────────────────────┼─►│    Python 3.11    │  │
          │  │   TypeScript      │  │        WebSocket            │  │    Async/Await    │  │
          │  └───────────────────┘  │                             │  └───────────────────┘  │
          │                         │                             │           │             │
          │  ┌───────────────────┐  │                             │           ▼             │
          │  │   TanStack Query  │  │                             │  ┌───────────────────┐  │
          │  │   React Context   │  │                             │  │   SQLAlchemy 2.0  │  │
          │  │   Zustand         │  │                             │  │   Alembic         │  │
          │  └───────────────────┘  │                             │  └───────────────────┘  │
          │                         │                             │           │             │
          │  ┌───────────────────┐  │                             │           ▼             │
          │  │   Tailwind CSS    │  │                             │  ┌───────────────────┐  │
          │  │   shadcn/ui       │  │                             │  │   PostgreSQL 15   │  │
          │  │   Framer Motion   │  │                             │  │   (Supabase)      │  │
          │  └───────────────────┘  │                             │  └───────────────────┘  │
          └─────────────────────────┘                             └─────────────────────────┘
                                                                              │
                                    ┌─────────────────────────────────────────┼─────────────────────────────────────────┐
                                    │                                         │                                         │
                                    ▼                                         ▼                                         ▼
                      ┌─────────────────────────┐             ┌─────────────────────────┐             ┌─────────────────────────┐
                      │     AI SERVICES         │             │    BACKGROUND JOBS      │             │   EXTERNAL APIS         │
                      │                         │             │                         │             │                         │
                      │  ┌───────────────────┐  │             │  ┌───────────────────┐  │             │  ┌───────────────────┐  │
                      │  │   OpenAI GPT-4    │  │             │  │   Celery          │  │             │  │   Alanube         │  │
                      │  │   Claude 3.5      │  │             │  │   Redis Queue     │  │             │  │   (e-CF DGII)     │  │
                      │  └───────────────────┘  │             │  └───────────────────┘  │             │  └───────────────────┘  │
                      │                         │             │                         │             │                         │
                      │  ┌───────────────────┐  │             │  ┌───────────────────┐  │             │  ┌───────────────────┐  │
                      │  │   Gemini Vision   │  │             │  │   Scheduled Tasks │  │             │  │   DGII API        │  │
                      │  │   (OCR)           │  │             │  │   APScheduler     │  │             │  │   (Validation)    │  │
                      │  └───────────────────┘  │             │  └───────────────────┘  │             │  └───────────────────┘  │
                      └─────────────────────────┘             └─────────────────────────┘             └─────────────────────────┘
```

## Database Schema (Simplified)

```
┌──────────────────┐       ┌──────────────────┐       ┌──────────────────┐
│     USERS        │       │    EMPRESAS      │       │  USER_EMPRESA    │
├──────────────────┤       ├──────────────────┤       ├──────────────────┤
│ id (UUID)        │       │ id (UUID)        │       │ user_id (FK)     │
│ email            │       │ nombre           │       │ empresa_id (FK)  │
│ hashed_password  │──────►│ rnc              │◄──────│ rol              │
│ full_name        │       │ direccion        │       │ is_active        │
│ role             │       │ telefono         │       └──────────────────┘
│ is_active        │       │ email            │
│ created_at       │       │ config_json      │
└──────────────────┘       └──────────────────┘
                                   │
                                   │
         ┌─────────────────────────┼─────────────────────────┐
         │                         │                         │
         ▼                         ▼                         ▼
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│    FACTURAS      │     │    TERCEROS      │     │   CUENTAS        │
├──────────────────┤     ├──────────────────┤     ├──────────────────┤
│ id (UUID)        │     │ id (UUID)        │     │ id (UUID)        │
│ empresa_id (FK)  │     │ empresa_id (FK)  │     │ empresa_id (FK)  │
│ tercero_id (FK)  │     │ tipo (C/P)       │     │ codigo           │
│ ncf              │     │ nombre           │     │ nombre           │
│ tipo_ncf         │     │ rnc_cedula       │     │ tipo             │
│ fecha            │     │ direccion        │     │ naturaleza (D/C) │
│ subtotal         │     │ telefono         │     │ padre_id         │
│ itbis            │     │ email            │     │ nivel            │
│ total            │     │ limite_credito   │     │ es_detalle       │
│ estado           │     │ dias_credito     │     └──────────────────┘
│ track_id         │     └──────────────────┘
└──────────────────┘              │
         │                        │
         │                        ▼
         │              ┌──────────────────┐
         │              │   MOVIMIENTOS    │
         │              ├──────────────────┤
         │              │ id (UUID)        │
         └─────────────►│ empresa_id (FK)  │
                        │ tercero_id (FK)  │
                        │ factura_id (FK)  │
                        │ tipo (pago/cobro)│
                        │ monto            │
                        │ fecha            │
                        │ metodo_pago      │
                        │ banco_id         │
                        └──────────────────┘
```

## Module Structure

```
backend/
├── src/
│   ├── main.py                    # Application entry point
│   ├── modules/
│   │   ├── ai_agents/             # AI orchestration
│   │   │   ├── router.py          # /api/v1/ai/*
│   │   │   ├── service.py         # LLM integration
│   │   │   ├── tools/             # Function calling tools
│   │   │   │   ├── query_tool.py  # Database queries
│   │   │   │   ├── report_tool.py # Report generation
│   │   │   │   └── action_tool.py # Business actions
│   │   │   └── prompts/           # System prompts
│   │   │
│   │   ├── auth/                  # Authentication
│   │   │   ├── router.py          # /api/v1/auth/*
│   │   │   ├── service.py         # JWT, password hashing
│   │   │   ├── models.py          # User, Session models
│   │   │   └── dependencies.py    # Auth middleware
│   │   │
│   │   ├── ventas/                # Sales & Invoicing
│   │   │   ├── router.py          # /api/v1/ventas/*
│   │   │   ├── service.py         # Invoice logic
│   │   │   ├── alanube_service.py # e-CF integration
│   │   │   ├── ncf_service.py     # NCF sequences
│   │   │   └── models.py          # Factura, DetalleFactura
│   │   │
│   │   ├── compras/               # Purchases & Expenses
│   │   ├── contabilidad/          # Accounting
│   │   ├── terceros/              # Customers & Vendors
│   │   ├── tesoreria/             # Treasury & Banking
│   │   ├── reportes/              # DGII Reports
│   │   ├── ocr/                   # Invoice scanning
│   │   ├── dashboard/             # Metrics & KPIs
│   │   └── empresas/              # Multi-tenant
│   │
│   └── shared/
│       ├── config.py              # Settings
│       ├── database.py            # DB connection
│       ├── security.py            # Auth utilities
│       └── exceptions.py          # Custom exceptions
│
├── alembic/                       # Database migrations
│   └── versions/                  # 23 migration files
│
└── tests/
    ├── unit/                      # Unit tests
    ├── integration/               # Integration tests
    └── e2e/                       # End-to-end tests
```

## API Design Patterns

### RESTful Endpoints
```
GET    /api/v1/ventas/facturas           # List invoices (paginated)
GET    /api/v1/ventas/facturas/{id}      # Get single invoice
POST   /api/v1/ventas/facturas           # Create invoice
PUT    /api/v1/ventas/facturas/{id}      # Update invoice
DELETE /api/v1/ventas/facturas/{id}      # Soft delete invoice
POST   /api/v1/ventas/facturas/{id}/emit # Emit e-CF to DGII
```

### Response Format
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "ncf": "E310000000001",
    "total": 5900.00,
    "estado": "emitida"
  },
  "meta": {
    "timestamp": "2024-12-11T10:30:00Z",
    "request_id": "req_abc123"
  }
}
```

### Pagination
```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total": 150,
    "pages": 8
  }
}
```

## Security Layers

```
┌─────────────────────────────────────────────────────────────────┐
│                     SECURITY ARCHITECTURE                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. NETWORK LAYER                                               │
│     ├── Cloudflare WAF (DDoS protection)                        │
│     ├── TLS 1.3 encryption                                      │
│     └── Rate limiting (100 req/min per IP)                      │
│                                                                  │
│  2. APPLICATION LAYER                                           │
│     ├── JWT Authentication (RS256)                              │
│     ├── Refresh token rotation                                  │
│     ├── CORS whitelist                                          │
│     └── Request validation (Pydantic)                           │
│                                                                  │
│  3. DATA LAYER                                                  │
│     ├── Password hashing (bcrypt, cost=12)                      │
│     ├── SQL injection prevention (ORM only)                     │
│     ├── Row-level security (empresa_id filter)                  │
│     └── Sensitive data encryption (AES-256)                     │
│                                                                  │
│  4. AUDIT LAYER                                                 │
│     ├── Request logging                                         │
│     ├── User activity tracking                                  │
│     └── Security event alerts                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Performance Optimizations

| Technique | Implementation | Impact |
|-----------|----------------|--------|
| **Database Indexing** | Composite indexes on frequent queries | 10x faster queries |
| **Query Optimization** | Eager loading, select specific columns | 50% less DB load |
| **Caching** | Redis for session, frequent data | 80% cache hit rate |
| **Connection Pooling** | SQLAlchemy pool_size=20 | Handles 1000+ concurrent |
| **Async I/O** | FastAPI async endpoints | 3x throughput |
| **CDN** | Static assets on Cloudflare | 200ms global latency |
| **Compression** | Gzip responses > 1KB | 70% bandwidth reduction |

## Deployment Pipeline

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Commit  │───►│  GitHub  │───►│   CI     │───►│  Review  │───►│  Deploy  │
│          │    │  Actions │    │  Tests   │    │  Stage   │    │  Prod    │
└──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
                     │               │               │               │
                     ▼               ▼               ▼               ▼
               ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
               │  Lint    │   │  Unit    │   │  E2E     │   │  Health  │
               │  Format  │   │  Tests   │   │  Tests   │   │  Check   │
               │  Types   │   │  Coverage│   │  Preview │   │  Monitor │
               └──────────┘   └──────────┘   └──────────┘   └──────────┘
```
