# 🤖 Proyecto LARC OPEN 2026: Sistema de Triple Núcleo (NXT)

Este documento presenta el ecosistema de software del robot recolector de café para la competencia **LARC 2026**, destinado al uso interno del equipo **PUMA Nexus Coffea**, del Laboratorio de Algoritmos para la Robótica (LAR).  

El sistema utiliza una arquitectura de procesamiento distribuido sobre tres ladrillos LEGO Mindstorms NXT, comunicados mediante Bluetooth para gestionar tareas en paralelo: **Navegación, Recolección y Logística.**

El desarrollo ha sido realizado bajo el **liderazgo principal de Enrique**, con los desarrolladores principales **Odette (Crowley)** y **Balam**, y la participación activa de otros miembros del equipo. Además, se reconoce la invaluable labor de la **Responsable Académica del LAR, Lic. Miriam Hernández Ramírez**, cuyo apoyo en ingeniería y programación ha sido fundamental.  

Toda la lógica de control, los cálculos vectoriales de la base omnidireccional y los algoritmos de seguimiento de línea se han desarrollado y optimizado a partir del código proporcionado por **Luis Armando**, cuya contribución fue de gran relevancia para este proyecto, a pesar de pertenecer a una categoría distinta dentro de la competencia. Los códigos no son definitivos y continuarán evolucionando con la participación de todo el equipo.

---

<details>
<summary>🏗️ Arquitectura del Sistema (Hardware 31x36 cm)</summary>

El robot se divide en tres nodos especializados para maximizar la eficiencia y reducir la latencia de respuesta:

### 1. 🛰️ Ladrillo "BASE" (Maestro)
* **Función:** Inteligencia central y navegación.
* **Actuadores:** 3 Motores (Puertos A, B, C) en configuración omni-wheel.
* **Sensores:** 2 Sensores de Luz (S1, S2) para seguimiento de línea de 19 mm.
* **Rol BT:** Maestro de la red (Slot 1: Garra | Slot 2: Almacén).

### 2. 🏗️ Ladrillo "GARRA" (Esclavo 1)
* **Función:** Recolección vertical de precisión.
* **Actuadores:** 2 Motores sincronizados (A, B) para elevador de tijera; 1 Motor (C) para pinza.
* **Sensores:** 1 Sensor Ultrasónico (S1) para validación de captura.

### 3. 📦 Ladrillo "ALMACENAMIENTO" (Esclavo 2)
* **Función:** Clasificación y descarga de frutos.
* **Actuadores:** 1 Motor (A) clasificador; 2 Motores (B, C) para compuertas de descarga.
* **Sensores:** S1 (Color Interno), S2 (Color Externo), S3 (Ultrasonido de Proximidad).

</details>

<details>
<summary>🧪 Decisiones de Ingeniería y Justificación de Valores</summary>

| Parámetro | Valor | Justificación Técnica (Mundo Real) |
| :--- | :--- | :--- |
| **Potencia Base** | `25%` | Control de inercia para un chasis de gran volumen. Evita el derrape en curvas. |
| **Umbral Luz** | `35` | Punto de corte óptico entre el blanco de la lona y el negro de la cinta oficial. |
| **Sync Ratio** | `0` | Obliga a los motores del elevador a rotar idénticos, evitando que la tijera se trabe. |
| **Wait (Caída)** | `1500ms` | Tiempo necesario para que la gravedad venza la fricción del grano en la rampa. |
| **US Threshold** | `12cm` | Margen de maniobra para alinear las compuertas con los depósitos externos. |

</details>

<details>
<summary>🚀 Instructivo de Despliegue Global</summary>

### Paso 1: Configuración de Red (Bluetooth)
1. Renombre los ladrillos esclavos como `GARRA` y `ALMACEN`.
2. Desde el menú del ladrillo **BASE**, busque y vincule a `GARRA` en el **Contacto 1** y a `ALMACEN` en el **Contacto 2**.
3. Asegúrese de que el Bluetooth esté en modo **Visible** en los tres dispositivos.

### Paso 2: Secuencia de Encendido
1. **Ejecutar Esclavos:** Inicie primero los programas en los ladrillos de la Garra y Almacenamiento. Verá el mensaje "ESPERANDO MAESTRO".
2. **Ejecutar Maestro:** Inicie el programa en la Base.
3. **Validación:** El sistema emitirá un **doble pitido** cuando el enlace de triple núcleo esté activo y sincronizado.

### Paso 3: Operación en Pista
* Posicione el robot centrado en la línea de salida.
* Presione el botón naranja del Maestro para iniciar la rutina autónoma.

</details>

<details>
<summary>⚠️ Resolución de Problemas (Troubleshooting)</summary>

* **Pérdida de Trayectoria:** Verifique que los sensores de luz estén a 1 cm del suelo. Si hay mucha luz ambiental, reduzca el umbral a `30`.
* **Elevador Trabado:** Verifique que los cables no obstruyan el movimiento de las tijeras. Asegure el uso de `OnFwdSync`.
* **Falla de Clasificación:** Limpie los sensores de color internos; las sombras del chasis pueden alterar la lectura del rojo/café.

</details>

---

> **Nota de Versión:** Código optimizado para el reglamento oficial LARC 2026.