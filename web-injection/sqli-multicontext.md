# SQLi — cubrir todos los contextos, no solo comilla simple

## El gap más común en detectores caseros

Un confirmador de SQLi que solo prueba contexto de comilla simple (`1' AND '1'='1`) se pierde la mayoría del SQLi ciego real, porque la mayoría de parámetros vulnerables son **numéricos** (IDs: `?id=1`). En contexto numérico la comilla simple rompe la query de una forma distinta a como la rompería en contexto de texto — sin error visible en producción, el hueco queda completamente invisible para un check que solo conoce comillas.

## Contextos a probar (boolean-based)

- Numérico sin comillas: `?id=1 AND 1=1` vs `?id=1 AND 1=2`
- Comilla simple: `1' AND '1'='1`
- Comilla doble: `1" AND "1"="1`
- Paréntesis + comentario: `1) AND (1=1)-- -`

Confirmación: comparar respuesta TRUE vs baseline (deben coincidir) y FALSE vs baseline (debe diferir) — nunca solo "hubo un error SQL", porque error-based tiene su propio catálogo de firmas por motor.

## Error-based

Mantener un catálogo amplio de firmas de error por motor, no solo MySQL/Postgres genéricos — motores enterprise (MSSQL, DB2, MS Access/Jet, JDBC) tienen firmas de error propias y distintivas que valen la pena tener explícitas.

## Time-based

Payloads por backend (MySQL `SLEEP()`, Postgres `pg_sleep()`, MSSQL `WAITFOR DELAY`, SQLite). **Doble confirmación del delay** (repetir la medición y exigir que el delay se mantenga ≥ baseline + margen fijo en ambas corridas) — mide una vez y confías en jitter de red te da falsos positivos esporádicos.

## UNION-based / stacked queries

Siguiente nivel de cobertura tras cubrir boolean/error/time en todos los contextos. UNION necesita descubrir el número de columnas antes de poder inyectar una marca de confirmación única (ver `false-positive-catalog.md` — el valor de confirmación no debe poder aparecer por eco/coincidencia).

## Escalado por intensidad

El número de contextos y payloads probados debe escalar con el nivel de autorización/agresividad (passive prueba menos, active/deep prueban el catálogo completo) — nunca el alcance de qué se puede tocar, solo cuánto se prueba dentro de lo ya autorizado. Ver `methodology/scope-and-authorization.md`.
