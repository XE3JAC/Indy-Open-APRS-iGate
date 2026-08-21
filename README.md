# Indy Open APRS — Documentación iGate y Tracker

**Firmware APRS abierto para ESP8266 basado en la plataforma de hardware TA1KNN**

Indy Open APRS es un firmware desarrollado para una plataforma APRS formada por **ESP8266 / NodeMCU + Nano TNC basada en ATmega**. El proyecto proporciona una interfaz web para configuración, diagnóstico, mensajería APRS, telemetría, actualización OTA, respaldo y operación como iGate.

- **Proyecto:** Indy Open APRS
- **Firmware actual:** XE3JAC
- **Firmware TNC actual:** XE3JAC
- **Firmware TNC original:** SQ9MDD
- **Hardware de referencia:** TA1KNN

# WEBFLASHER - Indy Open iGate o Indy Open Tracker
**https://xe3jac.github.io/Indy-Open-APRS-iGate/webflasher**

# WEBFLASHER - TNC
**https://xe3jac.github.io/Indy-Open-APRS-iGate/tnc-webflasher**
# Indy Open iGate Manual
**https://xe3jac.github.io/Indy-Open-APRS-iGate/**

---

## Contenido

- [1. Descripción general](#1-descripción-general)
- [2. Arquitectura del sistema](#2-arquitectura-del-sistema)
- [3. Perfiles de firmware: iGate y Tracker](#3-perfiles-de-firmware-igate-y-tracker)
- [4. Accesos y seguridad](#4-accesos-y-seguridad)
- [5. Configuración general](#5-configuración-general)
- [6. Mensajes APRS](#6-mensajes-aprs)
- [7. Estado y contactos](#7-estado-y-contactos)
- [8. Actualización OTA](#8-actualización-ota)
- [9. Respaldo y restauración](#9-respaldo-y-restauración)
- [10. Registro](#10-registro)
- [11. Pantalla OLED](#11-pantalla-oled)
- [12. Instalación inicial por USB](#12-instalación-inicial-por-usb)
- [13. Nano TNC basada en ATmega](#13-nano-tnc-basada-en-atmega)
- [14. Instalación mediante Web Flasher](#14-instalación-mediante-web-flasher)
- [15. Historia y créditos](#15-historia-y-créditos)
- [16. Recomendaciones](#16-recomendaciones)

---

## 1. Descripción general

Indy Open APRS dispone de una interfaz web desde la que se pueden configurar y supervisar las principales funciones del sistema.

Las secciones disponibles son:

- **Accesos**
- **Configuración**
- **Mensajes**
- **Estado**
- **Contactos**
- **Actualizar**
- **Respaldo**
- **Registro**
- **Ayuda**

Los valores mostrados en las capturas —indicativos, direcciones IP, coordenadas, rutas, servidores y lecturas— son ejemplos de una estación configurada.

### Primera puesta en marcha recomendada

1. Configurar las credenciales del punto de acceso y del acceso OTA.
2. Seleccionar el modo de operación.
3. Configurar Wi-Fi.
4. Configurar APRS-IS cuando el modo seleccionado lo requiera.
5. Configurar indicativo, posición, símbolo, comentario, intervalo de baliza y ruta RF.
6. Revisar TNC, NTP, offset UTC y telemetría.
7. Guardar la configuración.
8. Comprobar el funcionamiento en **Estado** y **Registro**.
9. Descargar un respaldo cuando la instalación sea estable.

---

## 2. Arquitectura del sistema

La arquitectura de referencia separa el procesamiento principal y las funciones de radio:

**Radio ⇄ Nano TNC / ATmega ⇄ ESP8266 / NodeMCU ⇄ Wi-Fi / APRS-IS**

El ESP8266 ejecuta Indy Open APRS y se encarga de la interfaz web, red, APRS-IS, OLED, telemetría, registro y configuración.

La Nano TNC utiliza su propio firmware y realiza las funciones TNC/KISS necesarias para la comunicación APRS por RF.

---


## 3. Perfiles de firmware: iGate y Tracker

Indy Open APRS dispone de dos perfiles principales para el ESP8266. Comparten la plataforma **ESP8266 / NodeMCU + Nano TNC basada en ATmega**, pero sus funciones son diferentes.

| Función | Indy Open iGate | Indy Open Tracker |
|---|---|---|
| APRS por RF | Sí | Sí |
| APRS-IS | Sí | No |
| Balizas | RF y APRS-IS | RF mediante GPS |
| Mensajes | RF, Internet o ambos; hasta 7 programados | Sólo RF; sin programación |
| Digipeater | New-N limitado + WIDE1-1 opcional | Fill-in WIDE1-1 |
| GPS | No es la base de operación descrita | Sí |
| SmartBeaconing | No documentado en este perfil | Sí |
| Indy Open APRS Notify | Sí | No |

### 3.1 Indy Open iGate

Indy Open iGate recibe APRS por RF, lo reporta a APRS-IS y transmite balizas por RF y APRS-IS. Las salidas son independientes: RF puede transmitir sin APRS-IS validado; APRS-IS requiere `logresp ... verified`.

**Digipeater iGate**

- **Activar Digipeater APRS** es el interruptor principal.
- Atiende `WIDE2-1` y `WIDE2-2`.
- No repite `WIDE3-n` ni superiores.
- Conserva digipeaters ya usados y evita duplicados y loops.
- **Permitir Digipeater de Relleno WIDE1-1** sólo funciona si el Digipeater principal está activo.
- La Ruta RF de las balizas propias no determina qué tráfico procesa el DIGI.

**RF → APRS-IS**

Las tramas RF válidas se publican añadiendo `,qAR,<INDICATIVO-IGATE>`. Para evitar bucles se omiten TCPIP/TCPXX, NOGATE, RFONLY, rutas qxx, consultas genéricas, tráfico propio repetido y duplicados.

**Indy Open APRS Notify**

1. Pulsa **Generar Código**.
2. Abre el bot **Indy Open APRS Notify** en Telegram.
3. Envía `/link CODIGO`.

El código dura 10 minutos. La `device_key` es interna y nunca se muestra. El ESP8266 no utiliza Telegram directamente: las notificaciones pasan por Indy Open Notify Relay.

### 3.2 Indy Open Tracker

Indy Open Tracker genera balizas APRS de posición y telemetría por **RF** a partir del GPS. **No usa APRS-IS ni Indy Open Notify/Telegram.**

**Red y portal**

- **Activar Wi-Fi para configuración** abre el portal local mediante Wi-Fi.
- SSID y contraseña bastan para DHCP.
- IP estática, máscara, gateway y DNS sólo se usan para IP fija.
- El Access Point `IndyOpenTracker` permanece disponible para configuración.
- OTA conserva configuración e historiales y debe utilizar únicamente un binario Tracker.

**GPS**

Baudios disponibles: `4800`, `9600`, `19200`, `38400`, `57600` y `115200`. El valor predeterminado es **9600**. Al guardar el valor, el puerto GPS se reabre inmediatamente.

Las balizas normales requieren fijación GPS válida.

**Ubicación de Prueba**

- Vive sólo en RAM durante 30 minutos.
- Se utiliza únicamente para **Baliza de Prueba RF**.
- Desaparece al reiniciar.
- Desaparece al recibir GPS real.

**Balizas y Ruta RF**

Indicativo APRS, símbolo, comentario, Estado APRS, intervalo y Ruta RF construyen las balizas. La ruta inicial es:

`WIDE1-1,WIDE2-1`

Una ruta vacía transmite directa.

**SmartBeaconing**

Controla velocidades, intervalos, giro, tiempo estacionado y símbolo estacionado. Si se desactiva, se utiliza el intervalo fijo.

**Digipeater Tracker**

**Activar Digipeater APRS** es el único control. Activado funciona exclusivamente como **Fill-in WIDE1-1**:

- Repite una solicitud `WIDE1-1` pendiente.
- Conserva el resto del path.
- No consume `WIDE2-1`, `WIDE2-2` ni otros `WIDEn-N`.
- No funciona como New-N.
- Mantiene protección contra trama propia, loops y duplicados de 30 segundos.
- Si coinciden una baliza propia y un DIGI pendiente, el DIGI tiene prioridad.

**Mensajes y estadísticas**

Los mensajes del Tracker son **sólo RF** y no hay programación de mensajes.

Las estadísticas RF conservan estaciones, paquetes, DIGI, DX, nivel RX, audio dBV y última RX.

---

## 4. Accesos y seguridad

![Pantalla Accesos y seguridad](images/manual_01.png)

La pantalla **Accesos** permite configurar:

- **Nombre AP:** nombre de la red inalámbrica creada por el equipo.
- **Contraseña AP:** contraseña del punto de acceso.
- **Usuario OTA:** usuario para operaciones protegidas.
- **Contraseña OTA:** contraseña asociada al acceso OTA.

### Credenciales predeterminadas

En una instalación nueva, las credenciales predefinidas son:

| Acceso | Usuario / SSID | Contraseña |
|---|---|---|
| **Acceso OTA** | `admin` | `indygate` |
| **Wi-Fi / Access Point** | `admin` | `indygate` |

> **Importante:** estas son las credenciales iniciales. Por seguridad, se recomienda cambiarlas desde la sección **Accesos** después de la primera configuración.

> **Precaución:** conserve las credenciales OTA en un lugar seguro.

---

## 5. Configuración general

![Configuración de red, APRS-IS y estación](images/manual_02.png)

### Modo de operación

El selector determina la función principal del equipo. La interfaz incluye modos como:

- **iGate ON (APRS-IS)**
- **Digipeater**
- **Tracker**

### Red

- Wi-Fi SSID
- Wi-Fi password
- IP estática o DHCP
- Máscara
- Gateway
- DNS

### APRS-IS

- Indicativo APRS-IS
- Passcode
- Servidor
- Puerto
- Filtro APRS-IS

### Estación

- Latitud APRS
- Longitud APRS
- Símbolo
- Comentario
- Estado APRS
- Minutos entre beacons
- Ruta RF
- Baliza de prueba
- Posición de prueba

![Configuración de TNC, tiempo, sistema y telemetría](images/manual_03.png)

### TNC y tiempo

- Baud rate TNC
- Retardo DIGI
- Reinicio automático
- Offset UTC
- Servidor NTP
- Potencia del AP Wi-Fi

### Sistema y telemetría

La interfaz permite habilitar o deshabilitar funciones como beep, orientación OLED, reenvío APRS-IS → RF, telemetría, monitor AFSK y opciones relacionadas con PTT.

---

## 6. Mensajes APRS

![Pantalla de mensajes APRS](images/manual_04.png)

![Selección de ruta de mensajes](images/manual_05.png)

Para enviar un mensaje:

1. Introduzca el indicativo en **Destino APRS**.
2. Escriba el texto.
3. Seleccione la ruta:
 - Por RF
 - Por Internet
 - RF + Internet
4. Pulse **Enviar mensaje**.
5. Consulte el historial para verificar el estado.

El historial puede mostrar estados como **Pendiente**, **Confirmado**, **Reenviado** o **Sin respuesta**.

---

## 7. Estado y contactos

![Estado general del sistema](images/manual_06.png)

La pantalla **Estado** permite revisar:

- Tiempo encendido
- Wi-Fi
- Estado APRS-IS
- TNC/KISS
- RX RF
- TX RF
- PMS5003/PMS7003
- BME280
- Voltaje
- OLED

![Contactos y actividad RF](images/manual_07.png)

La sección **Contactos** muestra actividad reciente por RF e información de estaciones recibidas.

---

## 8. Actualización OTA

![Actualización OTA](images/manual_08.png)

La actualización OTA permite instalar una nueva versión del firmware del ESP8266 desde la interfaz web.

1. Seleccione el archivo `.bin`.
2. Verifique que corresponde al ESP8266.
3. Pulse **Instalar actualización**.
4. No desconecte la alimentación ni cierre la página.
5. Espere hasta que el proceso termine.

> Utilice únicamente firmware creado para el hardware correspondiente.

---

## 9. Respaldo y restauración

![Respaldo y restauración](images/manual_09.png)

### Descargar respaldo

Genera una copia de la configuración actual.

### Restaurar respaldo

Permite seleccionar un respaldo previamente generado y sustituir la configuración actual.

### Borrar configuraciones

Elimina configuración, mensajes, registros y contactos, pero conserva el firmware.

> **Precaución:** descargue un respaldo antes de borrar la configuración.

---

## 10. Registro

![Registro del sistema](images/manual_10.png)

El registro facilita el diagnóstico. Los filtros disponibles incluyen:

- Todo
- Sistema
- APRS-IS
- RF / DIGI
- Errores

También se puede consultar `log.txt`.

---

## 11. Pantalla OLED

![Ayuda integrada y referencia OLED](images/manual_11.png)

El equipo utiliza una OLED de **0.96 pulgadas, 128×64 píxeles**.

Las tres pantallas de operación son **fijas**. Para cambiar de una pantalla a la siguiente se debe pulsar el **switch o botón físico del sistema**. Después de la tercera pantalla se vuelve a la primera.

Las vistas muestran información como:

1. Resumen del sistema, indicativo, IP y APRS-IS.
2. Estado TNC/KISS y actividad RF.
3. Sensores PMS/BME e información I2C/OLED.

La pantalla OLED también puede mostrar un bitmap monocromático de **128×64 píxeles** durante el arranque.

---

## 12. Instalación inicial por USB

El firmware del ESP8266 y el firmware de la Nano TNC son independientes.

### ESP8266 / NodeMCU

1. Conecte el ESP8266 a la computadora con un cable USB de datos.
2. Identifique el puerto COM.
3. Seleccione el firmware correspondiente.
4. Inicie la programación.
5. Espere hasta que termine correctamente.
6. Reinicie el ESP8266.
7. Acceda al AP de Indy Open APRS y realice la configuración inicial. Las credenciales predeterminadas son `admin` / `indygate`.

> No desconecte el USB mientras se está escribiendo el firmware.

Para la primera conexión recuerde las credenciales predefinidas: **usuario/SSID `admin` y contraseña `indygate`**.

---

## 13. Nano TNC basada en ATmega

La Nano TNC realiza la interfaz APRS por RF y utiliza firmware independiente.

Procedimiento general:

1. Conecte la Nano por USB.
2. Identifique el puerto serie.
3. Verifique el microcontrolador/bootloader correspondiente.
4. Seleccione el firmware TNC.
5. Programe la Nano.
6. Espere la confirmación.
7. Vuelva a instalarla en el sistema.
8. Compruebe que Indy Open APRS muestra el TNC activo.

La publicación histórica de la plataforma acredita el firmware TNC a **SQ9MDD**.

---

## 14. Instalación mediante Web Flasher

El Web Flasher permite realizar la instalación inicial del firmware del ESP8266 directamente desde un navegador compatible.

### Requisitos

- Computadora
- Navegador compatible con puerto serie
- Cable USB de datos
- ESP8266 / NodeMCU
- Web Flasher oficial de Indy Open APRS

### Procedimiento

1. Conecte el ESP8266 por USB.
2. Abra el Web Flasher.
3. Pulse el botón de conexión/instalación.
4. Seleccione el puerto serie.
5. Autorice la conexión.
6. Seleccione la versión correcta del firmware.
7. Inicie la instalación.
8. Espere hasta el 100 %.
9. Reinicie el equipo.
10. Realice la configuración inicial.

### Web Flasher vs OTA

| Método | Conexión | Uso |
|---|---|---|
| Web Flasher | USB | Primera instalación o recuperación |
| OTA | Red | Actualización de un equipo ya operativo |
| Programación USB tradicional | USB | Instalación o recuperación manual |

> La Nano TNC es independiente y no se debe asumir que puede programarse desde el mismo Web Flasher del ESP8266.

---

## 15. Historia y créditos

Indy Open APRS utiliza como plataforma de referencia la arquitectura de hardware de la **PCB TA1KNN**.

La publicación histórica del proyecto APRS iGATE acredita:

| Elemento | Autor / referencia |
|---|---|
| Firmware actual | **XE3JAC** |
| Firmware actual TNC | **XE3JAC** |
| PCB / hardware | **TA1KNN** |
| Firmware original TNC | **SQ9MDD** |

Indy Open APRS es un firmware posterior y propio para ESP8266. 
La referencia a **PCB TA1KNN** describe la plataforma física utilizada como base y no implica autoría del firmware Indy Open APRS.

---

## 16. Recomendaciones

- Configure primero accesos y red.
- Cambie las credenciales predeterminadas `admin` / `indygate` después de la primera configuración.
- Verifique el modo de operación antes de transmitir.
- Revise posición, símbolo, comentario y ruta RF.
- Utilice **Estado** y **Registro** para diagnóstico.
- Haga un respaldo después de alcanzar una configuración estable.
- Mantenga alimentación estable durante OTA o Web Flasher.
- No mezcle el firmware del ESP8266 con el firmware de la Nano TNC.

---

## Créditos

**Indy Open APRS** 
Indy Open Project by **XE3JAC**

Documentación basada en la interfaz y hardware de referencia del proyecto.


## ☕ Apoya el proyecto

Indy Open es un proyecto comunitario desarrollado por interés en la radioafición, la experimentación y el hardware/software abierto.

Si quieres ayudar a continuar desarrollando placas, probando hardware y creando nuevas funciones, podrás apoyar voluntariamente el proyecto.

❤️ **Las donaciones son completamente opcionales.**

[![Donar con PayPal](https://www.paypalobjects.com/es_XC/MX/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/donate?hosted_button_id=2RX7AT3HP4RNG)

**¡Gracias por apoyar Indy Open APRS! 73 de XE3JAC**
