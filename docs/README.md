# Documentación 1: Simulación del módulo de la camara.

**Juan Carrillo** [cedula]

**Sebastian Betancourt** 1016089847

**Sara Ramos** [cedula]

Para esta primera entrega, se nos pidió hacer lo siguiente:

- Crear el módulo `cam_read.v`, encargado de hacer dos tareas:

  - Capturar los datos generados por la cámara en formato RGB565
  - Transformar el formato RGB565 a RGB332

- Instanciar el módulo dentro del proyecto completo `test_cam.xise`, de tal forma que fuese posible simular y probar el funcionamiento del módulo.

La creación del módulo `cam_read.v` se hizo basada en un otro módulo utilizado para la cámara OV7670 al cuál se le hicieron cieras modificaciones, funciona de la siguiente manera:

Tiene un diagrama de caja negra que es el siguiente:

<img src="https://github.com/unal-edigital2-2019-2/work02-simulation-grupo-2/blob/master/docs/figs/caja_negra.png?raw=true" width = "750" >
Teniendo en cuenta la imagen, `pclk`, `vsync` y `href` son puertos que reciben las señales generadas por la cámara para la sincronización de los datos. `px_data` recibe los datos generados por la cámara. `mem_px_addr` envía a la memoria ram la dirección en la que debe ir  ubicada cierta información de los pixeles. `mem_px_data` es el puerto que envía los datos. `px_wr` es una señal que autoriza la escritura de datos en la memoria ram.

- Funciona con una máquina de estados que tiene cuatro estados:
  - `WAIT_FRAME_START`, que es el estado inicial, y es el encargado de detectar cuando se empieza un fotograma, detectando cambios en la señal `vsync`
  - `ROW_CAPTURE`, que es el estado en el cuál se capturan los datos de una fila de pixeles
  - `DATA_OUT_RANGE`, que es un estado que avisa cuando ya se tiene la totalidad de los datos de una fila
  - `DONE`, que avisa cuando un fotograma ya está completo y se puede pasar a capturar otro

Cada pixel tiene dos Bytes de información, los cuales la cámara envía a traves de un puerto de un Byte, por lo que la información completa de un pixel se recibe luego de dos ciclos de reloj. Para hacer el almacenamiento y envío correcto de esta información se hicieron se crearon dos cosas en el módulo:

- Un registro temporal `temp_rgb`, que tiene el tamaño suficiente para almacenar un pixel completo
- Una señal llamada `pixel_half`, la cuál está encargada de mostrar 0 si el registro `temp_rgb` tiene la información de medio pixel, o 1 si tiene la información de un pixel completo, estando lista para ser enviada.

Una vez ya está la información de un pixel lo siguiente es hacer una compresión de la imágen, pasandola de `RGB565` a `RGB332`, esto fue hecho en código a traves de una concatenación, tomando partes específicas del registro `temp_rgb` y asignandolas a la salida `mem_px_data`, que las envía a la memoria ram de dos puertos:

```verilog
mem_px_data = {temp_rgb[15:13], temp_rgb[10:8],temp_rgb[4:3]};
```


La máquina de estados funciona teniendo en cuenta las señales de control de la cámara `vsync` y `href`, las cuales se encargan de mostrar los momentos en los que la cámara está haciendo captura de imágenes.

El módulo está configurado para hacer captura de imágenes en tamaño `QQCIF` (120 x 160). Si se desea usarlo para otro formato como `QCIF` O `CIF`, se deben configurar los parámetros `Maxwidthimage` y `Maxlengthimage` a los valores deseados.
