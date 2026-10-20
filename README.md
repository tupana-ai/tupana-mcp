# Tupana MCP

Servidor [MCP](https://modelcontextprotocol.io) remoto de **Tupana**: conecta Claude, ChatGPT, Cursor o cualquier agente compatible con tu facturación electrónica chilena (SII), cobranza, conciliación bancaria y contabilidad.

```
https://mcp.tupana.ai/mcp
```

- **Transporte:** Streamable HTTP.
- **Autenticación:** OAuth 2.1 con PKCE y registro dinámico de clientes. Al conectar, el cliente te abre el navegador para que inicies sesión en Tupana y autorices el acceso. No necesitas copiar ninguna API key.
- **Scopes:** `tupana:read` y `tupana:write`.
- **Privacidad:** cada usuario ve solo las empresas a las que tiene acceso en Tupana; todas las herramientas resuelven documentos y empresas dentro de ese alcance.
- **Seguridad:** las acciones irreversibles (emitir al SII, enviar correos, conciliar) están diseñadas para pedir confirmación. La emisión es un paso separado: el agente crea un borrador, te muestra la vista previa en PDF y solo emite con `emit_document` después de tu visto bueno.

¿Aún no tienes cuenta? Créala en [developers.tupana.ai](https://developers.tupana.ai).

## Qué puedes pedirle

- "¿Cuánto facturé este mes y quiénes son mis 10 mayores clientes?"
- "Hazle una factura a 76.123.456-7 por 3 horas de asesoría a $45.000 + IVA, con referencia a la OC 4500123."
- "¿Qué facturas tengo pendientes de cobro con más de 30 días?"
- "Concilia los movimientos del banco de ayer con las facturas que calcen."
- "Muéstrame el resumen de IVA y el F29 de septiembre."
- "Agrégale facturación electrónica chilena a este SaaS." (el MCP también enseña a construir la integración)

## Conectar

### Claude (claude.ai y Claude Desktop)

1. Ve a **Settings → Connectors → Add custom connector**.
2. Nombre: `Tupana`. URL: `https://mcp.tupana.ai/mcp`.
3. Haz clic en **Connect**, inicia sesión en Tupana y autoriza.

En planes Team y Enterprise, un administrador puede agregar el conector para toda la organización.

### Claude Code

```bash
claude mcp add --transport http tupana https://mcp.tupana.ai/mcp
```

Luego, dentro de Claude Code, ejecuta `/mcp`, elige `tupana` y completa el login en el navegador. Para compartirlo con tu equipo, agrégalo con `--scope project` (queda en `.mcp.json`).

### Cursor

En `~/.cursor/mcp.json` (global) o `.cursor/mcp.json` (proyecto):

```json
{
  "mcpServers": {
    "tupana": {
      "url": "https://mcp.tupana.ai/mcp"
    }
  }
}
```

Cursor detecta que el servidor pide OAuth y te muestra el botón para iniciar sesión.

### ChatGPT

1. Activa **Settings → Apps & Connectors → Advanced settings → Developer mode** (disponible según tu plan).
2. En **Apps & Connectors**, crea un conector nuevo con la URL `https://mcp.tupana.ai/mcp` y autenticación **OAuth**.
3. Autoriza con tu cuenta de Tupana y actívalo en el chat.

### VS Code (GitHub Copilot)

En `.vscode/mcp.json`:

```json
{
  "servers": {
    "tupana": {
      "type": "http",
      "url": "https://mcp.tupana.ai/mcp"
    }
  }
}
```

### Otros clientes

Cualquier cliente con soporte de MCP remoto (Streamable HTTP + OAuth 2.1) funciona con la misma URL. El servidor publica sus metadatos en:

- `https://mcp.tupana.ai/.well-known/oauth-protected-resource`
- `https://mcp.tupana.ai/.well-known/oauth-authorization-server`

Si tu cliente solo soporta stdio, usa un puente como [`mcp-remote`](https://www.npmjs.com/package/mcp-remote):

```json
{
  "mcpServers": {
    "tupana": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.tupana.ai/mcp"]
    }
  }
}
```

### Desde tu propio agente

En [tupana-ai/tupana-examples](https://github.com/tupana-ai/tupana-examples/tree/main/ai-agent-mcp) hay un agente en Python (API de Claude + cliente MCP) que factura pidiéndote confirmación antes de emitir.

## Herramientas

### Documentos y facturación

| Herramienta | Qué hace |
| --- | --- |
| `get_documents` | Consulta documentos tributarios (facturas, boletas, notas de crédito…). |
| `search_documents` | Busca documentos emitidos por cliente, monto o folio. |
| `get_document_link` | Link (PDF) de un documento emitido. |
| `create_sale_invoice` | Crea un **borrador** de factura afecta (33) o exenta (34), con referencias (OC 801, HES). |
| `create_purchase_invoice` | Crea un borrador de factura de compra (46). |
| `create_export_invoice` | Crea un borrador de factura de exportación (110). |
| `create_credit_note` | Crea una nota de crédito (61) que referencia una factura (anulación o corrección). |
| `get_document_preview` | Vista previa en PDF de un borrador, para que la revises. |
| `emit_document` | **Emite al SII** un borrador ya revisado. Exige vista previa previa. |
| `send_document_email` | Reenvía un documento emitido por correo. |
| `create_scheduled_invoice` | Crea una factura programada (recurrente). |
| `get_scheduled_documents` | Lista los documentos programados. |
| `parse_document_to_invoice` | Extrae ítems de una orden de compra o PDF para facturarla. |

### Clientes, proveedores y empresas

| Herramienta | Qué hace |
| --- | --- |
| `get_entity_info` | Tus empresas en Tupana. |
| `find_clients` | Busca clientes (receptores). |
| `enrich_client_from_rut` | Datos del contribuyente en el SII a partir de su RUT. |
| `get_suppliers` | Proveedores según las facturas de compra recibidas. |
| `get_top_clients` | Ranking de clientes por monto o cantidad facturada. |
| `get_top_products` | Ranking de productos o ítems vendidos. |

### Reportes, libros e impuestos

| Herramienta | Qué hace |
| --- | --- |
| `report_documents` | Reportes y análisis sobre documentos. |
| `get_monthly_revenue` | Serie mensual de ventas. |
| `get_sii_books` | Libros de compra y venta del SII. |
| `get_f29_summary` | Resumen del F29 del período. |
| `get_tax_summary` | Resumen de IVA crédito y débito fiscal. |
| `generate_chart` | Genera un gráfico a partir de datos. |
| `generate_multi_series_chart` | Gráfico con varias series (ej. ventas vs. compras). |

### Cobranza y conciliación bancaria

| Herramienta | Qué hace |
| --- | --- |
| `get_pending_invoices` | Facturas pendientes de pago (cuentas por cobrar y por pagar). |
| `get_bank_movements` | Movimientos bancarios y su estado de conciliación. |
| `get_payment_documents` | Documentos de pago (tipo 100). |
| `get_reconciliation_suggestions` | Sugerencias de conciliación con puntaje automático. |
| `reconcile_movement` | Concilia un movimiento con una o más facturas. |
| `get_reconciliations` | Historial de conciliaciones. |

### Contabilidad

| Herramienta | Qué hace |
| --- | --- |
| `get_journal_entries` | Libro diario: asientos en un rango de fechas. |
| `get_ledger` | Libro mayor de una cuenta, con saldo acumulado. |
| `get_balance_sheet` | Balance por sección (activo, pasivo, patrimonio). |
| `get_income_statement` | Estado de resultados. |
| `get_chart_of_accounts` | Plan de cuentas. |
| `get_account_balance` | Saldo actual de una cuenta. |
| `create_journal_entry` | Crea un asiento contable manual. |

### Base pública de empresas del SII

| Herramienta | Qué hace |
| --- | --- |
| `search_sii_companies` | Busca empresas de Chile en la nómina pública del SII. |
| `find_sii_prospects` | Encuentra clientes potenciales por rubro, región y tamaño. |
| `find_growing_sii_companies` | Empresas que crecieron entre dos años. |
| `get_sii_company_history` | Trayectoria de una empresa en la nómina. |
| `get_sii_market_summary` | Tamaño de un mercado (empresas y trabajadores). |
| `get_sii_companies_metadata` | Filtros disponibles (años, tramos, regiones, rubros). |

### Construir facturación en tu SaaS

| Herramienta | Qué hace |
| --- | --- |
| `get_capabilities` | Qué puede hacer Tupana y con qué herramienta empezar. |
| `get_invoice_flow` | El flujo completo de facturación, paso a paso. |
| `get_invoice_ui_spec` | Especificación del frontend (campos, tipos, condiciones). |
| `get_invoice_validation_schema` | Reglas de validación del borrador por paso y campo. |
| `validate_invoice_draft` | Valida un borrador con las mismas reglas de la API. |
| `get_customer_lookup_flow` | De RUT a razón social, actividades y direcciones. |
| `get_invoice_item_rules` | Reglas de las líneas de la factura. |
| `get_invoice_reference_rules` | Reglas de las referencias según el tipo de documento. |
| `get_invoice_preview_flow` | Cómo pedir y mostrar la vista previa. |
| `get_invoice_issuance_flow` | Cómo emitir, qué estados hay y qué devuelve. |
| `get_async_strategy` | Webhook + WebSocket o polling, según tu stack. |
| `get_integration_setup` | API key, enrolamiento, webhooks y sandbox. |
| `get_invoice_contract_findings` | Qué reglas están confirmadas en código y cuáles son declaradas. |
| `get_error_resolution` | Qué significa un error de Tupana o del SII y qué hacer. |
| `get_design_system` | Tokens del design system de Tupana. |

### Conexión de empresas y navegación

| Herramienta | Qué hace |
| --- | --- |
| `get_tupana_connect_guide` | Cómo enrolar una empresa con Tupana Connect. |
| `open_tupana_connect` | Abre Tupana Connect y devuelve el link (facturación SII, lectura SII o banco). |
| `get_app_link` | Link directo a la pantalla de la app para lo que se hace por interfaz. |
| `get_app_guide` | Guía de navegación de la app. |
| `list_capabilities` | Responde "¿qué puedes hacer?". |
| `how_to` | Tutorial paso a paso para una tarea. |

> La lista refleja el servidor al momento de publicar este repositorio. La fuente de verdad es `tools/list` del propio servidor.

## Directorios

- **MCP Registry oficial:** [`server.json`](server.json) (`io.github.tupana-ai/tupana`). Para publicarlo o actualizarlo, ejecuta el workflow **Publicar en el MCP Registry** (Actions → Run workflow), que se autentica con GitHub OIDC; o desde tu máquina:

  ```bash
  brew install mcp-publisher
  mcp-publisher login github   # con una cuenta miembro de la organización tupana-ai
  mcp-publisher publish
  ```

- **Smithery:** se publica por URL en [smithery.ai/new](https://smithery.ai/new) (o `smithery mcp publish "https://mcp.tupana.ai/mcp" -n @tupana/tupana`); no requiere archivo de configuración en el repositorio.

## Recursos

- Documentación: [docs.tupana.ai](https://docs.tupana.ai)
- Portal de desarrolladores: [developers.tupana.ai](https://developers.tupana.ai)
- SDKs: [Node.js](https://github.com/tupana-ai/tupana-node) · [Python](https://github.com/tupana-ai/tupana-python)
- Ejemplos: [tupana-ai/tupana-examples](https://github.com/tupana-ai/tupana-examples)
- Soporte: [contacto@tupana.ai](mailto:contacto@tupana.ai)

## Licencia

La documentación y los archivos de este repositorio están bajo licencia [MIT](LICENSE). El servidor MCP es un servicio alojado por Tupana.

---

## English summary

Remote MCP server for **Tupana** — Chilean electronic invoicing (SII), collections, bank reconciliation and accounting — at `https://mcp.tupana.ai/mcp` (Streamable HTTP, OAuth 2.1 with PKCE and dynamic client registration; scopes `tupana:read` / `tupana:write`). Add it as a custom connector in Claude, with `claude mcp add --transport http tupana https://mcp.tupana.ai/mcp` in Claude Code, via `mcp.json` in Cursor or VS Code, or as a developer-mode connector in ChatGPT. Irreversible actions (issuing to the SII, sending email, reconciling) are designed to require confirmation: the agent creates a draft, shows a PDF preview and only issues with `emit_document` after you approve. Registry metadata lives in [`server.json`](server.json) (`io.github.tupana-ai/tupana`).
