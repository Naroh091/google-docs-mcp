# Migración a Workspace MCP oficiales (enforcement 2027-02-01)

Fecha: 2026-11-25. Origen: aviso Google 24-nov-2026 (tiers Standard/Scaled + MCP obligatorio).

## Qué cambia

- Desde 2026-11-24: tiers Standard (gratis, con cuotas) y Scaled (más cuota, cobro desde 2027-04-01 solo por uso sobre franquicia diaria). Gmail, Drive y Calendar primero.
- Cuota Standard conocida: 5000 req/día por proyecto/producto (identificada en paso anterior; uso real de clawdbot-488800 pendiente de dashboard).
- Desde 2027-02-01: enforcement técnico. Los agentes IA deben usar MCP oficiales o CLI de Google, no REST directo. Los MCP comunitarios que envuelven REST (este repo, gmail-mcp, google-calendar-mcp) quedarán bloqueados salvo migración.
- Addendum AI Tools: prohíbe almacenamiento prolongado de datos derivados, entrenamiento de modelos con datos Workspace y scraping de APIs. Retención limitada y borrado al desconectar el usuario.
- OAuth: los MCP oficiales rechazan clientes no agénticos desde 11/12-nov-2026. Hay que crear un cliente nuevo marcado como "Use this client for an AI-powered agent". No reutilizar el mismo client para REST no-agente y MCP agente; separar.

## Inventario verificado (solo lectura, máquina local)

1. `gmail-maldita / gmail-naroh091 / gmail-yo-naroh` → `@artymclabin/gmail-mcp@1.2.3` (comunitario, `googleapis` = REST directo). Migrar a `https://gmailmcp.googleapis.com/mcp/v1`.
2. `google-calendar` → `@cocal/google-calendar-mcp@2.6.3` (comunitario, REST directo). Migrar a `https://calendarmcp.googleapis.com/mcp/v1`.
3. `google-docs / google-docs-maldita / google-docs-naroh091` → `/home/naroh/work/google-docs-mcp/dist/index.js` (este repo, `googleapis` = REST directo en `src/clients.ts`, `src/tools/*`, `googleDocsApiHelpers.ts`, `googleSheetsApiHelpers.ts`). Migrar a:
   - `https://docsmcp.googleapis.com/mcp/v1`
   - `https://drivemcp.googleapis.com/mcp/v1`
   - `https://sheetsmcp.googleapis.com/mcp/v1`
   - `https://slidesmcp.googleapis.com/mcp/v1`
4. Skills con REST directo (no MCP):
   - `productivity/google-workspace/scripts/google_api.py`, `google_api_multi.py`, `setup.py`, `setup_multi.py`, `docs-comment-monitor.py`
   - `productivity/email-accounts/scripts/gmail_search.py`, `gmail_send.py`, `gmail_get_full.py`, `test_tokens.py`
   - `cron-monitors/references/google-docs-comments.md`
   - Todos hablan a `www.googleapis.com` / `gmail.googleapis.com`. Deben pasar a MCP oficial o a Workspace CLI (`gws` / CLI oficial) antes del 2027-02-01.
5. Proyectos GCP citados en el seguimiento: `clawdbot-488800` (uso real frente a Standard pendiente, sin acceso al dashboard), `malditageneral` (analytics-mcp), `warm-mediator-489311-n1` (sin evidencia de si usa REST; queda como unknown, lo revisa el owner).

## URLs oficiales (Google-hosted, GA 24-nov-2026)

- Gmail: `https://gmailmcp.googleapis.com/mcp/v1`
- Drive: `https://drivemcp.googleapis.com/mcp/v1`
- Docs: `https://docsmcp.googleapis.com/mcp/v1`
- Sheets: `https://sheetsmcp.googleapis.com/mcp/v1`
- Slides: `https://slidesmcp.googleapis.com/mcp/v1`
- Calendar: `https://calendarmcp.googleapis.com/mcp/v1`
- Chat: `https://chatmcp.googleapis.com/mcp/v1`
- People: `https://people.googleapis.com/mcp/v1`
- Classroom: `https://classroommcp.googleapis.com/mcp/v1`
- Universal Search: `https://workspacemcp.googleapis.com/mcp/v1`

Ref: https://developers.google.com/workspace/guides/configure-mcp-servers y https://workspace.google.com/blog/product-announcements/built-for-trust-and-control-new-ai-agent-protections-and-developer-tools-in-google-workspace

## Pasos (owner)

1. En cada proyecto GCP (incl. `clawdbot-488800` si es el que usa Hermes): activar las APIs `gmailmcp.googleapis.com`, `drivemcp.googleapis.com`, `docsmcp.googleapis.com`, `sheetsmcp.googleapis.com`, `slidesmcp.googleapis.com`, `calendarmcp.googleapis.com` (+ `chatmcp`, `people`, `tasksmcp`, `classroommcp`, `workspacemcp` según uso) con `gcloud services enable`.
2. Auth Platform > Clients: crear cliente OAuth nuevo marcado como agéntico. No marcar el cliente REST existente; separar agente vs no-agente.
3. En Hermes `config.yaml` (`mcp_servers`): sustituir los bloques `gmail-*`, `google-calendar`, `google-docs*` comunitarios por servidores remotos oficiales (ejemplo abajo). Pedir `access_type=offline` para refresh token.
4. Sustituir llamadas de los scripts `google_api*.py` / `gmail_*.py` por herramientas MCP oficiales o CLI oficial. Mantener `google-workspace` skill como fallback solo para funciones no-agente que sigan en REST.
5. Revisar Addendum: quitar retenciones largas de derivados y cualquier scraping programado (monitores cron que paginen Gmail/Drive).
6. Dashboard: comprobar uso real de `clawdbot-488800` y `malditageneral` frente a cuota Standard para decidir tier antes del 2027-04-01. Sin acción, throttling a Standard. Sin acceso en este paso: pendiente del owner.

Ejemplo Hermes (esquema, sin secretos):

```yaml
mcp_servers:
  gmail-official:
    url: https://gmailmcp.googleapis.com/mcp/v1
  drive-official:
    url: https://drivemcp.googleapis.com/mcp/v1
  docs-official:
    url: https://docsmcp.googleapis.com/mcp/v1
  sheets-official:
    url: https://sheetsmcp.googleapis.com/mcp/v1
  calendar-official:
    url: https://calendarmcp.googleapis.com/mcp/v1
```

## Estado de este repo

Este fork comunitario (`googleapis` directo) no cumple el requisito MCP de 2027-02-01. Opciones: archivar en favor de MCP oficiales, o mantenerlo solo para uso no-agente REST (apps deterministas, que Google dice que siguen en REST). No usarlo como vía de agente tras la fecha.

## Pendiente (requiere owner)

- Uso real en dashboard de `clawdbot-488800` / `malditageneral` vs Standard para elegir tier.
- Confirmar si `warm-mediator-489311-n1` usa REST directo.
- Crear cliente OAuth agéntico y rotar `config.yaml` (con secretos, fuera de este paso de solo lectura).
