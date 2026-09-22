# Enumeración de subdominios — cobertura, no solo velocidad

## La lección central

Enumerar solo por Certificate Transparency (CT) + Shodan **deja hosts fuera del inventario**, y ahí puede vivir el activo más sensible (en un caso real: un portal de pacientes completo).

Por qué se pierden:
- CT solo lista nombres con **certificado propio emitido**. Un host que sirve **certificado ajeno** (ej. detrás de un balanceador multi-tenant, IP compartida con otro cliente de la nube) nunca aparece en CT.
- Hosts bajo **certificado comodín** (`*.dominio.com`) no exponen sus nombres individuales en CT — solo se ve el wildcard, no qué subdominios existen debajo.
- Un simple DNS-brute con wordlist revela subdominios que las fuentes pasivas no tienen forma de conocer.

## Baseline mínimo obligatorio

CT (Certspotter u equivalente) + Shodan + subfinder (agregador de fuentes pasivas) + **fuerza bruta DNS con wordlist**. Las primeras tres son gratis y rápidas pero pasivas — la cuarta es la única que encuentra lo que nadie más publicó.

Validar en vivo cada host de login encontrado: certificado (`openssl s_client`) y cadena de redirección completa (`curl -skIL`) antes de cerrar el inventario.

## GitHub recon — no es solo buscar secretos en el HEAD

Dos fallas distintas de profundidad, no de cobertura:

1. **No leer el código encontrado.** Un repo público correlacionado al target puede filtrar topología interna completa (endpoints on-prem, esquema de base de datos, arquitectura, tenant de identidad) sin que haya ni un solo "secreto" con patrón de API key. Despachar un repo como "sin secretos" sin leer el código es un miss de disclosure, no solo de credenciales.
2. **No revisar el historial de git.** Un commit "fix(security)" que remueve una credencial del HEAD no la borra del historial — sigue recuperable con `git log -p -S<término>` o un secret-scan sobre todo el historial. Clonar con `--depth 1` garantiza perder esto.

**Regla dura:** en cualquier repo público hallado y correlacionado al target, (1) leer el código en busca de disclosure de arquitectura/datos, (2) clonar completo (sin `--depth`) y correr secret-scan sobre TODO el historial, (3) buscar service-account JSON, API keys, datos de prueba sensibles.

## Gotcha de búsqueda de leaks: términos de proyecto cortos inundan de ruido

Al ampliar una búsqueda de leaks más allá del dominio literal (porque un repo de cliente casi nunca menciona el dominio exacto, se encuentra por el nombre del proyecto/slug), un término genérico o una parte de slug corta (<5 caracteres) empieza a matchear repos completamente ajenos — READMEs de terceros, wordlists de dorks, JWTs de demo (`jwt.io`), documentación con claves de ejemplo (`AKIAIOSFODNN7EXAMPLE` es la key doc de AWS, no un leak real).

**Triple gate necesario** cuando se amplía así:
1. Excluir palabras genéricas comunes y partes de slug demasiado cortas como subject de búsqueda.
2. Descartar por ruta del archivo (README/docs/examples/wordlists/`.md`).
3. Exigir que el archivo referencie el dominio o el slug COMPLETO en su contenido, no que la query haya matcheado un término suelto — la relevancia se prueba con la coincidencia real, no con el hit de la búsqueda.
4. Rechazar sintaxis de dorks, `localhost`, JWTs de demo conocidos como contenido "secreto".

Sin este gate, ampliar cobertura de leaks produce falsos positivos CRITICAL — peor que no buscar.
