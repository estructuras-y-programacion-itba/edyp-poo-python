# Métodos Mágicos (Dunder Methods)

Ya conocés los métodos de instancia (acceden al estado del objeto), los métodos de clase (operan sobre la clase) y los estáticos (funciones de dominio sin estado). Los métodos mágicos son una categoría aparte: Python los llama automáticamente en respuesta a operadores, funciones built-in y protocolos del lenguaje — nunca los llamás directamente.

Los métodos mágicos, también llamados **dunder methods** (de *double underscore*), son métodos especiales con la forma `__nombre__`. Python los llama automáticamente en respuesta a operadores, funciones built-in y otras operaciones del lenguaje — vos nunca los llamás directamente.

Su propósito es permitir que tus clases se integren con el lenguaje de manera natural: que tus objetos respondan a `len()`, `print()`, los operadores `+`, `==`, `in`, y más, exactamente igual que los tipos built-in de Python.

## ¿Por qué existen?

Cuando escribís `len([1, 2, 3])`, Python llama internamente a `list.__len__()`. Cuando escribís `"hola" + " mundo"`, Python llama a `str.__add__()`. Los dunder methods son el mecanismo por el cual este protocolo se extiende a tus propias clases.

```python
# Esto...
print(len([1, 2, 3]))   # 3
# ...es equivalente a esto:
print(list.__len__([1, 2, 3]))  # 3
```

## Los más importantes

### `__init__`: el constructor

Ya lo conocés: inicializa el estado del objeto al crearlo.

```python
def __init__(self, numero_serie: str, material: str) -> None:
    self.numero_serie = numero_serie
    self.material = material
```

### `__repr__`: representación para desarrolladores

Retorna una cadena que debería, idealmente, ser una expresión Python válida para recrear el objeto. Es lo que ve el intérprete interactivo, los logs y los debuggers. **Siempre implementalo.**

```python
def __repr__(self) -> str:
    return f"Pieza({self.numero_serie!r}, {self.material!r})"
# Nota: !r aplica repr() al string, agregando comillas → 'P-0001'
```

### `__str__`: representación para humanos

Retorna una cadena amigable para mostrar al usuario final. Es lo que usa `print()` y `str()`. Si no lo implementás, Python usa `__repr__` como fallback.

```python
def __str__(self) -> str:
    return f"Pieza {self.numero_serie} — {self.material}, {self.peso:.2f} kg"
```

**Regla práctica:** implementá siempre `__repr__`. Implementá `__str__` cuando la representación legible para humanos deba ser diferente de la representación de debug.

### `__len__`: soporte para `len()`

Permite llamar `len(objeto)` sobre tus clases contenedoras.

```python
def __len__(self) -> int:
    return len(self._estaciones)
```

### `__eq__` y `__hash__`: igualdad y uso en sets/dicts

`__eq__` define qué significa que dos objetos sean *iguales* (operador `==`). Por defecto, dos objetos son iguales solo si son el mismo objeto en memoria (misma identidad).

`__hash__` define cómo se calcula el hash del objeto, necesario para usarlo como clave en un `dict` o elemento de un `set`. Si definís `__eq__`, Python automáticamente hace `__hash__ = None` (el objeto se vuelve no hasheable) a menos que también definas `__hash__`.

```python
def __eq__(self, otro: object) -> bool:
    if not isinstance(otro, type(self)):
        return NotImplemented
    return self.numero_serie == otro.numero_serie

def __hash__(self) -> int:
    return hash(self.numero_serie)

> **Nota para estudiantes:**
>
> Cuando implementás `__eq__`, si el otro objeto no es del tipo esperado, debés **retornar** `NotImplemented` (no levantar una excepción ni devolver `False`). Esto le indica a Python que pruebe la comparación inversa o que decida el resultado según las reglas del lenguaje. ¡No uses `raise NotImplementedError` ni `raise NotImplemented` acá! Simplemente devolvé `NotImplemented` como en el ejemplo.
```

### `__contains__`: soporte para el operador `in`

Permite escribir `item in objeto` de forma natural.

