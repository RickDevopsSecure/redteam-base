# Bug bounty de pre-lanzamiento en infraestructura para agentes de IA: el bug crítico no lo encontró el scanner, lo encontró leer el flujo completo

**Sector:** infraestructura para agentes de IA — categoría de producto donde un agente autónomo aprovisiona y paga su propio cómputo sin registro tradicional ("pagar ES el registro"). **Tipo de engagement:** programa de beta que invitaba explícitamente a intentar comprometerlo, black-box, alcance limitado a recursos propios del tester — nunca tocar cuentas ni servidores de otro usuario de la beta. Sin nombres, dominios ni evidencia identificable — el detalle técnico es real, el objetivo no.

## Contexto

Producto en fase beta: un agente (o un humano actuando como agente) paga con stablecoin o tarjeta y recibe un servidor provisionado, sin flujo de registro previo. El aprovisionamiento inicial solo entrega un identificador de recurso, sin token de gestión — la autenticación para administrar ese servidor (ejecutar comandos, reiniciarlo, destruirlo) se obtiene después, mediante un código de un solo uso (OTP) enviado por correo.

## Hallazgo 1 (CRÍTICO) — Account takeover completo por fuerza bruta de OTP en un endpoint no documentado

La especificación pública de la API (OpenAPI) no listaba el endpoint real de verificación del OTP — estaba activo pero fuera de la documentación expuesta. Ese endpoint aceptaba un código de 6 dígitos, válido por 10 minutos, **sin límite de intentos ni bloqueo tras fallos repetidos**. Veinticinco intentos con código incorrecto devolvieron veinticinco respuestas de error idénticas — ninguna señal de throttling, ningún bloqueo temporal de la cuenta.

**La cadena completa:** el aprovisionamiento entrega el recurso sin credencial de gestión → la única forma de obtener esa credencial es reclamar el recurso con el OTP correcto → sin límite de intentos, un espacio de 6 dígitos es trivialmente agotable en un tiempo razonable → quien complete el claim se queda con control administrativo total, incluyendo ejecución de comandos sobre el servidor y su destrucción. Toma de control total de la infraestructura de otro usuario de la beta, sin haber comprometido nada del lado de esa víctima — el hueco estaba en la primitiva de autenticación, no en el objetivo.

**Remediación:** rate-limit y bloqueo tras N intentos fallidos en cualquier endpoint de verificación de código de un solo uso, sin excepción por estar "fuera" de la documentación pública — lo no documentado sigue siendo superficie de ataque real.

## Hallazgo 2 (MEDIO) — Exposición de metadata sin autenticación (BOLA)

El endpoint de consulta de un recurso por identificador respondía sin autenticación, exponiendo metadata operativa (estado, plan, fecha de expiración, consumo de banda) de cualquier recurso cuyo identificador se conociera o adivinara. El identificador tenía entropía razonable (~60-70 bits en un formato alfanumérico de longitud fija) — no trivialmente adivinable en bloque, pero cualquier identificador filtrado por otro medio (logs, referrers, soporte) exponía esa metadata sin ningún control adicional.

## Hallazgo 3 (MEDIO) — Autenticación de correo rota en el dominio que entrega los OTP

El subdominio de envío de correo del servicio no tenía SPF ni DMARC configurados; el dominio principal tenía SPF pero no DMARC. Esto compone directamente con el Hallazgo 1: el canal que entrega el código de reclamo es spoofeable y vulnerable a que el correo real caiga en spam — un atacante no solo puede fuerza-brutear el OTP, también puede intentar interceptarlo o suplantar al remitente para phishing dirigido a un usuario que está esperando exactamente ese tipo de correo.

## Lo que sí estaba bien hecho

