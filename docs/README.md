# Documentación 1: Simulación del módulo de la camara.

**Juan Carrillo** [cedula]

**Sebastian Betancourt** 1016089847

**Sara Ramos** 1020837947

Para esta segunda entrega, se nos pidió hacer lo siguiente:

- Crear el módulo `cam_read.v`, encargado de hacer dos tareas:

  - Capturar los datos generados por la cámara en formato RGB565
  - Transformar el formato RGB565 a RGB332

- Instanciar el módulo dentro del proyecto completo `test_cam.xise`, de tal forma que fuese posible simular y probar el funcionamiento del módulo.

La creación del módulo `cam_read.v` se hizo basada en un otro módulo utilizado para la cámara OV7670 al cuál se le hicieron cietras modificaciones:

- Tiene un diagrama de caja negra que es el siguiente:

<img src="https://github.com/unal-edigital2-2019-2/work02-simulation-grupo-2/blob/master/docs/figs/caja_negra.png?raw=true" width = "750" >
- Y funciona de la siguiente manera:

Teniendo en cuenta la imagen, `pclk`, `vsync` y `href` son puertos que reciben las señales generadas por la cámara para la sincronización de los datos. `px_data` recibe los datos generados por la cámara. `mem_px_addr` envía a la memoria ram la dirección en la que debe ir  ubicada cierta información de los pixeles. `mem_px_data` es el puerto que envía los datos. `px_wr` es una señal que autoriza la escritura de datos en la memoria ram.

- Funciona con una máquina de estados que tiene cuatro estados:
  - `WAIT_FRAME_START`, que es el estado inicial, es el encargado de detectar cuando se empieza un fotograma, observando cambios en la señal `vsync`
  - `ROW_CAPTURE`, es el estado en el cuál se capturan los datos de una fila de pixeles.
  - `DATA_OUT_RANGE`, es un estado que avisa cuando ya se tiene la totalidad de los datos de una fila.
  - `DONE`, avisa cuando un fotograma ya está completo y se puede pasar a capturar otro.

Cada pixel tiene dos Bytes de información, los cuales la cámara envía a traves de un puerto de un Byte, por lo que la información completa de un pixel se recibe luego de dos ciclos de reloj. Para hacer el almacenamiento y envío correcto de esta información se crearon dos elementos en el módulo:

- Un registro temporal `temp_rgb`, que tiene el tamaño suficiente para almacenar un pixel completo.
- Una señal llamada `pixel_half`, la cuál está encargada de mostrar 0 si el registro `temp_rgb` tiene la información de medio pixel, o 1 si tiene la información de un pixel completo, estando lista para ser enviada.

Una vez ya está la información de un pixel lo siguiente es hacer una compresión de la imágen, pasandola de `RGB565` a `RGB332`, esto fue hecho en código a traves de una concatenación, tomando partes específicas del registro `temp_rgb` y asignandolas a la salida `mem_px_data`, que las envía a la memoria ram de dos puertos:

```verilog
mem_px_data = {temp_rgb[15:13], temp_rgb[10:8],temp_rgb[4:3]};
```


La máquina de estados funciona teniendo en cuenta las señales de control de la cámara `vsync` y `href`, las cuales se encargan de mostrar los momentos en los que la cámara está haciendo captura de imágenes.

El módulo está configurado para hacer captura de imágenes en tamaño `QQCIF` (120 x 160). Si se desea usarlo para otro formato como `QCIF` O `CIF`, se deben configurar los parámetros `Maxwidthimage` y `Maxlengthimage` a los valores deseados.

## Proceso de simulación

Habiendo instanciado el módulo con el resto del hardware, procedimos a hacer la simulación.

La simulación fue realizada gracias a una herramienta online creada por Eric Eastwood. Disponible en https://ericeastwood.com/lab/vga-simulator/. Este simulador cuenta con todas las consideraciones necesarias para simular,  requiere un archivo del cuál toma los datos para el primer frame de imagen, los siguientes frames ya dependen de la información otorgada por la cámara, que en este caso será iformación contenida en el testbench.

Al hacer la simulación y dejarla correr cierto tiempo, empezó a generar imágenes, las cuales se muestran a continuación:

<img src="https://github.com/unal-edigital2-2019-2/work02-simulation-grupo-2/blob/master/docs/figs/double.png?raw=true" width = "750" >

En esta imágen se puede ver que el software generó en el primer frame la cámara, con los colores azul y morado, los cuales se definen poniendolos como información en hexadecimal en el documento `image.mem`. El segundo frame contiene la imagen que debe salir de la cámara, la cuál se encuentra definida y puede ser modificada en el testbench.

Al acercar la imágen del segundo frame podemos ver lo siguiente:

<img src="https://github.com/unal-edigital2-2019-2/work02-simulation-grupo-2/blob/master/docs/figs/red_close.png?raw=true" width = "750" >

Esto lo interpretamos como puntos que quedaron sin información en la memoria ram. Se puede ver a traves de ellos y lo que se ve detras es la imágen del frame anterior.

## Protocolo UART

UART son las siglas de *Universal Asynchronous Receiver-Transmitter* que traducido es **Transmisor Receptor Asíncrono Universal**. Este protocolo es utilizado para comunicación serie entre dispositivos digitales. El UART toma bytes de datos y transmite los bits individuales de forma secuencial. En el destino, un segundo UART reensambla los bits en bytes completos. Las funciones principales de chip UART son: manejar las interrupciones de los dispositivos conectados al puerto serie y convertir los datos en formato paralelo, transmitidos al bus de sistema, a datos en formato serie, para que puedan ser transmitidos a través de los puertos y viceversa.
