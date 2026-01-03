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
    - flujo_caja: Cash flow statement
    - estado_cambios_patrimonio: Changes in equity

    Formats: json, csv, txt (DGII format), excel, pdf
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

## Batch OCR Invoice Scanning

### Batch Processing Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       BATCH OCR PROCESSING FLOW                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   User uploads 10 invoices (JPG/PNG/PDF)                                    │
│                          │                                                   │
│                          ▼                                                   │
│   ┌──────────────────────────────────────────┐                              │
│   │      PARALLEL PROCESSING QUEUE            │                              │
│   │  • Extract images from PDFs               │                              │
│   │  • Validate file formats                  │                              │
│   │  • Check file sizes                       │                              │
│   └──────────────────────────────────────────┘                              │
│                          │                                                   │
│           ┌──────────────┼──────────────┐                                   │
│           ▼              ▼              ▼                                    │
│   ┌─────────────┐ ┌─────────────┐ ┌─────────────┐                          │
│   │  Invoice 1  │ │  Invoice 2  │ │  Invoice N  │                          │
│   │   GEMINI    │ │   GEMINI    │ │   GEMINI    │                          │
│   │   VISION    │ │   VISION    │ │   VISION    │                          │
│   └─────────────┘ └─────────────┘ └─────────────┘                          │
│           │              │              │                                    │
│           ▼              ▼              ▼                                    │
│   ┌─────────────────────────────────────────┐                               │
│   │      REGEX PARSER                       │                               │
│   │  • Extract RNC (regex patterns)         │                               │
│   │  • Extract NCF (format validation)      │                               │
│   │  • Extract amounts (decimal patterns)   │                               │
│   │  • Extract line items                   │                               │
│   └─────────────────────────────────────────┘                               │
│                          │                                                   │
│                          ▼                                                   │
│   ┌─────────────────────────────────────────┐                               │
│   │     AI CLASSIFICATION                   │                               │
│   │                                          │                               │
│   │  NCF Type Analysis:                     │                               │
│   │  • B01/E31 → COMPRA (Credit Invoice)    │                               │
│   │  • B02/E32 → GASTO (Consumer Invoice)   │                               │
│   │  • B14/E34 → COMPRA (Special Regime)    │                               │
│   │  • B15/E35 → COMPRA (Government)        │                               │
│   └─────────────────────────────────────────┘                               │
│                          │                                                   │
│                          ▼                                                   │
│   ┌─────────────────────────────────────────┐                               │
│   │     DGII RNC VALIDATION                 │                               │
│   │  • Query DGII API for RNC               │                               │
│   │  • Validate name match                  │                               │
│   │  • Mark as "validated" or "failed"      │                               │
│   └─────────────────────────────────────────┘                               │
│                          │                                                   │
│                          ▼                                                   │
│   ┌─────────────────────────────────────────┐                               │
│   │     QUALITY VALIDATION                  │                               │
│   │                                          │                               │
│   │  Mark as FAILED if:                     │                               │
│   │  • Monto total = 0 or missing           │                               │
│   │  • Proveedor name is generic:           │                               │
│   │    - "cliente"                           │                               │
│   │    - "consumidor final"                  │                               │
│   │    - "publico general"                   │                               │
│   │  • Missing critical fields              │                               │
│   └─────────────────────────────────────────┘                               │
│                          │                                                   │
│                          ▼                                                   │
│   ┌─────────────────────────────────────────┐                               │
│   │     RESULTS TABLE                       │                               │
│   │  • SUCCESS: Green, "Guardar"            │                               │
│   │  • FAILED: Red, "Reintentar"            │                               │
│   │  • Super Admin: "Debug" button          │                               │
│   └─────────────────────────────────────────┘                               │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Super Admin Debug Mode

For troubleshooting OCR extraction issues, super admin users see a **Debug** button that reveals:

