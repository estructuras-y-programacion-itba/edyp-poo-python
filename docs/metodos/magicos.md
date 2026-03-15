# Métodos Mágicos (Dunder Methods)

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
def __init__(self, nombre: str) -> None:
    self.nombre = nombre
```

### `__repr__`: representación para desarrolladores

Retorna una cadena que debería, idealmente, ser una expresión Python válida para recrear el objeto. Es lo que ve el intérprete interactivo, los logs y los debuggers. **Siempre implementalo.**

```python
def __repr__(self) -> str:
    return f"Producto({self.nombre!r}, ${self.precio:.2f})"
# Nota: !r aplica repr() al string, agregando comillas → 'Laptop'
```

### `__str__`: representación para humanos

Retorna una cadena amigable para mostrar al usuario final. Es lo que usa `print()` y `str()`. Si no lo implementás, Python usa `__repr__` como fallback.

```python
def __str__(self) -> str:
    return f"{self.nombre} — ${self.precio:.2f}"
```

**Regla práctica:** implementá siempre `__repr__`. Implementá `__str__` cuando la representación legible para humanos deba ser diferente de la representación de debug.

### `__len__`: soporte para `len()`

Permite llamar `len(objeto)` sobre tus clases contenedoras.

```python
def __len__(self) -> int:
    return len(self._items)
```

### `__eq__` y `__hash__`: igualdad y uso en sets/dicts

`__eq__` define qué significa que dos objetos sean *iguales* (operador `==`). Por defecto, dos objetos son iguales solo si son el mismo objeto en memoria (misma identidad).

`__hash__` define cómo se calcula el hash del objeto, necesario para usarlo como clave en un `dict` o elemento de un `set`. Si definís `__eq__`, Python automáticamente hace `__hash__ = None` (el objeto se vuelve no hasheable) a menos que también definas `__hash__`.

```python
def __eq__(self, otro: object) -> bool:
    if not isinstance(otro, type(self)):
        return NotImplemented
    return self.codigo == otro.codigo

def __hash__(self) -> int:
    return hash(self.codigo)
```

### `__contains__`: soporte para el operador `in`

Permite escribir `item in objeto` de forma natural.

```python
def __contains__(self, item: object) -> bool:
    return item in self._items
```

### Operadores de comparación: `__lt__`, `__le__`, `__gt__`, `__ge__`

Permiten usar los operadores `<`, `<=`, `>`, `>=` y por extensión, funciones como `sorted()` y `min()`/`max()`.

```python
def __lt__(self, otro: "Producto") -> bool:
    return self.precio < otro.precio
```

### Operadores aritméticos: `__add__`, `__mul__`

Permiten usar `+` y `*` entre objetos de tu clase.

```python
def __add__(self, otro: "Vector") -> "Vector":
    return Vector(self.x + otro.x, self.y + otro.y)
```

## Ejemplo completo: clase `Inventario`

```python
from __future__ import annotations


class Producto:
    """
    Producto con código, nombre y precio.

    Args:
        codigo: Código único del producto.
        nombre: Nombre del producto.
        precio: Precio unitario.
    """

    def __init__(self, codigo: str, nombre: str, precio: float) -> None:
        self.codigo = codigo
        self.nombre = nombre
        self.precio = precio

    def __repr__(self) -> str:
        return f"Producto({self.codigo!r}, {self.nombre!r}, ${self.precio:.2f})"

    def __str__(self) -> str:
        return f"[{self.codigo}] {self.nombre} — ${self.precio:.2f}"

    def __eq__(self, otro: object) -> bool:
        if not isinstance(otro, Producto):
            return NotImplemented
        return self.codigo == otro.codigo

    def __hash__(self) -> int:
        return hash(self.codigo)

    def __lt__(self, otro: Producto) -> bool:
        """Permite ordenar productos por precio."""
        return self.precio < otro.precio


class Inventario:
    """
    Colección de productos con comportamiento de contenedor.

    Implementa los dunder methods necesarios para que el inventario
    se comporte como un contenedor de Python nativo.
    """

    def __init__(self) -> None:
        self._productos: list[Producto] = []

    def agregar(self, producto: Producto) -> None:
        """Agrega un producto al inventario."""
        if producto in self:  # usa __contains__
            raise ValueError(f"El producto {producto.codigo!r} ya existe en el inventario")
        self._productos.append(producto)

    # ── Dunder methods ────────────────────────────────────────────────────────

    def __repr__(self) -> str:
        """Representación técnica: útil en debuggers y logs."""
        return f"Inventario({len(self)} productos)"

    def __str__(self) -> str:
        """Representación legible: útil para mostrar al usuario."""
        if not self._productos:
            return "Inventario vacío"
        lineas = [f"=== Inventario ({len(self)} productos) ==="]
        lineas.extend(f"  {p}" for p in self._productos)
        return "\n".join(lineas)

    def __len__(self) -> int:
        """Permite: len(inventario)."""
        return len(self._productos)

    def __contains__(self, item: object) -> bool:
        """Permite: producto in inventario."""
        if isinstance(item, Producto):
            return any(p.codigo == item.codigo for p in self._productos)
        if isinstance(item, str):  # también busca por código
            return any(p.codigo == item for p in self._productos)
        return False

    def __iter__(self):
        """Permite iterar: for producto in inventario."""
        return iter(self._productos)

    def __getitem__(self, codigo: str) -> Producto:
        """Permite: inventario['LAP001']."""
        for p in self._productos:
            if p.codigo == codigo:
                return p
        raise KeyError(f"Producto no encontrado: {codigo!r}")


# Demostración de todos los dunder methods

inv = Inventario()
laptop = Producto("LAP001", "Laptop", 1500.0)
mouse = Producto("MOU001", "Mouse", 45.0)
teclado = Producto("TEC001", "Teclado", 80.0)

inv.agregar(laptop)
inv.agregar(mouse)
inv.agregar(teclado)

# __str__ y __repr__
print(inv)        # muestra la versión legible (usa __str__)
print(repr(inv))  # Inventario(3 productos)

# __len__
print(f"Total de productos: {len(inv)}")  # 3

# __contains__
print(laptop in inv)    # True (busca por objeto, usa __eq__ de Producto)
print("LAP001" in inv)  # True (busca por código)
print("XXX999" in inv)  # False

# __iter__
productos_ordenados = sorted(inv)   # usa __lt__ de Producto para ordenar por precio
for p in productos_ordenados:
    print(p)

# __getitem__
print(inv["LAP001"])  # [LAP001] Laptop — $1500.00

try:
    inv["XXX999"]
except KeyError as e:
    print(f"Error: {e}")  # Error: 'Producto no encontrado: 'XXX999''

# __eq__ en Producto: dos objetos con mismo código son iguales
p1 = Producto("LAP001", "Laptop Pro", 2000.0)
p2 = Producto("LAP001", "Laptop", 1500.0)
print(p1 == p2)  # True — mismo código → misma identidad de negocio

# __hash__: se pueden usar en sets y como claves de dict
catalogo = {laptop: "disponible", mouse: "agotado"}
print(catalogo[laptop])  # "disponible"
```

> **En la práctica:** el error más frecuente con dunder methods es definir `__eq__` sin definir `__hash__`. Cuando Python detecta que definiste `__eq__`, automáticamente hace que el objeto no sea hasheable (para evitar inconsistencias). Si querés que el objeto sea usable en sets y como clave de dict, siempre definí ambos juntos. La regla: si dos objetos son iguales según `__eq__`, su `__hash__` debe retornar el mismo valor.

---
