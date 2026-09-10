# WoW Perú — análisis técnico de referencia para AYNI

Este repositorio documenta un análisis **de caja negra y de frontend** del ecosistema web/launcher de WoW Perú, usando únicamente material accesible desde el navegador y archivos suministrados para una cuenta de prueba autorizada.

El objetivo no es copiar código, contenido, assets ni diseño propietario. El objetivo es entender patrones de arquitectura, flujos de cuenta, integración web↔API↔juego, launcher, operaciones de tienda y herramientas administrativas que puedan inspirar una implementación **propia y original** para AYNI.

## Estado

**Fase 1 — Panel web: completada.**

Se analizaron estas capturas de frontend:

- `wow-peru-panel.html.html`
- `scripts.js.descarga`
- `redesign.js.descarga`
- `perucoins-ingame.js.descarga`

No se almacenaron credenciales, cookies, tokens de sesión ni tokens de Cloudflare en este repositorio.

## Hallazgos principales

- El sitio funciona como HTML multipágina con comportamiento de aplicación en el panel mediante JavaScript/jQuery.
- El frontend consume una API same-origin bajo `/api`.
- El cliente principal conserva un token de sesión en `localStorage`, lo adjunta como `X-Wowperu-Session` y además usa credenciales same-origin.
- El panel se hidrata desde `/api/player-panel` y se refresca periódicamente, además de refrescar al volver el foco/visibilidad.
- El panel integra cuenta, personajes, tiempo jugado, recompensas, tienda, Perúcoins, donaciones, referidos, foro, soporte y descarga del launcher.
- Hay funciones de staff/GM presentes en el mismo frontend, pero su visibilidad depende del rol devuelto por el servidor. La seguridad real debe verificarse del lado backend; ocultar UI no equivale a autorización.
- La capa `redesign.js` es de mejora progresiva: animaciones, carruseles, accesibilidad y microinteracciones sin cambiar la API.
- `perucoins-ingame.js` es una herramienta administrativa separada para consulta de saldos/personajes/visuales y ajustes controlados.

## Documentos

- [`docs/01-arquitectura-panel-web.md`](docs/01-arquitectura-panel-web.md) — arquitectura y flujo del panel.
- [`docs/02-superficie-api.md`](docs/02-superficie-api.md) — inventario funcional de endpoints observables en frontend.
- [`docs/03-lecciones-para-ayni.md`](docs/03-lecciones-para-ayni.md) — qué conviene adoptar, mejorar o evitar en AYNI.
- [`docs/04-analisis-pendiente.md`](docs/04-analisis-pendiente.md) — lo que todavía falta estudiar para completar el panorama.

## Alcance y límites

Este análisis **no demuestra** cómo está implementado el backend, la base de datos, el servidor de juego, la autorización real de roles ni la lógica interna del launcher. Esas partes requieren artefactos adicionales o captura de tráfico autorizada y saneada.
