# Sustentación en vivo · duración propuesta de nueve minutos
Distribución sugerida para exponer. Los aportes acreditados por Git figuran por separado en la diapositiva 19.

1. Jefferson (2 min). Presentar el problema y abrir el README y el historial del equipo. En el anfitrión ejecutar `make qemu CPUS=4 TOOLPREFIX=riscv64-linux-gnu-` desde la raíz. Señalar los mensajes de los harts y el prompt de xv6.
2. Kengy (3 min). Abrir kernel/proc.c en otra ventana y localizar scheduler(). Señalar el filtro RUNNABLE, la máscara y el lock individual. Explicar que mientras se suelta el lock otra CPU puede escoger al candidato, de ahí la segunda comprobación. En xv6 ejecutar `cpustat` y `taskset -c 1 hog 10`. Leer la máscara 0x2 y la distribución de ticks.
3. Frank (3 min). En xv6 ejecutar `schedtest`. Interpretar T5 (concentración frente a reparto), T6 (primer turno con y sin aging) y T7 (migraciones después de fijar afinidad). Si aparece FAIL, conservar la salida y repetir solo el caso con schedtest seguido del número. Diferenciar las cifras de hoy de las capturas.
4. Cierre conjunto (1 min). Mostrar la diapositiva 19. Cada integrante abre su commit, explica el cambio y contesta una pregunta sobre él. Cerrar QEMU con Ctrl+A y después X.

Preguntas para ensayar: ¿por qué CPU1 usa la máscara 0x2? El bit 1 tiene valor 2. ¿Por qué puede haber una migración al fijar afinidad? El proceso puede comenzar en otro núcleo. ¿Qué establece T8? Los 48 procesos terminaron en esa corrida. ¿Qué queda fuera del experimento? Las ganancias de caché en hardware real.
