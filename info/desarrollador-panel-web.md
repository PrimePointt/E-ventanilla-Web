# E-Ventanilla: conexión del panel web con la API y PostgreSQL del VPS

Guía para el desarrollador del **panel interno de administración**. Preparada el **11 de octubre de 2026**, a partir del código del proyecto y de la configuración de despliegue existente.

Este documento no contiene contraseñas, tokens, claves privadas ni cadenas de conexión reales. La preparación de pgAdmin y las cuentas individuales de Bryan y Brandon ya se aplicaron en el VPS. Los valores entre `<...>` son datos pendientes de asignar; deben sustituirse antes de ejecutar los comandos.

## 1. Cómo se compartirán los datos con Android

La aplicación Android y el panel utilizarán **la misma API y la misma base PostgreSQL**. La conexión habitual será: navegador del panel → backend del panel → API NestJS → PostgreSQL.

El backend del panel puede servir las páginas y actuar como intermediario de la API. Este patrón se suele llamar BFF. La lógica institucional, los permisos, las transacciones y las operaciones sobre la base deben permanecer en la API compartida.

El navegador no recibe credenciales PostgreSQL. No hay que crear otra base para los datos operativos del panel ni duplicar las cuentas, contraseñas o citas de Android.

**Para empezar a consumir la API pública desde su equipo, el desarrollador no necesita acceso SSH al VPS.** SSH y pgAdmin son accesos adicionales para inspección, mantenimiento y tareas que se autoricen expresamente.

## 2. Direcciones y configuración existente

| Elemento | Dirección o valor |
| --- | --- |
| Repositorio | `https://github.com/Daesrock/E-Ventanilla.git` |
| API por HTTPS | `https://api-ventanilla.daesrock.dev/v1/` |
| pgAdmin | `https://bd-ventanilla.daesrock.dev` |
| Organización de Cloudflare Access | `daesrock.cloudflareaccess.com` |
| IP pública del VPS registrada para el proyecto | `160.34.221.146` |
| Código en el VPS | `/home/ubuntu/e-ventanilla/app` |
| Configuración Docker | `/home/ubuntu/e-ventanilla/app/deployment/compose.yaml` |
| Nombre del proyecto Compose | `eventanilla` |
| Servicio PostgreSQL en Docker | `database` |
| Base de datos | `eventanilla` |
| Esquema principal | `public` |
| Puerto PostgreSQL dentro de Docker | `5432` |
| Runtime JavaScript/TypeScript del proyecto | Bun **1.4.2** |
| API | NestJS, TypeScript y Prisma **7.10.0** |
| PostgreSQL | **18**, imagen compatible con ARM64 |

Nginx Proxy Manager gestiona HTTPS de la API; Cloudflare publica su DNS. pgAdmin se publica mediante Cloudflare Tunnel y Access. PostgreSQL no tiene un puerto publicado al host en el Compose actual.

`database` es un nombre de servicio dentro de Docker. No se resuelve desde la computadora del desarrollador. El dominio de pgAdmin sirve una interfaz web, no una conexión PostgreSQL: no debe usarse como host de `DATABASE_URL`.

El dominio del nuevo panel y su framework todavía deben confirmarse. No se ha creado una carpeta de aplicación web ni un servicio Docker para el panel en esta entrega.

## 3. Lo disponible y lo que falta para administrar

El esquema Prisma contiene **23 tablas del dominio**, además de la tabla técnica `_prisma_migrations`. Contempla usuarios, roles y permisos, ciudadanos, personal, municipios, sedes, ventanillas, trámites, horarios, citas, asignaciones, atención, registros, notificaciones y autenticación.

Archivos principales que debe revisar el desarrollador después de clonar:

- `database/prisma/schema.prisma`: modelos y relaciones.
- `database/prisma/migrations/`: estructura, restricciones, triggers y cambios existentes.
- `database/permissions/runtime.sql`: permisos de la cuenta utilizada por la API.
- `apps/api/src/features/auth/`: acceso, verificación, recuperación y representación pública de usuarios.
- `apps/api/src/features/sessions/`: sesiones compartidas.
- `apps/api/src/features/appointments/appointment.repository.ts`: persistencia interna de citas y transacciones.
- `contracts/auth-fields.json`: reglas de campos compartidas con Android.

