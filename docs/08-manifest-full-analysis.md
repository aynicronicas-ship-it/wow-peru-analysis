# Análisis de `manifest-full.json`

## Resumen

Se analizó el manifest completo distribuido por el launcher de WoW Perú.

Metadatos observados:

- `version`: `20260903-alas-3dias`
- `gameVersion`: `3.3.5a`
- `realmList`: `set realmlist login.wow-peru.lat`
- `generatedAt`: `2026-09-03T07:36:54.7490333Z`
- archivos activos: **217**
- archivos retirados declarados: **3**
- tamaño total declarado: **17,426,326,454 bytes** ≈ **17.426 GB** ≈ **16.229 GiB**

El total explica prácticamente de forma exacta el texto de ~`16.2 GB` mostrado por el launcher: la UI está expresando el volumen en GiB y redondeándolo.

## Estructura por área

| Área | Archivos | Tamaño aprox. | Proporción |
|---|---:|---:|---:|
| `Data/` | 25 | 16.083 GiB | 99.10% |
| `Interface/` | 182 | 0.122 GiB | 0.75% |
| raíz del cliente | 10 | 0.025 GiB | 0.15% |

La descarga está dominada totalmente por los archivos MPQ de `Data/`. La gran cantidad de archivos de `Interface/` apenas afecta al tamaño total.

## Archivos más pesados

1. `Data/patch.MPQ` — 3.730 GiB
2. `Data/common.MPQ` — 2.687 GiB
3. `Data/lichking.MPQ` — 2.404 GiB
4. `Data/expansion.MPQ` — 1.791 GiB
5. `Data/common-2.MPQ` — 1.690 GiB
6. `Data/patch-2.MPQ` — 1.307 GiB
7. `Data/patch-3.MPQ` — 0.564 GiB
8. `Data/esES/patch-esES.MPQ` — 0.530 GiB
9. `Data/esES/speech-esES.MPQ` — 0.376 GiB
10. `Data/esES/expansion-speech-esES.MPQ` — 0.222 GiB

Hay **8 archivos de 500 MB o más** y **6 archivos de 1 GB o más**. Esto explica por qué el sistema de descarga segmentada/reanudable observado dentro del launcher tiene tanta importancia práctica.

## Contrato de integridad

Cada entrada del manifest especifica principalmente:

```text
path
size
sha256
url
label (en la mayoría de entradas)
```

El patrón permite al launcher:

1. resolver el destino relativo dentro del cliente;
2. conocer el tamaño esperado;
3. descargar el archivo desde una URL concreta;
4. validar el resultado mediante SHA-256;
5. decidir si un archivo local ya coincide con la versión publicada.

No se observa en este manifest un sistema de chunks global con hashes por bloque: la unidad lógica publicada es el **archivo**. El launcher puede segmentar la transferencia de archivos grandes, pero la verificación final sigue asociada al SHA-256 del archivo completo.

## Distribución

Todos los archivos observados se publican bajo:

`https://download.wow-peru.lat/client/`

El manifest funciona así como catálogo y tabla de direccionamiento para el CDN/servidor de archivos. No contiene el binario en sí; contiene metadatos y URLs.

## Archivos retirados

El manifest declara tres rutas retiradas:

- `Data/patch-pe.MPQ`
- `Data/patch-zzzzzzzzzzzz-wowperu-turkeymount-20260717.MPQ`
- `Data/patch-zzzzzzzzzzzzzz-wowperu-turkeymount-full-20260717.MPQ`

Las tres llevan como razón una sustitución por el refresh `20260719-wowpe-data-refresh`.

Esto demuestra un mecanismo explícito de **tombstones/retired files**: una actualización no solo agrega o reemplaza archivos; también puede indicar al launcher que ciertos archivos antiguos ya no deben permanecer instalados.

Para AYNI conviene conservar esta idea, pero formalizarla como una operación de manifest (`delete`) versionada y auditable.

## Personalización observable

El manifest deja ver varias piezas claramente asociadas al servidor, sin necesidad de inferir contenido interno de MPQ:

- `Data/patch-Z-WOWPERU.MPQ` (~14.1 MiB), archivo con branding explícito.
- `Interface/AddOns/WowPeruVisualShop/`, con catálogo, UI y recursos de alas.
- múltiples iconos `wingXX_*.blp` usados por la tienda visual.
- archivos de `Interface/GlueXML/` para la pantalla/flujo de login.
- una amplia colección de `Interface/GLUES/LOADINGSCREENS/` distribuida fuera de MPQ.

El directorio `WowPeruVisualShop` contiene **20 archivos** y, junto con `patch-Z-WOWPERU.MPQ`, las rutas explícitamente marcadas con `WoWPeru` suman aproximadamente **44.3 MiB**. Esta cifra NO representa toda la personalización real, porque un archivo de nombre genérico como `patch.MPQ` puede contener contenido modificado y el manifest no permite determinarlo sin comparar contra una referencia limpia.

## Tipos de archivo

Conteos observados:

- `.blp`: 141
- `.mpq`: 24
- `.pub`: 23
- `.lua`: 13
- `.dll`: 7
- `.toc`: 2
- `.xml`: 2
- `.exe`: 2
- `.wtf`: 1
- `.txt`: 1
- `.manifest`: 1

Por tamaño, prácticamente todo el cliente reside en los 24 MPQ: ~16.083 GiB.

## Ejecutables y runtime

El manifest también distribuye la raíz ejecutable/runtime del cliente, incluyendo:

- `Wow.exe`
- `WowError.exe`
- `Battle.net.dll`
- `dbghelp.dll`
- `DivxDecoder.dll`
- `ijl15.dll`
- `msvcr80.dll`
- `Scan.dll`
- `unicows.dll`
- `Microsoft.VC80.CRT.manifest`

Esto confirma que la instalación nueva no presupone una copia previa del juego: el manifest describe un cliente completo y arrancable.

## Qué aprendemos para AYNI

### Mantener

- manifest declarativo y versionado;
- tamaño + hash criptográfico + URL por artefacto;
- lista explícita de archivos retirados;
- separar contenido opcional del cliente base;
- descarga reanudable para paquetes grandes;
- verificación antes de marcar una versión como lista.

### Mejorar

Para un juego nuevo como AYNI no conviene repetir literalmente un catálogo de grandes contenedores monolíticos. Conviene diseñar el sistema desde el principio para parches incrementales:

```text
Release
 ├─ manifest firmado
 ├─ paquetes/chunks direccionados por hash
 ├─ grupos base
 ├─ grupos opcionales (HD, idioma, audio)
 ├─ tombstones
 └─ rollback metadata
```

Idealmente el launcher AYNI debería poder reutilizar chunks sin volver a bajar un archivo entero cuando cambien pocos datos dentro de un paquete grande.

También conviene firmar criptográficamente el manifest, no depender únicamente del HTTPS y de hashes declarados por el propio manifest. El launcher debe validar la firma del manifest antes de confiar en URLs, paths, tamaños y hashes.

## Conclusión

`manifest-full.json` confirma la arquitectura observada dentro de `app.asar`: el launcher es un gestor de instalación/actualización basado en manifest. El cliente estándar mostrado como **16.2 GB** corresponde a aproximadamente **16.229 GiB** de archivos declarados.

La siguiente evidencia con mayor valor es comparar:

1. `repair-manifest.json` contra este manifest completo;
2. `manifest-hd.json` para conocer exactamente qué agrega la opción HD;
3. `addons.json` para separar los addons opcionales del contenido base.

No hace falta descargar los 16.2 GB del cliente para completar esas tres comparaciones; bastan esos manifests JSON.
