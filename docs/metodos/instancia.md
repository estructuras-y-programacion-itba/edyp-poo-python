# Métodos de Instancia

Los métodos de instancia son el tipo más común de métodos en Python. Son funciones definidas dentro de una clase que operan sobre un objeto específico: cada llamada a un método de instancia tiene acceso al estado particular de *ese* objeto.

## `self`: el nexo entre el método y el objeto

El primer parámetro de todo método de instancia se llama `self` por convención. Python lo pasa automáticamente cuando llamás el método sobre un objeto: `carrito.agregar_item(producto)` es equivalente a `Carrito.agregar_item(carrito, producto)`.

A través de `self`, el método puede:

- Leer atributos del objeto: `self.items`
- Modificar atributos del objeto: `self._total = 0`
- Llamar a otros métodos del mismo objeto: `self.calcular_total()`

```python
class Carrito:
    def __init__(self) -> None:
        self._items: list[dict] = []  # self da acceso al estado del objeto

    def agregar_item(self, nombre: str, precio: float) -> None:
        self._items.append({"nombre": nombre, "precio": precio})
        # self._items es EL estado de ESTE carrito, no de otro
```

## Métodos que leen vs. métodos que modifican el estado

Los métodos de instancia se dividen en dos grandes grupos según su efecto sobre el estado del objeto:

| Tipo | Efecto | Ejemplo |
| --- | --- | --- |
| **Consulta / getter** | Lee el estado, no lo modifica | `calcular_total()`, `esta_vacio()` |
| **Comando / mutador** | Modifica el estado del objeto | `agregar_item()`, `vaciar()` |

Esta distinción importa para el diseño. Un principio útil es el **Command-Query Separation** de Bertrand Meyer: un método debería *o bien* modificar el estado (comando), *o bien* retornar información (consulta), pero no ambas cosas a la vez. Mezclarlas dificulta el razonamiento sobre el código.

## Convenciones de nomenclatura

- Nombres en **snake_case**: `calcular_total()`, `agregar_item()`.
- Verbos para métodos que realizan acciones: `agregar`, `quitar`, `vaciar`, `confirmar`.
- Sustantivos o adjetivos para métodos que retornan información: `total()`, `esta_vacio()`, `cantidad_items()`.
- Prefijo `_` para métodos internos que no forman parte de la API pública: `_validar_item()`.

## Llamada a métodos

Cuando llamás `objeto.metodo()`, Python busca `metodo` en la clase del objeto y lo llama pasando el objeto como primer argumento (`self`). No tenés que pasarlo vos.

```python
carrito = Carrito()
carrito.agregar_item("Laptop", 1500.0)  # Python pasa carrito como self
```

## Ejemplo completo: clase `Carrito`

