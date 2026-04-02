# 📖 Get_next_line

[🇪🇸 Español](#español) | [🇬🇧 English](#english)

---

## Español

**Get_next_line** es el proyecto de la escuela 42 que aborda la lectura secuencial de archivos línea por línea, introduciéndonos de lleno en el manejo avanzado de punteros, la gestión de memoria dinámica y el uso fundamental de las **variables estáticas**.

### 🎯 De qué trata el proyecto
Imagina un archivo de texto enorme o una entrada de datos que necesitas procesar. Leer todo su contenido de un golpe puede ser fatal en términos de memoria o, en algunos casos, completamente imposible.
Para solucionarlo, necesitas una función que lea **solo una línea a la vez** cada vez que sea invocada.

El reto: Diseñar una función `char *get_next_line(int fd);` que sea capaz de recordar exactamente dónde se quedó leyendo en el archivo (asociado a su file descriptor) después de cada llamada, lográndolo todo sin generar fugas de memoria (memory leaks).

### ⚙️ Uso y Compilación

Este proyecto no cuenta (o no debería usar por norma) un `Makefile` como tal, pero para usar y testear el código junto con tus propios archivos `main.c`, debes añadir la flag especial `-D BUFFER_SIZE=n` que determina cuántos bytes intentará leer `read()` a la vez:

```bash
gcc -Wall -Wextra -Werror -D BUFFER_SIZE=42 main.c get_next_line.c get_next_line_utils.c -o gnl
./gnl
```

**Parámetros detallados a considerar:**
- `BUFFER_SIZE`: Cantidad de bytes en memoria que la función lee en cada llamada al sistema `read()`. Puede ser 1 byte o 9999 bytes. Tu programa debe funcionar sin importar qué valor tome.
- `fd` (File Descriptor): El número que el sistema operativo asigna a un archivo abierto o a una entrada estándar (como `0` para `stdin`).

**Ejemplo de prueba en un `main.c`:**
```c
#include <fcntl.h>
#include <stdio.h>
#include "get_next_line.h"

int main(void)
{
    int fd = open("archivo.txt", O_RDONLY);
    char *line;

    while ((line = get_next_line(fd)) != NULL)
    {
        printf("%s", line);
        free(line);
    }
    close(fd);
    return (0);
}
```

### 📂 Estructura del Código

El proyecto limita de manera estricta la cantidad de archivos y cómo están fragmentadas las funciones, dejándolo así:

```text
Get_next_line/
├── get_next_line.c           # Función troncal y extracción de cada línea leída
├── get_next_line_utils.c     # Utilidades para manipulación de strings (strlen, strchr, strjoin...)
├── get_next_line.h           # Cabecera: prototipos, librerías y definición local del BUFFER_SIZE
├── get_next_line_bonus.c     # Archivo extra: lógica para múltiples descriptores simultáneos
├── get_next_line_utils_bonus.c# Archivo extra: utilidades de apoyo al bonus
└── get_next_line_bonus.h     # Cabecera para las variables y funciones bonus
```

### 📈 Comportamiento Esperado

Cada vez que invocamos `get_next_line` desde nuestro programa principal, la función retorna en la salida algo con el siguiente formato estricto:

`[texto de la línea]\n`

Mensajes esperados:
- `La línea correctamente extraída y siempre preservando su \n natural.`
- La última línea del fichero devuelta correctamente (incluya o no `\n` al acabarse el texto).
- `null`: Cuando se alcanza el final absoluto del archivo (EOF) o si ocurre algún error de lectura.

Dificultades encontradas:

### 🛑 Variables estáticas & Fugas de memoria

El hilo conductor de este proyecto es saber retener información de una llamada de la función a la siguiente y no perder datos en cada ciclo del buffer. Nos enfrentamos a dos desafíos serios:

1. **El Resto no consumido (Variables Estáticas):** Ocurren cuando `read()` lee, digamos, un bloque de 42 bytes, pero la línea real del archivo termina en el byte 15. Los 27 bytes adicionales que ya han sido leídos no pueden desecharse. Tenemos que salvarlos temporalmente en una variable estática (static) para que la siguiente invocación de `get_next_line` "vuelva" a ese pequeño remanente antes de mandar llamar nuevamente a `read()`. 
2. **Leaks (Problemas de memoria):** Expandir constantemente el tamaño de nuestra cuerda (string) usando `malloc` obligatoriamente involucra desechar correctamente (liberar) el arreglo viejo. Una mala gestión, pisarse las huellas u olvidarse de aplicar `free()` si el programa tropieza o retorna en error es sinónimo de que el programa colapse a la larga cuando los descriptores mueren por saturación. 

### ⚠️ Reglas y Control de Errores
- **No lseek ni globals:** Está terminantemente prohibido usar `lseek` para retroceder el cursor de lectura del archivo, así como usar variables globales. Todo se maneja exclusivamente a pulso gracias a la cualidad de las variables *static*.
- **Gestión de memoria blindada:** No se permite cruzar fugas de memoria (memory leaks); ni a mitad de ejecución, ni en errores de archivo ni al final del programa. Todo debe liberar su puntero adecuadamente y revisarse con `valgrind`.
- **Single Static Variable:** Un reto exigente del proyecto es intentar condensar toda la retentiva en un único puntero de origen estático.
- **Bonus Track:** La versión con bonus eleva la dificultad garantizando que un hilo pueda estar leyendo del descriptor 3, intercalar al 4, ir al 5, y volver al 3 conservando rigurosamente toda la sintaxis y resaca de lecturas pendientes de forma paralela usando un array del límite abierto de archivos (suele ser 1024 o 4096 dependiendo del OS).

---

## English

Have you ever wondered how to read heavy files in C sensibly without consuming all available memory? **Get_next_line** is a 42 school project that tackles sequential file reading line by line while diving deep into pointer handling, dynamic memory allocation, and the pivotal concept of **static variables**.

### 🎯 Project Overview
Imagine dealing with an impossibly large text file or manipulating an ongoing input stream. Reading the entire content dynamically in memory at a single sweep can be highly inefficient or outright impossible.
To solve this, you need a function that reads and isolates **strictly one line at a time** continuously.

The challenge: Design a `char *get_next_line(int fd);` function that flawlessly remembers exactly where it paused inside the file descriptor stream trip after trip, while effectively preventing any memory or memory leak problems.

### ⚙️ Usage & Compilation

This specific 42 task typically doesn't ship with a full `Makefile`. To build the executable compiling the code together with your own `main.c`, you use the explicit compiler flag `-D BUFFER_SIZE=n` to manually determine the byte length evaluated in a solitary loop reading cycle:

```bash
gcc -Wall -Wextra -Werror -D BUFFER_SIZE=42 main.c get_next_line.c get_next_line_utils.c -o gnl
./gnl
```

**Parameter Breakdown:**
- `BUFFER_SIZE`: Bytes threshold injected simultaneously into the native `read()` function call. This size matters: it can be trivially tiny like 1 byte, or massive like 9999. The implementation's math should support all arbitrary widths seamlessly.
- `fd` (File Descriptor): A numeric handle given by the Operating System to access a physical open file or standard incoming payload (`0` for `stdin`).

**Execution Example via `main.c`:**
```c
#include <fcntl.h>
#include <stdio.h>
#include "get_next_line.h"

int main(void)
{
    int fd = open("archivo.txt", O_RDONLY);
    char *line;

    while ((line = get_next_line(fd)) != NULL)
    {
        printf("%s", line);
        free(line);
    }
    close(fd);
    return (0);
}
```

### 📂 Codebase Structure

Files available are extremely limited in number by subject rules resulting in a focused scheme:

```text
Get_next_line/
├── get_next_line.c           # Core functional logic and extraction of each individual line
├── get_next_line_utils.c     # String-handling sidekick duties (strlen, strjoin...)
├── get_next_line.h           # Header: function signatures, macros, and standard imports
├── get_next_line_bonus.c     # Bonus code introducing the multiple concurrent FDs feature
├── get_next_line_utils_bonus.c# Assistant helper tasks exclusive to the bonus subset
└── get_next_line_bonus.h     # Dedicated Header to tackle complex multi-array definitions
```

### 📈 Logging Format / Expectations

With each iterative call triggering `get_next_line`, it outputs back a fully assembled substring shaped like this:

`[read string line]\n`

Allowed and Expected Return State messages:
- `Accurately fetches the read string whilst preserving its native \n newline anchor.`
- Processes string leftovers properly returning the file's very last line securely (regardless if it actually holds a literal newline anchor character at EOF).
- `null`: Signals the complete End Of File exhaustion threshold (EOF) or immediately catches failing invalid read procedures.

Challenges Faced:

### 🛑 Static Variables & Memory Leaking

At the heart of GNL lies carrying state-of-fact persistence spanning across successive calls and effectively preventing the destruction of dangling unused characters throughout cycles. Two major threats must be conquered:

1. **Unconsumed Character Carryovers (Static state saving):** This occurs natively whenever your `read()` snatches, say, 42 straight chunks together, but abruptly, the text’s physical visual delimiter is localized at byte 15. Discarding the remaining 27 characters is out of the question. You stash them safely into a durable layout (a `static` variable placeholder string backup), meaning subsequent call requests to `get_next_line` check this residue log prior to ever poking `read()` for a completely fresh fetch again.
2. **Memory Leaks:** Constantly stretching or resizing dynamically our accumulated sentences using `strjoin` and `malloc` inevitably carries a strict duty: discarding the obsolete array trace footprint (`free`). Slipping up pointer tracking, omitting memory frees mid-execution, and discarding checks whenever an abrupt code error triggers equates drastically to suffocating the program under dead memory over time.

### ⚠️ Rules & Constraints
- **Zero lseek & globals permitted:** Reverting the active read pointer cursor using `lseek` mechanics or adopting standard macro global variables triggers an automatic fail. Persistence remains rigorously bound to `static` traits only.
- **Battle-hardened Memory bounds:** Zero tolerance for leaks; `valgrind` tests shouldn’t report memory dropping in scenarios spanning from a fully nominal execution logic to abruptly invalid File Descriptors interrupting procedures.
- **Single Static Resource Limit:** Reaching ultimate excellence demands implementing the tool depending almost exclusively on configuring a solitary `static` master index structure across your execution code.
- **Bonus Multitasking Track:** Delivering the bonus tier forces your function to operate across overlapping multi-file scenarios natively. Intermittently reading `fd(3)`, jumping to `fd(4)`, switching into `fd(5)`, then requesting `fd(3)` back without permanently corrupting or splicing foreign texts together implies crafting complex static-array layouts capped at standard OS file limits (1024+ typically).