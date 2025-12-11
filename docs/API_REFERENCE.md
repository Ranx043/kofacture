# KOFACTURE - API Reference

## Base URL
```
Production: https://api.kofacture.com/api/v1
Development: http://localhost:8000/api/v1
```

## Authentication

All endpoints (except `/auth/login` and `/auth/register`) require JWT authentication.

```http
Authorization: Bearer <access_token>
```

### Token Refresh
```http
POST /auth/refresh
Content-Type: application/json

{
  "refresh_token": "eyJhbGciOiJIUzI1NiIs..."
}
```

---

## Endpoints Overview

| Module | Endpoints | Description |
|--------|-----------|-------------|
| **Auth** | 5 | Authentication & user management |
| **Sales** | 14 | Invoices, NCF, e-CF |
| **Purchases** | 8 | Vendor invoices, expenses |
| **Accounting** | 17 | Chart of accounts, journal entries |
| **Treasury** | 19 | Banks, payments, cash flow |
| **Reports** | 16 | DGII 606/607/608/609 |
| **OCR** | 4 | Invoice scanning |
| **AI** | 3 | Conversational assistant |
| **Dashboard** | 4 | Metrics & KPIs |

**Total: 100+ endpoints**

---

## Auth Module

### Login
```http
POST /auth/login
Content-Type: application/x-www-form-urlencoded

username=user@example.com&password=secret123
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "bearer",
  "user": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "user@example.com",
    "full_name": "John Doe",
    "role": "admin"
  }
}
```

### Get Current User
```http
GET /auth/me
Authorization: Bearer <token>
```

---

## Sales Module

### List Invoices
```http
GET /ventas/facturas?page=1&per_page=20&estado=emitida
Authorization: Bearer <token>
```

**Query Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| `page` | int | Page number (default: 1) |
| `per_page` | int | Items per page (default: 20, max: 100) |
| `estado` | string | Filter by status (borrador, emitida, anulada) |
| `fecha_desde` | date | Filter from date |
| `fecha_hasta` | date | Filter to date |
| `tercero_id` | uuid | Filter by customer |

**Response:**
```json
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "ncf": "E310000000001",
      "tipo_ncf": "E31",
      "fecha": "2024-11-15",
      "tercero": {
        "id": "...",
        "nombre": "Comercial López",
        "rnc": "101-23456-7"
      },
      "subtotal": 50000.00,
      "itbis": 9000.00,
      "total": 59000.00,
      "estado": "emitida",
      "track_id": "ECF-2024-001234"
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total": 150,
    "pages": 8
  }
}
```

### Create Invoice
```http
POST /ventas/facturas
Authorization: Bearer <token>
Content-Type: application/json

{
  "tercero_id": "550e8400-e29b-41d4-a716-446655440000",
  "tipo_ncf": "E31",
  "fecha": "2024-11-15",
  "condicion_pago": "credito",
  "dias_credito": 30,
  "lineas": [
    {
      "descripcion": "Servicio de consultoría",
      "cantidad": 1,
      "precio_unitario": 50000.00,
      "itbis_incluido": false
    }
  ],
  "notas": "Factura por servicios de noviembre"
}
```

### Emit e-CF (Electronic Invoice)
```http
POST /ventas/facturas/{id}/emitir
Authorization: Bearer <token>
```

**Response:**
```json
{
  "success": true,
  "data": {
    "ncf": "E310000000001",
    "track_id": "ECF-2024-001234",
    "estado": "emitida",
    "fecha_emision": "2024-11-15T10:30:00Z",
    "xml_url": "https://storage.../factura_E310000000001.xml",
    "pdf_url": "https://storage.../factura_E310000000001.pdf"
  }
}
```

---

## Purchases Module

### Register Purchase
```http
POST /compras
Authorization: Bearer <token>
Content-Type: application/json

{
  "tercero_id": "...",
  "ncf_proveedor": "B0100000123",
  "fecha": "2024-11-10",
  "subtotal": 25000.00,
  "itbis": 4500.00,
  "retencion_isr": 2500.00,
  "retencion_itbis": 0,
  "lineas": [...]
}
```

### Upload Invoice Image (OCR)
```http
POST /compras/escanear
Authorization: Bearer <token>
Content-Type: multipart/form-data

file: <invoice_image.jpg>
```

