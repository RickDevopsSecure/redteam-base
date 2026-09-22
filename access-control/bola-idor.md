# BOLA/IDOR — confirmación determinística

## Patrón de detección

Comparar el acceso a un recurso propio contra el mismo tipo de recurso con un identificador que pertenece a otro usuario/cuenta (creada específicamente para la prueba, sin privilegios). Confirmar que la respuesta contiene datos reales del otro usuario, no un 403/404 disfrazado de 200.

## Autenticación autenticada detrás de token (D8 en la metodología propia)

Los mismos confirmadores usados sin sesión (que reciben 401 y descartan todo) hay que correrlos también CON un token propio (Bearer, obtenido por registro+login del propio harness). Vulnerabilidad que vive detrás de auth es invisible para un scan que nunca se autentica. No reinventar la lógica de detección — correr la ya probada con un contexto de sesión distinto.

## Mass-assignment: por qué no es apto para automatización

Confirmar mass-assignment de forma limpia (sin falso positivo) requiere que el check realmente escale un privilegio — crear una cuenta con `role:admin` inyectado en el payload de registro y verificar que el privilegio quedó activo. Eso dos cosas: (1) deja un artefacto privilegiado permanente en el sistema del cliente, y (2) sin ese write, solo se prueba que el endpoint "acepta" el campo extra, que no es lo mismo que haber escalado privilegio real — ambigüedad que viola el principio de prueba-o-no-se-reporta.

**Regla:** todo lo que sea de verdad read-only o cree solo artefactos throwaway sin privilegio es apto para automatización sin supervisión. Cualquier cosa que exija escribir privilegio persistente va a engagement manual, con consentimiento explícito de write + limpieza post-prueba documentada. No es un "pendiente técnico", es una decisión de diseño.

## Enumeración de usuarios como vector colateral de BOLA

Un endpoint de enumeración de usuarios (ej. una API tipo `/api/users` o `/jsonapi/user/user` en CMS con API REST/JSONAPI expuesta) puede filtrar nombres en claro sin ser técnicamente un BOLA en el sentido de "acceder a OTRO recurso" — es exposición directa de un listado. Verificar explícitamente si cualquier API de listado de usuarios requiere autenticación y si expone campos sensibles (nombre completo, email, rol) a un usuario no privilegiado o anónimo.
