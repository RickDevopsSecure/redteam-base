# Reconocimiento 100% pasivo en un grupo de telecomunicaciones: el DNS delata al grupo, y las credenciales filtradas importan más que el firewall

**Sector:** telecomunicaciones / ISP (grupo con marca de cara al consumidor, marca corporativa, y un brazo tecnológico/MSP propio). **Tipo de engagement:** evaluación de postura (CVP) estrictamente pasiva — cero tráfico intrusivo, todo verificado con fuentes públicas (DNS, TLS, HTTP status, inteligencia de credenciales filtradas). Sin nombres, dominios ni evidencia identificable — el detalle técnico es real, el objetivo no.

## Contexto

Grupo con al menos tres propiedades digitales distintas: una marca de consumo (front-end público), una marca corporativa (donde vive el riesgo real), y un brazo tecnológico/MSP interno del propio grupo. El engagement fue deliberadamente pasivo — sin enviar un solo payload, sin autenticarse en nada — para demostrar cuánta señal de riesgo real se puede generar solo leyendo lo que ya es público.

## Hallazgo 1 — Confirmar que dos marcas son el mismo grupo, sin más fuente que el DNS

Antes de correlacionar cualquier hallazgo entre las dos marcas del grupo, hacía falta confirmar que efectivamente pertenecían a la misma organización (evitar atribuir a un cliente el riesgo de otro). La confirmación no vino de una fuente externa — vino de leer el registro DMARC de una de las marcas: el campo de reporte agregado (`rua`) apuntaba a un buzón en el dominio de la OTRA marca del grupo.

**Por qué vale la pena documentarlo como técnica:** es una correlación gratuita, pasiva y difícil de negar — nadie configura el buzón de reportes DMARC de un dominio para que apunte al dominio de un tercero no relacionado. Es una forma confiable de mapear qué dominios pertenecen al mismo grupo cuando la relación societaria no es obvia desde afuera, sin depender de WHOIS (frecuentemente privatizado) ni de adivinar por el nombre.

## Hallazgo 2 (CRÍTICO real del engagement) — Identidad corporativa ya comprometida por filtraciones históricas

La marca corporativa del grupo usa autenticación única (SSO) tipo M365 para todo el personal. Cruzando el dominio corporativo contra fuentes de inteligencia de credenciales filtradas: miles de registros agregados asociados al dominio, de los cuales más de un centenar eran cuentas corporativas con contraseña en texto claro, y varios miles adicionales correspondían a credenciales de clientes/personal capturadas por malware de tipo infostealer (no filtraciones de terceros — credenciales robadas directamente del dispositivo de la víctima). Una porción de las cuentas corporativas en texto claro seguía activa y circulando en canales de venta de credenciales al momento de la verificación.

**Por qué esto pesa más que un hallazgo técnico de infraestructura:** con SSO corporativo, una sola credencial corporativa válida es la llave de todo lo que ese SSO protege. Ningún hardening de red detiene esto — el vector no es la infraestructura, es la identidad de la persona. Es el tipo de hallazgo que un pentest activo tradicional, enfocado en la aplicación, ni siquiera busca, y que en la práctica es más explotable que la mayoría de las vulnerabilidades técnicas del mismo informe.

## Hallazgo 3 — Ambientes de desarrollo de pago y autenticación, expuestos y alcanzables

Enumeración de subdominios (vía certificate transparency) de la marca corporativa devolvió el inventario esperado de producción, pero también varios ambientes que por nombre eran claramente de desarrollo/staging — y que respondían con HTTP 200 a cualquiera, sin autenticación: un bucket de almacenamiento de objetos sirviendo contenido de un ambiente de "pago en línea" de desarrollo, un servidor web de desarrollo para un flujo de pagos, un portal de pagos de desarrollo servido desde almacenamiento de objetos, un endpoint de autenticación de desarrollo detrás de un balanceador de carga, y un ambiente de pruebas corriendo la configuración por defecto de un servidor web IIS.

