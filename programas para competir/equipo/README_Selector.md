# Ladrillo SELECTOR: Clasificacion y Deposito Autonomo (v3.0)

## Descripcion General

Nodo completamente independiente (no recibe comandos BT). Permanece activo durante toda la ronda y actua en dos momentos: cuando detecta una pelota entrante proveniente de la garra, y cuando el robot se posiciona frente a una caja del beneficiadero. Garantiza que cada grano llegue al contenedor correcto, evitando penalizaciones.

La clasificacion usa tonalidad relativa R/G/B del sensor HiTechnic v2 para identificar tanto el tipo de pelota interna como el color de la caja externa.

## Especificaciones de Hardware

| Componente | Puerto | Funcion Tecnica |
| :--- | :--- | :--- |
| Motor selector | OUT_A (MA) | Dirige la pelota: 0 grados neutro, +45 canal rojo, -45 canal azul |
| Motor compuerta izquierda | OUT_B (MB) | Abre 90 grados para depositar en caja azul (sobremaduros) |
| Motor compuerta derecha | OUT_C (MC) | Abre 90 grados para depositar en caja roja (maduros) |
| Sensor ultrasonico | IN_1 (S1) | Detecta presencia de caja a menos de 15 cm |
| Sensor RGB HiTechnic v2 | IN_2 (S2) | Lee color de pelota entrante y color de caja externa |

S1 y S2 sirven para detectar las cajas donde se depositaran las pelotas.

## Decisiones de Ingenieria

| Parametro | Valor | Justificacion |
| :--- | :--- | :--- |
| Angulo selector | +/-45 grados | Minimo giro para alinear rampa con cada canal de salida |
| Angulo compuerta | 90 grados | Apertura completa para que la pelota ruede sin obstruccion |
| Wait caida | 1500 ms | Tiempo probado para que gravedad venza friccion del PLA en rampa > 15 grados |
| Distancia caja | 15 cm | Margen para alinear compuerta sobre el contenedor (450 mm de lado) |
| Umbral rojo | 50 | Canal R dominando por encima de 50 = caja/pelota roja |
| Umbral azul | 50 | Canal B dominando por encima de 50 = caja/pelota azul |
| Umbral verde | 40 | Canal G dominando por encima de 40 = verde (ignorar) |

### Logica de no-penalizacion

El codigo compara tipo_pelota_interna == tipo_caja_externa antes de abrir cualquier compuerta. Si no coinciden, el selector permanece cerrado y espera al siguiente cuadro. Esto implementa directamente la regla del reglamento:

| Situacion | Puntos | Selector abre? |
| :--- | :--- | :--- |
| Maduro en caja roja | +7 | SI |
| Sobremaduro en caja azul | +5 | SI |
| Maduro en caja azul | -7 | NO (bloqueado) |
| Sobremaduro en caja roja | -5 | NO (bloqueado) |

### Clasificacion por tonalidad relativa (HiTechnic v2)

Se leen los tres canales R, G, B mediante I2C directo (registros 0x43-0x45) y se determina cual domina:

- R > umbral AND R > G AND R > B -> Caja roja / pelota madura (rojo/naranja/amarillo)
- B > umbral AND B > R AND B > G -> Caja azul / pelota sobremadura (azul/negro)
- G > umbral AND G > R AND G > B -> Verde -> ignorar (no deberia llegar al selector)
- Ninguno domina -> ignorar

Este metodo es robusto ante variaciones de iluminacion porque trabaja con dominancia relativa, no con un valor absoluto de color.

### Lectura I2C mejorada

En v3.0 se usa LowspeedCheckStatus() para esperar que los datos I2C esten disponibles antes de leer, en lugar de un Wait() fijo. Esto hace la lectura mas robusta y rapida.

## Flujo de Operacion

```
[Siempre activo - bucle cada 50 ms]
       |
  Lee US (S1) y RGB (S2)
       |
  pelota_actual == NINGUNA  Y  dist > 15 cm?
       | SI -> Fase A: detectar pelota entrante
       |   Clasificar color RGB
       |   Si MADURA o SOBREMAD -> guardar tipo
       |
  pelota_actual != NINGUNA  Y  dist < 15 cm?
       | SI -> Fase B: intentar deposito
       |   Clasificar color de la caja
       |   tipo_pelota == tipo_caja?
       |     SI -> girar selector, abrir compuerta, esperar, cerrar
       |     NO -> no abrir, esperar siguiente cuadro
       |
  Repetir
```

## Cambios respecto a v2.0

1. Puertos corregidos con constantes IN_1 e IN_2 (no S1/S2 que son macros de valor)
2. Mejorada lectura I2C con LowspeedCheckStatus() y timeout de 50 ms
3. Variable pelota_actual inicializada explicitamente fuera del main
4. Comentarios limpios sin caracteres especiales Unicode

## Guia de Ajuste en Pista

1. **Confunde color de caja:** Limpiar sensor S2 y ajustar UMBRAL_ROJO / UMBRAL_AZUL. En entornos con luz calida (amarilla) subir el umbral de rojo a 60.
2. **Pelota no sale completamente:** Verificar que la rampa fisica tenga inclinacion > 15 grados. Si persiste, aumentar T_CAIDA a 2000 ms.
3. **Abre compuerta antes de llegar a la caja:** Reducir DIST_CAJA a 10 cm.
4. **Selector no regresa al neutro:** Verificar que el angulo de retorno coincida exactamente con el de salida (siempre +/-ANG_SELECTOR).
5. **Confunde pelota interna con caja externa:** Crear un escudo fisico alrededor del sensor S2 para que solo vea hacia afuera cuando el robot esta en el beneficiadero.
6. **Sensor no lee (valores 0,0,0):** Verificar cable I2C en S2, reconectar y reiniciar.

## Secuencia de Inicio

1. Encender este ladrillo antes que el Maestro.
2. Pantalla mostrara "SELECTOR v3 / ACTIVO / Sin pelota".
3. No requiere vinculo BT; opera de forma completamente autonoma.
4. Permanece activo durante toda la ronda sin intervencion.

## Penalizaciones Evitadas

| Situacion | Penalizacion | Mecanismo de proteccion |
| :--- | :--- | :--- |
| Grano maduro en caja azul | -7 pts | tipo_pelota == tipo_caja antes de abrir |
| Grano sobremaduro en caja roja | -5 pts | tipo_pelota == tipo_caja antes de abrir |
| Grano abandonado en escenario | -7 pts | Las compuertas solo abren frente a caja verificada |
| Grano verde recogido | -7 pts | Protegido por la garra (no recoge verdes) |
