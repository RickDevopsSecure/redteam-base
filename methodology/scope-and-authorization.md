# Alcance y autorización — reglas no negociables

Reglas que deben ir en CUALQUIER herramienta o agente que dispare tráfico activo, propio o de terceros.

## Gate de autorización airtight

- Off por default. Requiere flag explícito de entorno para siquiera aceptar el modo activo/de red.
- Exige `authorized=true` (acuse de permiso escrito) en cada request, no una sola vez a nivel de sesión.
- Targets se validan por forma antes de tocar red: CIDR / IP / rango corto (`a-b`). **Nunca nombres DNS** como target de red interna (evita DNS-rebind: resolver una vez y atacar otra IP).
- IPs de metadata de nube (`169.254.169.254`, `100.100.100.200`, `fd00:ec2::254`, equivalentes GCP/Azure) **siempre excluidas**, aunque el CIDR declarado las contenga.
- Cap duro de hosts (ej. `/20` máx) rechazado ANTES de enumerar — nunca se debe poder iterar un `/8` por accidente de scope.
- Network/broadcast address excluidos del barrido.
- Un único punto de verdad (`scope.contains(ip)`) que TODA acción de host debe pasar — no reimplementar el check en cada módulo.
- Defensa en profundidad: el gate no solo filtra el discovery inicial, también debe estar disponible como check antes de cada conexión cruda (vía contextvar o equivalente), para que un pivote lateral no se salga del alcance declarado.

## Principio advisory / read-only

- El scanner/agente detecta, prioriza y recomienda. No modifica infraestructura del cliente.
- Checks de credenciales por defecto = login + cierre inmediato, nunca ejecución remota.
- Un finding que requiere escribir privilegio permanente en el objetivo (ej. mass-assignment que crea una cuenta admin) **no es apto para automatización sin supervisión** — eso va a engagement manual con consentimiento explícito de write + limpieza post-prueba. No confundir "se puede automatizar" con "se debe".

## Prueba-o-no-se-reporta (0-FP como principio de diseño)

- Un hallazgo solo se reporta si hay confirmación activa (respuesta del sistema, no solo "el patrón parece sospechoso").
- Cuando se amplía la superficie de búsqueda (ej. más queries, más términos, más contextos), la ganancia de recall casi siempre viene acompañada de más ruido — cada ampliación necesita su propio gate de relevancia, no solo "más señales := más cobertura". Ver `web-injection/false-positive-catalog.md`.
- Un CRITICAL falso en un reporte de cliente es peor que no reportar nada. El sesgo del sistema debe ser hacia el silencio, no hacia la alarma.

## Escalado de intensidad, no de alcance

Patrón reusable: separar "qué tan agresivo soy dentro del alcance ya autorizado" (passive/active/deep — más payloads, más profundidad de crawl, más contextos de inyección) de "qué alcance tengo permitido tocar" (el gate de arriba). Escalar intensidad nunca debe ampliar el alcance.
