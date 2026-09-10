# Superficie API observable desde el frontend

> Inventario derivado del JavaScript entregado. No implica que todas las rutas estén disponibles para todos los usuarios ni que el comportamiento del backend sea idéntico a lo que sugiere la interfaz.

## Autenticación y cuenta

- `POST /api/auth/login`
- `POST /api/auth/logout`
- `POST /api/auth/register`
- `POST /api/auth/password-reset/request`
- `POST /api/auth/password-reset/confirm`
- `POST /api/auth/username-recovery/request`
- `GET /api/session/me`
- `GET /api/security/config`
- `POST /api/account/password`
- `POST /api/account/email-change/request`
- `POST /api/account/pin`
- `POST /api/account/pin/create`
- `POST /api/account/pin/reset`

## Panel

- `GET /api/player-panel?account=...&_live=...`

## Personajes y servicios

- `POST /api/character-services`
- `POST /api/character-services/reset-cooldown`
- `GET /api/characters/search?q=...`
- `POST /api/recovery-items`
- `POST /api/validate-target`

## Tienda, promociones y tiempo jugado

- `POST /api/shop-order`
- `POST /api/promos`
- `POST /api/playtime/redeem`

## Recluta a un amigo

- `POST /api/recruit/invite`
- `POST /api/recruit/claim`

## Donaciones

- `POST /api/donations`
- `POST /api/donations/yape-plin`
- `POST /api/donations/square-checkout`
- `POST /api/donations/square-confirm`
- `POST /api/donations/stripe-checkout`
- `POST /api/donations/stripe-confirm`
- `POST /api/donations/paypal-checkout`
- `POST /api/donations/paypal-confirm`

## Launcher

- `GET /api/downloads/launcher?format=json`
- `GET /api/downloads/launcher`

## Foro

Se observan rutas para índice, temas, respuestas y administración de tableros:

- `GET /api/forum`
- `GET /api/forum/topics?board=...`
- `GET /api/forum/topic?id=...`
- rutas bajo `/api/forum/boards/...`
- `POST /api/forum/topics`
- `POST /api/forum/replies`

## Tickets GM

- `GET /api/gm-tickets`
- `POST /api/gm-tickets`
- operaciones sobre `/api/gm-tickets/<id>` y comentarios asociados

El frontend contempla crear, editar, asignar, cambiar estados, comentar y cerrar/archivar tickets, sujeto a permisos de staff.

## Administración de Perúcoins

- `GET /api/admin/perucoins/accounts?limit=25&q=...`
- `GET /api/admin/perucoins?account=...`
- `POST /api/admin/perucoins/adjust`
- `GET /api/admin/perucoins/manual-payments`
- operaciones de revisión/acreditación bajo `/api/admin/perucoins/manual-payments/<id>`

## Perúcoins ingame

Módulo separado:

- `GET /api/admin/perucoins/ingame?account=...`
- `POST /api/admin/perucoins/ingame/wings-reset`
- `POST /api/admin/perucoins/ingame/adjust`

Este módulo consulta personajes, saldo web, saldo ingame y visuales comprados. El frontend lo etiqueta como herramienta para GM rango 3.

## Observaciones arquitectónicas

1. La API parece ser el punto de integración entre web y datos del reino.
2. El frontend trata al backend como autoridad para sesión, personajes, saldo, cooldowns, compras y operaciones administrativas.
3. Varias operaciones generan `clientRequestId`, patrón útil para idempotencia y para evitar dobles cobros/ejecuciones.
4. Cloudflare Turnstile protege formularios sensibles en distintos flujos.
5. El inventario no permite deducir framework backend, motor de base de datos ni protocolo interno con el servidor de juego.
