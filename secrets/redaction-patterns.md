# Censura/redacción de PII y credenciales en evidencia y reportes

Principio: ningún dato personal o credencial real debe llegar a un reporte, log persistido o artefacto compartible, sin importar de qué fase/módulo venga el hallazgo.

## Patrones a enmascarar

- **Emails**: `j***@a***.com` (conserva primera letra de usuario y dominio, oculta el resto).
- **Passwords/tokens/secrets/API keys/client secrets/TOTP**: reemplazo total (`***REDACTED***`), tanto en valor JSON como en formato `k=v`.
- **Tarjetas de pago**: conservar solo los últimos 4 dígitos.
- **JWT**: mostrar solo el primer segmento (`eyJ***`) — nunca el payload ni la firma completos, ni siquiera truncados a 2 segmentos (un JWT de 2 segmentos sigue siendo decodificable).
- **Cloud API keys** (AWS, GitHub, etc.): mostrar solo el prefijo que identifica el tipo (`AKIA****`, `gh*_***`), nunca el resto.
- **Hashes/hex largos**: mostrar solo los primeros 4 caracteres.
- **Credenciales embebidas en URL** (`scheme://user:pass@host`): remover la parte de credenciales, conservar host/repo para que la evidencia siga siendo útil.
- **Listados de usuarios/enumeración**: cuando el hallazgo trae un marcador de enumeración (endpoint tipo `/api/users`, `/jsonapi/user`, "user-listing"), enmascarar cada token entre comillas del listado — esto es un patrón aparte de emails/credenciales porque nombres en claro sin `@` no matchean esos regex. Gatear por la presencia del marcador para no sobre-censurar arrays benignos que no son de usuarios.

Explícitamente NO tocar: texto benigno que casualmente parece sensible (ej. "49" como número, "localhost" en config), placeholders de documentación conocidos (`AKIAIOSFODNN7EXAMPLE`, `your_api_key`, `example.com`, `changeme`).

## Dónde aplicar (belt-and-suspenders, no un solo punto)

1. **En el origen** — donde se genera el finding (evidencia de acceso a datos de otro usuario, remotes de git con credenciales embebidas, emails de committers).
2. **En el chokepoint de persistencia** — la función que escribe cualquier hallazgo a la base de datos debe pasar evidence/exploit_output/description por la censura ANTES de guardar, sin excepción, sin importar qué fase lo generó. Esto es lo que cierra el gap de "un hallazgo de una fase nueva no pasaba por el filtro pensado para otra fase". Idempotente (no romper si se aplica dos veces) y tolerante a nulos, y no debe bloquear el guardado si la función de censura falla — mejor guardar sin censurar-perfecto que perder el hallazgo, pero esto exige monitoreo de que la censura nunca falle silenciosamente en producción.
3. **En cada superficie de render** (PDF, HTML, cualquier exportación) — última línea de defensa: ningún dato en claro debe poder llegar a un artefacto final sin pasar por el filtro, incluso si algo se coló en el paso 1 o 2.

## Nota sobre forward-only

Un chokepoint agregado en el paso 2 solo protege hallazgos creados DESPUÉS del cambio — datos ya persistidos antes conservan el valor crudo en la base de datos. El render (paso 3) sigue censurando esos hallazgos viejos al mostrarlos, pero si hay acceso directo a la base de datos subyacente, contemplar una pasada de backfill si el caso lo amerita.
