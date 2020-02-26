# FrSkyDuino
## Prefacio
En este documento se asumen conocimientos generales básicos de electrónica, telecomunicaciones y programación en el lenguaje C++.
La abreviatura de bit es siempre b y la de bytes B.
## Contenido
- Contexto e intenciones
- Protocolo SBUS
	- Parámetros 
	- Formato de trama
- Dispositivos compatibles
	- FrSky Taranis SE 2019
	- FrSky X8R
	- FrSky R9M
	- FrSky R9
- Interfaz e implementación
- Código de ejemplos
- Modelos de conexión
- Licencia
## Contexto e intenciones
En la actualidad los drones se han colocado en el mercado de manera accesible. La popularización de estos vehículos teledirigidos ha facilitado el acceso a equipos anteriormente reservado a expertos y técnicos de la materia. Los sistemas domésticos completos son abundantes, mientras que las opciones para desarrolladores siguen estancadas. La mayoría de sistemas de comunicación empleados en estos aparatos utilizan tecnologías standard pero protocolos cerrados únicos para cada dispositivo.

Una de las marcas líderes en el mercado de media y alta gama es FrSky. Fabricante de multitud de productos destinados al control de vehículos teledirigidos. Emplea el protocolo SBUS para la conexión DTE-DCE y DCE-DTE, pero sus receptores cuentan con señales PWM y PPM lo que lo hace compatible con gran parte del sector.
Pese a esto, sigue sin existir una forma sencilla de utilizar Arduino, las placas de desarrollo líderes, junto a los dispositivos FrSky.

El objetivo  de esta librería es permitir esa conexión rápida y sin complicaciones que permitirá a los desarrolladores crear modelos teledirigidos de gran calidad.
## Protocolo SBUS
SBUS es un derivado del universalmente conocido TIA/EIA RS-232.
Permite encapsular 16 canales de alta precisión más 2 canales booleanos. Incorpora flags para activar modo aprueba de fallos e informar de tramas perdidas.
### Parámetros
La configuración es 8E2:
- 1 bit de inicio
- 8 bits de datos
- 1 bit de paridad par
- 2 bits de parada
El baudrate estipulado es 100000, no es ninguno de los estandard.
Utiliza lógica invertida, representa un bit '0' con un nivel de tensión alto y '1' con un nivel bajo.
La trama tiene 25 bytes, por tanto tarda 3 ms en transmitirse.
### Formato de trama
Los 16 primeros canales se representan con 11 bits, eso hace necesarios 22 bytes. La información de los canales está repartida en varios bytes, eso dificulta la implementación.
Los bytes se numeran comenzando por el 0, el último es el 24. Los bits se numeran desde el 0 empezando por la derecha. El bit 0 es  el menos significativo.
- Byte 0: Cabecera. 0x0F
- Byte 1-22: Datos de los 16 canales de 11 bits.
	- Canal 1: 1.0, 1.1, 1.2, 1.3, 1.4, 1.5, 1.6, 1.7, 2.0, 2.1, 2.2
	- Canal 2: 2.3, 2.4, 2.5, 2.6, 2.7, 3.0, 3.1, 3.2, 3.3, 3.4, 3.5
	- Canal 3: ...
- Byte 23: Dos canales de 1 bit y flags.
	- Bit 0: Canal nº 17
	- Bit 1: Canal nº 18
	- Bit 2: Trama perdida
	- Bit 3: Modo seguro
	- Bits 4-7: Siempre '0'
- Byte 24: Final de trama. 0x00

