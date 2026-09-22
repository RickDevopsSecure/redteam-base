# Catálogo de falsos positivos comunes en detección de inyección

Principio: **un miss es malo, pero un FP en un reporte de cliente es peor.** Cada patrón de abajo es una forma real y recurrente en la que una detección "por patrón" se equivoca.

## SSTI — números pequeños en contexto de precio
Un payload SSTI clásico usa aritmética (`{{7*7}}` → `49`) para confirmar evaluación de plantilla. Un sitio de e-commerce que YA tiene "49" en la página (precio, cantidad, SKU) puede hacer que un check ingenuo confunda coincidencia textual con ejecución real. **Fix:** el baseline (respuesta sin payload) debe compararse contra la respuesta con payload; solo cuenta si el número aparece en una posición nueva que corresponde exactamente a donde se inyectó el payload, no en cualquier parte de la página.

## LFI — "localhost" en bundles JS
Un intento de Local File Inclusion que prueba rutas típicas puede toparse con la palabra `localhost` ya presente en un bundle de JavaScript minificado (referencias a config de desarrollo, comentarios de build, URLs de fallback). Un check que solo busca la string "localhost" en la respuesta como señal de éxito dispara falso. **Fix:** confirmar por estructura del archivo esperado (ej. contenido real de `/etc/passwd` con el formato `root:x:0:0:`), no por presencia de una palabra común.

## Admin panels en SPAs — soft-404
Una Single Page Application sirve el mismo `index.html` (HTTP 200) para cualquier ruta, incluidas rutas de admin que no existen. Un scanner que solo mira código de status HTTP confunde "ruta servida con 200" con "panel de admin expuesto". **Fix:** verificar contenido real de la respuesta (¿es el mismo shell de la SPA que cualquier otra ruta inválida?, ¿hay contenido específico del panel?), no solo el status code.

## SQLi — eco genérico confundido con UNION exitoso
Al probar SQLi UNION-based, un valor que se refleja en la respuesta puede coincidir por casualidad con contenido que la aplicación ya mostraba (eco de parámetro, valor por defecto), sin que la UNION realmente haya inyectado una columna. **Fix:** el valor de confirmación debe ser algo que NO podría aparecer de otra forma en la respuesta (string único generado por el check, no un número o palabra común), y compararse contra baseline.

## Censura de evidencia — falsos negativos en enumeración de usuarios
Filtración de usuarios que llega desde una fase de explotación activa genérica (no desde el módulo de control de acceso dedicado) puede no matchear los patrones de censura pensados para emails/credenciales — un listado de nombres en claro sin `@` ni forma de password se cuela. **Fix:** cualquier hallazgo con marcador de enumeración (endpoints tipo `/api/users`, `/jsonapi/user`, "user-listing") debe pasar por un enmascarado de tokens entre comillas, no solo por los patrones de email/credential. Ver `secrets/redaction-patterns.md`.

## Ampliar la búsqueda de leaks sin gate de relevancia
Ver `recon/subdomain-enumeration.md` — un término de proyecto corto o genérico usado como subject de búsqueda en code search inunda de resultados de repos ajenos (wordlists, docs, demos). Cada ampliación de superficie de búsqueda necesita su propio gate de relevancia sobre el contenido real, no solo sobre el match de la query.

## Principio general para diseñar cualquier check nuevo
1. ¿Qué produciría este mismo resultado SIN que la vulnerabilidad exista? (contenido preexistente, coincidencia textual, comportamiento por diseño de la app)
2. ¿El check compara contra un baseline (respuesta sin payload) o asume que cualquier señal positiva es prueba?
3. ¿La confirmación es algo que un atacante real necesitaría (dato extraído, delay medido dos veces, string único reflejado) o solo "se parece a"?
