# Mapas de elecciones

Mapas (Highcharts Maps y GeoJSON) de las páginas de resultados de `sfEleccionesPlugin`.

## Publicación

Se sirven desde `https://uh.gsstatic.es/js/elecciones/` (S3 + CloudFront). La carpeta está en
`config/rsync_exclude.txt` del proyecto: **no se sube con rsync**, se sube el fichero a S3 y se
invalida su ruta en CloudFront.

`SECC_CE_20230101.json` (250 MB) está en `.gitignore` y no se publica.

## Cómo se usan

Cada elección indica su mapa en `elecciones_eleccion` (y, por edición, en
`elecciones_eleccion_circunscripcion`):

- `mapa`: clave con la que el fichero se registra en `Highcharts.maps[...]`.
- `url_mapa`: URL del fichero en uh.gsstatic.es.
- `peer_method_mapa`: método de `EleccionesCircunscripcionPeer` que da las circunscripciones a pintar.

**Cada zona se enlaza con su resultado por el nombre** (`properties.name` = `denominacion` de la
circunscripción). Si no coinciden exactamente (p. ej. «Ibiza» frente a «Eivissa»), la zona sale
en blanco sin dar error.

Los mapas por isla de la carga del backend están fijados en
`plugins/sfEleccionesPlugin/modules/elecciones_eleccion/actions/actions.class.php` (~línea 1210).

## Ficheros

| Fichero | Clave `Highcharts.maps` | Zonas | Contenido | Lo usan |
|---|---|---|---|---|
| `es-all.js` | `countries/es/es-all` | 53 | España por provincias (coordenadas proyectadas, EPSG:2062 + recuadro de Canarias) | Congreso, Europeas y Senado hasta 10N |
| `es-all-ci.js` | `countries/es/es-all-ci` | 60 | España por circunscripciones del Senado: provincias peninsulares + islas, con «Ibiza y Formentera» juntas | Senado 23J (55) |
| `es-all-ci-2026.js` | `countries/es/es-all-ci-2026` | 61 | Igual que `es-all-ci.js` con «Eivissa» y «Formentera» separadas (reforma de 2026: Formentera elige senador propio). Partido de la geometría de `es-all-ci.js`, sin reproyectar | Senado 29N (66) |
| `es-ca.js` | `countries/es/es-ca` | 4 | Cataluña por provincias | Catalanas 14F y 12M (56, 57) |
| `es-ib.js` | `countries/es/es-all` (!) | 1 | Illes Balears en un único polígono. **Se registra con la misma clave que `es-all.js`**: no cargar los dos en la misma página | — |
| `baleares.geojson` | `countries/es/ib-all` | 67 | Baleares por municipios (lon/lat, `name` + `CODIGOINE`) | Autonómicas, consells y municipales |
| `mallorca.geojson` | `countries/es/ib-mallorca` | 53 | Municipios de Mallorca | Autonómicas 2019 (23) y carga del backend |
| `menorca.geojson` | `countries/es/ib-menorca` | 8 | Municipios de Menorca | Carga del backend |
| `eivissa.geojson` | `countries/es/ib-eivissa` | 6 | Municipios de Eivissa y Formentera | Carga del backend |
| `palma.json` | — | 253 | Secciones censales de Palma (INE 2023: `CUSEC`, `CDIS`, `CSEC`…) | Sin uso todavía (mapa por barrios) |
| `palma-low.json` | — | 253 | Lo mismo, simplificado | Sin uso todavía |
| `baleares-mesas.json` | — | 670 | Secciones censales de Baleares (INE 2023) | Sin uso todavía |
| `catalunya_comarcas.geojson` | `countries/es/ca-comarcas` | 41 | Comarcas de Cataluña (`nom_comar`, `comarca`, `provincia`) | Sin uso en BD |
| `comarques-barcelona.geojson` | — | 13 | Comarcas de la provincia de Barcelona | Sin uso en BD |
| `SECC_CE_20230101.json` | — | — | Secciones censales de toda España (INE, 1-1-2023). Origen de `palma*.json` y `baleares-mesas.json` | No se versiona ni se publica |

## Fuentes

- Mapas `es-*.js`: Highcharts Map Collection (`countries/es/...`), adaptados.
- Comarcas de Cataluña:
  - https://github.com/aariste/GeoJSON-Mapas
  - https://vangdata.carto.com/tables/shapefiles_catalunya_comarcas/public/map
- Secciones censales: INE, «Cartografía digitalizada de secciones censales» a 1-1-2023
  (fichero `SECC_CE_20230101`).
- Municipios de Baleares: límites del IGN/CNIG.

## Para regenerar o ampliar

- Límites oficiales (municipios, provincias, comunidades): Centro de Descargas del CNIG, «Líneas
  límite municipales» (CC BY 4.0, citar IGN). Ya convertidos a TopoJSON con códigos INE:
  https://github.com/martgnz/es-atlas (`https://unpkg.com/es-atlas/es/provinces.json`, …).
- Recortar y simplificar con mapshaper, p. ej. una comunidad con sus provincias:
  `mapshaper provinces.json -filter 'id.slice(0,2)=="04"' -simplify 10% -o format=geojson`.
- Islas: no hay capa oficial por isla; se obtienen uniendo los municipios de cada isla
  (`-dissolve`). En la BD cada municipio cuelga (`parent`) de su circunscripción insular.
- Mantener las zonas con `name` igual a la `denominacion` de la circunscripción, o el mapa no
  enlaza con los resultados.

Análisis completo y mejoras pendientes: `docs/elecciones-mapas-analisis.md` del proyecto.
