# Metodología por fases (web/API)

Secuencia de fases usada como sweep determinístico en escaneo automatizado, y como checklist en pentest manual. La lección estructural: **cuanto más de esto corre como fase determinística en vez de "a discreción del agente/operador", más confiable es la cobertura.** Un tool disponible pero opcional (a criterio del LLM o de si el pentester se acuerda) tiene cobertura NO confiable — se demuestra corriendo el mismo scope dos veces y viendo qué desaparece.

## D1 — Recon
Enumeración de superficie: subdominios, tecnologías, puertos, endpoints. Ver `recon/`.

## D1.5 — Posture
Checks determinísticos sin necesidad de agente: cabeceras de seguridad ausentes (HSTS/CSP/X-Frame-Options/X-Content-Type-Options/Referrer-Policy/Permissions-Policy), HTTP sin forzar HTTPS, DMARC/SPF/CAA ausentes o débiles, DNSSEC ausente, zone transfer (AXFR) abierto, EOL runtime por banner (versión de PHP/OpenSSL/Apache/etc.), version disclosure, cookies sin flags seguros.

## D2 — Injection
SQLi, XSS, command injection, XXE, SSTI, NoSQLi. Ver `web-injection/`.

## D2.5 — Exposure
`.git` dumpable, backups/configs expuestos por dork, archivos sensibles servidos por accidente.

## D2.55 — Repo exposure
Búsqueda de leaks en repos públicos correlacionados al target (código, historial completo, no solo HEAD). Ver `recon/github-recon.md`.

## D2.6 — Traversal
Path traversal / LFI.

## D3 — Access control
BOLA/IDOR. Ver `access-control/`.

## D3.5 — Authflow
Registro, login, recuperación de contraseña, poisoning de reset-link vía Host header.

## D4 — BOLA determinístico
Confirmación automatizada de referencias a objetos de otro usuario.

## D5 — GraphQL
Introspección, queries/mutations no autorizadas.

## D6 — Authenticated (sesión propia)
Superficie que requiere sesión autenticada normal.

## D7 — JWT
Algoritmo `none`, confusión de firma, claims manipulables, expiración no validada.

## D8 — Authenticated API (Bearer)
Repite los confirmadores de D2 pero con token propio en header — caza vulnerabilidad que vive DETRÁS de auth y que D2 sin token recibe como 401 y descarta. Reusa la lógica de detección ya probada, no reinventa nada nuevo — el valor está en correrla con un contexto de sesión distinto.

## D9 / ATTACK — Explotación activa determinística
Sweep de command injection, SSRF (in-band + OOB vía callback), open redirect, XXE, CRLF injection, host header injection, NoSQL injection — contra los endpoints ya descubiertos, con intensidad escalable (passive/active/deep).

## Pentest de red interna (aparte, gate propio)
Discovery (ping sweep dentro de scope) → enum de puertos de servicio explícitos (no solo top-N, para no perder datastores en puerto no estándar) → clasificación (web/datastore/db/admin/ftp/plaintext) → checks determinísticos de auth (Redis/Memcached/Elasticsearch/CouchDB sin auth, FTP anónimo, SMB null/guest, MSSQL/MySQL/PostgreSQL default-creds vía login+cierre, nunca exec) → pivote: si aparece HTTP(S) interno, correr las fases D2–D8 contra ese endpoint también. Ver `methodology/scope-and-authorization.md` para el gate que blinda todo esto.

## Principio transversal: nunca single-shot en el confirmador
Un solo request fallido (timeout/reset/5xx transitorio) no debe traducirse en "0 findings" silencioso. Cualquier choke-point que decide si algo se reporta necesita reintento acotado (2-3x, backoff, solo en error transitorio — nunca reintentar un ReadTimeout como si fuera fallo de red, porque amplifica la lentitud del target en vez de recuperarse de ella). Ver `agentic-redteam/silent-failure-anti-pattern.md`.