```python
class ItemNoEncontradoError(Exception):
    """Se lanza cuando se intenta quitar un ítem que no existe en el carrito."""

    def __init__(self, nombre: str) -> None:
        self.nombre = nombre
        super().__init__(f"El ítem '{nombre}' no está en el carrito")


class Carrito:
    """
    Carrito de compras con manejo de ítems y cálculo de totales.

    Un carrito comienza vacío. Los ítems se agregan con su precio unitario
    y cantidad. El total se calcula a demanda, sin almacenarse para evitar
    inconsistencias.
    """

    def __init__(self) -> None:
        # Estado privado: solo accesible a través de los métodos
        self._items: dict[str, dict] = {}

    # ── Comandos (modifican estado) ──────────────────────────────────────────

    def agregar_item(self, nombre: str, precio: float, cantidad: int = 1) -> None:
        """
        Agrega un ítem al carrito. Si ya existe, suma la cantidad.

        Args:
            nombre: Nombre del producto.
            precio: Precio unitario del producto.
            cantidad: Cantidad a agregar (por defecto 1).

        Raises:
            ValueError: Si el precio o la cantidad no son positivos.
        """
        if precio <= 0:
            raise ValueError(f"El precio debe ser positivo, recibí: {precio}")
        if cantidad <= 0:
            raise ValueError(f"La cantidad debe ser positiva, recibí: {cantidad}")

        if nombre in self._items:
            self._items[nombre]["cantidad"] += cantidad
        else:
            self._items[nombre] = {"precio": precio, "cantidad": cantidad}

    def quitar_item(self, nombre: str) -> None:
        """
        Elimina completamente un ítem del carrito.

        Args:
            nombre: Nombre del producto a quitar.

        Raises:
            ItemNoEncontradoError: Si el ítem no existe en el carrito.
        """
        if nombre not in self._items:
            raise ItemNoEncontradoError(nombre)
        del self._items[nombre]

    def vaciar(self) -> None:
        """Elimina todos los ítems del carrito."""
        self._items.clear()

    # ── Consultas (retornan información, no modifican estado) ────────────────

    def calcular_total(self) -> float:
        """
        Calcula el total del carrito.

        Returns:
            Suma de precio * cantidad para todos los ítems.
        """
        return sum(
            item["precio"] * item["cantidad"]
            for item in self._items.values()
        )

    def cantidad_items(self) -> int:
        """Retorna la cantidad de tipos de productos distintos en el carrito."""
        return len(self._items)

    def esta_vacio(self) -> bool:
        """Indica si el carrito no tiene ningún ítem."""
        return len(self._items) == 0

    def listar(self) -> list[str]:
        """
        Retorna una lista con la descripción de cada ítem.

        Returns:
            Lista de strings con formato 'nombre: $precio x cantidad'.
        """
        return [
            f"{nombre}: ${data['precio']:.2f} x {data['cantidad']}"
            for nombre, data in self._items.items()
        ]

    # ── Métodos mágicos ──────────────────────────────────────────────────────

    def __repr__(self) -> str:
        return f"Carrito({self.cantidad_items()} ítems, total=${self.calcular_total():.2f})"

    def __str__(self) -> str:
        if self.esta_vacio():
            return "Carrito vacío"
        lineas = ["=== Carrito de Compras ==="]
        lineas.extend(self.listar())
        lineas.append(f"─────────────────────────")
        lineas.append(f"TOTAL: ${self.calcular_total():.2f}")
        return "\n".join(lineas)


# Demostración
carrito = Carrito()

# Comandos: modifican el estado
carrito.agregar_item("Laptop", 1500.0)
carrito.agregar_item("Mouse", 45.0, 2)
carrito.agregar_item("Teclado", 80.0)
carrito.agregar_item("Mouse", 45.0, 1)  # suma cantidad: ahora son 3 mouses

print(carrito)
# === Carrito de Compras ===
# Laptop: $1500.00 x 1
# Mouse: $45.00 x 3
# Teclado: $80.00 x 1
# ─────────────────────────
# TOTAL: $1715.00

# Consultas: leen sin modificar
print(f"Total: ${carrito.calcular_total():.2f}")  # $1715.00
print(f"Ítems: {carrito.cantidad_items()}")       # 3
print(f"Vacío: {carrito.esta_vacio()}")           # False

# Quitar un ítem
carrito.quitar_item("Teclado")
print(f"Después de quitar Teclado: ${carrito.calcular_total():.2f}")  # $1635.00

# Manejo de error
try:
    carrito.quitar_item("Monitor")  # no existe
except ItemNoEncontradoError as e:
    print(e)  # El ítem 'Monitor' no está en el carrito
```

## Los métodos de instancia y el encapsulamiento

Los métodos de instancia son la única vía legítima para interactuar con los atributos privados de un objeto. Esta es la esencia del encapsulamiento: el estado interno (`_items`) no se expone directamente; en cambio, los métodos de instancia (`agregar_item`, `quitar_item`, `calcular_total`) definen la API a través de la cual el mundo exterior puede interactuar con el carrito.

Si el estado fuera público, cualquier parte del código podría hacer `carrito._items["Laptop"]["precio"] = -999` sin pasar por ninguna validación. Los métodos de instancia son el guardianes de la consistencia del estado.

> **En la práctica:** una señal de buen diseño es que podés cambiar completamente la representación interna del estado (por ejemplo, pasar de una `list` a un `dict` para los ítems) sin que el código externo que usa el carrito se entere. Eso es encapsulamiento real funcionando.

---
