# Blueprint de launcher para AYNI

Este documento traduce patrones observados en el launcher estudiado a una arquitectura propia para AYNI. No copia código ni assets.

## Arquitectura propuesta

`AYNI Launcher UI` → `Preload/API local` → `Launcher Core` → `Release Service + CDN + Game Client`

Separar cinco responsabilidades:

1. **UI del launcher**: login opcional, noticias, estado del mundo, progreso, configuración y botón Jugar.
2. **Launcher Core**: acceso a disco, manifests, verificación, reparación, procesos y auto-update.
3. **Release Service**: publica metadata firmada de versiones y canales.
4. **CDN/Storage**: entrega archivos grandes del cliente por HTTPS con Range.
5. **Game Client/Gateway**: el launcher inicia el ejecutable propio de AYNI y el juego se autentica contra servicios de AYNI.

## Manifest recomendado

Para AYNI conviene usar un manifest versionado con al menos:

- `releaseId`
- `channel` (`stable`, `test`)
- `minimumLauncherVersion`
- `files[]`
  - `path`
  - `size`
  - `sha256`
  - `url` o clave CDN
  - `group` (`core`, `optional-hd`, `language`, etc.)
- `retiredFiles[]`
- firma digital del manifest

Mejora sobre el patrón estudiado: **verificar criptográficamente la firma del manifest**, no limitarse a detectar que exista un campo `signature`.

## Descarga y reparación

AYNI debería incorporar:

- descargas concurrentes con límite configurable;
- HTTP Range para reanudar;
- `.part` para descargas incompletas;
- descarga segmentada para archivos grandes;
- SHA-256 obligatorio después de cada descarga;
- reemplazo atómico del archivo;
- retries con backoff y jitter;
- pausa/reanudación/cancelación;
- estimación de velocidad y ETA;
- journal local para recuperar operaciones tras un cierre inesperado.

Para Godot, conviene evitar empaquetar todo el contenido en uno o dos archivos enormes si queremos parches eficientes. Dividir contenido por paquetes/regiones permite que una actualización no obligue a descargar decenas de GB.

## Instalación

Ruta administrada sugerida en Windows:

`C:\ProgramData\AYNI\Game`

o una carpeta elegida por el jugador, manteniendo el launcher separado del cliente.

Guardar configuración de launcher en una ubicación de usuario (`AppData`) y datos del juego en la ruta elegida.

## Integridad y seguridad

Reglas recomendadas:

- manifests únicamente desde HTTPS;
- allowlist de dominios CDN;
- bloquear rutas absolutas y `..`;
- nunca escribir fuera de la raíz del cliente;
- validar symlinks/reparse points antes de borrar carpetas;
- manifests firmados por una clave cuya pública esté embebida en el launcher;
- firma de código del instalador y ejecutable del launcher;
- no almacenar contraseñas en texto plano;
- renderer sin Node.js;
- `contextIsolation` y sandbox activados si usamos Electron;
- API IPC mínima y validación de tipos/rutas en el proceso principal.

## Auto-update del launcher

Separar completamente la versión del launcher de la del juego.

El launcher debe:

1. comprobar su propia actualización;
2. verificar firma/hash del instalador/update package;
3. descargarla en segundo plano;
4. pedir reinicio para instalar;
5. soportar rollback o al menos conservar el instalador de la versión anterior durante el cambio.

## Flujo de Jugar para AYNI

1. comprobar actualización del launcher;
2. obtener release actual del juego;
3. escanear diferencias;
4. reparar/descargar lo necesario;
5. validar integridad final;
6. obtener un **ticket de sesión corto** desde Auth Service;
7. iniciar el ejecutable del juego con un mecanismo seguro de handoff;
8. el cliente canjea el ticket contra Game Gateway;
9. el ticket expira y no sirve para volver a iniciar sesión.

Esto es preferible a pasar usuario/contraseña en la línea de comandos.

## Login y portal

AYNI puede permitir login en el launcher, pero el panel web y el launcher no deben compartir tokens de larga duración sin necesidad.

Propuesta:

- portal web: cookie segura HTTP-only;
- launcher: OAuth/device flow o credencial propia protegida por Windows Credential Manager;
- juego: ticket efímero emitido por el launcher/auth service.

## Noticias, comunidad y workshop

El launcher puede consumir JSON/API para:

- noticias;
- mantenimiento;
- estado de servidores;
- eventos;
- workshop/mods autorizados;
- notas de parche.

El Workshop de AYNI debería tener manifests independientes por mod/paquete y aislamiento de rutas, similar al principio de seguridad usado para addons, pero diseñado desde cero para nuestro formato.

## Observabilidad

Guardar logs estructurados y saneados:

- versión launcher/juego;
- fase de descarga;
- error de red;
- archivo afectado sin rutas personales;
- hashes esperados/resultado de integridad;
- status del gateway;
- crash id.

Nunca registrar contraseñas, refresh tokens o tickets de sesión.

## Tecnología

Electron sirve y ofrece una UI web muy flexible, pero AYNI también puede valorar **Tauri** si queremos menor tamaño y memoria. La decisión debe tomarse por requisitos del launcher, no por copiar la tecnología del servidor analizado.

Para nuestro proyecto actual en Godot, el launcher debería permanecer como aplicación separada del ejecutable del juego.

## MVP recomendado

Primera versión de AYNI Launcher:

- noticias + estado;
- seleccionar ruta;
- instalar juego;
- manifest firmado;
- descarga concurrente/reanudable;
- SHA-256;
- reparar;
- auto-update launcher;
- botón Jugar;
- logs diagnósticos;
- canal `stable/test`.

Después añadimos login integrado, workshop, contenido opcional y control de múltiples regiones/servidores.