Estructura de la trama en detalle. Los canales se expresan como: Canal.Bit (empezando desde el bit y el canal 0). Los valores entre comillas son fijos.
|Byte | 7   | 6	  | 5	| 4	  | 3   | 2	  | 1	| 0	  |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 00  | '0' | '0' | '0' | '0' | '1' | '1' | '1' | '1' |
| 01  |00.07|00.06|00.05|00.04|00.03|00.02|00.01|00.00|
| 02  |01.04|01.03|01.02|01.01|01.00|00.10|00.09|00.08|
| 03  |02.01|02.00|01.10|01.09|01.08|01.07|01.06|01.05|
| 04  |02.09|02.08|02.07|02.06|02.05|02.04|02.03|02.02|
| 05  |03.06|03.05|03.04|03.03|03.02|03.01|03.00|02.10|
| 06  |04.03|04.02|04.01|04.00|03.10|03.09|03.08|03.07|
| 07  |05.00|04.10|04.09|04.08|04.07|04.06|04.05|04.04|
| 08  |05.08|05.07|05.06|05.05|05.04|05.03|05.02|05.01|
| 09  |06.05|06.04|06.03|06.02|06.01|06.00|05.10|05.09|
| 10  |07.02|07.01|07.00|06.10|06.09|06.08|06.07|07.06|
| 11  |07.10|07.09|07.08|07.07|07.06|07.05|07.04|06.03|
| 12  |08.07|08.06|08.05|08.04|08.03|08.02|08.01|08.00|
| 13  |09.04|09.03|09.02|09.01|09.00|08.10|08.09|08.08|
| 14  |10.01|10.00|09.10|09.09|09.08|09.07|09.06|09.05|
| 15  |10.09|10.08|10.07|10.06|10.05|10.04|10.03|10.02|
| 16  |11.06|11.05|11.04|11.03|11.02|11.01|11.00|10.10|
| 17  |12.03|12.02|12.01|12.00|11.10|11.09|11.08|11.07|
| 18  |13.00|12.10|12.09|12.08|12.07|12.06|12.05|12.04|
| 19  |13.08|13.07|13.06|13.05|13.04|13.03|13.02|13.01|
| 20  |14.05|14.04|14.03|14.02|14.01|14.00|13.10|13.09|
| 21  |15.02|15.01|15.00|14.10|14.09|14.08|14.07|14.06|
| 22  |15.10|15.09|15.08|15.07|15.06|15.05|15.04|15.03|
| 23  | '0' | '0' | '0' | '0' |Safe Mode|Frame Lost| 17 | 16 |
| 25  | '0' | '0' | '0' | '0' | '0' | '0' | '0' | '0' |

