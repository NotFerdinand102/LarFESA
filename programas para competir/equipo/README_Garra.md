# Ladrillo GARRA: Recoleccion por Niveles con Clasificacion RGB (v3.0)

## Descripcion General

Nodo esclavo especializado en la recoleccion de granos de cafe. Recibe comandos del Maestro (Base) via Bluetooth y responde con ACK para indicar si recogio o no. Usa el sensor HiTechnic Color v2 en S4 para discriminar granos verdes (no recoger) de maduros y sobremaduros (recoger), cumpliendo la regla de no separar granos verdes del arbol.

La clasificacion se basa en INPUT_GREENCOLOR: si el canal verde domina, no recoger. Cualquier otro color, recoger.

## Especificaciones de Hardware

| Componente | Puerto | Funcion Tecnica |
| :--- | :--- | :--- |
| Motor pinza | OUT_A (MA) | Cierra (+45 grados) / Abre (-45 grados) desde posicion inicial |
| Motor elevador principal | OUT_B (MB) | Fwd = SUBE |
| Motor elevador secundario | OUT_C (MC) | Rev = SUBE (sincronizado inverso con MB) |
| Sensor RGB HiTechnic v2 | IN_4 (S4) | Detecta color de la pelota para decidir si recoger |

## Decisiones de Ingenieria

| Parametro | Valor | Justificacion |
| :--- | :--- | :--- |
| Elevador: angulo alto | 45 grados | Corresponde a la fila superior del arbol |
| Elevador: angulo medio | 25 grados | Corresponde a la fila media del arbol |
| Elevador: angulo bajo | 10 grados | Corresponde a la fila inferior del arbol |
| Pinza | +/-45 grados | Un cuarto de vuelta: suficiente para sujetar sin forzar |
| Potencia elevador | 40% | No aumentar; si no llega, aumentar el angulo |
| Potencia pinza | 50% | Agarre firme sin forzar el motor |
| Umbral verde | 40 | Canal G dominando por encima de 40 = pelota verde |

### Sincronizacion del elevador

MB=Fwd y MC=Rev producen el mismo sentido de giro mecanico en el mecanismo de tijera. Se usa RotateMotor independiente por ser angulos pequenios (<=45 grados). Para subir hasta arriba solo girar 45 grados.

### Clasificacion por canal verde (INPUT_GREENCOLOR)

El sensor HiTechnic v2 entrega valores separados de R, G, B mediante lectura I2C directa (registros 0x43-0x45). La logica es:

- `green > red AND green > blue AND green > 40` -> VERDE -> no recoger, bajar sin tocar
- Cualquier otro patron -> RECOGER (cubre rojo, naranja, amarillo, azul, negro)

### Comunicacion bidireccional (ACK)

Despues de cada intento de recoleccion la garra envia un ACK al maestro:
- ACK_RECOGIO (1) -> el maestro sabe que debe retroceder y enviar CMD_SOLTAR
- ACK_VACIO (0) -> el maestro avanza directamente a la siguiente posicion

## Comandos BT

### Recibidos (INBOX 1)

| Comando | Valor | Accion |
| :--- | :--- | :--- |
| CMD_ALTO | 10 | Sube 45 grados, lee color, recoge si no es verde, baja |
| CMD_MEDIO | 11 | Sube 25 grados, lee color, recoge si no es verde, baja |
| CMD_BAJO | 12 | Sube 10 grados, lee color, recoge si no es verde, baja |
| CMD_SOLTAR | 20 | Abre pinza para depositar al selector |

### Enviados (OUTBOX 0)

| ACK | Valor | Significado |
| :--- | :--- | :--- |
| ACK_RECOGIO | 1 | Pelota recogida exitosamente |
| ACK_VACIO | 0 | No recogio (verde, vacio, o ya tiene una pelota) |

La garra no recoge dos pelotas al mismo tiempo; si ya tiene una, envia ACK_VACIO hasta recibir CMD_SOLTAR.

## Ciclo de Recoleccion (por comando)

```
Recibe CMD_ALTO / MEDIO / BAJO
  |
  +-- Ya tiene pelota?
  |   SI -> enviar ACK_VACIO
  |   NO:
  |
  +-- Subir elevador al angulo del nivel
  +-- Esperar 400 ms (estabilizacion)
  +-- Leer RGB del sensor S4
  |
  +-- Canal verde domina?
  |   SI -> Bajar sin tocar -> enviar ACK_VACIO
  |   NO:
  |
  +-- Cerrar pinza (+45 grados)
  +-- Esperar 500 ms (agarre firme)
  +-- Bajar elevador con pelota sujeta
  +-- Enviar ACK_RECOGIO al maestro
  +-- Esperar CMD_SOLTAR
  +-- Abrir pinza -> pelota cae al selector
```

## Cambios respecto a v2.0

1. Refactorizado: eliminado codigo repetitivo en switch con funcion `procesar_recoleccion()` y `angulo_por_comando()`
2. Mejorada lectura I2C con `LowspeedCheckStatus()` en lugar de Wait fijo
3. Puerto del sensor documentado como IN_4 (constante NXC correcta)
4. Manejo robusto de timeout en lectura I2C (max 50 ms)

## Guia de Ajuste en Pista

1. **Pelota verde mal identificada:** Limpiar sensor con panio seco y ajustar UMBRAL_VERDE. Subir si confunde pelota de color como verde; bajar si no identifica el verde.
2. **Elevador no llega al tope:** Aumentar el angulo en pasos de 5 grados (ANG_ALTO, ANG_MEDIO, ANG_BAJO). No aumentar la potencia.
3. **Pelota se resbala de la pinza:** Aumentar ANG_PINZA de 45 a 60.
4. **Descenso brusco / vibracion:** Reducir VEL_ELEV a 30.
5. **Sensor no lee (valores 0,0,0):** Verificar que S4 este en modo Lowspeed (I2C). Revisar cable y reconectar.
6. **ACK no llega al maestro:** Verificar que OUTBOX sea 0 y que el maestro lea INBOX 0.

## Secuencia de Inicio

1. Encender este ladrillo antes que el Maestro.
2. Pantalla mostrara "GARRA v3 / ESPERANDO BT".
3. El Maestro se conectara automaticamente al confirmar el enlace.
4. La garra responde solo a comandos; no inicia movimiento sola.
