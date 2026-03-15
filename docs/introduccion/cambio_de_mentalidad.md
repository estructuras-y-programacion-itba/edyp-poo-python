# El Cambio de Mentalidad

Aprender la sintaxis de las clases en Python es la parte fácil. El verdadero desafío al aprender POO es otro: cambiar la manera en que pensás un problema antes de escribir una sola línea de código.

## De verbos a sustantivos

En programación estructurada, el diseño empieza con una pregunta: **¿qué pasos necesito para resolver esto?** La respuesta son funciones (verbos: calcular, guardar, mostrar, procesar).

En POO, la pregunta inicial es diferente: **¿qué entidades existen en este dominio y cómo se relacionan entre sí?** La respuesta son objetos (sustantivos: cliente, pedido, producto, factura).

Este cambio de verbos a sustantivos como unidad de diseño fue descripto por Alan Kay —inventor de Smalltalk y uno de los padres de la POO— quien enfatizaba que los objetos son entidades que *reciben mensajes* y *responden* a ellos. Las acciones existen, pero son responsabilidad de los objetos que las llevan a cabo.

## El error más común: código procedural dentro de clases

Cuando alguien aprende POO por primera vez y viene del paradigma procedural, es muy frecuente escribir algo así:

```python
# ❌ Esto NO es POO — es código procedural disfrazado de clase
class GestorDePedidos:
    def procesar(self, cliente_nombre, cliente_email, productos, cantidades, direccion):
        # 80 líneas de lógica mezclada...
        total = 0
        for i, producto in enumerate(productos):
            total += producto["precio"] * cantidades[i]
        # validar stock, calcular envío, enviar email, guardar en BD...
        return total
```

Este código tiene una clase, pero no tiene *objetos*. Es una función grande con un nombre de clase como disfraz. No hay estado encapsulado, no hay responsabilidades distribuidas, no hay abstracción real.

El diseño orientado a objetos correcto distribuye responsabilidades entre múltiples objetos, cada uno con un propósito claro:

```python
# ✅ Esto SÍ es POO — responsabilidades distribuidas entre objetos
class Cliente:
    """Conoce sus propios datos de contacto."""

    def __init__(self, nombre: str, email: str) -> None:
        self.nombre = nombre
        self.email = email


class Producto:
    """Conoce su precio y gestiona su propio stock."""

    def __init__(self, nombre: str, precio: float, stock: int) -> None:
        self.nombre = nombre
        self._precio = precio
        self._stock = stock

    @property
    def precio(self) -> float:
        return self._precio

    def tiene_stock(self, cantidad: int) -> bool:
        return self._stock >= cantidad


class LineaDePedido:
    """Representa un producto con su cantidad dentro de un pedido."""

    def __init__(self, producto: Producto, cantidad: int) -> None:
        self._producto = producto
        self._cantidad = cantidad

    def subtotal(self) -> float:
        return self._producto.precio * self._cantidad


class Pedido:
    """Agrupa líneas de pedido y calcula el total. No sabe de clientes ni de BD."""

    def __init__(self, cliente: Cliente) -> None:
        self._cliente = cliente
        self._lineas: list[LineaDePedido] = []

    def agregar(self, linea: LineaDePedido) -> None:
        self._lineas.append(linea)

    def total(self) -> float:
        return sum(linea.subtotal() for linea in self._lineas)
```

> **En la práctica:** cuando revisamos trabajos prácticos, el síntoma más frecuente de "mentalidad procedural" es una única clase con un único método enorme que hace todo. Si ese método tiene más de 20 líneas, casi siempre hay objetos que todavía no fueron identificados.

## Técnica: identificar sustantivos y verbos

Una técnica práctica para diseñar un sistema orientado a objetos desde una descripción en lenguaje natural es subrayar los **sustantivos** (candidatos a clases o atributos) y los **verbos** (candidatos a métodos).

Tomemos este enunciado:

> *"Un cliente realiza un pedido que contiene productos. Cada producto tiene un nombre, un precio y un stock disponible. El pedido calcula su total y puede ser confirmado o cancelado."*

Aplicando la técnica:

| Elemento | Tipo | ¿Qué es en código? |
| --- | --- | --- |
| **cliente** | Sustantivo | Clase `Cliente` |
| **pedido** | Sustantivo | Clase `Pedido` |
| **producto** | Sustantivo | Clase `Producto` |
| nombre, precio, stock | Sustantivos (datos) | Atributos de `Producto` |
| **realiza** | Verbo | Relación entre `Cliente` y `Pedido` |
| **contiene** | Verbo | `Pedido` tiene una lista de `Producto` |
| **calcula su total** | Verbo | Método `total()` en `Pedido` |
| **confirmar / cancelar** | Verbos | Métodos `confirmar()` y `cancelar()` en `Pedido` |

El diagrama de clases resultante sería:

```
Cliente  ──────────────────>  Pedido  ─────────────────>  Producto
         realiza (1..*)              contiene (1..*)
```

```python
from enum import Enum


class EstadoPedido(Enum):
    PENDIENTE = "pendiente"
    CONFIRMADO = "confirmado"
    CANCELADO = "cancelado"


class Producto:
    """
    Producto con nombre, precio y stock.

    Args:
        nombre: Nombre del producto.
        precio: Precio unitario (debe ser positivo).
        stock: Unidades disponibles.
    """

    def __init__(self, nombre: str, precio: float, stock: int) -> None:
        self.nombre = nombre
        self._precio = precio
        self._stock = stock

    @property
    def precio(self) -> float:
        return self._precio

    def tiene_stock(self, cantidad: int) -> bool:
        """Verifica si hay stock suficiente."""
        return self._stock >= cantidad

    def __repr__(self) -> str:
        return f"Producto({self.nombre!r}, ${self._precio:.2f})"


class Cliente:
    """
    Cliente con nombre y email.

    Args:
        nombre: Nombre completo del cliente.
        email: Dirección de correo electrónico.
    """

    def __init__(self, nombre: str, email: str) -> None:
        self.nombre = nombre
        self.email = email

    def __repr__(self) -> str:
        return f"Cliente({self.nombre!r})"


class Pedido:
    """
    Pedido de un cliente con una o más líneas de producto.

    Args:
        cliente: El cliente que realiza el pedido.
    """

    def __init__(self, cliente: Cliente) -> None:
        self._cliente = cliente
        self._productos: list[tuple[Producto, int]] = []
        self._estado = EstadoPedido.PENDIENTE

    def agregar_producto(self, producto: Producto, cantidad: int) -> None:
        """Agrega un producto al pedido."""
        if cantidad <= 0:
            raise ValueError("La cantidad debe ser positiva")
        self._productos.append((producto, cantidad))

    def total(self) -> float:
        """Calcula el total del pedido."""
        return sum(p.precio * cantidad for p, cantidad in self._productos)

    def confirmar(self) -> None:
        """Confirma el pedido si está en estado PENDIENTE."""
        if self._estado != EstadoPedido.PENDIENTE:
            raise ValueError(f"No se puede confirmar un pedido {self._estado.value}")
        self._estado = EstadoPedido.CONFIRMADO

    def cancelar(self) -> None:
        """Cancela el pedido si no fue ya confirmado."""
        if self._estado == EstadoPedido.CONFIRMADO:
            raise ValueError("No se puede cancelar un pedido ya confirmado")
        self._estado = EstadoPedido.CANCELADO

    def __repr__(self) -> str:
        return f"Pedido({self._cliente!r}, total=${self.total():.2f}, {self._estado.value})"


# Demostración
laptop = Producto("Laptop", 1500.0, 10)
mouse = Producto("Mouse", 45.0, 50)
cliente = Cliente("Carlos López", "carlos@ejemplo.com")

pedido = Pedido(cliente)
pedido.agregar_producto(laptop, 2)
pedido.agregar_producto(mouse, 3)
pedido.confirmar()

print(pedido)         # Pedido(Cliente('Carlos López'), total=$3135.00, confirmado)
print(pedido.total()) # 3135.0
```

## El cambio lleva tiempo

No te preocupés si al principio seguís pensando en "pasos". Es completamente normal. La mentalidad procedural está profundamente arraigada y el cambio al diseño orientado a objetos lleva semanas de práctica deliberada.

Una señal de progreso: cuando empezás a diseñar las clases *antes* de escribir el código, preguntándote qué entidades existen y cómo se relacionan, ya estás pensando orientado a objetos.

---