**Tener las tablas no significa que exista un CRUD administrativo publicado.** Actualmente hay controladores de autenticación y estado/catálogos; las rutas de administración de sedes, trámites, horarios, personal y citas siguen pendientes. El repositorio de citas tampoco equivale a una ruta HTTP de reserva disponible.

La migración inicial del dominio contempla los roles de aplicación `CITIZEN`, `COUNTER_STAFF`, `COUNTER_HEAD` y `DEPARTMENT_HEAD`. El registro público crea ciudadanos. No existe en el código revisado un flujo terminado para crear cuentas de personal desde el panel, ni guards que apliquen sus permisos administrativos.

Antes de publicar operaciones administrativas se debe implementar en la API:

1. Alta controlada de cuentas de personal y su asignación a sedes/ventanillas.
2. Comprobación de sesión, cuenta activa y requisitos de verificación.
3. Autorización por rol, permiso y ámbito de sede/ventanilla para cada operación.
4. DTO, validación, transacción y respuesta para cada endpoint administrativo.
5. Pruebas que demuestren que un ciudadano no puede ejecutar operaciones de personal.
6. Registro de acciones administrativas con actor, operación y resultado, sin credenciales ni códigos.

El backend debe consultar los permisos reales. Ocultar botones, enviar un rol desde el navegador o comprobar únicamente que el correo está verificado no concede autorización administrativa. Referencia: [autorización en NestJS](https://docs.nestjs.com/security/authorization).

## 4. Arranque del desarrollador web

```sh
git clone https://github.com/Daesrock/E-Ventanilla.git
cd E-Ventanilla
```

Clonar proporciona código y plantillas de configuración, no acceso a la base, al servidor ni cuentas administrativas. Usar Bun 1.4.2; conservar el archivo de bloqueo y las versiones elegidas.

La ubicación propuesta para la futura aplicación es `apps/web/`. Es una propuesta de organización; esa aplicación todavía no está implementada.

En el **backend** del panel, configurar una variable privada de servidor:

```dotenv
API_BASE_URL=https://api-ventanilla.daesrock.dev/v1/
```

La URL no es un secreto. Las contraseñas, cookies de sesión y tokens sí requieren protección. Nunca colocar `DATABASE_URL`, claves de Resend, secretos de códigos o credenciales del VPS en variables públicas de un framework, como `VITE_*` o `NEXT_PUBLIC_*`.

### Comprobación inicial sin cuentas ni escrituras

Los siguientes GET consultan únicamente disponibilidad y metadatos públicos:

```sh
curl --fail-with-body https://api-ventanilla.daesrock.dev/v1/health
curl --fail-with-body https://api-ventanilla.daesrock.dev/v1/ready
curl --fail-with-body https://api-ventanilla.daesrock.dev/v1/catalog/municipalities
```

`health` comprueba el proceso; `ready` comprueba que la API puede consultar PostgreSQL. Ninguno valida autorización administrativa. En PowerShell puede usarse `curl.exe` para evitar el alias de otras versiones.

Este archivo TypeScript de diagnóstico puede ejecutarse desde el servidor o equipo del desarrollador con `bun comprobar-api.ts`:

```ts
const configuredUrl = process.env.API_BASE_URL;
if (!configuredUrl) throw new Error('Configura API_BASE_URL.');

const base = new URL(configuredUrl);
if (base.protocol !== 'https:' || !base.pathname.endsWith('/')) {
  throw new Error('La API del VPS debe usar HTTPS y terminar con /.');
}

const response = await fetch(new URL('ready', base), {
  headers: { Accept: 'application/json' },
  signal: AbortSignal.timeout(10_000),
});
if (!response.ok) throw new Error(`La API no está disponible: HTTP ${response.status}.`);
console.log(await response.json());
```

Este diagnóstico no autentica personas ni consulta datos personales. No tiene que incorporarse al código del navegador.

## 5. Endpoints actuales y sesiones

Todas las rutas de esta tabla son relativas a `/v1/`:

| Método | Ruta | Uso actual |
| --- | --- | --- |
| GET | `health` | Disponibilidad del proceso |
| GET | `ready` | Disponibilidad de PostgreSQL a través de la API |
| GET | `catalog/municipalities` | Catálogo oficial de municipios |
| GET | `catalog/status` | Estado de corroboración de información institucional |
| POST | `auth/login` | Acceso con `identifier` y `password` |
| GET | `auth/me` | Datos públicos de la cuenta de la sesión |
| POST | `auth/logout` | Revocar la sesión |
| POST | `auth/register` | Registro de ciudadanos; no alta de personal |
| POST | `auth/verify-email` | Verificar correo mediante código |
| POST | `auth/resend-verification` | Solicitar explícitamente otro código |
| GET | `auth/verification-context` | Retomar seguimiento usando una sesión |
| GET | `auth/verification-status` | Estado de envío usando el token de seguimiento |
| POST | `auth/forgot-password` | Solicitar recuperación |
| POST | `auth/verify-reset-code` | Validar el código antes del cambio de contraseña |
| POST | `auth/reset-password` | Cambiar contraseña con identificador, código y nueva contraseña |
| GET | `auth/appointment-access` | Comprobar sesión y correo verificado; no crea citas |

`auth/login` devuelve `token`, `expiresAt` y `user` tras autenticar. El token de sesión es **opaco**, no un JWT. La API guarda su digest; el panel no debe intentar decodificarlo. `user` no devuelve actualmente el rol administrativo ni sus permisos.

Para `auth/me`, `auth/logout` y otros endpoints protegidos se envía:

```http
Authorization: Bearer <TOKEN_DE_SESION>
```

El token de seguimiento de correo es distinto: solo sirve para `auth/verification-status`. No es una sesión ni concede acceso administrativo. El flujo de recuperación sigue requiriendo el código al ejecutar `auth/reset-password`; validar el código no entrega una sesión de usuario.

Para el panel, el backend web deberá guardar el token de la API en una sesión de servidor, o protegerlo con una cookie `HttpOnly`, `Secure` y `SameSite` apropiada. Las operaciones con cookies deben comprobar origen y aplicar protección CSRF. No guardar tokens duraderos en `localStorage` ni incluirlos en direcciones o logs. Referencias: [sesiones de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html) y [prevención de CSRF](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html).

El inicio de sesión existente permite `allowPendingVerification`; su uso no elimina los requisitos de cada operación. Tampoco convierte a un ciudadano en personal administrativo.

### CORS y elección de arquitectura

El código actual de `apps/api/src/main.ts` **no habilita CORS**. Una SPA que llame directamente a la API desde otro origen necesitará una configuración explícita y una prueba de preflight; que Android o `curl` funcionen no demuestra que el navegador lo permita.

Con el BFF recomendado, el navegador consulta al backend de su propio panel y este llama por HTTPS a la API. CORS no se aplica a esa llamada entre servidores. Si se decide usar llamadas directas del navegador, habilitar únicamente los orígenes reales aprobados, métodos y cabeceras necesarios. No se ha aplicado ninguno de esos cambios en esta guía. Referencias: [CORS de NestJS](https://docs.nestjs.com/security/cors) y [CORS en navegadores](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS).

## 6. Acceso individual a pgAdmin

### Accesos preparados para el equipo

El propietario confirmó que añadió estos correos a Cloudflare Access. Se crearon las cuentas activas de pgAdmin y PostgreSQL, con contraseñas diferentes por persona y por servicio:

| Persona | Correo de pgAdmin | Usuario PostgreSQL |
| --- | --- | --- |
| Bryan | `bryanhernandezsolis.09@gmail.com` | `eventanilla_bryan` |
| Brandon | `brandonrgz06@gmail.com` | `eventanilla_brandon` |

Por indicación del propietario, ambos tienen rol **Administrator** en pgAdmin y los mismos permisos PostgreSQL que `eventanilla_manager`. Pueden consultar y modificar datos, crear y modificar tablas y administrar usuarios de pgAdmin. Sus roles PostgreSQL heredan `eventanilla_manager`; mantienen las mismas restricciones de ese usuario respecto a superusuario y administración de roles del clúster.

Cada cuenta tiene una conexión personal **E-Ventanilla · administración** ya registrada. Se autentica con su usuario PostgreSQL y, después, asume `eventanilla_manager`. Esto conserva el usuario individual de entrada y permite que los objetos creados desde esa conexión tengan un propietario común. Para una sesión SQL externa, ejecutar `SET ROLE eventanilla_manager;` antes de crear objetos. [Referencia de roles y propiedad](https://www.postgresql.org/docs/18/role-membership.html).

**Cómo entrar:** abrir `https://bd-ventanilla.daesrock.dev`, introducir su propio correo en Cloudflare y verificar el código recibido allí; iniciar sesión en pgAdmin con ese correo y su contraseña de pgAdmin; abrir la conexión preconfigurada y proporcionar su contraseña PostgreSQL. Las tablas aparecen en **Databases → eventanilla → Schemas → public → Tables**.

El propietario puede consultar las credenciales en su propia terminal del VPS, para entregarlas a cada destinatario por un canal privado:

```sh
sudo cat /home/ubuntu/e-ventanilla/secrets/developers/bryan.txt
sudo cat /home/ubuntu/e-ventanilla/secrets/developers/brandon.txt
```

Los archivos solo son accesibles para root; no están en Git ni montados en pgAdmin o la API. No pegar sus valores en este documento o en el chat. Se verificaron las contraseñas, la igualdad de permisos y la creación, modificación y eliminación de una tabla dentro de una transacción revertida. Las comprobaciones no cargaron ciudadanos ni dejaron tablas de prueba. La cuenta del propietario y los contenedores existentes se conservaron.

Estos accesos son para pgAdmin y PostgreSQL. Las cuentas del panel web futuro, sus permisos en la API y los accesos SSH individuales se gestionan por separado; esta preparación no los crea.

### Para incorporar otra persona

El propietario debe preparar los accesos antes de que una nueva persona pueda usar el panel de base:

1. Añadir su correo a **Include → Emails** en la política Allow de la aplicación de Cloudflare Access que protege `bd-ventanilla.daesrock.dev`. Conservar el correo del propietario. Cada persona recibe el código en su propia cuenta.
2. Crear su cuenta de pgAdmin con autenticación Internal y contraseña propia. Usar **User** para trabajo con la base; **Administrator** también permite administrar cuentas de pgAdmin y requiere que ese acceso esté autorizado.
3. Crear su rol PostgreSQL individual desde el administrador del clúster y conceder solamente los permisos acordados.
4. Usar únicamente la contraseña del rol PostgreSQL de cada persona. La contraseña automática compartida ya se retiró.

**Preparación realizada el 11 de octubre de 2026:** se retiraron `PGPASSFILE`, el montaje y definición de `pgadmin_pgpass`, el arranque personalizado y el archivo compartido del volumen. También se eliminó el parámetro `passfile` de la conexión guardada y se recreó únicamente pgAdmin con su volumen existente. El preparador de secretos ya no genera ese archivo.

Se comprobó que la conexión exige contraseña, que la contraseña existente del propietario funciona y que su cuenta de pgAdmin se conservó. PostgreSQL, la API y el túnel mantuvieron sus contenedores y no se reiniciaron. La API responde por HTTPS y Cloudflare Access sigue protegiendo pgAdmin. La configuración anterior quedó respaldada en una carpeta privada del VPS, accesible solo para root.

### Contraseña PostgreSQL del propietario

Al abrir la conexión **E-Ventanilla · administración**, pgAdmin ahora solicitará la contraseña de `eventanilla_manager`. Es distinta de la contraseña de acceso a pgAdmin y del código de Cloudflare.

El propietario puede consultarla en **su propia terminal del VPS**:

```sh
sudo cat /home/ubuntu/e-ventanilla/secrets/manager_password
```

Pegar el valor únicamente en la solicitud de contraseña de PostgreSQL. No compartirlo con el compañero ni copiarlo al chat, al repositorio o a este documento. Para el compañero se creará un rol y una contraseña propios.

Una vez provisionado, el desarrollador entra a pgAdmin y registra una conexión:

| Campo | Valor |
| --- | --- |
| Nombre | E-Ventanilla |
| Host | `database` |
| Puerto | `5432` |
| Maintenance database | `eventanilla` |
| Username | `<ROL_PERSONAL_POSTGRESQL>` |
| Password | Su contraseña individual de PostgreSQL |

Ver tablas: **Databases → eventanilla → Schemas → public → Tables**. El ERD Tool permite revisar relaciones. La cuenta de pgAdmin y el rol de PostgreSQL son dos cuentas distintas; el código enviado por Cloudflare tampoco sustituye sus contraseñas.

Referencias: [políticas de Access](https://developers.cloudflare.com/cloudflare-one/access-controls/policies/), [usuarios de pgAdmin](https://www.pgadmin.org/docs/pgadmin4/9.18/user_management.html), [conexiones de pgAdmin](https://www.pgadmin.org/docs/pgadmin4/9.18/server_dialog.html) y [passfiles de PostgreSQL](https://www.postgresql.org/docs/18/libpq-pgpass.html).

## 7. Acceso SQL desde su equipo, si lo necesita

Para desarrollar el panel consumiendo la API no se necesita esta conexión. Si requiere un cliente SQL o revisar el esquema desde su computadora, preparar una cuenta SSH personal con su clave pública y un rol PostgreSQL individual. No compartir el usuario SSH del propietario, su clave privada ni credenciales de migraciones.

El propietario debe comprobar el usuario y puerto SSH, autorizar el forwarding necesario y proporcionar la IP privada actual del contenedor de PostgreSQL. Desde el VPS puede identificarla sin mostrar secretos:

```sh
cd /home/ubuntu/e-ventanilla/app/deployment
container_id=$(sudo docker compose --env-file .env ps -q database)
test -n "$container_id"
sudo docker inspect --format '{{range $name, $network := .NetworkSettings.Networks}}{{println $name $network.IPAddress}}{{end}}' "$container_id"
```

Seleccionar la dirección de la red interna de base de datos. La IP del contenedor puede cambiar al recrearlo; debe comprobarse antes de conectar. No usar `database` como destino del túnel, porque el servidor SSH del host no utiliza el DNS interno de Compose.

En su equipo, mantener abierta una terminal con:

```sh
ssh -N -T -p <PUERTO_SSH> -o ExitOnForwardFailure=yes -o ServerAliveInterval=30 -L 127.0.0.1:15432:<IP_PRIVADA_CONTENEDOR>:5432 <USUARIO_SSH_PERSONAL>@160.34.221.146
```

El cliente SQL se conecta a `127.0.0.1`, puerto `15432`, base `eventanilla` y su rol personal. Este comando no exige publicar el puerto PostgreSQL en Internet ni cambiar el Compose actual. La conexión del host a la IP privada y los permisos de forwarding deben comprobarse al provisionar la cuenta; este túnel no fue ejecutado en esta entrega.

SSH cifra el tramo entre la computadora y el VPS. No se afirma que PostgreSQL tenga TLS habilitado dentro de Docker. Referencias: [túneles SSH de PostgreSQL](https://www.postgresql.org/docs/18/ssh-tunnels.html) y [redes y publicación de puertos Docker](https://docs.docker.com/engine/network/port-publishing/).

### Variable privada y comprobación SQL con Bun

Solo para una herramienta de servidor o diagnóstico autorizado, configurar un `.env` privado. Si la contraseña tiene caracteres reservados, codificar usuario y contraseña para una URL:

```dotenv
DATABASE_URL=postgresql://<ROL_PERSONAL_POSTGRESQL>:<CONTRASENA_CODIFICADA_URL>@127.0.0.1:15432/eventanilla
```

Con el túnel abierto, esta comprobación solo consulta metadatos de conexión:

```ts
import { SQL } from 'bun';

const connection = process.env.DATABASE_URL;
if (!connection) throw new Error('Configura DATABASE_URL en un archivo privado.');
const db = new SQL(connection);

try {
  const result = await db`
    SELECT current_database() AS database, current_user AS role
  `;
  console.log(result);
} finally {
  await db.close();
}
```

Ejecutar con Bun 1.4.2. Este diagnóstico no forma parte del frontend y no debe publicarse como endpoint del panel. La API del proyecto sigue utilizando Prisma; este ejemplo de [Bun SQL](https://bun.sh/docs/runtime/sql) no sustituye su capa de persistencia.

El desarrollo y las pruebas de escritura deben usar una base local o un entorno de pruebas separado, no ciudadanos reales en producción. Todavía debe provisionarse un entorno compartido de pruebas si el equipo lo necesita.

## 8. Si se acuerda un backend web con acceso directo a PostgreSQL

Es una alternativa arquitectónica que debe acordarse antes de implementarla. La recomendación sigue siendo ampliar la API común para que Android y web compartan validación, permisos y reglas.

Si se aprueba acceso directo para un backend web dentro del VPS:

- Conectarlo a la red privada de PostgreSQL y usar `database:5432` desde ese contenedor.
- Crear un rol de servicio exclusivo para el backend web con permisos mínimos; no usar el rol personal del desarrollador ni `eventanilla_manager`.
- Montar su secreto mediante Docker. El código del navegador no recibe esa cadena de conexión.
- Mantener una sola autoridad sobre el esquema: `database/prisma/schema.prisma` y sus migraciones. No crear un segundo historial de migraciones para las mismas tablas.
- Conservar transacciones, restricciones y triggers. No escribir directamente autenticación ni duplicar hashes, sesiones, códigos o políticas de contraseñas.
- Implementar y probar autorización administrativa en cada operación de servidor antes de exponerla.

No se ha creado ese rol, ese backend ni esa conexión de servicio. Un backend web desplegado en otro proveedor necesitará una red privada o un enlace seguro de servidor a servidor; no debe resolverlo exponiendo PostgreSQL públicamente.

## 9. Permisos: persona, aplicación y base

| Identidad | Qué permite | Qué no concede por sí sola |
| --- | --- | --- |
| Correo autorizado en Access | Llegar a pgAdmin | Entrar a PostgreSQL o administrar el panel web |
| Usuario de pgAdmin | Usar la interfaz de base | Permisos de SQL que su rol PostgreSQL no tenga |
| Rol PostgreSQL personal | Inspección o mantenimiento acordado | Ser personal administrativo dentro de la aplicación |
| Cuenta SSH personal | Tareas/forwarding autorizados en el VPS | Acceso global automático a secretos o despliegue |
| Cuenta de personal en E-Ventanilla | Operaciones que autorice la API según rol y ámbito | Acceso SSH, pgAdmin o credenciales de base |
| Rol de servicio de la API | Consultas y escrituras de la API | Migraciones, administración del clúster o identidad del usuario final |

La API actual utiliza `eventanilla_api`; las migraciones utilizan `eventanilla_migrator`. El propietario administra mediante `eventanilla_manager`; el administrador del clúster es `eventanilla_admin`. No entregar esas contraseñas al desarrollador como sustituto de provisionar su cuenta.

Actualmente `eventanilla_api` solo puede consultar `Role`, `Permission`, `RolePermission` y `Municipality`; sus escrituras están revocadas. Si el panel necesita gestionar autorizaciones, habrá que diseñar el endpoint y revisar ese permiso de servicio expresamente. No resolverlo utilizando las credenciales de migraciones para servir solicitudes web.

Agregar a alguien al grupo Docker equivale a conceder capacidades de administrador del host. Si su tarea es frontend e integración con API, no necesita ese acceso. Los cambios de estructura deben revisarse y aplicarse mediante migraciones, no quedar únicamente como modificaciones manuales de producción. Referencia: [permisos del grupo Docker](https://docs.docker.com/engine/install/linux-postinstall/).

## 10. Publicación del futuro panel en el VPS

Una vez elegidos framework y dominio:

1. Implementar el panel y sus pruebas en `apps/web/` o en la ubicación acordada.
2. Preparar su Dockerfile ARM64 y servicio separado, sin incluir `.env` ni secretos en la imagen.
3. Conectar el servicio web a Nginx mediante `proxy_net`. El BFF puede llamar a la API usando su dirección HTTPS pública y no necesita entrar a la red de PostgreSQL.
4. Crear DNS en Cloudflare para el dominio aprobado y configurar el proxy y certificado en Nginx.
5. Aplicar autenticación de personal y autorización administrativa en la API. Access para el panel, si se decide usarlo, es una capa adicional y no sustituye esos permisos.
6. Publicar cambios compatibles con Android; verificar que ambos clientes usan la misma base mediante la API.
7. Comprobar logs, errores, aislamiento de secretos y recuperación antes de habilitar escrituras de producción.

Publicar solo el frontend no crea automáticamente endpoints de administración ni permite acceder a PostgreSQL. No ejecutar migraciones de producción desde el proceso que sirve la web. Antes de migraciones debe existir un respaldo recuperable y un procedimiento de reversión revisado.

## 11. Verificación y problemas frecuentes

| Situación | Comprobación |
| --- | --- |
| API devuelve 503 en `ready` | Revisar conexión de API a PostgreSQL y salud de servicios |
| Funciona con `curl` pero el navegador informa CORS | Usar el BFF o configurar y verificar los orígenes aprobados en API |
| HTTP 401 | Sesión ausente, inválida, vencida o cuenta inactiva; volver a autenticar |
| HTTP 403 con `EMAIL_VERIFICATION_REQUIRED` | Revisar el flujo de verificación; no significa permiso administrativo |
| HTTP 403 en futuros endpoints del panel | Revisar rol, permiso y ámbito autorizados en el servidor |
| HTTP 429 | Respetar el límite de solicitudes; evitar reintentos continuos |
| `database` no resuelve desde su laptop | Usar pgAdmin del VPS o el túnel SSH con IP privada confirmada |
| `Connection refused` en `127.0.0.1:15432` | Revisar si el túnel está abierto y el puerto local está disponible |
| Falla el destino del túnel | Revisar IP del contenedor, forwarding SSH y acceso desde el host |
| `password authentication failed` | Revisar rol individual, contraseña y codificación de la URL |
| `permission denied` en una tabla | Revisar grants con el propietario; no sustituir la cuenta por un superusuario |
| Cambiar un dato en web no aparece en Android | Comprobar API/base/entorno, refresco del cliente y respuesta de la operación |

Para el propietario, los logs de API se consultan en el VPS sin copiar secretos:

```sh
cd /home/ubuntu/e-ventanilla/app
sudo sh deployment/logs.sh api --follow
```

El acceso a logs del desarrollador debe autorizarse por separado. Evitar registrar cuerpos de login, contraseñas, tokens, códigos de verificación o resultados con datos personales.

Antes de habilitar el panel, comprobar: cuenta de personal válida; ciudadano rechazado en rutas administrativas; límites por sede; escritura autorizada visible desde el otro cliente; cuenta de PostgreSQL sin DDL en ejecución; ausencia de secretos en los archivos públicos; puertos privados; y respaldo restaurable.

## 12. Pendientes para desarrollar el panel

- El propietario debe entregar a Bryan y Brandon sus contraseñas por un canal privado y cada uno debe completar su primer acceso desde su navegador.
- Necesidad real de SSH; si aplica, clave **pública**, usuario, puerto y restricciones de forwarding.
- Framework web y dominio definitivo.
- Operaciones del primer módulo del panel y permisos de cada tipo de personal.
- Entorno aislado para desarrollo y pruebas de escritura.
- Persona responsable de revisar migraciones y publicar al VPS.

Las contraseñas y claves privadas se configuran por un canal privado y no se pegan en chats ni documentos compartidos. Este Markdown está bajo `docs/`, excluido por `.gitignore`; se puede entregar manualmente al desarrollador y no se publicará en el repositorio.