```
┌─────────────────────────────────────────────────────────────┐
│                    DEBUG MODAL                               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  📄 TEXTO OCR CRUDO (Google Vision)                         │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ COMERCIAL LOPEZ                                        │ │
│  │ RNC: 101-23456-7                                       │ │
│  │ NCF: E310000000123                                     │ │
│  │ Fecha: 15/11/2024                                      │ │
│  │ Subtotal: RD$5,000.00                                  │ │
│  │ ITBIS 18%: RD$900.00                                   │ │
│  │ TOTAL: RD$5,900.00                                     │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  🔍 DATOS EXTRAÍDOS (Regex Parser)                         │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ RNC Proveedor: 101-23456-7                             │ │
│  │ Nombre Proveedor: COMERCIAL LOPEZ                      │ │
│  │ NCF: E310000000123                                     │ │
│  │ Fecha Emisión: 2024-11-15                              │ │
│  │ Monto Total: 5900.00                                   │ │
│  │ Monto ITBIS: 900.00                                    │ │
│  │                                                         │ │
│  │ Items Detectados:                                       │ │
│  │ • Producto A | Cant: 2 | Precio: $2,500 | Total: $5k  │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ✅ VALIDACIÓN DGII (RNC)                                   │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Estado: ACTIVO                                          │ │
│  │ Nombre Registrado: COMERCIAL LOPEZ SRL                 │ │
│  │ Match: ✅ Coincide (similarity: 0.95)                   │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Debug mode only visible for**: Super Admin users (configured via `SUPER_ADMIN_EMAILS` environment variable)

### Processing Pipeline Comparison

| Step | Single OCR | Batch OCR |
|------|-----------|-----------|
| **Upload** | 1 file | 10+ files |
| **OCR Provider** | Google Vision | Google Vision (parallel) |
| **Parsing** | Regex patterns | Regex patterns (concurrent) |
| **Classification** | Manual | Automatic (NCF-based) |
| **DGII Validation** | RNC lookup | Batch RNC lookup |
| **Quality Check** | Manual | Automatic (generic names) |
| **Processing Time** | ~3 seconds | ~2 seconds per invoice |
| **Debug Mode** | Not available | Super Admin only |

### Extracted Fields (Batch OCR)

| Field | Example | Validation | Source |
|-------|---------|------------|--------|
| `rnc_proveedor` | 101-23456-7 | Luhn checksum | Regex pattern |
| `nombre_proveedor` | COMERCIAL LOPEZ | Not generic | OCR text |
| `ncf` | E310000000123 | DGII format | Regex + validation |
| `fecha_emision` | 2024-11-15 | Valid date | Date parser |
| `monto_total` | 5900.00 | > 0 required | Decimal regex |
| `monto_itbis` | 900.00 | 18% validation | Decimal regex |
| `items[]` | Product details | Array not empty | Line item parser |
| `tipo_documento` | COMPRA/GASTO | Auto-classified | NCF type logic |
| `dgii_validacion` | {status, nombre} | RNC match | DGII API |

### Classification Logic

```python
def classify_invoice(ncf: str) -> str:
    """
    Automatic classification based on NCF type.

    COMPRA (Purchase - has tax credit):
    - B01/E31: Credit Fiscal (between taxpayers)
    - B14/E34: Special Regime (zonas francas)
    - B15/E35: Government purchases
    - B16: Export

    GASTO (Expense - no tax credit):
    - B02/E32: Consumer Final (retail purchases)
    """
    if ncf.startswith(("B01", "E31", "B14", "E34", "B15", "E35", "B16")):
        return "COMPRA"
    elif ncf.startswith(("B02", "E32")):
        return "GASTO"
    else:
        return "GASTO"  # Default fallback
```

### Accuracy Metrics

| Document Type | Accuracy | Processing Time | Batch Capable |
|---------------|----------|-----------------|---------------|
| Standard invoices | 95%+ | 1-2 seconds | Yes |
| Handwritten | 75-85% | 2-3 seconds | Yes |
| Poor quality scans | 60-75% | 3-5 seconds | Yes |
| PDF (digital) | 99%+ | < 1 second | Yes |
| Batch (10 invoices) | 95%+ | ~20 seconds total | Yes |

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

### Batch OCR Upload
```
User: "Sube 15 facturas de compras de esta semana"

AI: Procesando lote de 15 facturas...

✅ **Resultados del Lote**
• Exitosas: 13 facturas
• Fallidas: 2 facturas

📊 **Resumen Automático**
• COMPRAS (crédito fiscal): 10 facturas → RD$125,450.00
• GASTOS (consumidor final): 3 facturas → RD$8,230.00
• Total ITBIS: RD$24,042.40

⚠️ **Atención Requerida**
• Factura #3: Proveedor genérico "Consumidor Final" → Revisar
• Factura #7: RNC no validado → Verificar con DGII

[Ver Detalles] [Guardar Todo] [Revisar Fallidas]
```

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

### OCR Quality Improvement
```
🔍 OCR Tip: 3 of your recent invoices had low confidence scores.
Consider:
• Using better lighting when taking photos
• Scanning documents instead of photographing
• Uploading PDF versions when available

[View Tips] [See Examples]
```

## Performance Optimizations

### Caching Strategy
- **RNC Validation**: Cached for 24 hours per RNC
- **Tax Calculations**: Cached until data changes
- **Report Generation**: Cached per period + empresa

### Parallel Processing
- **Batch OCR**: Up to 10 concurrent Gemini API calls
- **Database Queries**: Connection pooling + eager loading
- **Report Generation**: Async task queues for large datasets

### Error Handling
- **Gemini API Failures**: Automatic retry with exponential backoff
- **DGII API Timeouts**: Graceful degradation (validation skipped)
- **Invalid Data**: Detailed error messages for user correction
