# Análisis estático del launcher

## Resumen

Se analizó `app.asar` del launcher instalado de WoW Perú. El archivo es un ASAR de Electron y contiene la lógica principal distribuida al usuario final. Este documento resume arquitectura y comportamiento sin copiar código fuente propietario.

## Stack identificado

`package.json` del ASAR declara:

- aplicación: `launcher-wowperu`
- versión distribuida: `1.0.143`
- entrada principal: `electron/main.cjs`
- React `19.2.7`
- React DOM `19.2.7`
- `electron-updater` `6.8.9`
- `yauzl` `3.4.0`
- `lucide-react` como librería de iconos

Arquitectura observable:

`React renderer` → `preload/contextBridge` → `IPC` → `Electron main process` → disco/red/proceso del juego.

## Rutas y endpoints de distribución

Configuración distribuida en el ASAR:

- base de descargas: `https://download.wow-peru.lat`
- manifest normal: `/launcher/manifest.json`
- manifest instalación completa: `/launcher/manifest-full.json`
- manifest reparación: `/launcher/repair-manifest.json`
- manifest HD: `/launcher/manifest-hd.json`
- noticias: `/launcher/news.json`
- addons: `/launcher/addons.json`
- metadata launcher: `/launcher/launcher-update.json`
- feed de auto-update: `/launcher/updates/`
- realmlist: `set realmlist login.wow-peru.lat`
- status TCP: `login.wow-peru.lat:3724`

`app-update.yml` confirma también el feed genérico de actualización bajo `https://download.wow-peru.lat/launcher/updates/`.

## Instalación administrada

La ruta predeterminada del cliente es:

`C:\ProgramData\WoW Peru\Client`

El launcher permite además seleccionar una ruta personalizada o reutilizar un cliente existente. Evita instalar el cliente dentro de la propia carpeta del launcher.

La validación de cliente existente comprueba `Wow.exe` y marcadores clásicos de WoW 3.3.5a, incluyendo MPQ base como `common.MPQ`, `common-2.MPQ`, `expansion.MPQ` y `lichking.MPQ`.

## Sistema de manifests

Cada entrada válida del manifest debe declarar:

- ruta relativa segura;
- URL permitida;
- tamaño no negativo;
- SHA-256 de 64 caracteres hexadecimales.

El launcher bloquea rutas absolutas, letras de unidad y traversal `..`. También limita los dominios de descarga a una allowlist HTTPS.

Hay políticas diferentes para:

1. instalación completa;
2. reparación;
3. paquete HD opcional.

La reparación es deliberadamente restrictiva: evita sustituir indiscriminadamente archivos base críticos del cliente y permite principalmente contenido personalizado autorizado. Esto reduce el impacto de un manifest de reparación defectuoso o manipulado.

## Escaneo y reparación

El launcher compara archivo local contra manifest mediante tamaño y, cuando corresponde, SHA-256.

- En escaneo rápido prioriza tamaño para no hashear gigabytes innecesariamente.
- Configuración/realmlist recibe verificación más estricta.
- Para archivos de hasta 256 MiB, el escaneo normal puede calcular SHA-256.
- Después de una descarga sí verifica SHA-256 antes de aceptar el archivo.
- Si el hash falla, borra el resultado y realiza un segundo intento con cache-busting.

Cuando reemplaza ciertos archivos existentes durante reparación crea respaldo con sufijo `.wowperu.bak`.

El sistema también contempla archivos `retiredFiles`: si un parche fue retirado por el manifest, puede respaldarlo y eliminarlo de forma controlada.

## Motor de descargas

Parámetros predeterminados observados:

- hasta 4 archivos concurrentes;
- reintentos de red: hasta 10;
- timeout de conexión: 45 s;
- progreso emitido aproximadamente cada 250 ms;
- archivos grandes desde 512 MiB pueden usar descarga segmentada;
- segmentos predeterminados de 64 MiB;
- hasta 3 conexiones de segmentos por archivo;
- soporte de HTTP Range y reanudación mediante archivos `.part`.

Si el servidor no soporta Range, el launcher retrocede a descarga lineal.

