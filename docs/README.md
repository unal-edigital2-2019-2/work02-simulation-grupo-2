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

- Funciona con una máquina de estados que tiene cuatro estados:
  - `WAIT_FRAME_START`, que es el estado inicial, y es el encargado de detectar cuando se empieza un fotograma, detectando cambios en la señal `vsync`
  - `ROW_CAPTURE`, que es el estado en el cuál se capturan los datos de una fila de pixeles
  - `DATA_OUT_RANGE`, que es un estado que avisa cuando ya se tiene la totalidad de los datos de una fila
  - `DONE`, que avisa cuando un fotograma ya está completo y se puede pasar a capturar otro