Los endpoints de acciones privilegiadas (ejecutar comandos, reiniciar, apagar, destruir) sí exigían autenticación correctamente cuando se probaban de forma directa — devolvían 401 sin excepción, sin acceso cross-tenant anónimo. El problema nunca fue "falta autorización" en el sentido clásico; fue que esa autorización se podía obtener de forma fraudulenta por la puerta de atrás del OTP sin límite de intentos. Vale la distinción: un sistema puede tener el control de acceso bien diseñado y seguir siendo comprometible por completo si la forma de *adquirir* la credencial tiene un hueco.

## La lección que vale más que la lista de hallazgos

El hallazgo crítico (Hallazgo 1) se encontró por exploración manual y deliberada de la superficie real del servicio, no por un barrido automatizado contra lo que la especificación pública decía que existía — precisamente porque el endpoint vulnerable **no aparecía en esa especificación**. Un enfoque de descubrimiento que confía solo en lo oficialmente documentado (spec de API, sitemap, documentación pública) va a subcontar la superficie real exactamente en el lugar donde el operador del servicio decidió no documentar algo — que suele ser, no por casualidad, donde vive el riesgo.

Es el mismo principio que documento en [`recon/subdomain-enumeration.md`](../recon/subdomain-enumeration.md) para descubrimiento de subdominios (confiar solo en Certificate Transparency deja fuera del inventario justo los hosts que no tienen certificado propio): **cualquier fuente de descubrimiento que dependa de que el objetivo haya publicado/documentado algo, por diseño, no puede ver lo que el objetivo no publicó.** La única forma de cerrar ese gap es complementar con exploración activa que no dependa de la buena voluntad documental del objetivo — fuzzing de rutas, lectura de bundles de cliente, prueba de patrones de endpoint típicos de la misma familia de API.

Uso herramientas de IA para acelerar reconocimiento y triage en este tipo de trabajo, pero la verificación de que este endpoint realmente carecía de rate-limit — mandar los 25 intentos, confirmar que ninguno arrojó una señal de throttling — fue trabajo manual, deliberado, uno por uno. Es exactamente el punto que desarrollo en [`essays/el-prompt-no-es-el-pentest.md`](../essays/el-prompt-no-es-el-pentest.md): la parte que un prompt no resuelve solo es decidir qué merece la pena verificar a mano y efectivamente verificarlo.

## Disciplina ética del engagement

Todo el tráfico activo (intentos de OTP, pruebas de BOLA) se ejecutó exclusivamente contra recursos propios del tester — cuenta propia, servidor propio, correos propios. Cero interacción con cuentas o recursos de otros usuarios de la beta, incluso siendo un programa que invitaba activamente a intentar comprometer el servicio. Ver [`methodology/scope-and-authorization.md`](../methodology/scope-and-authorization.md) — la invitación abierta a "intenta romperlo" no es lo mismo que autorización para tocar el activo de un tercero, y esa línea se mantuvo incluso sin que nadie la estuviera vigilando activamente.

## Resumen de hallazgos

| Hallazgo | Severidad | Vector |
|---|---|---|
| ATO por fuerza bruta de OTP en endpoint no documentado | Crítico | Autenticación / superficie no documentada |
| Exposición de metadata sin auth (BOLA) | Medio | Control de acceso |
| SPF/DMARC ausentes en dominio de envío de OTP | Medio | Autenticación de correo |
| Autorización en acciones privilegiadas (exec/reboot/destroy) | — | Bien calibrado |

## Metodología y patrones relacionados en este repo

- [`recon/subdomain-enumeration.md`](../recon/subdomain-enumeration.md) — el mismo principio de "lo no documentado no es lo mismo que lo no explotable", aplicado ahí a subdominios y aquí a endpoints de API.
- [`methodology/scope-and-authorization.md`](../methodology/scope-and-authorization.md) — por qué una invitación abierta a atacar no autoriza tocar recursos de terceros.
- [`essays/el-prompt-no-es-el-pentest.md`](../essays/el-prompt-no-es-el-pentest.md) — la verificación manual como el paso que ninguna herramienta reemplaza.