**Response:**
```json
{
  "success": true,
  "data": {
    "rnc_emisor": "101-23456-7",
    "nombre_emisor": "Proveedor XYZ",
    "ncf": "B0100000123",
    "fecha": "2024-11-10",
    "subtotal": 25000.00,
    "itbis": 4500.00,
    "total": 29500.00,
    "lineas": [
      {
        "descripcion": "Producto A",
        "cantidad": 10,
        "precio_unitario": 2500.00
      }
    ],
    "confidence": 0.95
  }
}
```

---

## Accounting Module

### Chart of Accounts
```http
GET /contabilidad/cuentas?nivel=1
Authorization: Bearer <token>
```

### Create Journal Entry
```http
POST /contabilidad/asientos
Authorization: Bearer <token>
Content-Type: application/json

{
  "fecha": "2024-11-15",
  "descripcion": "Venta de servicios",
  "lineas": [
    {
      "cuenta_id": "...",
      "debito": 59000.00,
      "credito": 0
    },
    {
      "cuenta_id": "...",
      "debito": 0,
      "credito": 50000.00
    },
    {
      "cuenta_id": "...",
      "debito": 0,
      "credito": 9000.00
    }
  ]
}
```

---

## Reports Module (DGII)

### Generate Report 606
```http
POST /reportes/dgii/606/generar
Authorization: Bearer <token>
Content-Type: application/json

{
  "periodo": "202411",
  "formato": "txt"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "periodo": "202411",
    "rnc": "101234567",
    "total_registros": 47,
    "total_monto": 2450890.00,
    "total_itbis": 441160.20,
    "archivo_url": "https://storage.../606_101234567_202411.txt",
    "preview": [
      {
        "rnc_cedula": "101234567",
        "tipo_bienes_servicios": "01",
        "ncf": "B0100000001",
        "fecha_comprobante": "20241115",
        "monto_facturado": 50000.00,
        "itbis_facturado": 9000.00
      }
    ]
  }
}
```

### Available Reports
| Endpoint | Description |
|----------|-------------|
| `POST /reportes/dgii/606/generar` | Purchase report |
| `POST /reportes/dgii/607/generar` | Sales report |
| `POST /reportes/dgii/608/generar` | Voided invoices |
| `POST /reportes/dgii/609/generar` | Foreign payments |
| `GET /reportes/dgii/{tipo}/preview` | Preview before generation |
| `GET /reportes/dgii/{tipo}/download/{periodo}` | Download generated file |

---

## AI Assistant Module

### Send Message
```http
POST /ai/chat
Authorization: Bearer <token>
Content-Type: application/json

{
  "message": "¿Cuánto ITBIS debo pagar este mes?",
  "conversation_id": "optional-uuid"
}
```

**Response (Streaming):**
```
data: {"type": "thinking", "content": "Analyzing tax data..."}
data: {"type": "tool_call", "tool": "query_database", "args": {...}}
data: {"type": "tool_result", "result": {...}}
data: {"type": "message", "content": "Based on your November..."}
data: {"type": "done", "conversation_id": "..."}
```

### Get Conversation History
```http
GET /ai/conversations/{conversation_id}
Authorization: Bearer <token>
```

---

## Treasury Module

### List Bank Accounts
```http
GET /tesoreria/bancos
Authorization: Bearer <token>
```

### Register Payment
```http
POST /tesoreria/pagos
Authorization: Bearer <token>
Content-Type: application/json

{
  "tercero_id": "...",
  "factura_id": "...",
  "monto": 59000.00,
  "fecha": "2024-11-20",
  "metodo_pago": "transferencia",
  "banco_id": "...",
  "referencia": "TRF-001234"
}
```

### Cash Flow Report
```http
GET /tesoreria/flujo-caja?fecha_desde=2024-11-01&fecha_hasta=2024-11-30
Authorization: Bearer <token>
```

---

## Error Responses

### Standard Error Format
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  }
}
```

### Error Codes
| Code | HTTP Status | Description |
|------|-------------|-------------|
| `UNAUTHORIZED` | 401 | Invalid or expired token |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `NOT_FOUND` | 404 | Resource not found |
| `VALIDATION_ERROR` | 422 | Invalid request data |
| `RATE_LIMITED` | 429 | Too many requests |
| `INTERNAL_ERROR` | 500 | Server error |

---

## Rate Limiting

| Endpoint Type | Limit |
|---------------|-------|
| Authentication | 5 req/min |
| General API | 100 req/min |
| AI Chat | 20 req/min |
| Report Generation | 10 req/min |

Headers returned:
```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1699999999
```
