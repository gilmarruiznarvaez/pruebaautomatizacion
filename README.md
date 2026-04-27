# Cold Outreach Automation (n8n)

Workflow de prospección en frío: extrae leads de Apollo.io, redacta correo + propuesta con Gemini vía OpenRouter, genera PDF y envía por Gmail con el PDF adjunto.

## Importar en n8n

1. En tu instancia de n8n: **Workflows → Import from File** y selecciona `cold-outreach-workflow.json`.
2. Configura las credenciales (abajo).
3. Ajusta `Set Campaign Vars` con tu industria objetivo, nombre, empresa y enlace de calendario.
4. Ejecuta con **Test workflow**.

## Credenciales necesarias

| Nodo | Credencial | Cómo configurarla |
|------|------------|-------------------|
| `Apollo - Search Leads` | Variable de entorno `APOLLO_API_KEY` en n8n, o reemplaza `={{$env.APOLLO_API_KEY}}` por tu API key directamente en el header `X-Api-Key`. | Apollo → Settings → Integrations → API. |
| `OpenRouter - Gemini 3 Pro` | **HTTP Header Auth** con `Authorization: Bearer <OPENROUTER_KEY>`. | openrouter.ai → Keys. |
| `Generate PDF (PDFShift)` | **HTTP Basic Auth** — usuario: `api`, contraseña: tu API key de PDFShift. | pdfshift.io. |
| `Gmail - Send Email` | OAuth2 de Gmail nativo de n8n. | Crea credencial `Gmail OAuth2` y autoriza. |

## Flujo

```
Manual Trigger
  → Set Campaign Vars
  → Apollo: search por industria + cargos C-level
  → Split Out → Filter (con email) → Normalize Lead
  → OpenRouter (Gemini 3 Pro) — devuelve JSON {subject, email_body, proposal_title, proposal_text}
  → Code: arma HTML de propuesta
  → PDFShift: HTML → PDF binario
  → Gmail: envía con asunto + cuerpo HTML + PDF adjunto
```

## Notas

- Si tu modelo en OpenRouter tiene otro ID (`google/gemini-3-pro`, `google/gemini-2.5-pro`...), cámbialo en el `jsonBody` del nodo OpenRouter.
- Cold outreach está sujeto a GDPR/CAN-SPAM. Incluye opt-out y datos del remitente en producción.
