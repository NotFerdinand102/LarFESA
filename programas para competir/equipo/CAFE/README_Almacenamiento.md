# 📦 Ladrillo Almacenamiento: Clasificación y Gestión de Inventario

## 📝 Descripción General
Nodo esclavo encargado de la logística final. Clasifica los granos por madurez (Color) y gestiona la descarga selectiva en los contenedores del beneficiadero, maximizando la puntuación del Apéndice C.

## 🏗️ Especificaciones de Hardware
| Componente | Puerto | Función Técnica |
| :--- | :--- | :--- |
| **Motor Clasificador**| OUT_A | Distribución en rampas internas |
| **Motor Puertas** | OUT_BC | Apertura de compuertas laterales |
| **Sensor Color 1** | S1 | Identificación de grano interno |
| **Sensor Color 2** | S2 | Identificación de contenedor en pista |
| **Sensor US** | S3 | Alineación de descarga (Proximidad) |

## 🧠 Decisiones de Ingeniería (Design Log)
*   **Lógica de No-Penalización:** El código integra un comparador `Sensor(S2) == Sensor(S1)`. Esto garantiza que la compuerta **solo se abra** si el color de la caja externa coincide con el grano cargado, evitando el castigo de **-7 puntos**.
*   **Gravedad Asistida (Wait 1500ms):** 1.5 segundos es el tiempo probado para que el grano venza la fricción de la rampa de plástico. Menos tiempo causaría que el robot arranque con el grano aún a mitad del túnel.
*   **Alineación (12 cm):** Este valor compensa el ancho de las llantas omni, permitiendo que la compuerta quede exactamente sobre el centro del contenedor de descarga.

## 🛠️ Guía de Ajuste en Pista
1. **Error de Color:** Si confunde Rojo con Café, limpie los sensores S1/S2 con aire comprimido.
2. **Grano Atorado:** Si el grano no sale, verifique la inclinación física de la rampa (debe ser >15°).
3. **Descarga Fallida:** Si se abre la puerta antes de llegar, reduzca la distancia de proximidad en S3 a **10 cm**.