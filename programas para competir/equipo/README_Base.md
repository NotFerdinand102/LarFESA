# Ladrillo BASE: Maestro de Navegacion y Coordinacion (v3.0)

## Descripcion General

Cerebro central del robot PUMA Nexus Coffea para LARC OPEN 2026. Controla la navegacion omnidireccional con ruedas omni, coordina via Bluetooth bidireccional al ladrillo Garra (esclavo), y ejecuta la estrategia de barrido por nivel (primero fila alta en los 3 arboles, luego media, luego baja).

El bloque Selector funciona de manera completamente independiente y no requiere comandos del maestro.

## Especificaciones de Hardware

| Componente | Puerto | Funcion Tecnica |
| :--- | :--- | :--- |
| Motor frontal derecho | OUT_A (MA) | Traccion omni - eje principal de avance |
| Motor izquierdo | OUT_B (MB) | Traccion omni |
| Motor trasero | OUT_C (MC) | Traccion omni |
| Sensor linea frontal izquierdo | IN_1 (S1) | Deteccion de linea negra 19 mm |
| Sensor linea trasero izquierdo | IN_2 (S2) | Correccion de trayectoria |
| Sensor linea frontal derecho | IN_3 (S3) | Deteccion de cruces y correccion |
| Sensor ultrasonico frontal | IN_4 (S4) | Deteccion de obstaculos (albercas) a < 35 cm |

## Modelo de Movimiento Omni (basado en Movimiento.nxc)

Las funciones de movimiento fueron extraidas del archivo Movimiento.nxc probado fisicamente:

- **adelante()**: MB=Rev, MC=Fwd - avanza en el eje del motor MA
- **atras()**: MB=Fwd, MC=Rev - retrocede
- **derecha()**: MA=Rev, MB=Fwd/2-3, MC=Fwd/2 - giro a la derecha
- **izquierda()**: MA=Fwd, MB=Rev/2-3, MC=Rev/2 - giro a la izquierda (espejo)
- **girar_eje()**: rotacion sobre el centro del robot

## Flujo de Operacion (Estrategia Barrido por Nivel)

```
INICIO
  |
  +-- Espera BT con Garra (esclavo)
  |
  +-- FASE 1: Sale del cuadro inicial -> izquierda hasta linea negra
  |
  +-- FASE 2: Avanza hacia zona de cultivo
  |           (esquiva albercas, guarda ruta en matriz 5x2)
  |
  +-- FASE 3: Recoleccion por niveles
  |   +-- 3a: NIVEL ALTO  -> Arbol A (5 bolas) -> B (5) -> C (5)
  |   +-- 3b: NIVEL MEDIO -> Arbol A (6 bolas) -> B (6) -> C (6)
  |   +-- 3c: NIVEL BAJO  -> Arbol A (5 bolas) -> B (5) -> C (5)
  |   En cada bola:
  |     1. Envia CMD al Garra
  |     2. Espera ACK (1=recogio, 0=verde/vacio)
  |     3. Si recogio: retrocede, envia CMD_SOLTAR, avanza
  |     4. Desplaza lateralmente a siguiente posicion
  |
  +-- FASE 4: Regresa por ruta inversa (180 grados + esquives invertidos)
  |
  +-- FASE 5: Recorre 5 cuadros del beneficiadero (10 s c/u)
  |           -> Selector deposita autonomamente
  |
  +-- FASE 6: Regresa al cuadro inicial -> FIN
```

## Decisiones de Ingenieria

| Parametro | Valor | Justificacion |
| :--- | :--- | :--- |
| Potencia base (VEL) | 75% | Velocidad alta para aprovechar los 8 min |
| Potencia fina (VEL_LENTO) | 40% | Movimientos de precision frente al arbol |
| Umbral de linea | 35 | Punto optico entre blanco (~60) y negro (~20) |
| Distancia obstaculo | 35 cm | Margen para iniciar esquive sin colision |
| T_ESPERA_ACK | 8000 ms | Timeout esperando respuesta BT de la garra |
| T_DEPOSITO | 10000 ms | Tiempo para que el Selector detecte y deposite |
| T_LATERAL_BOLA | 80 ms | Desplazamiento lateral entre posiciones de bola |
| Matriz ruta[5][2] | [obstaculo, direccion] | Permite invertir ruta para el regreso exacto |

## Protocolo Bluetooth

| Comando | Valor | Destino | Accion |
| :--- | :--- | :--- | :--- |
| CMD_SUBIR_ALTO | 10 | Garra (Slot 1) | Sube a fila alta y detecta/recoge |
| CMD_SUBIR_MEDIO | 11 | Garra (Slot 1) | Sube a fila media y detecta/recoge |
| CMD_SUBIR_BAJO | 12 | Garra (Slot 1) | Sube a fila baja y detecta/recoge |
| CMD_SOLTAR | 20 | Garra (Slot 1) | Abre pinza, pelota cae al selector |

| ACK | Valor | Origen | Significado |
| :--- | :--- | :--- | :--- |
| ACK_RECOGIO | 1 | Garra -> Base | Pelota recogida exitosamente |
| ACK_VACIO | 0 | Garra -> Base | No recogio (verde, vacio, o ya tiene pelota) |

El Selector no recibe comandos BT; es completamente autonomo.

## Cambios respecto a v2.0

1. Eliminado caracter basura "e32" al inicio del archivo
2. Funciones de movimiento verificadas contra Movimiento.nxc
3. Agregada correccion de trayectoria usando sensor trasero (S2)
4. Variable `total_pasos` inicializada explicitamente a 0
5. Puertos documentados con constantes IN_1 a IN_4 para claridad
6. Comentarios limpios sin caracteres especiales (compatibilidad NXC)

## Guia de Ajuste en Pista

1. **Patinaje en arranque:** Reducir VEL de 75 a 60. Si persiste, ajustar T_GIRO_90.
2. **No detecta linea:** Mucha luz ambiental -> bajar UMBRAL_LINEA a 30. Sombras -> subirlo a 40.
3. **Bateria baja (<7.2V):** Incrementar VEL a 85 para mantener torque nominal.
4. **Esquive incorrecto de alberca:** Ajustar T_GIRO_90 en pasos de 50 ms.
5. **No llega al arbol:** Aumentar T_AVANCE.
6. **Bolas mal alineadas:** Ajustar T_LATERAL_BOLA en pasos de 10 ms.
7. **ACK no llega (timeout):** Verificar slot BT correcto y que la garra este encendida primero.

## Secuencia de Encendido

1. Encender y ejecutar Garra_Esclavo -> pantalla muestra "ESPERANDO BT"
2. Encender y ejecutar Selector_Esclavo -> queda activo autonomamente
3. Encender y ejecutar Base_Maestro -> se conecta al confirmar BT
4. Posicionar el robot centrado en su cuadro inicial del beneficiadero
5. El programa arranca automaticamente al confirmar enlace BT
