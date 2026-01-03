# KOFACTURE - AI-Powered Accounting Platform

![KOFACTURE Banner](https://img.shields.io/badge/KOFACTURE-AI%20Accounting-1E3A5F?style=for-the-badge&logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIyNCIgaGVpZ2h0PSIyNCIgdmlld0JveD0iMCAwIDI0IDI0IiBmaWxsPSJub25lIiBzdHJva2U9IiMwMEI0QTAiIHN0cm9rZS13aWR0aD0iMiIgc3Ryb2tlLWxpbmVjYXA9InJvdW5kIiBzdHJva2UtbGluZWpvaW49InJvdW5kIj48cGF0aCBkPSJNMyAzdjE4aDE4Ii8+PHBhdGggZD0ibTE5IDkgLTcgNy00LTQtNCA0Ii8+PC9zdmc+)

[![Built with AI](https://img.shields.io/badge/Built%20with-OpenAI%20%7C%20Gemini-412991?style=flat-square)](/)
[![Stack](https://img.shields.io/badge/Stack-FastAPI%20%7C%20Next.js%20%7C%20PostgreSQL-009688?style=flat-square)](/)
[![Timeline](https://img.shields.io/badge/Built%20in-30%20Days-success?style=flat-square)](/)

> **Full-stack SaaS accounting platform with AI assistant, batch OCR invoice scanning, 55+ government reports, and complete tax compliance. Built in under 30 days.**

---

## Overview

KOFACTURE is a complete cloud-based accounting system designed for businesses in the Dominican Republic. It features an AI-powered assistant, automatic batch invoice scanning with OCR, electronic invoicing (e-CF), 55+ tax and financial reports, and full compliance with local tax regulations (DGII).

### Key Highlights

- **AI Conversational Assistant** - Natural language queries about finances, taxes, and reports
- **Batch OCR Invoice Scanning** - Upload multiple photos/PDFs, get structured data in seconds
- **55+ Reports** - Financial statements, tax reports, inventory, treasury, and operational reports
- **Multi-tenant Architecture** - Support for accountants managing multiple businesses
- **Government Compliance** - Full DGII tax reporting (606, 607, 608, 609 formats)
- **Electronic Invoicing** - e-CF integration with Alanube (certified DGII provider)
- **Real-time Dashboard** - Financial metrics and business intelligence
- **Automatic Classification** - AI classifies invoices as COMPRA (purchases) or GASTO (expenses)
- **Super Admin Debug Mode** - Raw OCR text + extracted data + DGII validation for troubleshooting

---

## Screenshots

### Dashboard
![Dashboard](docs/screenshots/01-dashboard.png)
*Real-time financial overview with key metrics and alerts*

### AI Assistant
![AI Assistant](docs/screenshots/02-ai-assistant.png)
*Ask questions in natural language, get instant answers*

### Batch OCR Scanner
![Batch OCR](docs/screenshots/escanear-masivo.png)
*Upload multiple invoices at once, automatic classification and extraction*

### Sales & Invoicing
![Sales](docs/screenshots/03-ventas-facturas.png)
*Complete invoicing system with electronic invoicing (e-CF)*

### OCR Invoice Scanner
![OCR Scanner](docs/screenshots/04-ocr-scanner.png)
*Upload any invoice photo, extract data automatically*

### Tax Reports (DGII 606)
![Tax Reports](docs/screenshots/05-reporte-606.png)
*One-click generation of government-required reports*

### Centro de Reportes (55 Reports)
![Centro Reportes](docs/screenshots/centro-reportes.png)
*Complete reporting center: financial, tax, inventory, treasury, operational*

### Accounting
![Accounting](docs/screenshots/06-contabilidad.png)
*Chart of accounts and journal entries*

### API Documentation
![API Docs](docs/screenshots/07-api-swagger.png)
*150+ RESTful endpoints with Swagger documentation*

### Client Management
![Clients](docs/screenshots/08-clientes.png)
*Complete CRM for managing customers and vendors*

### Treasury & Banking
![Treasury](docs/screenshots/09-tesoreria-bancos.png)
*Bank accounts, payments, and cash flow management*

### Authentication
![Login](docs/screenshots/10-login.png)
*Secure JWT-based authentication system*

---

## Tech Stack

### Backend
| Technology | Purpose |
|------------|---------|
| **FastAPI** | High-performance Python API framework |
| **SQLAlchemy 2.0** | Modern ORM with async support |
| **PostgreSQL** | Primary database (Supabase hosted) |
| **JWT + bcrypt** | Secure authentication |
| **Pydantic V2** | Data validation and serialization |

### Frontend
| Technology | Purpose |
|------------|---------|
| **Next.js 14** | React framework with App Router |
| **TypeScript** | Type-safe development |
| **Tailwind CSS** | Utility-first styling |
| **shadcn/ui** | Modern component library |
| **TanStack Query** | Server state management |

### AI & Integrations
| Technology | Purpose |
|------------|---------|
| **OpenAI GPT-4** | Conversational AI assistant |
| **Claude 3.5 Sonnet** | Structured invoice extraction (fallback) |
| **Google Gemini Vision** | OCR invoice scanning |
| **Alanube API** | Electronic invoicing (e-CF) |

### DevOps & Infrastructure
| Technology | Purpose |
|------------|---------|
| **Vercel** | Frontend hosting (edge network) |
| **Railway** | Backend hosting (auto-deploy) |
| **Supabase** | PostgreSQL database (AWS us-east-1) |
| **Docker** | Containerization for local dev |
| **GitHub Actions** | CI/CD automation |

---

## Features

### AI Assistant
The AI assistant understands natural language and has full context of your financial data:

```
User: "How much do I owe in taxes this month?"

AI: Based on your November transactions:
    • Sales Tax Collected: $2,512.00
    • Tax Credits (purchases): $1,847.00
    • Net Tax Due: $665.00

    You have 3 invoices pending registration that could
    increase your tax credits by approximately $120.
```

### Batch OCR Invoice Scanning
Upload multiple invoice photos or PDFs and get structured data instantly:

- **Supported formats**: JPG, PNG, PDF
- **Batch processing**: Upload 10+ invoices at once
- **Extracted fields**: Vendor info, tax ID (RNC), NCF, line items, totals, tax amounts
- **Accuracy**: 95%+ for standard invoices
- **Processing time**: < 3 seconds per invoice
- **Automatic classification**: AI determines COMPRA (purchase) vs GASTO (expense) based on NCF type
- **Super admin debug**: View raw OCR text, parsed data, and DGII validation

### 55+ Reports Center
Complete reporting suite organized by category:

#### Financial Reports (13)
- Balance Sheet (General, Comparative, Consolidated)
- Income Statement (Simple, Multi-period, Budget vs Actual)
- Cash Flow (Direct, Indirect, Detailed)
- Changes in Equity
- General Ledger
- Trial Balance
- Account Statement

#### Tax Reports (DGII) (4)
- **606**: Purchase transactions
- **607**: Sales transactions
- **608**: Voided invoices
- **609**: Foreign payments

#### Inventory Reports (15)
- Stock Valuation (FIFO, LIFO, Average)
- Stock Movement
- Kardex (by product, by location)
- Low Stock Alerts
- Expiring Products
- Dead Stock Analysis
- Inventory Turnover
- ABC Classification
- Physical Count Variance
- Supplier Performance

#### Treasury Reports (14)
- Cash Flow Projection
- Bank Reconciliation
- Payment Schedule
- Collection Schedule
- Accounts Receivable Aging
- Accounts Payable Aging
- Cash Position
- Daily Cash Report
- Payment Methods Analysis
- Bank Movement Detail

#### Operational Reports (9)
- Sales by Product/Service
- Sales by Customer
- Sales by Period
- Purchase Analysis
- Top Customers
- Top Products
- Sales Commission
- Profitability by Product
- Monthly KPIs

### Multi-tenant Support
Perfect for accounting firms managing multiple clients:

- **Role-based access**: Admin, Accountant, Viewer
- **Company switching**: Seamless navigation between client accounts
- **Audit trails**: Complete activity logging
- **Data isolation**: Strict separation between tenants
- **Module activation**: Enable/disable features per company

### Electronic Invoicing (e-CF)
Integrated with Alanube for DGII-certified electronic invoicing:

- **NCF Generation**: Automatic fiscal sequence management
- **e-CF Emission**: Generate electronic invoices compliant with DGII
- **Real-time Validation**: RNC validation against DGII database
- **Status Tracking**: Monitor invoice registration status
- **Automatic Retry**: Handle API timeouts and retries
- **Supported NCF Types**: B01/E31 (Credit), B02/E32 (Consumer), B14/E34 (Special Regime)

### Tax Compliance
One-click generation of required government reports with validation:

| Report | Description | Validation |
|--------|-------------|------------|
| **606** | Purchase transactions | RNC verification, NCF format |
| **607** | Sales transactions | NCF sequence validation |
| **608** | Voided invoices | NCF cross-check |
| **609** | Foreign payments | Amount thresholds |

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         FRONTEND                                 │
│                    Next.js 14 + TypeScript                       │
│              ┌─────────────────────────────────┐                │
│              │  Dashboard │ AI Chat │ Reports  │                │
│              │  OCR Batch │ e-CF │ Accounting  │                │
│              └─────────────────────────────────┘                │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                          BACKEND                                 │
│                     FastAPI + Python                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │   Auth   │  │ AI Agent │  │OCR Batch │  │ Reports  │        │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │  Sales   │  │ Purchases│  │ Treasury │  │Accounting│        │
│  │  (e-CF)  │  │  (OCR)   │  │  (Banks) │  │ (Entries)│        │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘        │
└─────────────────────────────────────────────────────────────────┘
                              │
          ┌───────────────────┼───────────────────┐
          ▼                   ▼                   ▼
    ┌──────────┐        ┌──────────┐        ┌──────────┐
    │PostgreSQL│        │ AI APIs  │        │ External │
    │(Supabase)│        │ GPT-4    │        │ Alanube  │
    │          │        │ Gemini   │        │ DGII API │
    └──────────┘        └──────────┘        └──────────┘
```

---

## API Endpoints

The backend exposes **150+ RESTful endpoints** organized by module:

| Module | Endpoints | Description |
|--------|-----------|-------------|
| `/auth` | 5 | Authentication & authorization |
| `/sales` | 18 | Invoices, NCF sequences, e-CF emission |
| `/purchases` | 12 | Vendor invoices, expenses |
| `/ocr` | 8 | Invoice scanning (single + batch) |
| `/accounting` | 17 | Chart of accounts, journal entries |
| `/treasury` | 19 | Banks, payments, cash flow |
| `/reports/fiscales` | 16 | DGII tax reports (606-609) |
| `/reports/financieros` | 22 | Financial statements |
| `/reports/inventario` | 28 | Inventory reports |
| `/reports/tesoreria` | 25 | Treasury reports |
| `/reports/operacionales` | 18 | Operational reports |
| `/ai` | 3 | AI assistant conversations |

**Total**: 150+ endpoints with full Swagger/OpenAPI documentation.

---

## Development Timeline

This project was built in **under 30 days** as a solo developer:

| Week | Deliverables |
|------|--------------|
| **Week 1** | Database schema, auth system, core API, multi-tenancy |
| **Week 2** | Frontend dashboard, sales/purchase modules, e-CF integration |
| **Week 3** | AI integration, OCR scanning, tax reports, batch OCR |
| **Week 4** | 55 reports center, module activation, polish, testing |

---

## Security

- **Authentication**: JWT tokens with refresh rotation
- **Password hashing**: bcrypt with salt rounds
- **Rate limiting**: Protection against brute force
- **Input validation**: Pydantic V2 models + Zod schemas
- **SQL injection**: ORM-only queries (SQLAlchemy)
- **XSS protection**: React automatic escaping
- **CORS**: Configurable allowed origins
- **Field-level encryption**: Sensitive data encrypted with Fernet (AES-128)
- **Super admin bypass**: Owner email bypasses all limits for testing

---

## Performance

- **API response time**: < 100ms average
- **OCR processing**: < 3 seconds per invoice
- **Batch OCR**: 10 invoices in ~20 seconds
- **AI responses**: < 2 seconds
- **Dashboard load**: < 1.5 seconds
- **Database queries**: Optimized with indexes and eager loading
- **Frontend caching**: TanStack Query with stale-while-revalidate
- **Railway timeout**: 120s keep-alive for long-running API calls

---

## Contact

**Paul** - Full-Stack Developer

- Expertise: Python, TypeScript, AI/ML integrations, SaaS architecture
- Available for: Freelance projects, consulting
- Timeline: Can deliver complex SaaS in 30 days
- Email: ranx043@gmail.com

---

## Documentation

Detailed technical documentation:

| Document | Description |
|----------|-------------|
| [Architecture](ARCHITECTURE.md) | System design, database schema, security layers |
| [AI Features](AI_FEATURES.md) | AI assistant, OCR pipeline, batch processing |
| [API Reference](API_REFERENCE.md) | 150+ endpoints with examples |
| [Project Management](PROJECT_MANAGEMENT.md) | PMI framework, sprints, WBS, risk management |

---

## Project Stats

```
┌────────────────────────────────────────────────────────────────┐
│                    PROJECT METRICS                              │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│   📁 Backend (Python)                                          │
│      ├── 15 modules                                            │
│      ├── 150+ API endpoints                                    │
│      ├── 32 database migrations                                │
│      ├── 55+ test files                                        │
│      └── 93 Python files                                       │
│                                                                 │
│   🎨 Frontend (TypeScript)                                     │
│      ├── 58 pages                                              │
│      ├── 102 components                                        │
│      ├── Full dark/light theme                                 │
│      └── 96 TSX files                                          │
│                                                                 │
│   🤖 AI Integration                                            │
│      ├── OpenAI GPT-4 (Assistant)                              │
│      ├── Google Gemini Vision (OCR)                            │
│      ├── Claude 3.5 Sonnet (Parser fallback)                   │
│      └── 20+ function tools                                    │
│                                                                 │
│   📊 Database                                                  │
│      ├── 35+ tables                                            │
│      ├── Multi-tenant isolation                                │
│      ├── Row-level security                                    │
│      └── Field-level encryption                                │
│                                                                 │
│   📈 Reports                                                   │
│      ├── 55 total reports                                      │
│      │   ├── 13 Financial                                      │
│      │   ├── 4 Fiscal (DGII)                                   │
│      │   ├── 15 Inventory                                      │
│      │   ├── 14 Treasury                                       │
│      │   └── 9 Operational                                     │
│      └── Export formats: Excel, PDF, CSV                       │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

---

## Why This Project Matters

This project demonstrates:

1. **Full-Stack Expertise** - Complete system from database to UI
2. **AI Integration** - Production-ready LLM with function calling and OCR
3. **Domain Complexity** - Tax regulations, accounting standards, compliance
4. **Speed of Delivery** - Complex SaaS in under 30 days
5. **Enterprise Patterns** - Multi-tenancy, audit trails, security, scalability
6. **Batch Processing** - Efficient handling of bulk operations
7. **Government Integration** - Electronic invoicing with certified providers

---

## Production Deployment

The system is currently deployed and running:

- **Frontend**: https://frontend-eta-one-33.vercel.app (Vercel Edge Network)
- **Backend**: https://ecosistemacontable-production.up.railway.app (Railway)
- **Database**: Supabase PostgreSQL (AWS us-east-1)

**Auto-deployment**: Both frontend and backend auto-deploy on push to `main` branch.

---

## License

This project is proprietary software. Screenshots and documentation may be shared for portfolio purposes.

---

*Built with modern technologies and AI-powered features. Designed for scalability and compliance with Dominican Republic tax regulations.*
