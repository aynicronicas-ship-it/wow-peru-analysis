# Arquitectura observable del panel web

## 1. Tipo de frontend

El panel observado es una aplicación web multipágina construida sobre HTML tradicional, jQuery y JavaScript propio. La lógica principal reside en `scripts.js`; `redesign.js` funciona como capa visual aditiva; `perucoins-ingame.js` añade herramientas administrativas específicas.

El `<body>` del panel declara `data-api-base="/api"`, por lo que el frontend consume rutas same-origin bajo ese prefijo.

## 2. Flujo de autenticación

El login envía un `POST /api/auth/login` con usuario, contraseña y, cuando está activo, token de Cloudflare Turnstile.

Tras un login correcto, la respuesta completa se guarda en `localStorage` bajo `wowPeruSession`, junto con la marca `wowPeruLoggedIn=true`.

La sesión se considera válida en cliente cuando:

- existe `wowPeruLoggedIn=true`;
- `wowPeruSession.ok === true`;
- existe `session.user.username`;
- existe un token de más de 32 caracteres.

Cada llamada centralizada con `apiRequest(...)` agrega el token al header `X-Wowperu-Session` y también utiliza `credentials: 'same-origin'`.

El cliente consulta `/api/session/me` para volver a validar la sesión y obtener, entre otros datos, información de acceso GM.

> Nota: lo anterior describe el comportamiento del frontend. No prueba cómo valida ni almacena sesiones el backend.

## 3. Carga del panel

La pantalla privada se hidrata principalmente mediante:

`GET /api/player-panel?account=<CUENTA>&_live=<timestamp>`

La respuesta alimenta al menos estos bloques lógicos:

- `user`
- `security`
- `staff`
- `characters`
- `deletedCharacters`
- `playtimeCharacters`
- `playtime`
- `transactionHistory`
- `donationPackages`
- `referral`
- `recruitRewards`
- `shop`
- `wallet`
- `cloudflare`

El panel vuelve a consultar datos periódicamente y cuando la ventana recupera foco o visibilidad. En el código analizado, el intervalo configurado es de 30 segundos.

## 4. Flujo funcional

```text
Navegador
  │
  ├── Login / Registro / Recuperación
  │       │
  │       └── /api/auth/*
  │
  ├── Sesión
  │       ├── localStorage
  │       ├── X-Wowperu-Session
  │       └── /api/session/me
  │
  └── Panel privado
          │
          ├── /api/player-panel
          │      ├── Cuenta
          │      ├── Personajes
          │      ├── Wallet Perúcoin
          │      ├── Tiempo jugado
          │      ├── Tienda
          │      ├── Referidos
          │      └── Permisos staff
          │
          ├── Servicios de personaje
          ├── Compras / regalos
          ├── Donaciones
          ├── Recompensas por tiempo
          ├── Foro / soporte
          └── Descarga launcher
```

## 5. Servicios de personaje

El frontend unifica múltiples acciones en `POST /api/character-services`, diferenciándolas por `serviceType`. Se observaron flujos para:

- desbloquear;
- revivir;
- teletransportar a ciudad;
- renombrar;
- personalizar;
- cambiar raza;
- cambiar facción;
- boost a nivel 60;
- boost a nivel 70;
- transferir a otra cuenta;
- recuperar personaje;
- recuperar objetos;
- enviar regalos.

Algunas acciones exigen PIN de seguridad de 4 dígitos y otras aplican cooldowns, saldo Perúcoin o requisitos por nivel/clase.

## 6. Tienda y economía

La tienda selecciona un personaje destino y envía compras mediante `POST /api/shop-order`. El frontend genera un `clientRequestId`, usa la moneda `Perúcoin` y envía el item seleccionado con id, nombre, cantidad y precio.

El panel mantiene un wallet web separado y actualiza el saldo visible después de operaciones exitosas.

## 7. Tiempo de juego

El panel muestra estadísticas por personaje y un saldo agregado de tiempo. Los canjes de recompensas pasan por `POST /api/playtime/redeem` y requieren personaje, recompensa, `clientRequestId` y PIN.

El frontend implementa límites por recompensa y un cooldown visible de 7 días entre determinados canjes.

## 8. Donaciones

El frontend contempla varios proveedores/métodos:

- Yape / Plin manual;
- Square;
- Stripe;
- PayPal;
- flujo genérico `/donations`.

Para Square, Stripe y PayPal se crea primero un checkout; el navegador guarda identificadores temporales en `localStorage`; al regresar del proveedor se ejecuta una ruta de confirmación del pago.

Yape / Plin admite carga de comprobante como imagen codificada, nombre del pagador, importe y un identificador idempotente de petición.

## 9. Staff / GM

El mismo frontend contiene interfaces para:

- Tickets GM;
- Gestión de balances y procesos de Perúcoins;
- Revisión de pagos manuales;
- Perúcoins ingame;
- ajustes administrativos;
- retiro de visuales/alas.

Su visibilidad se decide con datos devueltos por el servidor. Esto es correcto como UX, pero AYNI deberá asegurar que **cada endpoint administrativo valide autorización en backend**, independientemente de que el botón esté oculto.

## 10. Capa visual

`redesign.js` declara explícitamente que es aditivo y no modifica la API ni los hooks funcionales. Aporta:

- accesibilidad (`skip link`);
- navegación activa;
- reveal por scroll;
- microinteracciones y spotlight;
- ayuda de transferencia;
- vídeo de portada;
- transiciones de página;
- carruseles;
- efectos de paralaje;
- índice dinámico de reglamento.

Es un patrón útil para AYNI: separar lógica funcional de mejoras visuales reduce el riesgo de que una animación rompa el flujo crítico.
