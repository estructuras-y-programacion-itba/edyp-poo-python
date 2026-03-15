# Clases y Objetos

La distinción entre clase y objeto es la primera y más fundamental de toda la POO. Confundirlos es el error conceptual más común en los primeros pasos.

## La clase: el molde, no la cosa

Una **clase** es una plantilla o molde que describe cómo serán los objetos que se creen a partir de ella. Define qué datos van a tener (atributos) y qué comportamiento van a exhibir (métodos). Por sí sola, una clase no ocupa espacio de datos útil ni ejecuta nada: es solo la *descripción*.

La analogía clásica es la del plano de arquitectura: el plano de una casa no es una casa. Es la descripción de cómo construir una. Podés construir diez casas idénticas a partir del mismo plano, y cada una va a ser independiente de las demás.

En Python, una clase se define con la palabra reservada `class` seguida del nombre en **PascalCase** (también llamado CamelCase: cada palabra empieza con mayúscula):

```python
class Producto:
    pass  # clase vacía por ahora
```

## El objeto: la instancia concreta

Un **objeto** es una instancia concreta de una clase, creada en tiempo de ejecución. Mientras la clase existe en el código fuente, el objeto existe en la memoria de la computadora mientras el programa se ejecuta.

Se crea invocando la clase como si fuera una función, con el operador `()`:

```python
producto_a = Producto()  # se crea una instancia de Producto
producto_b = Producto()  # una instancia diferente, independiente
```

`producto_a` y `producto_b` son dos objetos distintos, cada uno con su propia identidad en memoria.

## El constructor: `__init__`

El método `__init__` es el **constructor** de la clase. Python lo llama automáticamente cada vez que se crea un nuevo objeto. Su propósito es inicializar el estado del objeto: asignar valores a los atributos que va a tener.

```python
class Producto:
    """
    Producto con nombre y precio.

    Args:
        nombre: Nombre del producto.
        precio: Precio unitario (debe ser positivo).
    """

    def __init__(self, nombre: str, precio: float) -> None:
        self.nombre = nombre
        self.precio = precio
```

Cuando escribís `Producto("Laptop", 1500.0)`, Python crea un objeto vacío y llama a `__init__` pasando ese objeto como primer argumento (que dentro del método se llama `self`).

## `self`: la referencia al objeto actual

`self` es el primer parámetro de cualquier método de instancia y representa al objeto sobre el que se está operando. Python lo pasa automáticamente: cuando llamás `producto.describir()`, Python llama internamente a `Producto.describir(producto)`.

No es una palabra reservada (podría llamarse diferente), pero es una convención tan universal que nunca deberías cambiarla.

```python
class Producto:
    """
    Producto con nombre, precio y método de descripción.

    Args:
        nombre: Nombre del producto.
        precio: Precio unitario.
    """

    def __init__(self, nombre: str, precio: float) -> None:
        self.nombre = nombre   # atributo de instancia
        self.precio = precio   # atributo de instancia

    def describir(self) -> str:
        """Retorna una descripción legible del producto."""
        return f"{self.nombre}: ${self.precio:.2f}"
```

## Atributos de instancia: el estado de cada objeto

Los **atributos de instancia** son variables que pertenecen a cada objeto individualmente. Se crean asignándolos a través de `self` en `__init__`. Cada objeto tiene su propio espacio de memoria para estos atributos: modificar el atributo de un objeto no afecta al otro.

## Identidad, estado y comportamiento

Todo objeto tiene tres dimensiones:

- **Identidad**: lo que lo hace único en memoria (su dirección). En Python, `id(objeto)` la retorna.
- **Estado**: los valores actuales de sus atributos.
- **Comportamiento**: las operaciones que puede realizar (sus métodos).

```python
class Producto:
    """
    Producto con nombre, precio y método de descripción.

    Args:
        nombre: Nombre del producto.
        precio: Precio unitario.
    """

    def __init__(self, nombre: str, precio: float) -> None:
        self.nombre = nombre
        self.precio = precio

    def describir(self) -> str:
        """Retorna una descripción legible del producto."""
        return f"{self.nombre}: ${self.precio:.2f}"

    def aplicar_descuento(self, porcentaje: float) -> None:
        """Aplica un descuento al precio del producto.

        Args:
            porcentaje: Porcentaje de descuento (0-100).
        """
        if not 0 < porcentaje <= 100:
            raise ValueError("El porcentaje debe estar entre 0 y 100")
        self.precio *= (1 - porcentaje / 100)

    def __repr__(self) -> str:
        return f"Producto({self.nombre!r}, ${self.precio:.2f})"


# Instanciación: se crean dos objetos independientes
laptop = Producto("Laptop", 1500.0)
mouse = Producto("Mouse", 45.0)

# Identidad: son distintos objetos en memoria
print(id(laptop) == id(mouse))   # False

# Estado: cada objeto tiene su propio estado
print(laptop.describir())  # Laptop: $1500.00
print(mouse.describir())   # Mouse: $45.00

# Independencia: modificar uno no afecta al otro
laptop.aplicar_descuento(10)
print(laptop.precio)  # 1350.0
print(mouse.precio)   # 45.0  ← sin cambios
```

## Las instancias son independientes entre sí

Este punto merece énfasis: cada objeto es su propio mundo. Crear cien instancias de `Producto` produce cien objetos independientes. El estado de uno no interfiere con el de los demás.

```python
# Tres productos, tres estados independientes
productos = [
    Producto("Teclado", 80.0),
    Producto("Monitor", 400.0),
    Producto("Webcam", 60.0),
]

# Aplicar descuento solo al primero
productos[0].aplicar_descuento(20)

for p in productos:
    print(p)
# Producto('Teclado', $64.00)   ← modificado
# Producto('Monitor', $400.00)  ← sin cambios
# Producto('Webcam', $60.00)    ← sin cambios
```

> **En la práctica:** un error clásico de quienes empiezan es usar atributos de *clase* (definidos directamente en el cuerpo de la clase, fuera de `__init__`) pensando que son atributos de instancia. Los atributos de clase son compartidos por *todas* las instancias, lo cual raramente es lo que se quiere. Siempre asigná el estado inicial del objeto dentro de `__init__` usando `self`.

---
