# Clasificador de botellas

Aplicación web local para clasificar botellas en seis estados mediante la cámara del equipo o una imagen subida. El modelo fue entrenado en Teachable Machine y se ejecuta completamente en el navegador con TensorFlow.js: no hay backend, no se suben imágenes a ningún servidor y no se guarda ningún dato.

La interfaz es una app de una sola pantalla, con tema oscuro, cámara a la izquierda y panel de predicción a la derecha. Está pensada para escritorio y para que el contenido principal entre sin scroll en ventanas de altura normal.

## Ficha rápida

| Campo | Valor |
| --- | --- |
| Tipo | Aplicación web estática, sin build |
| Archivo principal | `index.html` |
| Modelo | Teachable Machine, imagen estándar, MobileNetV1 |
| Entrada del modelo | 224 × 224 × 3 |
| Clases | 6 |
| Captura de cámara | 480 × 480, espejo |
| Inferencia | En el navegador, cada ~90 ms en cámara |
| Backend | Ninguno |
| Persistencia | Ninguna |
| Servidor recomendado | `python3 -m http.server` |
| URL recomendada | `http://localhost:8000/` |

## Vista general

```text
┌─────────────────────────────────────────────────────────────────────┐
│ ┌───────────────────────────────────┐ ┌───────────────────────────┐ │
│ │ CÁMARA / IMAGEN                   │ │ PREDICCIÓN ACTUAL         │ │
│ │                                   │ │ Clase           Confianza │ │
│ │         video o imagen            │ │ Etiqueta normal      97 % │ │
│ │                                   │ │ [barra de confianza]      │ │
│ │                                   │ ├───────────────────────────┤ │
│ │ [Iniciar] [Detener] [Subir]       │ │ Todas  Buenas  Defectos   │ │
│ └───────────────────────────────────┘ ├───────────────────────────┤ │
│                                       │ Etiqueta normal   0 % ▓▓▓ │ │
│                                       │ Etiqueta caida    0 % ▓▓▓ │ │
│                                       │ ...                       │ │
│                                       └───────────────────────────┘ │
│ Modelo de Teachable Machine                                         │
└─────────────────────────────────────────────────────────────────────┘
```

En pantallas de hasta 900 px de ancho, las dos columnas se apilan: primero la cámara y después la predicción, los filtros y la lista de clases.

## Características

- Clasificación en vivo desde la cámara, con vista espejo y actualización periódica.
- Clasificación de una imagen local mediante el botón "Subir imagen".
- Precarga silenciosa del modelo al abrir la página para reducir la espera.
- Tarjeta de predicción con clase ganadora, confianza y barra de progreso.
- Lista de las seis clases con porcentaje y medidor de puntos.
- Filtros por categoría: Todas, Buenas, Defectos y Faltantes.
- Resaltado de la clase ganadora en la lista.
- Mensajes de error en español, con casos específicos para cámara y WebRTC.
- Procesamiento local: el video y las imágenes no salen del navegador.
- Diseño responsive y soporte para `prefers-reduced-motion`.
- Sin dependencias de build: HTML, CSS y JavaScript inline, más el modelo local.

## Clases del modelo

El orden de esta tabla es el orden real de `metadata.json` y del arreglo `CLASSES` de `index.html`. Es importante mantenerlo sincronizado.

| # | Etiqueta del modelo | Categoría | Texto de categoría en la fila |
| --- | --- | --- | --- |
| 1 | `Etiqueta normal` | `good` | Buen estado |
| 2 | `Etiqueta caida` | `defect` | Defecto |
| 3 | `Sin etiqueta` | `missing` | Faltante |
| 4 | `Sin tapa` | `missing` | Faltante |
| 5 | `Botella golpeada` | `defect` | Defecto |
| 6 | `Botella aplastada` | `defect` | Defecto |

Resumen de los filtros:

| Filtro | Clases incluidas | Cantidad |
| --- | --- | --- |
| Todas | Las seis | 6 |
| Buenas | `Etiqueta normal` | 1 |
| Defectos | `Etiqueta caida`, `Botella golpeada`, `Botella aplastada` | 3 |
| Faltantes | `Sin etiqueta`, `Sin tapa` | 2 |

Notas sobre las etiquetas:

- `Etiqueta caida` está escrita sin tilde porque así aparece en el metadata del modelo. Si se corrige a "Etiqueta caída", hay que cambiarla en `metadata.json` y en `CLASSES` al mismo tiempo.
- Las categorías (`good`, `defect`, `missing`) son una agrupación de la interfaz, no una salida del modelo. El modelo solo devuelve las seis etiquetas.
- La interpretación esperada es: etiqueta en buen estado, etiqueta despegándose, botella sin etiqueta, botella sin tapa, botella con golpes y botella aplastada. El modelo no incluye una clase de fondo ni de objeto desconocido.

## Requisitos

- Un navegador moderno con WebRTC y `navigator.mediaDevices.getUserMedia`.
- Chrome o Edge son la opción recomendada. Firefox y Safari modernos también deberían funcionar.
- Python 3 para el servidor local de desarrollo, o cualquier servidor estático equivalente.
- Conexión a internet para cargar TensorFlow.js, Teachable Machine y la fuente Manrope desde CDN.
- Cámara y permisos del navegador si se va a usar la clasificación en vivo.
- El modelo local en la carpeta `my_model/`.

