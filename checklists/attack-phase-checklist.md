# Checklist — fase ATTACK (explotación activa determinística)

Requiere scope autorizado y gate de `methodology/scope-and-authorization.md` ya pasado. Escalar intensidad (passive/active/deep) solo cambia cuánto se prueba, nunca qué se puede tocar.

- [ ] **Command injection** — canary echo + doble confirmación time-based blind.
- [ ] **SSRF in-band** — metadata cloud (AWS/GCP/Azure/DO/K8s), Redis vía gopher, con encodings de bypass. Confirmar por contenido real de la respuesta, no por status code.
- [ ] **SSRF/XXE/RCE/SSTI out-of-band** — vía callback controlado (interactsh o infraestructura OOB propia). Confirmar por interacción real recibida, degradar con elegancia si el servicio OOB no está disponible (no fallar en silencio, loguear explícito).
- [ ] **Open redirect** — `Location` header apuntando a dominio de control del atacante.
- [ ] **XXE** — extracción de archivo local conocido como confirmación (no solo eco del payload).
- [ ] **CRLF injection / response splitting** — cuidado con falsos positivos en bloques de contenido legítimo con saltos de línea (ver guard de edge-case en el propio detector).
- [ ] **Host header injection** — poisoning de link de reset de password reflejado en la respuesta.
- [ ] **NoSQL injection** — operadores tipo `$ne`/`$regex`/`$where`, confirmar bypass real de autenticación.
- [ ] **SQLi** — ver `web-injection/sqli-multicontext.md`, todos los contextos.
- [ ] **SSTI** — con guard de falso positivo de baseline (ver `web-injection/false-positive-catalog.md`, caso "49 en precio").
- [ ] **Version → CVE** — banner de servidor/runtime (PHP/OpenSSL/Apache/etc.) mapeado a CVEs conocidos con versión afectada exacta.
- [ ] **Subdomain takeover** — CNAME colgante apuntando a servicio no reclamado (S3/GitHub Pages/Heroku/etc.), confirmado por fingerprint de la respuesta del servicio, no solo por CNAME roto.
- [ ] **JWT** (D7) — algoritmo `none`, confusión de firma HS/RS, claims manipulables, expiración no validada.
- [ ] **BOLA/IDOR** (D3/D4) — ver `access-control/bola-idor.md`.
- [ ] **Autenticado (Bearer)** (D8) — repetir confirmadores de inyección con sesión propia.

## Antes de cerrar cualquier hallazgo de esta fase

- [ ] Pasó por censura/redacción de PII y credenciales (`secrets/redaction-patterns.md`).
- [ ] Tiene confirmación activa real, no solo coincidencia de patrón (`web-injection/false-positive-catalog.md`).
- [ ] Si requiere escribir privilegio persistente en el target (ej. mass-assignment con cuenta admin): NO se automatiza, se documenta como candidato a engagement manual con consentimiento explícito.
