# Guía de reproducción

Esta guía permite repetir la compilación, la ejecución y las pruebas del proyecto.

## 1. Entorno

Instala las [dependencias para tu distribución](../../README.md#requisitos). La guía usa el prefijo `riscv64-linux-gnu-` y cuatro CPU virtuales.

## 2. Opción A: clonar el repositorio final

```bash
git clone https://github.com/Jefferson-Conde/so-xv6-afinidad.git
cd so-xv6-afinidad
make qemu CPUS=4 TOOLPREFIX=riscv64-linux-gnu-
```

Resultado esperado: el kernel arranca, aparecen `hart 1 starting`, `hart 2 starting` y `hart 3 starting`, y luego el prompt `$`.
Para salir de QEMU, `Ctrl+A` y luego `X`.

## 3. Opción B: reconstruir desde xv6 limpio con los parches

Los cambios están en cuatro parches que se aplican en orden sobre el commit base `9e3161a` de xv6-riscv.
Cada paso compila por sí solo.

```bash
git clone https://github.com/mit-pdos/xv6-riscv.git xv6-limpio
cd xv6-limpio
git switch -c proyecto-final 9e3161a
git apply RUTA/paso1-planificador.patch
git apply RUTA/paso2-syscalls.patch
git apply RUTA/paso3-herramientas.patch
git apply RUTA/paso4-pruebas.patch
git add -N -- Makefile kernel user
git diff --stat 9e3161a -- Makefile kernel user
make qemu CPUS=4 TOOLPREFIX=riscv64-linux-gnu-
```

Sustituye `RUTA` por la ruta absoluta de `documento_TrabajoFinal/documentacion_complementaria/parches/` en tu copia del repositorio del equipo. El resultado debe coincidir con el código del repositorio final: 17 archivos distintos del commit base, con 1067 líneas añadidas y 23 eliminadas.

El bloque anterior marca los archivos nuevos con `git add -N` para que se incluyan en la comparación.
Como alternativa a aplicar los cuatro archivos por separado, `todo-junto.patch` contiene la secuencia completa.

| Parche | Contenido |
|---|---|
| `paso1-planificador.patch` | Campos nuevos en `struct proc`, planificador de dos fases, aging, afinidad y estadísticas por CPU |
| `paso2-syscalls.patch` | `setpriority`, `setaffinity`, `getptable`, `getcpu` y `setaging` |
| `paso3-herramientas.patch` | Programas `taskset`, `cpustat` y `hog` |
| `paso4-pruebas.patch` | Programa `schedtest` con las ocho pruebas |

## 4. Ejecutar las pruebas automáticas

Dentro de xv6:

```
$ schedtest
```

Tarda cerca de un minuto. La última línea debe ser `== Resultado: 8/8 pruebas PASS ==`.
Para una sola prueba: `schedtest 1` a `schedtest 8`.

Criterio de éxito: las ocho pruebas muestran `resultado: PASS`. Los valores numéricos cambian levemente entre ejecuciones
(por ejemplo, los ticks de T5 o T8), pero los criterios de cada prueba deben cumplirse siempre.

## 5. Demostración manual

```
$ cpustat
$ taskset -c 1 hog 10
```

`hog` debe mostrar `aff=0x2` y todos los ticks en `CPU1`. Luego, la carga mixta:

```
$ taskset -c 0 hog 100 &
$ taskset -c 0 hog 100 &
$ hog 100 &
$ cpustat
```

Resultado esperado tras unos 10 segundos: los dos procesos fijos a la CPU 0 terminan con unos 50 ticks cada uno en la CPU0,
y el proceso libre reparte sus ticks entre las otras CPUs y acumula muchas migraciones.

## 6. Problemas frecuentes

- **QEMU no arranca o falta el compilador:** comprobar que `qemu-system-riscv64 --version` y `riscv64-linux-gnu-gcc --version` funcionan en el anfitrión.
- **T5 no muestra mejora:** esa prueba supone que QEMU ejecuta las CPUs en paralelo (un hilo del anfitrión por CPU).
  Si el equipo tiene un solo núcleo disponible, la mejora no se verá.
- **`git push` en WSL falla con `git-credential-manager.exe: not found`:** la ruta del asistente de credenciales de Windows no
  existe en tu equipo. Se puede usar un token personal de GitHub con el alcance `repo` (y `workflow` si el repositorio incluye
  archivos de `.github/workflows`).
- **Salir de QEMU:** `Ctrl+A` y luego `X`.