**El error de criterio que esto expone:** "es solo un ambiente dev" no es una razón válida para no hardenizarlo cuando ese ambiente implementa lógica de pago o autenticación. Un ambiente de pruebas de pago alcanzable públicamente es, para efectos de riesgo, una segunda superficie de ataque de pagos — casi nunca recibe el mismo nivel de revisión de seguridad que producción, y con frecuencia comparte lógica de negocio real (aunque sea contra datos de prueba) que revela cómo funciona el flujo real.

## Hallazgo 4 (target relacionado: brazo tecnológico/MSP del grupo) — Superficie operativa interna expuesta

El brazo tecnológico/MSP del grupo (perfil de riesgo distinto a la marca de consumo: sin credenciales filtradas, DMARC en cumplimiento estricto) sí tenía problema de superficie operativa: decenas de subdominios vivos, entre ellos una plataforma de automatización de flujos de trabajo en pantalla de configuración inicial **sin reclamar** — quien complete esa configuración primero se queda con control administrativo total de esa instancia de automatización —, múltiples instancias de monitoreo de infraestructura expuestas, un portal de centro de operaciones de red, y un certificado TLS vencido desde hacía más de dos meses en un subdominio de soporte corriendo la página por defecto de IIS.

**Por qué importa aunque no sea la marca "principal":** el brazo tecnológico/MSP de un grupo suele tener acceso operativo a la infraestructura de las demás marcas del grupo (es, literalmente, su función). Una plataforma de automatización sin reclamar ahí no es un riesgo aislado de una propiedad secundaria — es un punto de pivote hacia el resto del grupo.

## Lo que estaba bien calibrado

Ambas marcas del grupo tenían DMARC en política `p=reject` con alineación estricta — protección real contra suplantación de dominio por correo, no una política cosmética en modo de solo monitoreo. Gestión de DNS/NS consistente y correctamente delegada, y el correo corporativo apuntando limpiamente a la infraestructura correcta. El perímetro de red, en el sentido clásico, estaba genuinamente bien construido — la exposición real no vivía ahí.

## Disciplina de alcance

El engagement excluyó deliberadamente la capa de infraestructura de telecomunicaciones propiamente dicha (el plano de gestión de red del proveedor como operador de servicio) — quedó explícitamente diferida a un assessment autorizado aparte, de mayor alcance. Un pase pasivo de reconocimiento no es la vía para tocar infraestructura de ese tipo de riesgo, y decirlo explícitamente en el entregable es tan parte del profesionalismo como cualquier hallazgo.

## Resumen de hallazgos

| Hallazgo | Severidad | Confirmado por |
|---|---|---|
| Cross-atribución de marcas vía DMARC `rua` | — (técnica, no hallazgo de riesgo) | Registro DNS público |
| Credenciales corporativas filtradas, subconjunto activo, SSO expuesto | Crítico | Inteligencia de credenciales filtradas |
| Credenciales de clientes/personal vía infostealer | Alto | Inteligencia de credenciales filtradas |
| Ambientes dev de pago/autenticación alcanzables públicamente | Medio (x3-5) | Enumeración de subdominios + status HTTP |
| Automatización interna sin reclamar (brazo tecnológico/MSP) | Alto | Enumeración de subdominios + inspección de pantalla pública |
| Certificado TLS vencido en subdominio de soporte | Bajo | Verificación TLS |
| DMARC `p=reject` en ambas marcas | — | Bien calibrado |

## Metodología y patrones relacionados en este repo

- [`recon/subdomain-enumeration.md`](../recon/subdomain-enumeration.md) — por qué certificate transparency solo, sin DNS-brute, se queda corto; aquí CT sí alcanzó porque el objetivo no dependía de wildcard.
- [`methodology/scope-and-authorization.md`](../methodology/scope-and-authorization.md) — el principio de excluir explícitamente lo que no se probó, no solo reportar lo que sí.
- [`web-injection/false-positive-catalog.md`](../web-injection/false-positive-catalog.md) — el mismo principio de "confirmar antes de reportar" aplicado aquí a inteligencia de credenciales: un registro filtrado no vale lo mismo si está muerto que si sigue circulando activo.
