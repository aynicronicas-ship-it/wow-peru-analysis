# Lecciones aplicables a AYNI

Este documento separa **patrones útiles** de decisiones que AYNI debería mejorar o evitar. La meta es diseñar una arquitectura propia, no reproducir código, nombres, assets ni UX de WoW Perú.

## Conviene adoptar como patrón

### 1. Portal de cuenta separado del cliente del juego

La web puede encargarse de:

- registro e inicio de sesión;
- seguridad de cuenta;
- personajes y servicios fuera del juego;
- historial;
- soporte;
- pagos/tienda;
- descarga del launcher.

Esto mantiene el cliente de juego enfocado en el mundo, combate, chat y gameplay.

### 2. Un endpoint agregado para el panel

Un endpoint tipo `player-panel` reduce viajes de red al cargar la cuenta. Para AYNI puede existir un BFF/API Gateway que agregue:

- perfil;
- personajes;
- monedas;
- eventos/notificaciones;
- inventario de servicios web;
- estado del mundo/servidores.

### 3. Idempotencia en operaciones económicas

WoW Perú genera identificadores únicos de petición en compras/canjes. AYNI debería formalizar este patrón con una **Idempotency-Key** o identificador transaccional del lado servidor para compras, regalos, transferencias y recompensas.

### 4. Separar capa funcional y capa visual

La idea de que las animaciones y microinteracciones sean progresivas y no rompan formularios ni API es buena. Para AYNI:

- funcionalidad base primero;
- animaciones como mejora;
- soporte a `prefers-reduced-motion`;
- UI operable aunque fallen recursos secundarios.

### 5. Historial y auditoría

Para moneda premium, staff y servicios de personajes, AYNI debería conservar un ledger/auditoría inmutable con:

- actor;
- acción;
- saldo antes/después;
- personaje/cuenta;
- motivo;
- timestamp;
- request id;
- origen web/juego/admin.

### 6. Roles de staff definidos por backend

La UI puede ocultar secciones por rango, pero el backend siempre debe volver a comprobar el permiso en cada operación. AYNI debería usar RBAC/ABAC explícito y no confiar en botones ocultos.

## Conviene mejorar para AYNI

### 1. Evitar tokens sensibles persistentes en `localStorage`

El frontend observado conserva su token en `localStorage`. Para AYNI preferiríamos, cuando la arquitectura lo permita:

- cookie `HttpOnly`;
- `Secure`;
- `SameSite` apropiado;
- rotación de sesión;
- expiración corta y refresh controlado;
- protección CSRF según el diseño.

Esto reduce exposición del token frente a XSS.

### 2. No confiar en `account=<nombre>` enviado por el cliente

Aunque sea útil como hint de UI, el servidor de AYNI debe resolver la cuenta desde la identidad autenticada y no autorizar datos por un nombre de cuenta controlado por el cliente.

### 3. Separar servicios administrativos del bundle normal

En el frontend observado existen vistas administrativas ocultas dentro del HTML/bundle general. Para AYNI sería más limpio:

- panel administrativo separado;
- bundle/ruta separados;
- autorización backend estricta;
- observabilidad y auditoría separadas.

Esto no sustituye la seguridad backend, pero reduce superficie y exposición accidental.

### 4. API tipada y contratos versionados

El frontend observado usa JavaScript dinámico. AYNI, con su stack propio, debería usar contratos tipados, por ejemplo:

- TypeScript + esquema OpenAPI/JSON Schema;
- validación servidor y cliente;
- errores con códigos estables;
- versionado de endpoints críticos.

### 5. Actualizaciones en tiempo real donde aporten valor

El panel observado refresca cada ~30 s. En AYNI podríamos usar:

- polling para datos poco urgentes;
- WebSocket/SSE para cola del servidor, presencia, mantenimiento o notificaciones;
- backoff y límites para no sobrecargar backend.

## Arquitectura objetivo sugerida para AYNI

```text
Web pública / Cuenta
        │
        ▼
API Gateway / BFF
        │
        ├── Identity/Auth service
        ├── Account service
        ├── Character service
        ├── Commerce/Wallet service
        ├── Support service
        └── Launcher/Release service
                 │
                 ▼
           Launcher AYNI
                 │
      ┌──────────┴──────────┐
      ▼                     ▼
Release/CDN            Login/Game Gateway
                              │
                              ▼
                         Mundo AYNI
```

## Principio clave

No necesitamos copiar cómo ellos lo hicieron internamente. Lo valioso del análisis es identificar **qué problemas resuelven**: cuenta, acceso, launcher, actualización, conexión, economía, soporte y operaciones del personaje. AYNI puede resolver esos mismos problemas con una arquitectura más moderna y diseñada desde el inicio para su propio MMORPG.
