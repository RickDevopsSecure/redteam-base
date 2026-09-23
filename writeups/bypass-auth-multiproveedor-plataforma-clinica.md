# Cuando el control de acceso existe, pero solo para un proveedor de login: brecha de datos de pacientes por bypass de auth multi-proveedor

**Sector:** salud — plataforma clínica secundaria de un grupo hospitalario (sub-sistema separado, sobre Firebase, distinto del sitio principal del grupo). **Tipo de engagement:** CVP (evaluación de postura) black-box, alcance AWS + Firebase + Google Workspace. Sin nombres, dominios ni evidencia identificable — el detalle técnico es real, el objetivo no.

## Contexto

Grupos grandes casi nunca tienen una sola superficie web — tienen la principal, bien vigilada, y dos o tres plataformas secundarias construidas por un equipo distinto, con otro stack, que reciben mucha menos atención. Este caso es exactamente eso: una plataforma clínica separada, construida sobre Firebase, que manejaba datos reales de pacientes, corriendo en paralelo al sitio principal del grupo (cubierto en [`recon/subdomain-enumeration.md`](../recon/subdomain-enumeration.md) — el reconocimiento de este mismo engagement, incluyendo cómo se descubrió esta plataforma, ya está documentado ahí; este write-up es sobre lo que se encontró una vez dentro).

## El hallazgo crítico — brecha de PHI por auto-registro, sin credenciales previas

Cualquier persona anónima en internet podía crear una cuenta en la plataforma y, con esa sola cuenta recién creada sin ningún privilegio otorgado, leer la base completa de pacientes: nombre, fecha de nacimiento, sexo, y número de expediente médico de aproximadamente un centenar de registros.

**La cadena tenía tres piezas, cada una razonable por separado, catastrófica junta:**

1. **El registro público de cuentas estaba habilitado** en el proyecto Firebase — cualquiera podía crear una cuenta con solo un correo y contraseña, sin invitación ni aprobación.
2. **Las reglas de seguridad de Firestore** (la base de datos) gateaban la colección de pacientes con una condición de "cualquier usuario autenticado" — no por rol, no por pertenencia a un grupo, solo por el hecho de tener sesión iniciada.
3. **El control que se suponía debía cerrar el hueco de la regla 2** — verificar que el usuario perteneciera a un grupo autorizado en el directorio corporativo (Azure AD) — estaba implementado, pero **solo se ejecutaba en el flujo de login vía SSO de Microsoft**. El flujo de registro directo por correo/contraseña nunca pasaba por esa verificación, así que una cuenta creada por ese camino quedaba automáticamente "autenticada" para efectos de la regla 2, sin que el check de grupo se hubiera ejecutado ni una sola vez.

Reproducible sin destruir nada: config pública del proyecto → registro de cuenta nueva → lectura directa de la colección de pacientes. Cero credenciales robadas, cero exploit de software — la brecha era 100% de configuración y de una verificación de autorización que existía pero no cubría todos los caminos de entrada.

## Por qué esto es más importante que "faltó una regla"

Es tentador leer esto como "les faltó una regla de Firestore" y quedarse ahí. La causa raíz real es otra: **un control de autorización que depende de qué proveedor de login usó el usuario, en vez de aplicarse de forma uniforme después de la autenticación, no es un control — es un control con un agujero del tamaño del proveedor que no se cubrió.**

Es la tercera vez que este patrón aparece en este repo, con una superficie distinta cada vez:

- En [`methodology/scope-and-authorization.md`](../methodology/scope-and-authorization.md): un gate de autorización de red necesita un único punto de verdad que toda acción debe pasar.
- En [`writeups/pentest-fintech-pagos-jwt-forjable-y-exposicion-masiva.md`](pentest-fintech-pagos-jwt-forjable-y-exposicion-masiva.md): el mismo check de rol de negocio, copiado en seis endpoints distintos, faltaba en un séptimo.
- Aquí: el mismo check de pertenencia a grupo, implementado para un proveedor de identidad, nunca se generalizó a los demás proveedores que el sistema igual aceptaba.

**La regla que se repite:** cuando un sistema acepta más de un camino para llegar al mismo estado ("usuario autenticado"), cualquier verificación de autorización que dependa de contexto específico de UN camino (qué proveedor, qué endpoint, qué mutación) va a fallar en los demás caminos, a menos que se mueva a un punto después de donde todos los caminos convergen. No es un bug de un desarrollador distraído — es lo que pasa por diseño cuando la autorización se acopla al mecanismo de autenticación en vez de ejecutarse después de él, sin importar cuál se haya usado.

## Hallazgos adicionales del mismo engagement (resumen, sin profundizar — ya cubiertos por patrones existentes en este repo)

| Hallazgo | Severidad | Nota |
|---|---|---|
| Bypass de auth multi-proveedor → brecha de PHI (self-registro) | Crítico | Detallado arriba |
| Dominio principal suplantable (sin SPF, DMARC en modo monitoreo) | Alto | Mismo patrón que `methodology/phases.md` D1.5 |
| PHI de pacientes recuperable en historial de git de un repo público | Alto | Mismo patrón que `recon/subdomain-enumeration.md` — no basta con mirar el HEAD |
| Credencial de empleado filtrada, con infección de endpoint activa confirmada | Alto | Mismo patrón que `writeups/osint-pasivo-telecom-identidad-corporativa-credenciales-filtradas.md` |
| Código fuente completo + topología interna expuestos vía repo público | Medio | Mismo patrón que `recon/subdomain-enumeration.md` |
| Runtime end-of-life (varios hosts) | Medio | — |
| Certificado TLS incorrecto + downgrade HTTPS→HTTP en un portal | Medio | — |

## Lo que estaba bien calibrado

El proyecto Firebase rechazaba correctamente llamadas anónimas directas a la API (403/404 sin sesión) — el hueco era específicamente la combinación registro-público + regla laxa + check parcial, no una ausencia total de controles. La infraestructura AWS del grupo, aparte de esta plataforma secundaria, tenía TLS wildcard válido, App Check habilitado, y dependabot activo — la higiene general no era mala, la brecha vivía en un sub-sistema con menos ojos encima.

## Metodología y patrones relacionados en este repo

- [`recon/subdomain-enumeration.md`](../recon/subdomain-enumeration.md) — cómo se descubrió esta plataforma secundaria y su repo público (mismo engagement).
- [`methodology/scope-and-authorization.md`](../methodology/scope-and-authorization.md) — el principio de único punto de verdad para autorización, ahora en su tercera variante documentada en este repo.
- [`writeups/pentest-fintech-pagos-jwt-forjable-y-exposicion-masiva.md`](pentest-fintech-pagos-jwt-forjable-y-exposicion-masiva.md) — la variante de negocio del mismo principio (check de rol repetido vs. centralizado).
