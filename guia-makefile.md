# Guía Completa de Makefile

## Conceptos Básicos

### ¿Qué es un Makefile?
Un Makefile es un archivo que define un conjunto de tareas (targets) que se deben ejecutar. Make lee este archivo para determinar automáticamente qué partes de un programa necesitan ser recompiladas.

### Estructura Básica
```makefile
target: dependencias
    comandos
```
- `target`: nombre del archivo a crear o acción a ejecutar
- `dependencias`: archivos necesarios para el target
- `comandos`: acciones a ejecutar (¡importante usar TAB, no espacios!)

## Sintaxis Fundamental

### Reglas Básicas
```makefile
programa: main.o util.o
    gcc -o programa main.o util.o

main.o: main.c
    gcc -c main.c

util.o: util.c
    gcc -c util.c
```

### Variables
```makefile
CC = gcc
CFLAGS = -Wall -O2
OBJS = main.o util.o

programa: $(OBJS)
    $(CC) -o programa $(OBJS)
```

### Variables Automáticas
- `$@`: nombre del target
- `$<`: primera dependencia
- `$^`: todas las dependencias
- `$*`: nombre del archivo sin extensión

### Patrones
```makefile
%.o: %.c
    $(CC) -c $(CFLAGS) $< -o $@
```

## Funciones Importantes

### Funciones de Texto
- `$(wildcard *.c)`: lista archivos .c
- `$(patsubst %.c,%.o,$(wildcard *.c))`: sustituye extensiones
- `$(dir archivo)`: extrae directorio
- `$(notdir archivo)`: extrae nombre de archivo

### Funciones de Control
```makefile
ifdef VARIABLE
    # código si VARIABLE está definida
else
    # código si VARIABLE no está definida
endif
```

## Características Avanzadas

### Targets Especiales
1. **.PHONY**
```makefile
.PHONY: clean all test
```
Los targets .PHONY son targets que no representan archivos reales. Se usan para:
- Evitar conflictos con archivos del mismo nombre
- Asegurar que los comandos siempre se ejecuten
- Definir targets de utilidad como 'clean', 'all', 'install'

2. **.DEFAULT**
```makefile
.DEFAULT:
    @echo "Target desconocido"
```
Se ejecuta cuando se solicita un target que no existe en el Makefile.

3. **.PRECIOUS**
```makefile
.PRECIOUS: %.o
```
Evita que Make borre archivos intermedios en caso de interrupción.

### Targets Múltiples
```makefile
programa programa-debug: $(OBJS)
    $(CC) -o $@ $^
```
Permite definir reglas para varios targets a la vez.

### Targets de Orden Superior
```makefile
DIRS = src lib tests
all: $(DIRS)
$(DIRS):
    $(MAKE) -C $@
```
Permite ejecutar makefiles en subdirectorios.

### Variables Exportadas
```makefile
export PATH := $(PATH):$(PWD)/bin
export VARIABLE := valor
```
Las variables exportadas están disponibles en submakes.

### Inclusión Condicional
```makefile
sinclude config.mk    # No falla si el archivo no existe
-include local.mk     # Igual que sinclude
include required.mk   # Falla si el archivo no existe
```

### Funciones de Shell
```makefile
FILES := $(shell ls *.c)
VERSION := $(shell git describe --tags)
CURRENT_DIR := $(shell pwd)
```
Permite ejecutar comandos de shell y capturar su salida.

### Targets con Dependencias Dinámicas
```makefile
SRCS = $(wildcard *.c)
-include $(SRCS:.c=.d)

%.d: %.c
    $(CC) -MM $< > $@
```
Genera y incluye archivos de dependencias automáticamente.

### Targets Paralelos
```makefile
.NOTPARALLEL: critical_target  # Este target no se ejecutará en paralelo
programa: $(OBJS)
    $(MAKE) -j 4              # Ejecuta hasta 4 jobs en paralelo
```

### Variables de Entorno
```makefile
ifdef RELEASE
    CFLAGS += -O2 -DNDEBUG
else
    CFLAGS += -g -DDEBUG
endif

# Uso de variables de entorno con valores por defecto
CC ?= gcc
INSTALL_DIR ?= /usr/local
```

### Targets con Patrones Múltiples
```makefile
%.pdf %.html: %.md
    pandoc -o $@ $<
```
Una regla que maneja múltiples tipos de salida.

### Targets de Solo Prerrequisitos
```makefile
.PRECIOUS: %.o

%.o: %.c %.h common.h
```
Define dependencias sin comandos asociados.

### Control de Errores
```makefile
critical_target:
    -$(CC) test.c    # El guión ignora errores
    $(MAKE) next_step || exit 0    # Continúa incluso si hay error
```

### Variables Recursivas vs. Simples
```makefile
# Variable recursiva (se expande cada vez que se usa)
CFLAGS = -Wall $(EXTRA_FLAGS)

# Variable simple (se expande una sola vez)
CFLAGS := -Wall $(EXTRA_FLAGS)
```

### Targets Secundarios
```makefile
.SECONDARY: %.o    # Preserva archivos intermedios
```
Evita la eliminación automática de archivos intermedios.

## Consejos y Buenas Prácticas

1. **Organización**
   - Mantén targets relacionados juntos
   - Usa variables para valores reutilizables
   - Documenta reglas complejas con comentarios

2. **Depuración**
   - Usa `make -n` para ver comandos sin ejecutarlos
   - Usa `make -d` para debugging detallado
   - Añade `.PHONY` para targets que no son archivos

3. **Optimización**
   - Usa patrones cuando sea posible
   - Evita duplicación de código
   - Aprovecha las variables automáticas

## Resolución de Problemas Comunes

1. **Error de Tabulación**
   - Asegúrate de usar TAB, no espacios
   - Verifica la configuración del editor

2. **Dependencias Circulares**
   - Revisa la lógica de las dependencias
   - Usa `make -d` para analizar el problema

3. **Archivos no Actualizados**
   - Verifica timestamps de archivos
   - Usa `touch` para forzar actualización

## Referencias Rápidas

### Variables Predefinidas
```makefile
MAKE        # comando make
MAKEFILE    # nombre del makefile
MAKEFLAGS   # flags de make
VPATH       # path de búsqueda
SHELL       # shell a usar
```

### Operadores Comunes
```makefile
:=    # asignación inmediata
=     # asignación diferida
+=    # añadir a variable
?=    # asignar si no definida
```
