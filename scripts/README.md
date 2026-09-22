# scripts/

Herramientas propias, standalone y reusables (no acopladas al código de ningún producto/cliente). Cada script debe poder correr solo, con su propio `--help`, sin depender de infraestructura de un engagement específico.

- `recon/` — enumeración de subdominios, GitHub recon, fingerprinting.
- `injection/` — confirmadores de SQLi/SSTI/etc. reusables fuera del contexto de un scanner completo.
- `network/` — discovery/enum de red interna, checks de credenciales por defecto.

Al agregar un script: quitar cualquier dato hardcodeado de cliente (dominios, IPs, tokens de prueba reales) y documentar en un comentario de cabecera de qué lección o caso salió (sin nombrar al cliente).
