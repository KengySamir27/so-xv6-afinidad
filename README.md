# xv6-riscv: afinidad de CPU y estadísticas por núcleo sobre un planificador con prioridades y aging

Trabajo final del curso **Sistemas Operativos (IS-380)**, Escuela Profesional de Ingeniería de Sistemas,
Universidad Nacional de San Cristóbal de Huamanga (UNSCH), semestre 2026-II.
Docente: Ing. Lizbeth Jaico Quispe.

**Integrantes**

| Integrante | Cuenta de GitHub |
|---|---|
| Oré Mischa, Kengy Samir | [KengySamir27](https://github.com/KengySamir27) |
| Conde Flores Jefferson Cesar | [Jefferson-Conde](https://github.com/Jefferson-Conde) |
| Huicho Solier, Frank Felix |Frank1807|

## Descripción

xv6-riscv es el sistema operativo didáctico del MIT. Su planificador original recorre la tabla de procesos y ejecuta el
primero que esté listo: no tiene prioridades, no permite elegir en qué núcleo corre un proceso y no registra qué hizo cada núcleo.
Este proyecto lo extiende, sobre el commit base `9e3161a`, con:

- **Afinidad de CPU dura:** cada proceso tiene una máscara de CPUs permitidas (el bit *i* activa la CPU *i*).
- **Prioridades con aging:** prioridad base de 0 a 9 (mayor valor, mayor prioridad). Un proceso que espera sube un nivel de
  prioridad efectiva cada `AGING_TICKS` ticks, lo que evita la inanición.
- **Estadísticas por núcleo:** por proceso, ticks consumidos en cada CPU, última CPU y número de migraciones.
- **Planificador de dos fases:** nunca retiene más de un lock de proceso a la vez, por lo que elimina por construcción la
  retención y espera entre CPUs (una de las cuatro condiciones de deadlock de Coffman et al., 1971).
- **Cinco llamadas al sistema, tres herramientas de usuario y una batería de ocho pruebas automáticas (`schedtest`).**

El informe, la presentación, las evidencias y el material de reproducción están en [`documento_TrabajoFinal`](documento_TrabajoFinal/).

## Requisitos

Se necesita QEMU 7.2 o superior, herramientas cruzadas RISC-V, GCC del anfitrión, GNU Make, Perl y `bc`.

### Ubuntu y Debian

```bash
sudo apt update
sudo apt install git build-essential bc perl qemu-system-misc \
    gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu
```

### Arch Linux y Artix Linux (x86_64)

```bash
sudo pacman -Syu --needed base-devel git bc perl qemu-system-riscv \
    riscv64-linux-gnu-gcc riscv64-linux-gnu-binutils
```

Consulta las versiones instaladas con `qemu-system-riscv64 --version` y `riscv64-linux-gnu-gcc --version`.

## Instalación

```bash
git clone https://github.com/Jefferson-Conde/so-xv6-afinidad.git
cd so-xv6-afinidad
```

El commit base es `9e3161a`. Para ver exactamente qué se cambió:

```bash
git diff 9e3161a main --stat
```

## Compilación y ejecución

```bash
make qemu CPUS=4 TOOLPREFIX=riscv64-linux-gnu-
```

Esto compila el kernel y los programas de usuario, y arranca xv6 en QEMU con cuatro CPUs. Deben aparecer tres mensajes
`hart N starting` y el prompt `$`. Para salir de QEMU: `Ctrl+A` y luego `X`.

Para limpiar los archivos generados: `make clean`.

## Uso

### Llamadas al sistema agregadas

| N.º | Llamada | Función |
|---|---|---|
| 23 | `setpriority(pid, prio)` | Fija la prioridad base (0 a 9). `pid` 0 es el proceso que llama. |
| 24 | `setaffinity(pid, mask)` | Fija la máscara de CPUs permitidas. Debe incluir al menos una CPU activa. |
| 25 | `getptable(tab, max)` | Copia las estructuras `pstat` de los procesos en uso (prioridad, afinidad, estadísticas). |
| 26 | `getcpu()` | Devuelve el número de la CPU en que corre el proceso. |
| 27 | `setaging(on)` | Activa o desactiva el aging. |

Los argumentos inválidos devuelven `-1`.

### Herramientas de usuario (dentro de xv6)

| Comando | Qué hace |
|---|---|
| `taskset -c LISTA comando` | Ejecuta un comando restringido a las CPUs de la lista (por ejemplo `0,2-3`). |
| `taskset -p PID [LISTA]` | Consulta o cambia la afinidad de un proceso existente. |
| `cpustat [pid]` | Tabla con estado, prioridad base y efectiva, afinidad, última CPU, migraciones y ticks por CPU. |
| `hog <ticks>` | Mantiene la CPU ocupada y al terminar muestra en qué CPUs corrió. |
| `schedtest [n]` | Ejecuta la batería de ocho pruebas, o solo la prueba *n*. |

Demostración manual:

```
$ cpustat
$ taskset -c 1 hog 10
$ taskset -c 0 hog 100 &
$ taskset -c 0 hog 100 &
$ hog 100 &
$ cpustat
```

## Pruebas

Dentro de xv6, `schedtest` ejecuta toda la batería y `schedtest 3` ejecuta solo la prueba T3. Cada prueba imprime su
entrada, el resultado esperado, el obtenido y un veredicto PASS o FAIL. La última línea resume: `== Resultado: 8/8 pruebas PASS ==`.

| Caso | Qué verifica | Entrada | Resultado esperado |
|---|---|---|---|
| T1 | Afinidad dura a una CPU | `setaffinity(0, CPU1)` y 15 ticks muestreando `getcpu()` | Todas las muestras en CPU1 |
| T2 | Máscara de varias CPUs | 3 procesos con afinidad {CPU1, CPU2} | Muestras solo en CPU1 y CPU2 |
| T3 | Validación de argumentos | Máscara 0, CPU inexistente, pid inexistente, prioridad −1 y 10; luego casos válidos | Los 5 inválidos devuelven −1 y los 2 válidos devuelven 0 |
| T4 | Herencia en `fork` | Padre con prioridad 7 y afinidad {CPU2} | El hijo ve prioridad 7, afinidad 0x4 y corre en CPU2 |
| T5 | Carga concentrada frente a repartida | 4 procesos CPU-bound: todos a la CPU0 o uno por CPU | La carga concentrada tarda al menos el doble |
| T6 | Aging frente a inanición | Proceso H (prioridad 9) ocupa la CPU0 60 ticks; L (prioridad 0) espera | Sin aging L corre tras 50 ticks o más; con aging, antes de 40 |
| T7 | Migraciones entre CPUs | 6 procesos CPU-bound, sin afinidad y fijos | Sin afinidad hay migraciones; con afinidad dura, 0 tras fijar |
| T8 | Estrés del planificador | 3 rondas de 16 procesos con prioridad y afinidad aleatorias | Los 48 procesos terminan con código 0 y el sistema no se congela |

Las capturas de cada ejecución están en [`documento_TrabajoFinal/evidencias`](documento_TrabajoFinal/evidencias/).

## Archivos modificados o creados

17 archivos respecto del commit base (12 modificados y 5 nuevos): `kernel/{param.h, pstat.h, proc.h, proc.c, defs.h, main.c, trap.c,
syscall.h, syscall.c, sysproc.c}`, `user/{user.h, usys.pl, taskset.c, cpustat.c, hog.c, schedtest.c}` y `Makefile`.
La tabla detallada está en el informe.

## Reproducir desde un xv6 limpio

Los cambios están separados en cuatro parches que se aplican en orden sobre el commit base. Ver
[`documento_TrabajoFinal/documentacion_complementaria/guia_reproduccion.md`](documento_TrabajoFinal/documentacion_complementaria/guia_reproduccion.md).

## Limitaciones

- QEMU emula los núcleos pero no modela las cachés: se mide la distribución de la carga y las migraciones, no el beneficio de caché.
- Un tick dura unos 0,1 s, por lo que los tiempos tienen un error de ±1 tick.
- Cada prueba se ejecutó una vez; los valores cambian levemente entre corridas.
- No se maneja la inversión de prioridades.

## Créditos y licencia

Basado en [xv6-riscv](https://github.com/mit-pdos/xv6-riscv) de Cox, Kaashoek y Morris (MIT), distribuido bajo licencia MIT
(ver `LICENSE`). El `README` original de xv6 se conserva sin cambios.
