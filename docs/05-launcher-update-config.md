# Configuración de actualización del launcher

Fuente analizada: `app-update.yml` del launcher instalado.

## Hallazgo principal

El launcher usa un proveedor de actualización `generic` y apunta a:

`https://download.wow-peru.lat/launcher/updates/`

También define el directorio de caché del updater como:

`launcher-wowperu-updater`

## Interpretación

Esto es consistente con un launcher Electron empaquetado con `electron-builder`/`electron-updater` o una solución compatible con su formato de configuración. El archivo por sí solo no demuestra el mecanismo exacto de actualización ni los nombres de manifest que consume, pero sí confirma que la distribución de actualizaciones del launcher está desacoplada de la web principal y servida desde el subdominio `download.wow-peru.lat`.

## Qué conviene verificar dentro de `app.asar`

- Dependencia y uso real de `electron-updater`.
- Evento de comprobación de actualizaciones y frecuencia.
- Si el launcher usa `latest.yml`, `latest.yml` equivalente o un manifest propio.
- Firma/verificación de binarios.
- Manejo de descargas parciales, errores y rollback.
- Carpeta de caché y limpieza de versiones anteriores.
- Diferencia entre actualización del launcher y actualización/instalación del cliente WoW.

## Lección para AYNI

Conviene separar claramente dos canales:

1. **Updater del launcher**: binario pequeño, versionado y firmado.
2. **Updater/instalador del juego**: cliente grande, manifest propio, descarga por bloques/archivos, hashes y reparación.

No debe asumirse que el mismo mecanismo de `electron-updater` es adecuado para distribuir el juego completo.
