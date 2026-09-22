# redteam-base

Catálogo de referencia propio: metodología, checklists, patrones de falsos positivos y scripts para red team (manual + agéntico). Destilado de trabajo real en KIRUX/XIPE y engagements de Capital Cloud, **genericizado** — sin nombres de cliente, dominios, IPs ni credenciales reales. Cada entrada es una lección reusable, no un reporte.

## Regla de oro del repo

Nada de esto lleva datos de un cliente específico. Si una lección viene de un caso real, se documenta el *patrón* (qué se vio, por qué se confundió, cómo se distingue), nunca el nombre, dominio o evidencia del cliente. Confidencialidad de cliente > valor del ejemplo.

## Estructura

- `methodology/` — fases de un engagement (recon → injection → access control → attack) y reglas de alcance/autorización.
- `recon/` — enumeración de superficie (subdominios, GitHub recon, gotchas de cobertura).
- `web-injection/` — SQLi/SSTI/XXE/SSRF, contextos de inyección, catálogo de falsos positivos comunes.
- `access-control/` — BOLA/IDOR, autenticación, autorización.
- `cloud-aws/` — triage de GuardDuty, pentest de red interna, patrones de FP en detección cloud.
- `secrets/` — patrones de censura/redacción de PII y credenciales en evidencia/reportes.
- `agentic-redteam/` — lecciones específicas de usar LLMs como agente ofensivo (tool-calling, fallos silenciosos, límites de modelos "uncensored").
- `scripts/` — herramientas propias reusables, organizadas por fase.
- `checklists/` — checklists operativos por fase de ataque.

## Cómo se alimenta

Después de cada engagement o sesión de mejora de KIRUX, extraer la lección genérica (qué patrón se confirmó, qué falso positivo se cazó, qué gap de cobertura se cerró) y añadirla aquí sin datos identificables. Esto es un catálogo vivo, no un archivo histórico.
