# Tratamiento-de-audio-digital-con-interfaz-AXI-Stream-e-implementaci-n-en-FPGA


Codigo en VHDL para la implementación de un modificador de audio, con tres etapas, control de volumen, filtrado digital y efecto de audio Delay con entrada y salida por PMOD I2S2

Se tiene un muestreo a 512x por medio de ADC sigma-delta y decimado a 44.1 kHz, se obtienen los datos a traves de una interfaz de audio I2S y la modificación de los mismos por una interfaz Maestro-Esclavo AXI-Stream, esta interfaz tiene un reloj maestro de 22.579 MHz en el que son sincronizados tres procesos, se usan DSP inferidos a traves de codigo de descripción VHDL y tambien memoria de bloques BRAM, la resolución de dato para el audio es un paquete formado por dos palabras de 24 bits codificadas en complemento a 2 de caracter entero, canal izquierdo y derecho respectivamente, se cuenta con tres filtros FIR manejados por modulos de Xilinx Fir_compiler 7.2 en funcionamiento de pasa bajas, pasa altas y pasa banda, estos contienen una ganacia adicional a la salida por lo que no se estandariza el nivel de sonido en su uso, el modulo de efecto de audio delay tiene una capacidad de generar este retrazo desde los 10 ms hasta 370 ms.

