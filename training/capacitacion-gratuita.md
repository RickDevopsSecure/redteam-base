# Roadmap de capacitación gratuita — preparación para OSCP/CRTO sin gastar antes de tiempo

Objetivo: construir la base de habilidades con recursos 100% gratuitos ANTES de pagar OSCP y CRTO (ver decisión de orden en el ensayo/notas de fase 2 de este proyecto). La lógica es simple: los exámenes pagados certifican que ya tienes el nivel — no son el lugar para aprender desde cero, son el lugar para demostrar que ya sabes. Todo lo de abajo está verificado como gratuito al momento de escribir esto; los tiers y precios de estas plataformas cambian, conviene reconfirmar en el sitio antes de asumir que sigue igual.

## Referencia constante (gratis siempre, se consulta, no se "termina")

- **[PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)** — catálogo de payloads y técnicas de bypass por vulnerabilidad, mantenido activamente por la comunidad. Es el equivalente a tener `web-injection/` de este mismo repo, pero a escala de toda la industria.
- **[HackTricks](https://github.com/HackTricks-wiki/hacktricks)** — wiki de técnicas de pentest/CTF, igual de viva. Útil sobre todo para lo que este repo todavía no cubre a fondo: post-explotación, escalación de privilegios Windows/Linux, técnicas específicas de AD.

## Aplicaciones web (mapea directo a `web-injection/` de este repo)

- **[PortSwigger Web Security Academy](https://portswigger.net/web-security)** — 100% gratis, labs interactivos + material de lectura, no requiere Burp Pro (la edición Community alcanza para casi todo). Cubre SQLi, XSS, CSRF, fallos de lógica de negocio, autenticación, OAuth, SSRF, carga de archivos, condiciones de carrera, NoSQLi, GraphQL. Es el mejor complemento gratuito posible a `web-injection/sqli-multicontext.md` y `web-injection/false-positive-catalog.md` de este repo — practicar aquí antes de tocar una máquina completa ahorra tiempo.

## Metodología de intrusión y práctica estilo examen (para OSCP)

- **[TryHackMe](https://tryhackme.com)** — tier gratis con 350+ rooms guiadas. Sirve para calentar en un tema puntual (30-60 min por room), no para simular el examen — el tier gratis limita tiempo de máquina y no da acceso a attack box.
- **[HackTheBox](https://www.hackthebox.com)** — tier gratis rota 5 máquinas activas (sin acceso a retiradas, esas son de paga). Más exigente y menos guiado que TryHackMe, más parecido al estilo de examen real.
- **[OffSec Proving Grounds Play](https://help.offsec.com/hc/en-us/articles/360048613751-Access-PG-Play)** — gratis, máquinas solo Linux enviadas por la comunidad (base VulnHub), 3h de acceso por ventana de 24h. Registrarse en la OffSec Learning Library también da acceso gratuito al curso **PEN-103 (Kali Linux Revealed)** — vale la pena aunque ya domines Linux, porque el examen OSCP asume ese nivel de fluidez con la distro.
- **Lista OSCP-like de TJ Null** — currículum curado (no oficial de OffSec) de máquinas de HTB/PG que replican el estilo y dificultad del examen OSCP. Es la referencia estándar de la comunidad para saber si ya estás listo — buscar "TJ Null OSCP-like machines" para la lista vigente, se actualiza con el tiempo.

## Active Directory (crítico para CRTO, y para el bloque AD de OSCP)

- **[GOAD — Game of Active Directory](https://github.com/Orange-Cyberdefense/GOAD)** — laboratorio de AD vulnerable, autohospedado, gratis (pagas tu propio cómputo/electricidad, no el software). Necesita una máquina con recursos reales (~115GB de disco, VMs Windows con licencia de evaluación de 180 días). Es el más realista disponible sin pagar un lab dedicado — entornos con 2 bosques/3 dominios (GOAD completo) o una versión ligera de 3 VMs (GOAD-Light) si el hardware no alcanza para el completo.
- **Sección de Active Directory del curso gratuito de TCM Security en YouTube** (parte de *Practical Ethical Hacking*, disponible gratis en su canal) — cubre construcción de lab propio y ataques: LLMNR poisoning, SMB relay, pass-the-hash, kerberoasting, token impersonation. Buena introducción antes de meterle a GOAD con más autonomía.

## Fundamentos y cross-training (CTF, no necesariamente ofensivo puro)

- **[picoCTF](https://picoctf.org/)** (ahora parte de CyLab Security Academy, Carnegie Mellon) — 100% gratis, sin tiers de pago. picoGym conserva todos los retos de años anteriores para practicar sin esperar una competencia activa. Cubre criptografía, forense, explotación web, binarios — útil para no llegar a OSCP/CRTO con puntos ciegos fuera de web/AD.

## Orden sugerido (no son fechas, es secuencia)

1. **HackTricks/PayloadsAllTheThings** quedan abiertos en una pestaña desde el día uno — no se "completan", se consultan todo el camino.
2. Si algún fundamento se siente débil (Linux, redes, scripting básico), **picoCTF** o unas rooms puntuales de **TryHackMe** para calentar sin comprometerse a una ruta completa.
3. **PortSwigger Web Security Academy** en paralelo con todo lo demás — es la que más se conecta con el contenido que ya tienes en `web-injection/` de este repo, y no compite por el mismo tipo de setup/hardware que las máquinas completas.
4. **HackTheBox free + Proving Grounds Play** para practicar metodología de intrusión completa (recon → explotación → escalación → reporte), con la lista OSCP-like de TJ Null como currículum de referencia una vez que ya rindes bien en máquinas sueltas.
5. **GOAD + el bloque de AD de TCM** al final de la preparación gratuita, específicamente porque el bloque de AD es el que más pesa en CRTO y una porción real de OSCP — mejor no dejarlo para el mismo mes del examen.

## Lo que ninguno de estos reemplaza (honesto, no venta)

Nada de esto simula la presión de un examen real de 23-24 horas corridas con reporte incluido. Nada de esto da acceso a Cobalt Strike (viene incluido con el propio curso de CRTO, no antes). Y el nivel de las máquinas retiradas de HTB (las más parecidas en dificultad a un red team real) sigue de paga. Lo gratuito construye la base amplia; el examen pagado certifica que ya la tienes — no es un atajo para saltarse el pago, es para no pagar por aprender lo que se puede aprender gratis primero.

## Fuentes

- [TryHackMe vs HackTheBox 2026: Pricing, Paths & Verdict, HackerDNA](https://hackerdna.com/blog/tryhackme-vs-hackthebox)
- [Web Security Academy: Free Online Training from PortSwigger](https://portswigger.net/web-security)
- [PortSwigger Web Security Academy: free and still the best, security.university](https://security.university/resources/courses/portswigger-web-security-academy/)
- [GOAD — GitHub, Orange-Cyberdefense](https://github.com/Orange-Cyberdefense/GOAD)
- [Game Of Active Directory, documentación oficial](https://orange-cyberdefense.github.io/GOAD/)
- [TCM Security — Free course access](https://tcm-sec.com/course-access/free/)
- [Getting Started with PG Play and Practice, OffSec Support Portal](https://help.offsec.com/hc/en-us/articles/360048318472-Getting-Started-with-PG-Play-and-Practice)
- [Access PG Play, OffSec Support Portal](https://help.offsec.com/hc/en-us/articles/360048613751-Access-PG-Play)
- [picoCTF.org is now CyLab Security Academy](https://picoctf.org/)
- [PicoCTF Beginner Guide 2026, HackerDNA](https://hackerdna.com/blog/picoctf)
- [PayloadsAllTheThings — GitHub, swisskyrepo](https://github.com/swisskyrepo/PayloadsAllTheThings)
- [HackTricks — GitHub, HackTricks-wiki](https://github.com/HackTricks-wiki/hacktricks)