No se necesita Node.js, npm, bundler ni compilación.

## Cómo ejecutar

1. Abrir una terminal en la carpeta del proyecto:

```bash
cd "/mnt/hdd/Escritorio/modelo ia"
```

2. Iniciar un servidor estático en el puerto 8000:

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

3. Abrir la aplicación en el navegador:

```text
http://localhost:8000/
```

Si `localhost` no responde, probar con la dirección IPv4:

```text
http://127.0.0.1:8000/
```

4. Pulsar "Iniciar cámara" y permitir el acceso cuando el navegador lo solicite.

5. Para detener el servidor, volver a la terminal y pulsar `Ctrl+C`.

Advertencias importantes:

- No abrir `index.html` con doble clic. En `file://` fallan la carga del modelo por `fetch` y el acceso a la cámara.
- No abrir la página desde una IP de red local por HTTP sin HTTPS. Los navegadores bloquean la cámara en contextos no seguros. `localhost` y `127.0.0.1` sí se consideran contextos seguros.
- Si el puerto 8000 está ocupado, usar otro, por ejemplo `8001`, y abrir `http://localhost:8001/`. El mensaje de error interno de la app menciona 8000; se puede ajustar si se cambia el puerto de forma permanente.
- Si hay un proxy configurado, las peticiones a `localhost` pueden fallar. En la terminal se puede comprobar con `curl --noproxy '*'`.

Comprobar que el servidor está sirviendo los archivos:

```bash
curl --noproxy '*' -I http://127.0.0.1:8000/index.html
curl --noproxy '*' -I http://127.0.0.1:8000/my_model/model.json
curl --noproxy '*' -I http://127.0.0.1:8000/my_model/metadata.json
curl --noproxy '*' -I http://127.0.0.1:8000/my_model/weights.bin
```

Las cuatro respuestas deben ser `200`.

## Uso

### Cámara en vivo

1. Pulsar "Iniciar cámara".
2. Aceptar el permiso de cámara.
3. Colocar una botella centrada y con buena iluminación.
4. Leer la clase y la confianza en la tarjeta de la derecha.
5. Usar "Detener" para apagar la cámara; la vista vuelve al estado inicial y las predicciones se reinician.

La cámara captura un cuadrado de 480 × 480 y lo muestra con `object-fit: contain`. Si la ventana no es cuadrada, aparecen barras arriba y abajo (o a los lados) en lugar de recortar la imagen. El chip "224 × 224" indica el tamaño de entrada del modelo, no la resolución de captura.

### Subir una imagen

1. Pulsar "Subir imagen".
2. Elegir un archivo de imagen.
3. La app detiene la cámara, muestra la vista previa y ejecuta una única predicción.
4. El chip de la cámara cambia a "Imagen" y el estado a "Imagen analizada".
5. Para volver a la cámara, pulsar "Iniciar cámara".

La vista previa usa `object-fit: contain`, así que muestra la imagen completa. Sin embargo, Teachable Machine recorta el centro al cuadrado antes de redimensionar a 224 × 224. Para imágenes no cuadradas, el modelo no ve los bordes laterales; conviene encuadrar la botella en el centro.

### Filtros y lista

- "Todas" muestra las seis clases.
- "Buenas" muestra solo `Etiqueta normal`.
- "Defectos" muestra `Etiqueta caida`, `Botella golpeada` y `Botella aplastada`.
- "Faltantes" muestra `Sin etiqueta` y `Sin tapa`.
- El filtro solo oculta filas de la lista; la tarjeta de predicción sigue mostrando la clase ganadora aunque su fila esté oculta.
- La fila de la clase ganadora se resalta con un borde color melocotón y el porcentaje en el mismo color.

### Cómo interpretar la confianza

- La confianza es la probabilidad softmax de la clase ganadora, redondeada al porcentaje entero.
- Las probabilidades de las seis clases suman aproximadamente 1. Los porcentajes redondeados pueden no sumar exactamente 100.
- No hay umbral de confianza ni clase "desconocido". El modelo siempre elige una de las seis etiquetas, incluso ante un objeto que no sea una botella.
- Para una lectura más prudente, comparar la clase ganadora con la segunda y repetir la captura con mejor luz y encuadre.

## Cómo funciona

### Flujo general

```text
Carga de la página
  ├─ Construir tabs y filas desde CLASSES / CATEGORIES
  ├─ Reiniciar predicciones y estado
  └─ Precargar el modelo en segundo plano (ensureModel(true))

Iniciar cámara
  ├─ Comprobar getUserMedia
  ├─ Asegurar el modelo
  ├─ Crear tmImage.Webcam(480, 480, true)
  ├─ setup() + play()
  ├─ Insertar el canvas en la vista
  └─ Arrancar requestAnimationFrame(loop)

loop()
  ├─ webcam.update()
  ├─ Cada ~90 ms: model.predict(webcam.canvas)
  └─ updatePredictions()

Subir imagen
  ├─ Detener la cámara
  ├─ Crear object URL y mostrar la vista previa
  ├─ Esperar decode() (o load como fallback)
  ├─ model.predict(<img>)
  ├─ updatePredictions()
  └─ Revocar el object URL
```

