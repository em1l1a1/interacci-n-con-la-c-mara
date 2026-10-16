# Especificaciones del proyecto de interacción

## 1. Visión general
Aplicación web de una sola página que usa la cámara web para detectar la mano en tiempo real y dibujar en el aire sobre un lienzo, controlada solo con gestos.

## 2. Formato y tecnología
- Un solo archivo `index.html` con HTML, CSS y JavaScript embebidos. Sin instalar nada.
- Dibujo en un elemento `<canvas>`.
- Detección de la mano con **MediaPipe Hands** cargado por CDN, ejecutándose directamente en el navegador y entregando la posición de los dedos.
- Código ordenado, con comentarios en español. Colores, tamaños y umbrales definidos como variables al inicio para modificarlos fácilmente.

## 3. Cámara y pantalla
- Al abrir la página, pide permiso para usar la cámara.
- La imagen de la cámara se ve en espejo horizontal (si muevo la mano a la derecha, el trazo va a la derecha).
- El canvas tiene proporción 16:9 (1280×720) y se ajusta al tamaño de la ventana manteniendo la proporción.
- La cámara se muestra de fondo, un poco atenuada, para que el dibujo resalte. Los trazos se dibujan encima.

## 4. Detección de gestos (MediaPipe Hands)
MediaPipe entrega 21 landmarks por mano con coordenadas normalizadas (0 a 1). Se usa la mano con mayor confianza si se detectan varias.

### Criterio de dedo extendido o doblado
- Se compara la punta de cada dedo con su articulación intermedia.
- Índice, medio, anular y meñique: extendido si la punta está más arriba (menor Y normalizada) que la articulación.
- Pulgar: extendido si la punta está más a la derecha (mayor X normalizada) que la articulación, por el efecto espejo.

### Definición de gestos
| Gesto | Condición | Acción |
|-------|-----------|--------|
| Dibujar | Índice extendido; medio, anular, meñique y pulgar doblados. | Dibuja siguiendo la punta del índice (landmark 8). |
| Pausa | Los cinco dedos extendidos. | No dibuja. Muestra un cursor circular en la punta del índice. |
| Borrar | Los cinco dedos doblados (puño cerrado). | Borra trazos cercanos a la palma (efecto borrador). |
| Sin mano | No se detecta ninguna mano. | Muestra el mensaje "Muestra tu mano a la cámara". |

Landmarks clave:
- Punta del índice: 8
- Articulación intermedia del índice: 6

## 5. Trazos
- Línea continua de un solo color y grosor fijo (fácil de cambiar en variables al inicio del código).
- **Suavizado**: interpolación lineal entre la posición anterior y la nueva (`posSuave = posAnterior + (posNueva - posAnterior) * factor`) para que el trazo no tiemble.
- **Unión de puntos**: curvas de Bézier cuadrática entre cada tres puntos consecutivos, no líneas rectas entrecortadas.
- **Finalización**: al levantar el dedo, cambiar de gesto o perder la detección, el trazo actual termina. Al volver a dibujar, empieza uno nuevo sin unirse al anterior.

### Estructura de datos de un trazo
```js
{
  puntos: [{ x, y }, ...], // coordenadas en píxeles del canvas
  color: string,           // ej. "#00ff00"
  grosor: number,          // ej. 4
  terminado: boolean       // true cuando el gesto cambia o se pierde la mano
}
```

## 6. Borrador
- Con el puño cerrado, se borra la parte de los trazos que toca el círculo del borrador.
- Se muestra un círculo semitransparente que indica el área que se está borrando.
- Radio del borrador: **40 px** (variable al inicio del código).
- El borrado corta el trazo en el punto donde el círculo lo toca, eliminando la parte afectada y manteniendo los fragmentos que quedan fuera del círculo.
- Si un trazo queda dividido en fragmentos, cada fragmento se conserva como trazo independiente.
- Si un trazo queda completamente dentro del círculo, se borra por completo.

## 7. Indicador en pantalla
- Texto pequeño en una esquina que dice el gesto actual: "Dibujando", "Pausa" o "Borrando".
- Un recuadro discreto con la lista de gestos:
  - Índice = dibujar
  - Mano abierta = pausa
  - Puño = borrar

## 8. Interfaz y estilo
- Diseño limpio y minimalista.
- Fondo oscuro, tipografía clara y legible, colores armónicos.
- Sin botones ni menús: todo funciona solo con la mano.

## 9. Rendimiento
- Meta de 30 cuadros por segundo en un computador normal.
- Si la cámara no está disponible o se niega el permiso, muestra un mensaje claro explicando qué hacer.

## 10. Mapeo de coordenadas
La cámara se muestra espejada (`transform: scaleX(-1)`) para que se vea desde tu perspectiva propia. Las coordenadas normalizadas de MediaPipe (0 a 1) se compensan para que el trazo aparezca donde ves tu dedo:
- `xCanvas = (1 - xNormalizado) * 1280`
- `yCanvas = yNormalizado * 720`
- Suavizado aplicado antes de dibujar cada punto.

## 11. Cómo abrir y probar
1. Guarda el código en un archivo `index.html`.
2. Abre la carpeta en Visual Studio Code.
3. Instala la extensión "Live Server" si no la tienes.
4. Haz clic en "Go Live" o abre `index.html` con Live Server.
5. Concede permiso para la cámara cuando el navegador lo pida.
6. Muestra tu mano a la cámara y usa los gestos para dibujar, hacer pausa o borrar.
