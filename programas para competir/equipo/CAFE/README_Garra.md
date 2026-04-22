# 🏗️ Ladrillo Garra: Sistema de Manipulación Vertical de Tijera

## 📝 Descripción General
Este módulo es un nodo esclavo especializado en la recolección de precisión. Utiliza un mecanismo de elevación tipo "tijera" impulsado por motores sincronizados para alcanzar las alturas de 10cm y 15cm especificadas en el reglamento.

## 🏗️ Especificaciones de Hardware
| Componente | Puerto | Función Técnica |
| :--- | :--- | :--- |
| **Motores Elevador** | OUT_AB | Elevación Sincronizada (Tijera) |
| **Motor Pinza** | OUT_C | Captura y sujeción del fruto |
| **Sensor Ultrasonido**| S1 | Validación de proximidad de captura |

## 🧠 Decisiones de Ingeniería (Design Log)
*   **Sincronización Crítica (`OnFwdSync`):** La elección de esta función es vital. En mecanismos de tijera, un desfase de incluso 5 grados entre motores puede doblar las vigas Technic o trabar el elevador.
*   **Seguridad Mecánica (Wait 2500ms):** El tiempo de ascenso está calculado para llegar al tope sin golpear el chasis. **No aumente la potencia**, aumente el tiempo si requiere más altura.
*   **Protocolo de Captura:** El sensor US actúa como un "disparador" (trigger). La pinza no cierra hasta que el objeto está a menos de **10 cm**, evitando capturas fallidas por errores de navegación.

## 🛠️ Guía de Ajuste en Pista
1. **Caída de Fruto:** Si el grano se resbala al subir, aumente `RotateMotor` de la pinza a **100°**.
2. **Descenso Brusco:** Si el robot vibra al bajar, reduzca la potencia de descenso a **20**.
3. **Obstrucciones:** Verifique que los cables no se enreden en las tijeras durante el ascenso.