### Constantes y estado

| Elemento | Función |
| --- | --- |
| `MODEL_URL` | Ruta base del modelo: `./my_model/` |
| `CLASSES` | Etiquetas, categorías y textos de las filas |
| `CATEGORIES` | Definición de los tabs y sus identificadores |
| `els` | Referencias a los elementos del DOM |
| `model` | Modelo cargado; se conserva entre usos |
| `modelPromise` | Evita cargas duplicadas y permite reintentar tras un error |
| `webcam` | Instancia de `tmImage.Webcam` |
| `running` | Indica si el loop de cámara está activo |
| `rafId` | Identificador de `requestAnimationFrame` para cancelar |
| `lastPredict` | Marca de tiempo de la última predicción |
| `activeFilter` | Filtro actual; por defecto `all` |
| `rowEls` | Mapa de nombre de clase a fila, porcentaje y medidor |

`MODEL_URL` se llama así a propósito: usar `const URL` sobrescribiría el constructor global `URL`, que se necesita para `URL.createObjectURL` y `URL.revokeObjectURL` en la subida de imágenes.

### Funciones principales

| Función | Responsabilidad |
| --- | --- |
| `errorText(error)` | Extrae un texto legible de un error |
| `setStatus(state, text, predText)` | Actualiza el estado de la tarjeta de predicción |
| `hideError()` | Oculta y limpia el banner de error |
| `showError(error)` | Registra el error, muestra el banner y marca el estado como error |
| `buildTabs()` | Crea los tabs y calcula sus cantidades |
| `buildRows()` | Crea las seis filas de clase y el mapa `rowEls` |
| `setFilter(id)` | Activa un tab y oculta o muestra filas |
| `resetPredictions()` | Vuelve la tarjeta y las filas a cero |
| `updatePredictions(prediction)` | Actualiza la tarjeta, la barra y todas las filas |
| `ensureModel(silent)` | Carga el modelo una vez y reutiliza la promesa |
| `cameraSupported()` | Comprueba `navigator.mediaDevices.getUserMedia` |
| `startCamera()` | Preflight, carga del modelo, creación de la webcam y arranque del loop |
| `loop(timestamp)` | Actualiza la webcam y predice cada ~90 ms |
| `stopCamera(showPlaceholder)` | Detiene la webcam, cancela el loop y restaura la UI |
| `handleFile(file)` | Valida la imagen, muestra la vista previa y predice |

`setStatus` conserva los parámetros `state` y `text` por compatibilidad, pero en la versión actual solo escribe `predText || text` en `els.predState`. El chip "En vivo" de la cámara se controla por separado.

### Predicción

`model.predict()` devuelve un arreglo de objetos con esta forma:

```js
[
  { className: "Etiqueta normal", probability: 0.97 },
  { className: "Etiqueta caida", probability: 0.01 }
]
```

`updatePredictions()` recorre el arreglo, busca la fila correspondiente por nombre y, si no la encuentra, usa la posición como respaldo. Luego:

- Actualiza el porcentaje de cada fila.
- Actualiza el medidor de puntos con la variable CSS `--p`.
- Quita el resaltado anterior y marca la fila ganadora.
- Escribe la clase y la confianza en la tarjeta.
- Ajusta el ancho de la barra de confianza.

La app no aplica suavizado temporal ni promedio de varios cuadros. La predicción cambia con cada captura; en condiciones de poca luz o movimiento puede oscilar.

### Cámara y recorte

- La cámara se crea con `new tmImage.Webcam(480, 480, true)`. El tercer argumento activa el espejo horizontal.
- El canvas de captura es cuadrado, por lo que el modelo no recorta los lados de la cámara.
- La vista usa `object-fit: contain` para mostrar el cuadrado completo, con fondo oscuro en las barras.
- Las imágenes subidas se pasan directamente a `model.predict()`. Teachable Machine recorta el centro al cuadrado y redimensiona a `imageSize` (224 × 224 en este modelo).
- Si el modelo rinde peor en cámara que con imágenes, probar `flip = false` para que la cámara no esté espejada. El entrenamiento pudo haberse hecho sin espejo.

### Errores y estados

La app usa un banner superior con `role="alert"` y el estado de la tarjeta de predicción. Estados visibles:

| Estado | Cuándo aparece | Chip de cámara |
| --- | --- | --- |
| `Sin señal` | Inicio o cámara detenida | Oculto |
| `Cargando` | Cargando modelo o activando cámara | Oculto |
| `En vivo` | Cámara activa | "En vivo" |
| `Analizando` | Procesando una imagen subida | "Imagen" |
| `Imagen` | Imagen analizada | "Imagen" |
| `Error` | Fallo de cámara, modelo o imagen | Según el flujo |

El error de WebRTC se traduce a un mensaje que recuerda abrir la página en `http://localhost` y revisar los permisos. Los errores de modelo o de red se muestran con el texto original del navegador y también se registran en la consola.

