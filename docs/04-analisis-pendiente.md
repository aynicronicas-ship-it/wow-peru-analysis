# Análisis pendiente

La fase de **frontend/panel web** ya está suficientemente cubierta para diseñar una primera arquitectura de AYNI. Lo que falta se concentra en las capas que el navegador no revela por sí solo.

## Prioridad 1 — Launcher

Es la pieza más importante que falta.

Queremos documentar, sin copiar código propietario:

- tecnología/framework del launcher;
- proceso de instalación;
- estructura de archivos;
- mecanismo de autoactualización;
- servidor/CDN de descargas;
- manifiestos de versión;
- hashes/verificación de integridad;
- reparación de archivos;
- configuración de ruta del juego;
- cómo inicia el ejecutable del juego;
- argumentos de lanzamiento;
- manejo de errores;
- almacenamiento de configuración;
- firma digital del ejecutable/instalador.

Artefactos útiles: instalador/launcher descargado legítimamente desde la cuenta de prueba y, si existe, archivos de configuración/manifiesto distribuidos con él.

## Prioridad 2 — Tráfico normal y autorizado del panel

Un HAR **saneado** del navegador permitiría confirmar:

- respuestas reales de `/api/player-panel`;
- formato de `/api/session/me`;
- cabeceras HTTP y caché;
- códigos de estado;
- cookies utilizadas;
- tiempos de respuesta;
- payloads normales de acciones que el usuario esté autorizado a ejecutar.

Antes de compartir un HAR deben eliminarse:

- `Cookie` / `Set-Cookie`;
- `Authorization`;
- `X-Wowperu-Session`;
- contraseñas;
- tokens Turnstile;
- tokens de recuperación;
- datos personales no necesarios.

No necesitamos probar endpoints administrativos con una cuenta sin permisos ni intentar saltar autorización.

## Prioridad 3 — Contrato backend observable

A partir de respuestas legítimas podemos inferir estructuras de datos, pero todavía no sabemos:

- framework/backend usado;
- base de datos;
- cómo se integra con el servidor del juego;
- colas/jobs para entrega de items;
- transacciones de wallet;
- estrategia de locking/concurrencia;
- validación backend de roles;
- almacenamiento de comprobantes;
- sistema de email.

Estas piezas solo deben documentarse si aparecen en material distribuido públicamente o facilitado con autorización.

## Prioridad 4 — Flujo launcher ↔ juego ↔ servidor

Queremos conocer a nivel de arquitectura:

1. cómo obtiene versiones el launcher;
2. cómo descarga/parchea;
3. cómo valida archivos;
4. cómo configura el cliente;
5. cómo lanza el juego;
6. qué endpoint/host de login utiliza el cliente;
7. qué ocurre cuando hay mantenimiento o versión incompatible.

Para AYNI, este bloque se traducirá después a nuestro propio `Release Service + CDN + Launcher + Game Gateway`.

## Prioridad 5 — Observabilidad y operación

Si el launcher o la web exponen material suficiente, revisar:

- logs de actualización;
- crash reporting;
- telemetría;
- status del reino;
- mantenimiento programado;
- rollback/versiones;
- soporte y diagnóstico del cliente.

## Qué NO falta para empezar AYNI

No necesitamos conocer el código fuente privado de WoW Perú ni su base de datos para avanzar. Con la fase web ya podemos definir para AYNI:

- portal de cuenta;
- modelo de sesión;
- panel del jugador;
- servicios de personaje;
- wallet/tienda;
- historial y auditoría;
- roles de staff;
- interfaz de descarga.

El próximo análisis con mayor retorno es el **launcher**, no seguir profundizando indefinidamente en el HTML.
