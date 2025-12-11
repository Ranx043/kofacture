# KOFACTURE - AI Features Deep Dive

## AI Assistant Architecture

The AI assistant uses a **ReAct (Reasoning + Acting) pattern** with function calling to perform complex accounting tasks.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           AI ASSISTANT FLOW                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   User: "How much ITBIS do I owe this month?"                               │
│                          │                                                   │
│                          ▼                                                   │
│   ┌──────────────────────────────────────────┐                              │
│   │         CONTEXT INJECTION                 │                              │
│   │  • User role & permissions                │                              │
│   │  • Current empresa context                │                              │
│   │  • Available tools                        │                              │
│   │  • Conversation history                   │                              │
│   └──────────────────────────────────────────┘                              │
│                          │                                                   │
│                          ▼                                                   │
│   ┌──────────────────────────────────────────┐                              │
│   │           LLM REASONING                   │                              │
│   │  "I need to calculate ITBIS. Let me:     │                              │
│   │   1. Query sales ITBIS for this month    │                              │
│   │   2. Query purchase ITBIS for this month │                              │
│   │   3. Calculate the difference"           │                              │
│   └──────────────────────────────────────────┘                              │
│                          │                                                   │
│                          ▼                                                   │
│   ┌──────────────────────────────────────────┐                              │
│   │         FUNCTION CALLING                  │                              │
│   │                                           │                              │
│   │  Tool: query_database                     │                              │
│   │  Args: {                                  │                              │
│   │    "query_type": "itbis_summary",        │                              │
│   │    "period": "2024-11",                  │                              │
│   │    "empresa_id": "uuid..."               │                              │
│   │  }                                        │                              │
│   └──────────────────────────────────────────┘                              │
│                          │                                                   │
│                          ▼                                                   │
│   ┌──────────────────────────────────────────┐                              │
│   │        TOOL EXECUTION                     │                              │
│   │                                           │                              │
│   │  Result: {                                │                              │
│   │    "itbis_ventas": 45230.00,             │                              │
│   │    "itbis_compras": 32150.00,            │                              │
│   │    "itbis_neto": 13080.00                │                              │
│   │  }                                        │                              │
│   └──────────────────────────────────────────┘                              │
│                          │                                                   │
│                          ▼                                                   │
│   ┌──────────────────────────────────────────┐                              │
│   │       RESPONSE GENERATION                 │                              │
│   │                                           │                              │
│   │  "Based on your November transactions:    │                              │
│   │   • ITBIS Ventas: RD$45,230.00           │                              │
│   │   • ITBIS Compras: RD$32,150.00          │                              │
│   │   • ITBIS a Pagar: RD$13,080.00"         │                              │
│   └──────────────────────────────────────────┘                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Available AI Tools

### 1. Database Query Tool
```python
@tool
def query_database(query_type: str, params: dict) -> dict:
    """
    Execute predefined safe queries against the database.

    Supported query_types:
    - sales_summary: Total sales by period
    - purchase_summary: Total purchases by period
    - itbis_summary: ITBIS collected vs paid
    - customer_balance: Outstanding customer balances
    - vendor_balance: Outstanding vendor balances
    - cash_flow: Cash flow summary
    - aging_report: Accounts receivable/payable aging
    """
```

### 2. Report Generation Tool
```python
@tool
def generate_report(report_type: str, period: str, format: str) -> dict:
    """
    Generate DGII-compliant reports.

    Supported report_types:
    - dgii_606: Purchase report
    - dgii_607: Sales report
    - dgii_608: Voided invoices
    - dgii_609: Foreign payments
    - balance_general: Balance sheet
    - estado_resultados: Income statement

    Formats: json, csv, txt (DGII format)
    """
```

### 3. Business Action Tool
```python
@tool
def execute_action(action_type: str, params: dict) -> dict:
    """
    Execute business actions with user confirmation.

    Supported actions:
    - create_invoice: Create draft invoice
    - register_payment: Register payment received
    - register_expense: Register expense
    - send_reminder: Send payment reminder email
    """
```

## Example Conversations