### Precarga del modelo

Al final del script se ejecuta `ensureModel(true).catch(...)`. Esto descarga el modelo en segundo plano sin cambiar la interfaz. Si falla, solo se registra una advertencia; al pulsar "Iniciar cámara" o "Subir imagen" se vuelve a intentar. El modelo se conserva en memoria después de detener la cámara.

## Modelo

### Metadata

| Campo | Valor |
| --- | --- |
| `modelName` | `tm-my-image-model` |
| `tfjsVersion` | `1.7.4` |
| `tmVersion` | `2.4.19` |
| `packageName` | `@teachablemachine/image` |
| `packageVersion` | `0.8.4-alpha2` |
| `imageSize` | `224` |
| `timeStamp` | `2026-11-23T01:09:16.809Z` |
| `labels` | `Etiqueta normal`, `Etiqueta caida`, `Sin etiqueta`, `Sin tapa`, `Botella golpeada`, `Botella aplastada` |

### Arquitectura

| Parte | Detalle |
| --- | --- |
| Base convolucional | MobileNetV1 (`model3`), 155 capas internas |
| Pooling | `GlobalAveragePooling2D` |
| Clasificador | `Dense(100, relu)` → `Dense(6, softmax)` |
| Entrada | `(null, 224, 224, 3)`; batch dinámico |
| Salida | 6 probabilidades |
| Pesos | 263 tensores en un único `weights.bin` |
| Versión de Keras/TFJS | `tfjs-layers 1.7.4` |

El modelo es un clasificador de imagen estándar de Teachable Machine. No es un detector de objetos: no devuelve cajas ni varias botellas por imagen. Asume que el objeto de interés está centrado y ocupa una parte importante del encuadre.

No se incluye el dataset de entrenamiento ni el script de entrenamiento. El repositorio contiene el modelo exportado, la app y este README.

### Archivos del modelo

| Archivo | Tamaño aproximado | Contenido |
| --- | --- | --- |
| `model.json` | 91,818 bytes | Topología y manifiesto de pesos |
| `metadata.json` | 332 bytes | Etiquetas, tamaño de entrada y versiones |
| `weights.bin` | 2,155,632 bytes | Pesos del modelo (263 tensores) |
| `tm-my-image-model.zip` | 2,248,107 bytes | Export original de Teachable Machine |

La app sirve el modelo desde `my_model/`. Las copias de `model.json`, `metadata.json` y `weights.bin` en la raíz son idénticas a las de `my_model/` y funcionan como respaldo. Si se edita una copia, hay que copiar el cambio a la carpeta que se sirve.

El ZIP original conserva la etiqueta antigua `Botella golpeada de 1/3`. Las copias de trabajo en la raíz y en `my_model/` usan `Botella golpeada`. Si se vuelve a descomprimir el ZIP sobre `my_model/`, se revierte ese nombre; hay que actualizar `metadata.json` o volver a copiar la versión corregida.

### Sobre la precisión

- El modelo depende de la iluminación, el fondo, el ángulo y la distancia.
- La resolución de entrada es 224 × 224; los detalles pequeños (golpes finos, bordes de etiqueta) pueden perderse.
- No hay clase de fondo ni de "desconocido". Un objeto que no sea una botella también recibirá una de las seis etiquetas.
- La confianza softmax no está calibrada como probabilidad real; es una señal relativa del modelo.
- Para control de calidad real, conviene validar con un conjunto de imágenes propio, definir un umbral y considerar una clase negativa o de fondo.

## Estructura del proyecto

```text
modelo ia/
├── index.html                  # App principal: UI, CSS y lógica
├── index-simple.html           # Versión mínima de respaldo
├── metadata.json               # Metadata de respaldo en la raíz
├── model.json                  # Topología de respaldo en la raíz
├── weights.bin                 # Pesos de respaldo en la raíz
├── tm-my-image-model.zip       # Export original de Teachable Machine
├── my_model/
│   ├── metadata.json           # Metadata que sirve la app
│   ├── model.json              # Topología que sirve la app
│   └── weights.bin             # Pesos que sirve la app
└── README.md                   # Este documento
```

`index-simple.html` es una página de respaldo, deliberadamente mínima: cámara de 200 × 200, sin subida de imagen, sin filtros y con estilos básicos. Usa `const URL` para la ruta del modelo; no se debe reutilizar ese nombre si se añaden subidas con `URL.createObjectURL`.

## Diseño de la interfaz

### Layout

- `body` ocupa todo el ancho y tiene `min-height: 100vh`; el fondo base es `#0C0B0A`.
- `.window` ocupa el 100% del ancho, tiene `min-height: 100vh`, `overflow: hidden` y un fondo oscuro con glows radiales muy sutiles en verde azulado y melocotón.
- `.content` es una columna flex con `padding: 14px 28px` y `gap: 12px`.
- `.top-grid` es una grilla de dos columnas: `minmax(0, 1.6fr)` para la cámara y `minmax(340px, 1fr)` para el panel derecho, con `gap: 18px`.
- La columna derecha (`.side-col`) contiene la tarjeta de predicción, los tabs y la lista de clases, con `gap: 10px`.
- La tarjeta de predicción tiene `flex: 1`, por lo que ocupa el espacio vertical disponible.
- El pie de página queda al final y solo muestra "Modelo de Teachable Machine".

