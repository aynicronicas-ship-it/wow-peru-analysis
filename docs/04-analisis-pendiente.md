# Análisis pendiente

La fase de **frontend/panel web** y el **análisis estático del launcher** ya están suficientemente cubiertos para diseñar una primera arquitectura completa de AYNI.

## Completado — Launcher estático

Con `app-update.yml` y `app.asar` ya se confirmó:

- Electron + React;
- estructura renderer/preload/main;
- auto-update con `electron-updater`;
- feed de updates;
- manifests de instalación, reparación y HD;
- servidor de descargas;
- SHA-256 de archivos y SHA-512 para paquetes de auto-update;
- escaneo, reparación y backups;
- descarga concurrente, reanudable y segmentada;
- pausa/reanudación/cancelación;
- selección de cliente existente;
- ruta administrada del cliente;
- configuración automática de realmlist/WTF;
- limpieza de cache antes de jugar;
- lanzamiento de `Wow.exe` sin argumentos;
- gestión de addons;
- diagnóstico local;
- controles de seguridad de Electron.

Ver `06-launcher-static-analysis.md`.

## Prioridad 1 — Manifests reales vigentes

El ASAR revela las URLs y el contrato esperado, pero no incluye necesariamente una copia actual de los manifests de producción.

Todavía sería útil obtener de forma legítima y de solo lectura:

- `manifest-full.json`;
- `repair-manifest.json`;
- `manifest-hd.json`;
- `addons.json`;
- metadata actual del auto-updater.

Eso permitiría confirmar tamaños actuales, número de archivos, agrupación real, nombres de paquetes y si existe/firma válida de manifests.

No necesitamos descargar todos los archivos del cliente para esto.

## Prioridad 2 — Cliente instalado / diferencias reales

Si queremos cerrar el flujo launcher → juego, basta con inspeccionar una instalación legítima del cliente, no copiarla completa al repositorio.

Interesa documentar:

- árbol de primer nivel;
- `Data/` y parches personalizados;
- `Interface/AddOns` gestionado;
- `WTF/Config.wtf` y `realmlist.wtf` saneados;
- presencia/ausencia de archivos HD opcionales;
- qué archivos cambia el launcher tras reparar o instalar HD.

No es necesario subir los ~16 GB del juego.

## Prioridad 3 — Comportamiento de red durante una instalación pequeña

Una captura autorizada y saneada del launcher podría confirmar:

- uso real de HTTP Range;
- número de conexiones simultáneas;
- reanudación después de pausa;
- comportamiento ante hash fallido/retry;
- cabeceras CDN;
- cache-control y redirects.

Esto es opcional: el código distribuido ya permite diseñar nuestro propio motor de descargas.

## Prioridad 4 — Tráfico normal y autorizado del panel web

Un HAR **saneado** permitiría confirmar respuestas reales de `/api/player-panel`, `/api/session/me` y acciones normales de usuario.

Antes de compartir un HAR deben eliminarse:

- `Cookie` / `Set-Cookie`;
- `Authorization`;
- `X-Wowperu-Session`;
- contraseñas;
- tokens Turnstile;
- tokens de recuperación;
- datos personales no necesarios.

No necesitamos probar endpoints administrativos con una cuenta sin permisos ni intentar saltar autorización.

## Prioridad 5 — Backend y servidor de juego

El frontend y launcher no demuestran:

- framework del backend;
- base de datos;
- integración interna con Auth/World server;
- colas/jobs de entrega de items;
- locking/concurrencia de wallet;
- validación backend de roles;
- almacenamiento de comprobantes;
- sistema de correo;
- arquitectura interna del servidor de juego.

Estas piezas solo deben documentarse si aparecen en material público o material facilitado con autorización.

## Qué ya NO hace falta investigar para comenzar AYNI

No necesitamos conocer el código fuente privado de WoW Perú, su base de datos ni descargar todo su cliente para avanzar.

Ya tenemos suficiente evidencia para definir en AYNI:

- portal de cuenta;
- panel del jugador;
- modelo de servicios y wallet;
- launcher separado;
- Release Service;
- manifests firmados;
- CDN;
- instalación/reparación;
- auto-update;
- integridad de archivos;
- workshop/addons;
- handoff seguro launcher → juego;
- observabilidad y diagnósticos.

El siguiente análisis con mayor retorno, si queremos seguir estudiando WoW Perú, es **obtener los manifests vigentes o inspeccionar una instalación real del cliente**, no seguir desensamblando el launcher indefinidamente.