```python
def __contains__(self, item: object) -> bool:
    return item in self._estaciones
```

### Operadores de comparación: `__lt__`, `__le__`, `__gt__`, `__ge__`

Permiten usar los operadores `<`, `<=`, `>`, `>=` y por extensión, funciones como `sorted()` y `min()`/`max()`.

```python
def __lt__(self, otro: "Pieza") -> bool:
    return self.peso < otro.peso
```

### Operadores aritméticos: `__add__`, `__mul__`

Permiten usar `+` y `*` entre objetos de tu clase.

```python
def __add__(self, otro: "LineaDeMontaje") -> "LineaDeMontaje":
    nueva = LineaDeMontaje(f"{self._nombre}+{otro._nombre}")
    for est in list(self._estaciones.values()) + list(otro._estaciones.values()):
        nueva.agregar_estacion(est)
    return nueva
```

## Ejemplo completo: clases `Pieza` y `LineaDeMontaje`

```python
from __future__ import annotations


class Pieza:
    """
    Pieza industrial con número de serie, material y peso.

    Args:
        numero_serie: Identificador único de la pieza.
        material: Tipo de material (acero, aluminio, etc.).
        peso: Peso en kilogramos.
    """

    def __init__(self, numero_serie: str, material: str, peso: float) -> None:
        self.numero_serie = numero_serie
        self.material = material
        self.peso = peso

    def __repr__(self) -> str:
        return f"Pieza({self.numero_serie!r}, {self.material!r}, {self.peso:.2f}kg)"

    def __str__(self) -> str:
        return f"[{self.numero_serie}] {self.material} — {self.peso:.2f} kg"

    def __eq__(self, otro: object) -> bool:
        if not isinstance(otro, Pieza):
            return NotImplemented
        return self.numero_serie == otro.numero_serie

    def __hash__(self) -> int:
        return hash(self.numero_serie)

    def __lt__(self, otro: Pieza) -> bool:
        """Permite ordenar piezas por peso."""
        return self.peso < otro.peso


class EstacionTrabajo:
    """Estación de trabajo básica."""

    def __init__(self, nombre: str) -> None:
        self.nombre = nombre

    def __repr__(self) -> str:
        return f"EstacionTrabajo({self.nombre!r})"


class LineaDeMontaje:
    """
    Línea de montaje con comportamiento de contenedor.

    Implementa los dunder methods necesarios para que la línea
    se comporte como un contenedor de Python nativo.
    """

    def __init__(self, nombre: str) -> None:
        self._nombre = nombre
        self._estaciones: list[EstacionTrabajo] = []

    def agregar_estacion(self, estacion: EstacionTrabajo) -> None:
        """Agrega una estación a la línea."""
        if estacion in self:  # usa __contains__
            raise ValueError(f"La estación '{estacion.nombre}' ya está en la línea")
        self._estaciones.append(estacion)

    # ── Dunder methods ────────────────────────────────────────────────────────

    def __repr__(self) -> str:
        """Representación técnica: útil en debuggers y logs."""
        return f"LineaDeMontaje({self._nombre!r}, {len(self)} estaciones)"

    def __str__(self) -> str:
        """Representación legible: útil para mostrar al usuario."""
        if not self._estaciones:
            return f"Línea '{self._nombre}' (vacía)"
        nombres = ", ".join(est.nombre for est in self._estaciones)
        return f"Línea '{self._nombre}': {nombres}"

    def __len__(self) -> int:
        """Permite: len(linea)."""
        return len(self._estaciones)

    def __contains__(self, item: object) -> bool:
        """Permite: estacion in linea."""
        if isinstance(item, EstacionTrabajo):
            return any(e.nombre == item.nombre for e in self._estaciones)
        if isinstance(item, str):  # también busca por nombre
            return any(e.nombre == item for e in self._estaciones)
        return False

    def __iter__(self):
        """Permite iterar: for estacion in linea."""
        return iter(self._estaciones)

    def __getitem__(self, nombre: str) -> EstacionTrabajo:
        """Permite: linea['CNC-01']."""
        for est in self._estaciones:
            if est.nombre == nombre:
                return est
        raise KeyError(f"Estación no encontrada: {nombre!r}")

    def __add__(self, otra: LineaDeMontaje) -> LineaDeMontaje:
        """Fusiona dos líneas en una nueva: linea_a + linea_b."""
        nueva = LineaDeMontaje(f"{self._nombre}+{otra._nombre}")
        for est in self._estaciones:
            nueva.agregar_estacion(est)
        for est in otra._estaciones:
            if est not in nueva:
                nueva.agregar_estacion(est)
        return nueva


# Demostración de todos los dunder methods

linea_a = LineaDeMontaje("Línea A")
linea_b = LineaDeMontaje("Línea B")

cnc = EstacionTrabajo("CNC-01")
mig = EstacionTrabajo("MIG-01")
paint = EstacionTrabajo("PAINT-01")
robot = EstacionTrabajo("ROBOT-01")

linea_a.agregar_estacion(cnc)
linea_a.agregar_estacion(mig)
linea_b.agregar_estacion(paint)
linea_b.agregar_estacion(robot)

# __str__ y __repr__
print(linea_a)        # Línea 'Línea A': CNC-01, MIG-01
print(repr(linea_a))  # LineaDeMontaje('Línea A', 2 estaciones)

# __len__
print(f"Estaciones en A: {len(linea_a)}")  # 2

# __contains__
print(cnc in linea_a)       # True (busca por objeto)
print("CNC-01" in linea_a)  # True (busca por nombre)
print("ROBOT-01" in linea_a)  # False

# __iter__
for est in linea_a:
    print(est)

# __getitem__
print(linea_a["MIG-01"])  # EstacionTrabajo('MIG-01')

try:
    linea_a["LASER-99"]
except KeyError as e:
    print(f"Error: {e}")  # Error: 'Estación no encontrada: 'LASER-99''

# __add__: fusionar líneas
linea_completa = linea_a + linea_b
print(linea_completa)   # Línea 'Línea A+Línea B': CNC-01, MIG-01, PAINT-01, ROBOT-01
print(len(linea_completa))  # 4

# Pieza: __eq__ y __hash__
p1 = Pieza("P-0001", "acero", 2.3)
p2 = Pieza("P-0001", "aluminio", 1.0)  # mismo número de serie
print(p1 == p2)  # True — mismo número de serie → misma identidad de negocio

# __hash__: se pueden usar en sets y como claves de dict
catalogo = {p1: "en proceso", Pieza("P-0002", "acero", 3.1): "completada"}
print(catalogo[p1])  # "en proceso"

# __lt__: ordenar piezas por peso
piezas = [
    Pieza("P-0003", "acero", 5.0),
    Pieza("P-0001", "aluminio", 1.2),
    Pieza("P-0002", "acero", 3.4),
]
piezas_ordenadas = sorted(piezas)  # usa __lt__ para ordenar por peso
for p in piezas_ordenadas:
    print(p)
# [P-0001] aluminio — 1.20 kg
# [P-0002] acero — 3.40 kg
# [P-0003] acero — 5.00 kg
```

> **En la práctica:** el error más frecuente con dunder methods es definir `__eq__` sin definir `__hash__`. Cuando Python detecta que definiste `__eq__`, automáticamente hace que el objeto no sea hasheable (para evitar inconsistencias). Si querés que el objeto sea usable en sets y como clave de dict, siempre definí ambos juntos. La regla: si dos objetos son iguales según `__eq__`, su `__hash__` debe retornar el mismo valor.

## Ver también

Con el kit de herramientas de métodos completo, el próximo desafío es diseñar cómo las clases se relacionan entre sí en sistemas más complejos:

- [Relaciones entre Clases](../modelado/relaciones.md) — taxonomía de relaciones: asociación, agregación, composición, herencia
- [Encapsulamiento](../fundamentos/encapsulamiento.md) — complemento natural: `__eq__` y `__hash__` trabajan de la mano con atributos privados

---