No hay cabecera visible, semáforos de macOS, marca "Nagare", pill de estado superior derecha ni tarjeta informativa de procesamiento. Esos elementos se eliminaron durante el desarrollo.

### Paleta

| Token / color | Valor | Uso |
| --- | --- | --- |
| `--peach` | `#F4B398` | Acento principal, fila ganadora, barra |
| `--peach-light` | `#F9C9B2` | Token reservado; el botón usa `#F9C9B2` directamente |
| `--cream` | `#F4F0E8` | Texto principal y extremo de la barra |
| `--muted` | `#9D968D` | Texto secundario |
| `--muted-2` | `#7F7972` | Texto terciario y metadatos |
| `--window` | `#211F1D` | Token reservado para la ventana |
| `--line` | `rgba(244, 240, 232, .08)` | Token reservado para líneas |
| `--card` | `rgba(255, 255, 255, .045)` | Token reservado para tarjetas |
| Verde en vivo | `#7ED9A8` | Punto del chip "En vivo" |
| Teal de imagen | `#8FD6D2` | Punto del chip "Imagen" y extremo de la barra |
| Fondo base | `#0C0B0A` | `body` |
| Fondo de cámara | `#141312` / `#0E0D0C` | Viewport y canvas |
| Error | `#E07A6A` con texto `#F4C9C2` | Banner de error |

Algunos componentes usan valores hexadecimales directos además de los tokens. Si se cambia el tema, conviene buscar los valores `#F4B398`, `#F9C9B2`, `#8FD6D2` y `#7ED9A8` en el CSS.

### Tipografía

La app usa Manrope (Google Fonts) con pesos 400, 500, 600, 700 y 800. Si la fuente no carga, el fallback es `system-ui`, `-apple-system`, `Segoe UI`, `sans-serif`.

| Elemento | Tamaño | Peso | Notas |
| --- | --- | --- | --- |
| Confianza grande | `clamp(40px, 4.4vw, 56px)` | 800 | `line-height: .9` |
| Clase ganadora | `clamp(22px, 2.4vw, 32px)` | 800 | `line-height: 1.04` |
| Nombre de fila | 13.5 px | 700 | Una línea en escritorio |
| Tabs | 12 px | 600 | 11 px entre 901 y 1250 px |
| Etiqueta "Clase"/"Confianza" | 10.5 px | 600 | Gris terciario |
| Eyebrow "Predicción actual" | 10.5 px | 700 | Mayúsculas, tracking amplio |
| Metadatos de la tarjeta | 11 px | 400/700 | Modelo y número de clases |
| Pie | 10.5 px | 400 | Texto secundario |

### Componentes

- **Tarjeta de cámara:** borde redondeado de 20 px, fondo translúcido, viewport flexible con `min-height: 320px`, controles inferiores y chips superpuestos.
- **Vista de cámara:** canvas 480 × 480 con `object-fit: contain`; viñeta y línea de escaneo animada solo cuando la cámara está activa.
- **Placeholder:** icono de cámara, título "Cámara apagada" y texto de ayuda.
- **Controles:** botón principal "Iniciar cámara", botón fantasma "Detener" y botón fantasma "Subir imagen" alineado a la derecha en escritorio.
- **Tarjeta de predicción:** clase, confianza, barra degradada y metadatos "Modelo MobileNet" / "6 clases".
- **Tabs:** píldora con contadores; se pueden desplazar horizontalmente si no caben y ocultan la barra de scroll.
- **Filas de clase:** nombre, categoría, porcentaje y medidor de puntos. En escritorio son de una sola línea; en móvil se apilan.
- **Medidor de puntos:** se dibuja con gradientes radiales y máscaras CSS; el relleno usa `clip-path` y la variable `--p`.
- **Banner de error:** fondo rojizo translúcido, borde y `role="alert"`.

### Responsive

| Rango | Comportamiento |
| --- | --- |
| Más de 1250 px | Layout base de dos columnas |
| 901–1250 px | Tabs más compactos |
| Hasta 900 px | Las columnas se apilan; la cámara usa `aspect-ratio: 16 / 10` y la tarjeta de predicción tiene `min-height: 220px` |
| Hasta 640 px | Cámara en `4 / 3`; padding de contenido 14 px; clase y confianza más pequeñas; filas a una columna; medidores de 24 px; pie con wrap |
| `prefers-reduced-motion` | Se desactivan animaciones y transiciones |

En móvil el layout apilado puede requerir scroll vertical. El objetivo de "todo en una vista" aplica al layout de escritorio.

### Sin scroll en escritorio

La versión compacta reduce el espacio vertical para que el contenido principal entre en una ventana de escritorio normal:

- `content`: padding `14px 28px`, gap 12 px.
- `top-grid`: gap 18 px; la cámara y el panel derecho comparten la altura disponible.
- Tarjeta de predicción: padding 14 px, gap 9 px.
- Tabs: padding 3 px; tabs de 5 px vertical.
- Filas de clase: padding `5px 12px`, una sola línea, medidor de 18 px.
- Columna derecha: gap 10 px.
- Pie: `padding-top: 8px`.