## Dispositivos compatibles
Este proyecto se considerará exitoso si los siguientes dispositivos tienen compatibilidad total con la librería.
No son los únicos compatibles con SBUS, ni de la marca ni del resto del mercado. Se agregarán características y notas de otros dispositivos al momento de probarlos.
### Características generales
Todos los transmisores y receptores de FrSky cuentan con un puerto Smart, ubicado por lo general cerca de Vin. Mediante este puerto los mandos y drones especiales pueden programar y comunicarse con el transmisor o receptor más aya de las capacidades permitidas por SBUS. El puerto SMART es una conexión serial full-duplex propiedad de FrSky.
El puerto SBUS es simplex, pero siempre es posible conectar el SBUS IN y el SBUS OUT a RX y TX para una conexión Half-Duplex.
### Dispositivos concretos
#### FrSky Taranis SE 2019
Mando, transmisor 2.4 GHz, programador SMART y programable USB.
Especificaciones eléctricas pendientes.
**Compatibilidad pendiente**
#### FrSky R8X
Receptor con MIMO y telemetría para emisores en la banda de los 2.4 GHz. Salida PWM de 8 canales.
Vin DC 3.5V~10V, consumo 100mA@5V, rango máximo 1.5km.
**Compatibilidad pendiente**
#### FrSky R9M
Transmisor de largo alcance y alta precisión. Utiliza la banda de los 900 MHz con una potencia TX variable de 25mw, 200mw o 500mw. Compatible con toda la serie R9.
Vin DC 4V~12.6V, se recomiendan baterías de litio 2S nunca 3S.
**Compatibilidad pendiente**
#### FrSky R9
Receptor con MIMO y telemetría para el emisor R9M. Salida PWM de 8 canales.
Vin DC 3.5V~10V, consumo 100mA@5V, rango máximo 10km.
**Compatibilidad pendiente**
## Interfaz e implementación
### Interfaz pública.
Esta librería añade una clase SBUS con una interfaz similar a el Serial por defecto.
#### Constructor
Requiere dos parámetros, el pin TX y el pin RX.
```C++
SBUS miSBUS(RX, TX);
```
```C++
SBUS miSBUS = new SBUS(RX, TX);
```
#### Método Begin
Inicia la comunicación. Necesario antes de enviar o recibir.
```C++
miSBUS.begin();
```
#### Método Close
Finaliza la comunicación. Se ejecutará automáticamente cuando se destruya el objeto.
```C++
miSBUS.close();
```
#### Método Receive
Actualiza el contenido del buffer RX.
No bloquea la ejecución hasta recibir la trama completa, solo almacena los datos hasta el momento. Los datos de entrada se actualizan cuando una trama completa es recibida.
```C++
miSBUS.receive();
```
#### Método Send
Envía el contenido del buffer TX.
```C++
miSBUS.send();
```
#### Método GetChanel
Devuelve el contenido del canal RX.
```C++
miSBUS.getChanel(canal);
```
#### Método SetChanel
Actualiza el contenido del canal TX, pero no envía los datos.
```C++
miSBUS.setChanel(canal, valor);
```
#### Método GetFrameLost
Devuelve el estado del bit Frame Lost de RX.
```C++
miSBUS.getFrameLost();
```
#### Método SetFrameLost
Actualiza el valor del bit Frame Lost de TX.
```C++
miSBUS.setFrameLost(valor);
```
#### Método GetSafeMode
Devuelve el estado del bit Safe Mode de RX.
```C++
miSBUS.getSafeMode();
```
#### Método SetSafeMode
Actualiza el valor del bit Safe Mode de TX.
```C++
miSBUS.setSafeMode(valor);
```

### Constantes
Para facilitar el uso del hardware anteriormente citado, se han mapeado del canales que usa cada elemento de los mandos.

|Constante | Control asociado   |
|:---:|:---:|
| X9D_STICK_LEFT_X | Eje horizontal del joystick izquierdo en X9D |
| X9D_STICK_LEFT_Y | Eje vertical del joystick izquierdo en X9D |
| X9D_STICK_RIGHT_X | Eje horizontal del joystick derecho en X9D |
| X9D_STICK_RIGHT_Y | Eje vertical del joystick derecho en X9D |

### Detalles de la implementación.
La comunicación se realiza mediante SoftwareSerial ya que permite lógica inversa sin necesidad de componentes físicos para. El Serial por Hardware necesitaría inversores ya que no acepta lógica inversa.
Los datos son temporalmente almacenados en arrays de 25 chars, para contener los 25 bytes de una trama. Se tiene uno para RX y otro para TX. El buffer RX se actualiza al ejecutar el método read. Cuando el buffer RX se actualiza por completo la trama se parsea para sacar el valor de cada canal a fin de poder devolverlo con mayor velocidad. El buffer de TX no se actualiza al modificar un canal o bit, sino al momento de enviar los datos.

## Código de ejemplos
### Mostrar por monitor serial el valor de los canales
```C++
void Setup() ...
```
### Mover algunos servos y enviar potenciómetros
```C++
void Setup() ...
```
## Modelos de conexión
Imágenes y esquemáticos de la conexión física y lógica
### Mínimo para recibir y enviar telemetría
### Utilización de las señales PWM en Arduino o actuadores
## Licencia
Todo el código está bajo GNU GPL 3.
Toda persona puede obtener modificar o distribuir este código proporcionando siempre la fuente. Más información en el archivo de licencia.

Si vas a utilizar esta librería en un proyecto, me gustaría estar al corriente, es un honor servir a la comunidad.
