# Triage de GuardDuty — familias de falso positivo (detección correcta, narrativa equivocada)

Regla de oro: **antes de escalar cualquier finding de tipo C&C/DNS-malo, revisar el tag `Name`/función del recurso + la evidencia cruda (dominio DNS, IP remota, API observada) — no solo el tipo de finding.** Instancias que son appliances (firewall/proxy/DNS/DC) manejan tráfico malicioso downstream **por diseño** y disparan estos findings como FP. Aislarlas tumba la red del cliente sin necesidad.

## (a) DNS malo desde appliance/DNS del dominio → C2 falso positivo
El appliance agrega y filtra el DNS de todo lo downstream; GuardDuty ve el dominio malo salir de la IP del appliance y marca la instancia como comprometida — el origen real es un endpoint downstream, no el appliance. **No aislar.** Hallar el endpoint real con DNS query logging en el appliance/DC.

Candidatos típicos a marcar como "appliance conocido" por tag/función: firewall perimetral (FortiGate y similares), proxy (pfSense, Squid), servidor DNS (BIND), Domain Controller.

Descartes útiles cuando el finding cita un IOC público conocido (ej. una campaña de supply-chain documentada): verificar arquitectura (el backdoor de la campaña corre en Windows/una plataforma específica — un firewall Linux no puede ejecutarlo) y verificar que las máquinas realmente alcanzables no tienen el software vulnerable instalado.

## (b) DNS a streaming/ads/phishing pirata → falso positivo de "usuario navegando"
Dominios maliciosos **reales**, pero el patrón de tráfico es de usuario humano, no de beacon C2: ráfagas concentradas en horario laboral + silencios largos (días/semanas), sin exfiltración de datos.

**Gotcha de decodificación:** subdominios "aleatorios" en este tipo de infraestructura suelen ser base64 en minúsculas de nombres de canal/stream — al decodificar, probar combinaciones de mayúsculas porque DNS es case-insensitive y el encoding original se pierde.

**Gotcha operativo:** algunas protecciones anti-malware de endpoint (ej. blocklists de SO) bloquean copy/paste de estos dominios y pueden interferir con archivos que los contengan — defanguear (reemplazar `.` por `[.]`) antes de manipular evidencia.

## (c) Credential exfiltration con solo APIs de agente de gestión → falso positivo de egress híbrido
Un finding de exfiltración de credencial "fuera de AWS" que muestra SOLO llamadas de un agente de gestión (heartbeat, reporte de compliance, inventario) desde una sola IP suele ser egress híbrido: el tráfico de gestión de la instancia sale por la red del cliente (colo/VPN) en vez de por la red pública de AWS directamente, no un robo real. Confirmar con logs de auditoría por AccessKeyId que la credencial NO hizo enumeración real (listado de buckets, roles IAM, identidad, describe de recursos).

## (d) Acceso público anómalo con nombre alarmante → hosting estático intencional
Revisar el CONTENIDO real antes de escalar: un bucket/recurso con acceso público y nombre que suena sensible puede ser simplemente un frontend estático compilado servido por diseño — el riesgo real solo existe si el contenido incluye secretos embebidos.

## (e) Findings de muestra generados desde consola
Sample findings de la consola (`GeneratedFinding*`, instance-id de ejemplo, flags EICAR) son ruido, no señal. Filtrar por flag de "sample" — ojo que este flag puede venir anidado como string JSON dentro de un campo de metadata en vez de un booleano de primer nivel, y el placeholder de recurso puede venir en formatos y capitalización distintos según el tipo de servicio (contenedor/base de datos/función/instancia). Verificar antes de archivar automáticamente que ningún sample coincida por casualidad con algo que sería grave de ser real.

## (f) Comportamiento anómalo de usuario IAM sobre rol de administración federado
Sesión de administración vía SSO/Identity Center que ejecutó APIs "de impacto" (borrado, cambios de organización) casi siempre es un administrador legítimo o el propio proveedor de servicios gestionados conteniendo un incidente. Confirmar con logs de auditoría el nombre de sesión, qué APIs, desde qué IP. **Cuidado:** una variante que use una credencial de usuario IAM normal (no federada por SSO) sí merece revisión aparte con más cuidado — el mismo tipo de finding con origen distinto tiene prioridad distinta.

## (g) Cadena "multi-etapa" en un solo endpoint → estación de trabajo de desarrollo
La combinación de runtime de scripting + entorno tipo WSL/shell + herramientas de automatización + túneles de exposición (tipo ngrok) + acceso a almacenes de credenciales de navegador es el combo que dispara motores de detección de comportamiento sin parar (se ve como movimiento lateral → C2 → ejecución → acceso a credenciales → persistencia, en cadena). Discriminar mirando el dispositivo real (¿es una estación de desarrollo no gestionada, o un servidor de producción?) y las líneas de comando reales. El hallazgo real de fondo suele ser de menor severidad: estación de desarrollo sin gestión centralizada con admin local + túnel expuesto — llevar a gestión de dispositivos (MDM/Intune), no a contención de incidente.

## Contraste: hallazgos colaterales reales suelen ser más serios que la alerta que disparó el triage
En la práctica, revisar el grupo de seguridad/firewall del recurso flageado como FP suele revelar el hallazgo real: puertos de base de datos (SQL, RDP en puerto no estándar) abiertos a cualquier origen, con miles de probes de reconocimiento activo. El proceso de triage de una alerta ruidosa es también la oportunidad de auditar la exposición real del recurso.