La altura mínima estimada del contenido es de aproximadamente 465 px. Por encima de ese alto de ventana, el layout de escritorio debería entrar sin scroll. Por debajo, con zoom alto o con una ventana muy baja, puede aparecer un scroll mínimo; se prefiere eso a recortar contenido. Si se necesitara cero scroll de forma absoluta, se podría fijar `height: 100vh` y `overflow: hidden`, pero se correría el riesgo de ocultar la lista o el pie en ventanas pequeñas.

### Accesibilidad

- Las secciones y la navegación tienen `aria-label`.
- El banner de error usa `role="alert"`.
- Los botones son elementos `<button>` con texto visible.
- La vista previa de la imagen tiene un `alt`.
- Se respeta `prefers-reduced-motion`.
- Mejoras pendientes: anunciar la predicción con `aria-live`, usar `aria-selected`/`role="tablist"` en los filtros, añadir estilos de foco visibles, aumentar el tamaño táctil de los tabs y revisar el contraste de los textos más apagados.

## Decisiones y cambios

| Cambio | Estado actual | Motivo |
| --- | --- | --- |
| Fondo escénico rosado/azul con dunas SVG, estrellas y grano | Eliminado; fondo oscuro neutro con glows sutiles | Pedido del usuario |
| Ventana tipo macOS con semáforos y marca "Nagare" | Eliminada; app a pantalla completa | Pedido del usuario |
| Cabecera "Clasificador de botellas" y subtítulo | Eliminada; solo queda el `<title>` del documento | Pedido del usuario |
| Pill de estado "En vivo / Cámara apagada" arriba a la derecha | Eliminada; el estado vive en la tarjeta y en el chip de cámara | Pedido del usuario |
| Tarjeta "Todo se procesa aquí / El video no se envía..." | Eliminada | Pedido del usuario |
| Texto "Procesamiento local" en el pie | Eliminado; queda "Modelo de Teachable Machine" | Pedido del usuario |
| Layout con scroll | Compactado: filas de una línea, menos padding y gaps | Pedido de ver todo en una vista |
| Cámara 640 × 400 | 480 × 480 | El modelo recorta al cuadrado; una captura cuadrada evita recortes |
| `object-fit: cover` en la cámara | `object-fit: contain` | Mostrar el encuadre completo, con barras si hace falta |
| `const URL` para la ruta del modelo | `MODEL_URL` | Evitar sobrescribir el constructor global `URL` usado en la subida |
| Etiqueta "Botella golpeada de 1/3" | "Botella golpeada" | Nombre más claro en la interfaz y el metadata |
| Vista previa de imagen sin fallback de decodificación | `decode()` si existe; `load` como fallback si no existe | Compatibilidad con navegadores que no implementan `decode` |

## Personalización

### Cambiar el modelo

1. Entrenar un proyecto de imagen en Teachable Machine y exportarlo como TensorFlow.js.
2. Descomprimir el export en una carpeta temporal.
3. Copiar `model.json`, `metadata.json` y todos los archivos de pesos (`weights.bin` o los shards indicados por `model.json`) a `my_model/`.
4. Abrir `index.html` y actualizar el arreglo `CLASSES` para que coincida con las etiquetas y el orden de `metadata.json`.
5. Actualizar los textos estáticos si cambia el número de clases o el tamaño de entrada: "6 clases" en la tarjeta y el chip "224 × 224".
6. Si se añaden categorías nuevas, actualizar `CATEGORIES` y asignar el `cat` correspondiente en `CLASSES`.
7. Recargar con `Ctrl+F5` para evitar el modelo en caché.

Checklist de etiquetas:

- [ ] `metadata.json` de `my_model/` tiene las etiquetas nuevas.
- [ ] `CLASSES` tiene exactamente los mismos nombres, en el mismo orden.
- [ ] Cada `cat` existe en `CATEGORIES`.
- [ ] Los textos `catLabel` son los deseados.
- [ ] El número de clases de la tarjeta coincide.
- [ ] El chip de tamaño coincide con `imageSize` si cambió.

Si solo se renombra una etiqueta, hay que cambiarla en `metadata.json` y en `CLASSES` a la vez. Si los nombres no coinciden, la app intenta emparejar por posición, pero la fila puede conservar el nombre antiguo y la categoría puede quedar mal asignada.

### Cambiar colores y tipografía

- Los tokens principales están en `:root`, al inicio del `<style>`.
- El fondo de la ventana y sus glows están en `.window`.
- La barra de confianza y el medidor usan degradados con `--peach`, `--cream` y `#8FD6D2`.
- El botón principal usa un degradado melocotón directo.
- La fuente se carga con el `<link>` de Google Fonts y se aplica con `--font`.
- Si se cambia el tema, revisar también los colores del chip en vivo, el chip de imagen, el banner de error y la fila ganadora.

### Cambiar textos