### Tax Calculation
```
User: "¿Cuánto ITBIS debo pagar este mes?"

AI: Basándome en tus transacciones de noviembre 2024:

📊 **Resumen ITBIS**
• ITBIS Ventas: RD$45,230.00
• ITBIS Compras: RD$32,150.00
• **ITBIS a Pagar: RD$13,080.00**

💡 Tienes 3 facturas de compra pendientes de registrar que podrían
   aumentar tu crédito fiscal en aproximadamente RD$2,400.

¿Deseas que genere el reporte 606 para este período?
```

### Invoice Creation
```
User: "Crea una factura para Comercial López por RD$50,000 + ITBIS"

AI: Voy a crear la factura con los siguientes datos:

📄 **Nueva Factura**
• Cliente: Comercial López, S.R.L.
• RNC: 101-23456-7
• Subtotal: RD$50,000.00
• ITBIS (18%): RD$9,000.00
• **Total: RD$59,000.00**
• NCF: E310000000045 (próximo disponible)

¿Confirmo la creación de esta factura?

[Confirmar] [Modificar] [Cancelar]
```

### Report Generation
```
User: "Genera el 607 de octubre"

AI: Generando Formato 607 - Ventas de Octubre 2024...

✅ **Reporte 607 Generado**
• Período: 202410
• Registros: 47 facturas
• Total Ventas: RD$2,450,890.00
• Total ITBIS: RD$441,160.20

📁 Archivo: 607_101234567_202410.txt

El archivo está listo para subir al portal de la DGII.

[Descargar] [Vista Previa] [Enviar por Email]
```

## OCR Invoice Scanning

### Processing Pipeline

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   UPLOAD     │────►│  PREPROCESS  │────►│    OCR       │────►│   EXTRACT    │
│  (Image/PDF) │     │  (Enhance)   │     │  (Gemini)    │     │   (Parse)    │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
                                                                      │
                                                                      ▼
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   CONFIRM    │◄────│   DISPLAY    │◄────│   VALIDATE   │◄────│  STRUCTURE   │
│   (User)     │     │   (Preview)  │     │   (Rules)    │     │   (JSON)     │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
```

### Extracted Fields

| Field | Example | Validation |
|-------|---------|------------|
| `rnc_emisor` | 101-23456-7 | Luhn checksum |
| `ncf` | E310000000123 | DGII format |
| `fecha` | 2024-11-15 | Valid date |
| `subtotal` | 5000.00 | Numeric |
| `itbis` | 900.00 | 18% of subtotal |
| `total` | 5900.00 | subtotal + itbis |
| `lineas[]` | Product details | Array of items |

### Gemini Vision Prompt
```
Analyze this Dominican Republic invoice image and extract:
1. Vendor information (RNC, name, address)
2. Invoice number (NCF format: B01/E31/E32, etc.)
3. Date
4. Line items with quantities, descriptions, unit prices
5. Subtotal, ITBIS (18% tax), and Total
6. Customer information if visible

Return as structured JSON. Flag any fields you're uncertain about.
```

### Accuracy Metrics

| Document Type | Accuracy | Processing Time |
|---------------|----------|-----------------|
| Standard invoices | 95%+ | 1-2 seconds |
| Handwritten | 75-85% | 2-3 seconds |
| Poor quality scans | 60-75% | 3-5 seconds |
| PDF (digital) | 99%+ | < 1 second |

## Smart Suggestions

The AI proactively suggests actions based on patterns:

### Cash Flow Alerts
```
⚠️ Alert: Based on your payment patterns, you may have a cash
shortage next week. Consider:
• Following up on RD$125,000 in overdue receivables
• Negotiating extended terms with top 3 vendors
```

### Tax Optimization
```
💡 Tax Tip: You have RD$45,000 in unregistered purchase invoices
from this month. Registering them before month-end could reduce
your ITBIS payment by RD$8,100.
```

### Compliance Reminders
```
📅 Reminder: DGII Report 606/607 submission deadline is in 3 days.
Your reports are ready for review. [Generate Reports]
```
