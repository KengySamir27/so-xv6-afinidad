# Texto para publicar desde la cuenta de Jefferson
¿Qué ocurre si cuatro procesos compiten por un solo núcleo? En nuestro trabajo «Afinidad de CPU y estadísticas por núcleo sobre un planificador con prioridades y aging en xv6 multinúcleo», quisimos controlar dónde se ejecuta cada proceso y medir cómo se reparte el trabajo.

En Sistemas Operativos de la UNSCH modificamos el planificador de xv6-riscv. Una máscara indica qué núcleos puede usar cada proceso. Si un proceso listo espera, su prioridad efectiva aumenta. Para elegir al siguiente proceso, el planificador examina un candidato a la vez y vuelve a comprobarlo antes de ejecutarlo.

Con cuatro CPUs en QEMU, la misma carga terminó en 11 ticks cuando concentramos los procesos en CPU0 y en 3 cuando los repartimos. En otra prueba, el proceso de menor prioridad empezó en el tick 60 sin aging y en el 20 con aging. Las ocho pruebas de la batería pasaron. Las capturas muestran las condiciones de esas mediciones.

Equipo: Jefferson César Conde Flores, Kengy Samir Oré Mischa y Frank Felix Huicho Solier. Docente: Lizbeth Jaico Quispe.

https://github.com/Jefferson-Conde/so-xv6-afinidad

#SistemasOperativos #xv6 #GitHub #IngenieriaDeSoftware #UNSCH

---
Antes de publicar, seleccionar las menciones reales de ambos compañeros y de la docente (https://pe.linkedin.com/in/lizbeth-jaico-quispe-19a604170/) en el editor. Adjuntar 01-compilacion.png, 04-demo-carga.png y 09-resumen.png desde evidencias/. Jefferson realiza la publicación original. Kengy y Frank la comparten o interactúan con ella. Cada integrante entrega al docente el enlace de la publicación o de su republicación y verifica que sea visible. El enlace del perfil por sí solo no acredita la publicación.