- Título de la pestaña: `<title>` en el `<head>`.
- Placeholder: "Cámara apagada" y su descripción.
- Botones: "Iniciar cámara", "Detener", "Subir imagen".
- Chips: "En vivo", "Imagen", "224 × 224".
- Tarjeta: "Predicción actual", "Clase", "Confianza", "Modelo", "6 clases".
- Pie: "Modelo de Teachable Machine".
- Mensajes de estado y error: cadenas dentro del JavaScript.
- Nombres y categorías: `CLASSES` y `CATEGORIES`.

### Cambiar cámara y frecuencia

- Resolución de captura: `new tmImage.Webcam(480, 480, true)`. Mantener el cuadrado si se quiere evitar el recorte del modelo.
- Espejo: el tercer argumento `true`; probar `false` si el modelo rinde mejor sin espejo.
- Frecuencia de predicción: `timestamp - lastPredict > 90`; un valor mayor reduce carga, uno menor aumenta la reactividad.
- El chip "224 × 224" es informativo; no cambia la captura.
- La vista de la cámara se controla con `.viewport` y `.webcam canvas` en el CSS.

### Fijar versiones de las librerías

La página carga las librerías con `@latest` desde jsDelivr:

```html
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@latest/dist/tf.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image@latest/dist/teachablemachine-image.min.js"></script>
```

Para producción o para reproducibilidad, conviene fijar versiones y, preferiblemente, servir los archivos localmente. El metadata del modelo indica TensorFlow.js 1.7.4, Teachable Machine 2.4.19 y el paquete `@teachablemachine/image` 0.8.4-alpha2; usar un conjunto de versiones que hayas validado. `@latest` puede cambiar y romper la app sin que se toque el código.

### Evitar caché del modelo

- Recargar con `Ctrl+F5` (Windows/Linux) o `Cmd+Shift+R` (macOS).
- En DevTools, activar "Disable cache" mientras se prueba.
- Para producción, versionar la carpeta del modelo (por ejemplo, `my_model/v2/`) y actualizar `MODEL_URL`, o configurar cabeceras de caché adecuadas.
- El servidor `python3 -m http.server` envía `Last-Modified`; el navegador puede reutilizar archivos antiguos si no se fuerza la recarga.

## Privacidad y seguridad

- La cámara solo se activa cuando el usuario pulsa "Iniciar cámara".
- El video se procesa en el navegador; no se envía a ningún servidor.
- Las imágenes subidas se leen con un object URL local y se revocan al terminar.
- La app no usa backend, cookies, `localStorage`, analítica ni base de datos.
- Las únicas peticiones de red son las librerías de TensorFlow.js/Teachable Machine, la fuente Manrope y los archivos del modelo local.
- Para producción, servir las librerías y la fuente localmente, fijar versiones, añadir Subresource Integrity (SRI) y una Content Security Policy (CSP) adecuada.
- No cargar modelos de fuentes no confiables: un modelo es código/datos que se ejecuta en el contexto de la página.
- No hay límite de tamaño para las imágenes subidas; una imagen enorme puede consumir memoria y tardar en decodificar.

## Limitaciones y mejoras

Limitaciones actuales:

- Clasificador de una sola imagen; no detecta varias botellas ni devuelve cajas.
- No hay clase de fondo, "desconocido" ni umbral de confianza.
- No hay suavizado temporal; la predicción puede oscilar.
- Depende de la iluminación, el fondo, el ángulo y la calidad del entrenamiento.
- No hay selector de cámara; se usa la predeterminada del sistema.
- No hay arrastrar y soltar, carga por lotes, historial ni exportación.
- No hay backend ni persistencia.
- Requiere internet para las librerías y la fuente si no se alojan localmente.
- En móvil, la cámara sobre una IP de red local requiere HTTPS; el servidor simple no lo proporciona.
- El loop no maneja una desconexión de cámara posterior al arranque; si `webcam.update()` falla, puede detenerse la actualización.
- No hay pruebas automatizadas ni verificación visual con navegador headless en este entorno.

Mejoras recomendadas:

- Añadir un umbral de confianza y mostrar "No concluyente" cuando la probabilidad sea baja.
- Añadir una clase de fondo/negativo y reentrenar si se usa en un entorno real.
- Promediar varias predicciones para reducir el parpadeo.
- Añadir `aria-live` para anunciar la clase y la confianza.
- Añadir estilos de foco visibles y mejorar el tamaño táctil de los tabs.
- Añadir selector de cámara, arrastrar y soltar, carga por lotes e historial.
- Manejar la desconexión de la cámara y reiniciar el loop.
- Servir librerías y fuente localmente; fijar versiones y añadir SRI/CSP.
- Añadir pruebas end-to-end con Playwright y un servidor de prueba.
- Versionar el modelo y mostrar su versión en la interfaz.

## Solución de problemas