Las descargas pueden pausarse, reanudarse y cancelarse. El cierre del launcher durante una transferencia solicita confirmación.

## Configuración automática del cliente

Antes de jugar actualiza el `realmlist.wtf` de un locale existente (`esES`, `esMX`, `enUS` o `enGB`; si no existe crea `esES`).

También modifica `WTF/Config.wtf` para dejar configurado el host del reino y valores de aceptación/arranque necesarios. Antes de modificar archivos de configuración crea respaldo cuando existe un archivo previo.

## Parches HD

Existe un manifest separado para contenido HD y la interfaz ofrece instalarlo de manera opcional.

La política HD admite:

- MPQ bajo `Data/`;
- contenido bajo `Interface/`;
- nombres concretos de componentes de motor opcional (`WoW HD.exe`, `d3d9.dll`, `dxvk.conf`).

Sin embargo, la versión analizada elimina esos componentes de motor opcional antes de iniciar el juego y ejecuta `Wow.exe`. Por tanto, el comportamiento distribuido actualmente parece orientado a conservar assets HD compatibles y evitar depender de un ejecutable/motor alternativo en el arranque normal.

## Lanzamiento del juego

Flujo observado:

1. valida ruta y presencia de `Wow.exe`;
2. confirma que parece cliente 3.3.5a;
3. elimina componentes HD de motor opcional si existen;
4. elimina la carpeta `Cache` con comprobaciones contra symlinks/path escape;
5. reescribe realmlist/configuración;
6. intenta registrar preferencia de GPU de alto rendimiento en Windows si el usuario no tiene una preferencia previa;
7. ejecuta `Wow.exe` con `spawn`, sin argumentos adicionales, `cwd` en la carpeta del cliente y proceso desacoplado.

No se observa paso de usuario/contraseña al proceso del juego.

## Addons

El launcher consume un catálogo JSON de addons. Cada addon puede declarar ZIP, tamaño, SHA-256 y carpetas esperadas.

Antes de instalar:

- descarga y verifica el ZIP;
- extrae en carpeta temporal;
- valida rutas del ZIP para impedir traversal;
- busca carpetas de addon válidas;
- copia únicamente a `Interface\AddOns`.

La eliminación también restringe el destino a carpetas de addon declaradas.

## Auto-update del launcher

Usa `electron-updater` con provider `generic`.

Controles visibles:

- no permite downgrade ni reinstalar la misma versión;
- exige metadata con archivos;
- exige SHA-512 del paquete de actualización;
- restringe URLs de actualización a host permitido;
- `autoDownload=false` inicialmente, pero al recibir evento `update-available` valida metadata y dispara descarga;
- no instala automáticamente al cerrar;
- cuando queda descargada, la UI puede pedir reinicio y ejecutar `quitAndInstall`.

## Seguridad del renderer Electron

`BrowserWindow` se crea con:

- `contextIsolation: true`
- `nodeIntegration: false`
- `sandbox: true`
- `webSecurity: true`
- contenido inseguro deshabilitado
- DevTools deshabilitadas en build empaquetado salvo flag explícito

Además:

- bloquea `webview`;
- bloquea permisos del navegador;
- bloquea descargas iniciadas por el renderer;
- abre enlaces externos solo si pertenecen a una allowlist HTTPS;
- rechaza errores de certificado;
- el renderer solo accede a capacidades privilegiadas mediante un API reducido expuesto por `preload.cjs`.

## Diagnóstico local

El launcher mantiene eventos diagnósticos en memoria y en un JSONL bajo `userData/diagnostics/launcher-events.jsonl`.

Antes de registrar datos aplica redacción de tokens, cookies, authorization, password, rutas locales e IPs. También permite copiar un reporte diagnóstico desde la UI.

## Conclusión

La parte del launcher que antes faltaba ya está suficientemente entendida a nivel estático. Es un launcher Electron relativamente completo: manifest-driven, verificación criptográfica, reparación controlada, descargas concurrentes/segmentadas, auto-update, addons, configuración automática del cliente y medidas explícitas de seguridad del proceso Electron.

Lo que aún no puede demostrarse solo con `app.asar` está listado en `04-analisis-pendiente.md`.
