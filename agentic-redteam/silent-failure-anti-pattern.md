# Fallo silencioso — el riesgo #1 en un producto/agente de seguridad

## La lección madre

Un dict "válido" con un error adentro (o un status `"ok"` mentiroso) pasa cualquier gate que solo mida *forma* o *calidad de output*, no si la operación realmente sucedió. Para un producto de seguridad esto es lo peor posible: una integración rota se ve idéntica a una sana-pero-callada, y "0 findings" se lee como "está limpio" en vez de "no pudimos revisar".

## Formas recurrentes de fallo silencioso

- `except: pass` o `except: continue` alrededor de una llamada que puede fallar transitoriamente.
- Un flag de "sincronizado OK" que se escribe incluso cuando la sincronización falló.
- Verificación de firma/auth que hace fallback a `True` cuando falta el token en vez de rechazar.
- Una tarea en background cuyo fallo no se propaga — el usuario ve "scan completo" de un scan que nunca corrió de verdad (scan fantasma).
- Un error que se clasifica como severidad INFO en vez de fallo real.
- Marcar algo como "leído"/"procesado" antes de confirmar que se persistió.
- Un clasificador/triage que devuelve un objeto con la forma correcta pero con el error metido adentro de un campo — pasa cualquier validación de esquema.
- Un solo request fallido (timeout/reset/5xx transitorio) en un punto de decisión que determina si algo se reporta, sin reintento — "0 findings" indistinguible de "el target realmente no tiene nada".

## Corolario operativo: los harnesses deben medir LIVENESS, no solo forma

Un test/benchmark que solo valida "el output tiene la forma esperada" (campos presentes, tipo correcto) no detecta un fallo silencioso — el output roto también tiene la forma correcta. El harness debe afirmar explícitamente que la llamada SUCEDIÓ: sin flag de error interno, con el campo que solo se llena en éxito real presente, con un resumen que no sea el mensaje de error disfrazado de resultado.

Ejemplo real del patrón: un cambio de configuración de modelo (un parámetro deprecado en una versión nueva) rompió un clasificador durante días — nada lo cachó porque todos los checks medían forma del output, no si la llamada al modelo había tenido éxito.

## Retry acotado como mitigación, no bala de plata

Cuando el choke-point es "una sola petición decide si algo se reporta": reintento acotado (2-4x, backoff) SOLO en error transitorio real (excepción de conexión, reset, 5xx, 429) — nunca en timeout de lectura (`ReadTimeout`), porque reintentar la lentitud de un target real solo re-consume el mismo presupuesto de tiempo sin resolver la causa. Una respuesta HTTP definitiva (incluyendo error de aplicación <500) debe regresar de inmediato sin reintentar — el retry es para "no llegó respuesta", no para "la respuesta no me gustó".

Cuando el objetivo mismo no es alcanzable (target caído/muy lento), el sistema debe reportarlo explícitamente ("objetivo inalcanzable, no se pudo evaluar") en vez de dejar que la ausencia de datos se lea como "limpio". Es la diferencia entre "no encontramos nada" y "no pudimos buscar".

## No confundir flake de infraestructura de prueba con regresión de código

Un target de prueba inestable (ej. una app de demo mono-hilo que se bloquea bajo carga) puede fallar y pasar de forma intermitente con el mismo código exacto. Antes de invertir en "arreglar" el código por un fallo intermitente, confirmar con una re-corrida si el fallo es determinístico (mismo input, mismo resultado) o depende de la salud del target de prueba en ese momento. La mitigación correcta para un target flaky de CI es un retry a nivel del step de prueba (con el gate exigiendo que AMBOS intentos fallen para marcar rojo), no relajar la detección del código.