| Síntoma | Causa probable | Solución |
| --- | --- | --- |
| La cámara no se activa | Página abierta con `file://` o desde una IP sin HTTPS | Abrir `http://localhost:8000/` o `http://127.0.0.1:8000/` |
| La cámara no se activa | Permiso denegado o cámara ocupada | Revisar permisos del sitio y cerrar otras apps que usen la cámara |
| Error que menciona WebRTC | Contexto no seguro o restricción del navegador | Usar localhost/HTTPS, permitir la cámara y probar Chrome/Edge |
| El modelo no carga | Servidor apagado, ruta incorrecta o archivos faltantes | Iniciar el servidor y comprobar `my_model/model.json`, `metadata.json` y `weights.bin` |
| `Failed to fetch` | `file://`, servidor caído o proxy | Servir por HTTP local y revisar la consola/red |
| `tf is not defined` o `tmImage is not defined` | CDN bloqueado o sin internet | Revisar la red, el bloqueador de anuncios o alojar las librerías localmente |
| La página se ve sin la fuente Manrope | Google Fonts bloqueado | Es solo tipografía; la app sigue funcionando con la fuente del sistema |
| Predice siempre lo mismo | Poca luz, fondo confuso, objeto fuera de encuadre o modelo sesgado | Mejorar iluminación, centrar una sola botella y probar con imágenes de validación |
| La imagen subida parece recortada | Teachable Machine recorta el centro al cuadrado | Usar una imagen cuadrada o centrar la botella |
| La confianza es baja o cambia mucho | Modelo poco seguro, sin suavizado temporal | Repetir la captura, mejorar el encuadre o añadir un umbral/promedio |
| Hay scroll en escritorio | Ventana más baja que ~465 px, zoom alto o contenido adicional | Reducir zoom, ampliar la ventana o compactar más los estilos |
| El puerto 8000 está ocupado | Otro servidor en uso | Usar `8001` o detener el proceso con `lsof -i :8000` / `ss -ltnp` |
| Los cambios no aparecen | Caché del navegador o del modelo | `Ctrl+F5`, "Disable cache" o versionar la ruta del modelo |
| Las filas no coinciden con las predicciones | `CLASSES` desincronizado con `metadata.json` | Igualar nombres, orden y categorías |
| El ZIP revierte el nombre de una etiqueta | El export original usa "Botella golpeada de 1/3" | Usar las copias de `my_model/` o actualizar el metadata |
| En el móvil no hay cámara | HTTP sobre IP de red local | Usar HTTPS o un túnel; el servidor simple no ofrece HTTPS |

## Validación y desarrollo

La app no tiene build ni suite de pruebas. Estas son las comprobaciones usadas durante el desarrollo:

Comprobar la sintaxis del JavaScript inline:

```bash
python3 - <<'PY'
import re
html = open('index.html').read()
js = re.findall(r'<script type="text/javascript">(.*?)</script>', html, re.S)[0]
open('/tmp/app.js', 'w').write(js)
PY
node --check /tmp/app.js
```

Comprobar el balance de llaves del CSS:

```bash
python3 - <<'PY'
import re
html = open('index.html').read()
css = re.search(r'<style>(.*?)</style>', html, re.S).group(1)
print(css.count('{'), css.count('}'))
PY
```

Validar el HTML:

```bash
xmllint --html --noout index.html
```

Comprobar que el servidor responde:

```bash
curl --noproxy '*' -o /dev/null -w '%{http_code} %{size_download}\n' http://127.0.0.1:8000/index.html
```

Pruebas manuales recomendadas:

1. Abrir la app y comprobar que no hay errores en la consola.
2. Iniciar la cámara, aceptar el permiso y verificar el chip "En vivo".
3. Detener la cámara y comprobar que vuelve el placeholder y se reinician las predicciones.
4. Subir una imagen cuadrada y una no cuadrada; revisar la vista previa y la predicción.
5. Probar los cuatro filtros y comprobar los contadores.
6. Redimensionar la ventana: escritorio sin scroll, ≤900 px apilado, ≤640 px móvil.
7. Denegar el permiso de cámara y comprobar el banner de error.
8. Apagar el servidor y recargar para comprobar el error de carga del modelo.
9. Cambiar una etiqueta en `metadata.json` y `CLASSES`, recargar con `Ctrl+F5` y verificar.
10. Repetir con el modelo original del ZIP para confirmar que la etiqueta vuelve a "Botella golpeada de 1/3" si se sobrescribe.

En el entorno de desarrollo no había un navegador headless disponible, así que la verificación visual y de cámara es manual. Las comprobaciones estáticas (sintaxis, llaves, HTML y respuesta HTTP) sí se ejecutaron.

## Créditos

- Modelo entrenado y exportado con Teachable Machine (Google).
- Inferencia en el navegador con TensorFlow.js y `@teachablemachine/image`.
- Tipografía Manrope, disponible en Google Fonts.
- Librerías servidas desde jsDelivr.
- Referencia visual de diseño: `../Referencias/Design de referencia.jpg` (no es necesaria para ejecutar la app).
- El modelo y las imágenes de entrenamiento son provistos por el autor del proyecto; no se incluye el dataset.

## Licencia

Este repositorio no incluye un archivo `LICENSE`. Antes de distribuirlo, añade la licencia que corresponda al código y al modelo. Las dependencias externas tienen sus propias licencias: TensorFlow.js y Teachable Machine son proyectos de código abierto de Google, y Manrope se distribuye bajo SIL Open Font License. Revisa las licencias de las versiones que uses y no redistribuyas la imagen de referencia si no tienes derechos sobre ella.
