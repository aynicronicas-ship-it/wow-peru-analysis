# WoW Perú — análisis técnico de referencia para AYNI

Este repositorio documenta un análisis **de caja negra, frontend y launcher distribuido** del ecosistema WoW Perú, usando material accesible desde el navegador y archivos suministrados para una cuenta de prueba autorizada.

El objetivo no es copiar código, contenido, assets ni diseño propietario. El objetivo es entender patrones de arquitectura y traducirlos a una implementación **propia y original** para AYNI.

## Estado

- **Fase 1 — Panel web: completada.**
- **Fase 2 — Análisis estático del launcher: completada.**
- **Fase 3 — Manifest completo del cliente: completada.**
- **Fase 4 — Comparación repair/HD/addons: pendiente y opcional.**

Material estudiado:

- `wow-peru-panel.html.html`
- `scripts.js.descarga`
- `redesign.js.descarga`
- `perucoins-ingame.js.descarga`
- `app-update.yml`
- `app.asar`
- `manifest-full.json` (contenido suministrado como captura de texto)

No se almacenan en este repositorio credenciales, cookies, tokens de sesión, tokens Turnstile ni copias del código/ASAR propietario.

## Hallazgos principales

### Panel web

- API same-origin bajo `/api`.
- Sesión de panel manejada desde el frontend con token y verificación de `/session/me`.
- Panel de cuenta, personajes, tiempo jugado, recompensas, tienda, Perúcoins, donaciones, referidos, foro, soporte y descarga.
- Funciones GM presentes en frontend pero condicionadas por permisos devueltos por servidor.

### Launcher

- Electron + React.
- Versión analizada del launcher: `1.0.143`.
- `electron-updater` `6.8.9`.
- Ruta administrada del cliente: `C:\ProgramData\WoW Peru\Client`.
- Descargas desde `https://download.wow-peru.lat`.
- Manifests separados para instalación completa, reparación y HD.
- Verificación SHA-256 de archivos descargados.
- Auto-update exige SHA-512 de paquete.
- Descargas concurrentes, reanudables y segmentadas para archivos grandes.
- Reparación con política de rutas permitidas y backups `.wowperu.bak`.
- Gestión segura de ZIP/addons.
- Configuración automática de realmlist/WTF.
- Lanzamiento de `Wow.exe` sin argumentos adicionales.
- Renderer Electron aislado mediante preload/IPC, sandbox y `nodeIntegration: false`.

### Manifest completo

- 217 archivos activos y 3 rutas retiradas.
- Tamaño declarado: ~17.426 GB / **16.229 GiB**, coherente con los `16.2 GB` mostrados en la UI.
- ~99.1% del peso está en `Data/`; `Interface/` representa ~0.75%.
- 24 archivos MPQ contienen prácticamente todo el volumen del cliente.
- Cada artefacto declara `path`, `size`, `sha256` y `url`, habilitando descarga y verificación por archivo.
- Existe un mecanismo explícito de `retiredFiles` para eliminar contenido sustituido.
- Hay personalización observable como `Data/patch-Z-WOWPERU.MPQ` y `Interface/AddOns/WowPeruVisualShop/`.

## Documentos

- [`docs/01-arquitectura-panel-web.md`](docs/01-arquitectura-panel-web.md)
- [`docs/02-superficie-api.md`](docs/02-superficie-api.md)
- [`docs/03-lecciones-para-ayni.md`](docs/03-lecciones-para-ayni.md)
- [`docs/04-analisis-pendiente.md`](docs/04-analisis-pendiente.md)
- [`docs/05-launcher-update-config.md`](docs/05-launcher-update-config.md)
- [`docs/06-launcher-static-analysis.md`](docs/06-launcher-static-analysis.md)
- [`docs/07-blueprint-launcher-ayni.md`](docs/07-blueprint-launcher-ayni.md)
- [`docs/08-manifest-full-analysis.md`](docs/08-manifest-full-analysis.md)

## Resultado para AYNI

Ya existe suficiente información para diseñar nuestro propio flujo:

`Portal/Auth → Release Service → CDN → AYNI Launcher → cliente Godot → Game Gateway`

El launcher de AYNI debe usar manifests firmados, verificación de integridad, reparación incremental, auto-update separado del juego, handoff de sesión efímero y workshop/mods aislados.

## Alcance y límites

Este análisis no demuestra la implementación privada del backend, base de datos ni servidor de juego de WoW Perú. Tampoco necesitamos conocerlos para construir AYNI.
