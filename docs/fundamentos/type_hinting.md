# Type Hinting en Python

En los ejemplos anteriores escribimos métodos con atributos e instancias. El type hinting es la herramienta que hace explícito el contrato de esos métodos: qué tipos reciben, qué tipos retornan. No cambia cómo funciona el código en runtime, pero mejora radicalmente la legibilidad, el soporte del IDE y la capacidad de las herramientas de análisis estático para detectar errores antes de ejecutar.

Python es un lenguaje de tipado dinámico: no necesitás declarar el tipo de una variable antes de usarla. Eso da flexibilidad, pero también puede generar confusión cuando el código crece o cuando alguien más tiene que entenderlo.

El *type hinting* (o anotaciones de tipo) es una forma de declarar explícitamente qué tipos esperan recibir y retornar tus funciones y métodos. No cambia cómo ejecuta el programa —Python las ignora en tiempo de ejecución—, pero sí cambia radicalmente qué tan fácil es leer, mantener y trabajar con el código.

---

## ¿Por qué se usa cada vez más?

Tres razones concretas explican la adopción masiva del type hinting en proyectos profesionales:

**Las herramientas lo aprovechan.** Editores como VS Code o PyCharm usan las anotaciones para autocompletar, detectar errores antes de ejecutar y navegar el código. Sin anotaciones, el editor no puede saber qué métodos tiene un objeto que recibe una función.

**Las IAs lo generan por defecto.** Modelos como Copilot, ChatGPT o Claude producen código con type hints de forma automática. Si no sabés leerlos e interpretarlos, el código que te sugiere la IA se vuelve más difícil de revisar y adaptar.

**La documentación queda en el código.** En lugar de escribir un comentario diciendo "esta función recibe una lista de máquinas", el tipo lo dice directamente. Es documentación que no se puede desincronizar del código porque *es* el código.

---

## Sintaxis básica

### Variables

```python
nombre: str = "CNC-01"
capacidad: int = 100
temperatura: float = 72.5
activa: bool = True
```

Las anotaciones en variables son válidas pero opcionales. Donde realmente aportan valor es en funciones y métodos.

### Funciones y métodos

```python
def calcular_eficiencia(piezas: int, minutos: float) -> float:
    if minutos <= 0:
        return 0.0
    return piezas / minutos * 60
```

La sintaxis es:

- `param: Tipo` para cada parámetro
- `-> Tipo` para el valor de retorno
- `-> None` cuando la función no retorna nada

```python
def activar_maquina(nombre: str) -> None:
    print(f"Activando {nombre}")
```

---

## Tipos de colecciones

Anotar que algo es una `list` no alcanza — lo útil es saber *qué contiene* esa lista.

### Listas

```python
# Lista de strings
nombres: list[str] = ["CNC-01", "Soldadora-02"]

# Lista de enteros
turnos: list[int] = [1, 2, 3]
```

### Diccionarios

```python
# dict[tipo_clave, tipo_valor]
config: dict[str, float] = {"velocidad": 200.0, "temperatura_max": 85.0}
```

### Tuplas

```python
# Tupla con tipos fijos por posición
coordenada: tuple[float, float] = (3.14, 2.71)

# Tupla de longitud variable con un solo tipo
lecturas: tuple[float, ...] = (22.1, 23.4, 21.8)
```

---

## Referenciar clases propias

Cuando trabajás con POO, lo más común es usar tus propias clases como tipos. Funciona exactamente igual.

```python
class Maquina:
    def __init__(self, nombre: str, capacidad: int) -> None:
        self.nombre = nombre
        self._capacidad = capacidad

    def capacidad(self) -> int:
        return self._capacidad


class LineaDeMontaje:
    def __init__(self) -> None:
        self._maquinas: list[Maquina] = []  # lista de instancias de Maquina

    def agregar(self, maquina: Maquina) -> None:  # recibe una Maquina
        self._maquinas.append(maquina)

    def buscar(self, nombre: str) -> Maquina | None:  # retorna Maquina o None
        for m in self._maquinas:
            if m.nombre == nombre:
                return m
        return None

    def todas(self) -> list[Maquina]:  # retorna lista de Maquinas
        return list(self._maquinas)
```

Tres patrones importantes del ejemplo:

- `list[Maquina]` — lista de instancias de tu clase
- `Maquina | None` — puede retornar una `Maquina` o `None` (el operador `|` reemplaza a `Optional` de versiones anteriores)
- `self._maquinas: list[Maquina] = []` — anotás el atributo de instancia en `__init__`

---

## Referencia circular entre clases

A veces dos clases se referencian mutuamente. Si `LineaDeMontaje` menciona a `Maquina` y `Maquina` menciona a `LineaDeMontaje`, podés tener un error de nombre porque una clase todavía no está definida cuando Python lee la otra.

La solución es usar strings como anotación (referencia diferida):

```python
class Maquina:
    def asignar_linea(self, linea: "LineaDeMontaje") -> None:
        self._linea = linea


class LineaDeMontaje:
    def agregar(self, maquina: Maquina) -> None:
        self._maquinas.append(maquina)
```

En Python 3.10+ también podés activar la evaluación diferida automática con:

```python
from __future__ import annotations
```

Esto hace que todas las anotaciones se traten como strings internamente, eliminando el problema sin necesidad de comillas.

---

## Tipos especiales útiles

### `None` como retorno

```python
def limpiar_buffer(self) -> None:
    self._buffer.clear()
```

### Valor opcional (`X | None`)

```python
def buscar_sensor(self, sensor_id: str) -> str | None:
    return self._sensores.get(sensor_id)
```

### Tipo `Any`

Cuando el tipo genuinamente puede ser cualquier cosa. Usalo con cuidado: anuncia que no hay información de tipo en ese punto.

```python
from typing import Any

def registrar(self, dato: Any) -> None:
    self._log.append(dato)
```

### Callable

Para pasar funciones como argumento:

```python
from typing import Callable

def ejecutar_con_callback(pieza: str, callback: Callable[[str], None]) -> None:
    resultado = self.procesar(pieza)
    callback(resultado)
```

---

## Verificación estática con mypy

Las anotaciones de tipo no generan errores en tiempo de ejecución. Para detectar inconsistencias antes de ejecutar, existe `mypy`:

```bash
uv add mypy
uv run mypy src/
```

`mypy` analiza el código estáticamente y reporta si pasás un `str` donde se espera un `int`, si un método no existe en el tipo declarado, o si una función podría retornar `None` sin que el código que la llama lo maneje.

En proyectos grandes, integrarlo a la pipeline de CI es una práctica estándar.

---

## Resumen

| Patrón | Sintaxis |
| ------ | -------- |
| Parámetro simple | `def f(x: int) -> str` |
| Retorno vacío | `-> None` |
| Lista tipada | `list[str]`, `list[Maquina]` |
| Diccionario tipado | `dict[str, float]` |
| Valor opcional | `Maquina \| None` |
| Clase propia | mismo nombre de la clase |
| Referencia circular | `"NombreClase"` entre comillas |
| Atributo de instancia | `self._attr: list[X] = []` en `__init__` |

## Ver también

El type hinting documenta el contrato de tus métodos. El siguiente concepto protege el estado que esos métodos manejan:

- [Encapsulamiento](encapsulamiento.md) — convenciones de acceso y decoradores `@property`
- [Clases y Objetos](clases_y_objetos.md) — contexto sobre atributos de instancia y de clase que se anotan con tipos
