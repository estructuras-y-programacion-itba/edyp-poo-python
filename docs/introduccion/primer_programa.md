# Primer Programa Orientado a Objetos

Antes de descomponer la POO en sus partes —clases, atributos, métodos, encapsulamiento—
conviene ver cómo funciona el conjunto. Esta sección muestra un programa completo y
ejecutable que introduce todos esos conceptos de una sola vez, sin profundizar en
ninguno todavía.

## El problema: datos sin estructura

Imaginá que en una planta industrial necesitás llevar el registro de las piezas en
inventario. Un primer intento en código procedural podría verse así:

```python
# Pieza 1
pieza1_codigo = "P-001"
pieza1_nombre = "Bulón M8"
pieza1_cantidad = 500
pieza1_costo_unitario = 0.45

# Pieza 2
pieza2_codigo = "P-002"
pieza2_nombre = "Tuerca M8"
pieza2_cantidad = 480
pieza2_costo_unitario = 0.30

# Pieza 3
pieza3_codigo = "P-003"
pieza3_nombre = "Arandela M8"
pieza3_cantidad = 1200
pieza3_costo_unitario = 0.10

# ¿Cómo calcular el valor total del stock de la pieza 2?
valor_pieza2 = pieza2_cantidad * pieza2_costo_unitario
print(f"Valor en stock: ${valor_pieza2:.2f}")
```

Este enfoque funciona para tres piezas. Con cincuenta piezas distintas, el código se
vuelve inmanejable: no hay forma de agrupar los datos de una misma pieza, es fácil
mezclar las variables de una pieza con las de otra, y agregar operaciones (como
registrar ingresos o egresos) implica escribir funciones sueltas que reciben los datos
como parámetros.

## La solución orientada a objetos

La POO resuelve esto agrupando los datos y las operaciones que los manipulan en una
sola unidad llamada **objeto**. El molde para crear esos objetos se llama **clase**.

```python
class PiezaEnInventario:
    """Representa una pieza en el inventario de la planta."""

    def __init__(self, codigo: str, nombre: str, cantidad: int, costo_unitario: float) -> None:
        self.codigo = codigo
        self.nombre = nombre
        self.cantidad = cantidad
        self.costo_unitario = costo_unitario

    def registrar_ingreso(self, cantidad: int) -> None:
        """Registra el ingreso de unidades al inventario."""
        self.cantidad += cantidad

    def registrar_egreso(self, cantidad: int) -> None:
        """Registra el egreso de unidades del inventario."""
        self.cantidad -= cantidad

    def calcular_valor_total(self) -> float:
        """Retorna el valor total del stock (cantidad × costo unitario)."""
        return self.cantidad * self.costo_unitario

    def __str__(self) -> str:
        return (
            f"[{self.codigo}] {self.nombre} — "
            f"{self.cantidad} unidades @ ${self.costo_unitario:.2f} "
            f"= ${self.calcular_valor_total():.2f}"
        )
```

## Crear objetos y operar sobre ellos

La clase es el molde. Los **objetos** son las piezas concretas que se crean a partir
de ese molde:

```python
bulon = PiezaEnInventario("P-001", "Bulón M8", 500, 0.45)
tuerca = PiezaEnInventario("P-002", "Tuerca M8", 480, 0.30)
arandela = PiezaEnInventario("P-003", "Arandela M8", 1200, 0.10)

# Cada objeto tiene su propio estado independiente
print(bulon)
print(tuerca)
print(arandela)
```

```text
[P-001] Bulón M8 — 500 unidades @ $0.45 = $225.00
[P-002] Tuerca M8 — 480 unidades @ $0.30 = $144.00
[P-003] Arandela M8 — 1200 unidades @ $0.10 = $120.00
```

Ahora registramos un ingreso de 200 bulones y un egreso de 50 tuercas:

```python
bulon.registrar_ingreso(200)
tuerca.registrar_egreso(50)

print(bulon)
print(tuerca)
```

```text
[P-001] Bulón M8 — 700 unidades @ $0.45 = $315.00
[P-002] Tuerca M8 — 430 unidades @ $0.30 = $129.00
```

Modificar `bulon` no afectó a `tuerca` ni a `arandela`. Cada objeto mantiene su
propio estado.

## ¿Qué acaba de pasar?

Este programa introduce, sin nomenclatura formal todavía, los conceptos centrales de
la POO:

| Concepto | En el ejemplo |
| --- | --- |
| **Clase** | `PiezaEnInventario` — el molde que define cómo son todas las piezas |
| **Objeto** | `bulon`, `tuerca`, `arandela` — instancias concretas de ese molde |
| **Atributos** | `codigo`, `nombre`, `cantidad`, `costo_unitario` — el estado de cada objeto |
| **Métodos** | `registrar_ingreso`, `registrar_egreso`, `calcular_valor_total` — el comportamiento |
| **Constructor** | `__init__` — el método que inicializa cada objeto al crearlo |

La idea clave: los datos y las operaciones que los usan viven juntos en el mismo
objeto. `calcular_valor_total()` no recibe `cantidad` y `costo_unitario` como
parámetros — los lee de `self` porque ya los tiene.

Las secciones que siguen profundizan cada uno de estos elementos.

!!! tip "¿Podés ejecutarlo?"
    Copiá el código completo en una celda de Google Colab y ejecutalo. Experimentá
    cambiando los valores iniciales o agregando una cuarta pieza. La mejor forma de
    entender un concepto nuevo es modificar algo y ver qué cambia.

## Ver también

Este programa introdujo, de forma integrada, todos los conceptos centrales de la POO. Las secciones que siguen profundizan cada uno por separado:

- [Clases y Objetos](../fundamentos/clases_y_objetos.md) — distinción formal entre clase e instancia, atributos de clase vs. de instancia
- [Métodos de Instancia](../fundamentos/instancia.md) — cómo los métodos acceden al estado a través de `self`
- [Encapsulamiento](../fundamentos/encapsulamiento.md) — cómo controlar el acceso a los atributos
