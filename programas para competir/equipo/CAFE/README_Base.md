# 🤖 Ladrillo Base: Arquitectura de Control y Navegación Central

## 📝 Descripción General
Este módulo constituye el "Cerebro Central" del robot para la competencia **LARC OPEN 2026**. Implementa una arquitectura de control maestro-esclavo mediante Bluetooth, gestionando la navegación omnidireccional y la sincronización de tareas de recolección y logística.

Toda la lógica de movimiento y los cálculos vectoriales de este módulo son el resultado del desarrollo y optimización técnica de **Armando**.

## 🏗️ Especificaciones de Hardware
| Componente | Puerto | Función Técnica |
| :--- | :--- | :--- |
| **Motor A, B, C** | OUT_ABC | Tracción Omnidireccional (Cálculos de Armando) |
| **Sensor Luz 1** | S1 | Seguimiento de línea y detección de estaciones |
| **Sensor Luz 2** | S2 | Corrección de trayectoria y validación de cruces |

## 🧠 Decisiones de Ingeniería (Design Log)
*   **Gestión de Inercia:** Se ha limitado el PWM al **25%**. Dado que el chasis mide **31x36 cm**, una velocidad superior generaría un momento lineal difícil de corregir en las curvas cerradas del Apéndice A.
*   **Filtro de Contraste (Umbral 35):** Este valor es el "Gold Standard" para la lona oficial. Representa el límite donde el ruido lumínico ambiental no interfiere con la detección del negro profundo de la cinta.
*   **Topología de Red:** El Maestro mantiene dos canales activos (Slots 1 y 2). El sistema bloquea el inicio hasta que ambos periféricos confirman el *handshake*.

## 🛠️ Guía de Ajuste en Pista
1. **Patinaje:** Si las ruedas omni patinan, reduzca `OnFwd` a **20**.
2. **Falsos Positivos:** Si el robot se detiene donde no hay árboles, baje el umbral a **30**.
3. **Batería:** Con voltajes menores a **7.2V**, incremente la potencia a **30** para mantener el torque nominal.