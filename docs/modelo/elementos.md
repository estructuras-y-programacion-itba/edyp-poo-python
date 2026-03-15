# Elementos del Modelo de Objetos

Este modelo consta de cuatro elementos principales:

1. Abstracción
2. Encapsulación
3. Modularidad
4. Jerarquía

## Abstracción

Identificar las características esenciales de un objeto e ignorar los detalles irrelevantes para el contexto es, precisamente, la definición de abstracción. Según Booch, *"una abstracción denota las características esenciales de un objeto que lo distinguen de todos los demás tipos de objetos y, por lo tanto, proporcionan límites conceptuales claramente definidos en relación con la perspectiva del observador."*

![La abstracción se centra en las características esenciales de un objeto según la perspectiva del observador](../img/abstraccion.png)

Para ilustrar esto, pensá en cómo manejás un auto: usás el volante, el acelerador y los frenos sin necesitar saber nada sobre cómo funciona el motor internamente. El auto expone una interfaz simple y oculta la complejidad del mecanismo. En POO hacemos exactamente lo mismo con las clases: exponemos lo necesario y ocultamos lo demás.

Llevado al código Python, la abstracción formal se implementa con clases abstractas del módulo `abc`. Una clase abstracta define *qué* operaciones debe ofrecer un objeto, sin especificar *cómo* las implementa cada subclase concreta.

```python
from abc import ABC, abstractmethod

class Figura(ABC):
    """Abstracción de una figura geométrica."""

    @abstractmethod
    def area(self) -> float:
        """Calcula el área de la figura."""
        ...

    @abstractmethod
    def perimetro(self) -> float:
        """Calcula el perímetro de la figura."""
        ...

    def describir(self) -> str:
        return f"Área: {self.area():.2f}, Perímetro: {self.perimetro():.2f}"


class Circulo(Figura):
    def __init__(self, radio: float) -> None:
        self.radio = radio

    def area(self) -> float:
        import math
        return math.pi * self.radio ** 2

    def perimetro(self) -> float:
        import math
        return 2 * math.pi * self.radio


class Rectangulo(Figura):
    def __init__(self, ancho: float, alto: float) -> None:
        self.ancho = ancho
        self.alto = alto

    def area(self) -> float:
        return self.ancho * self.alto

    def perimetro(self) -> float:
        return 2 * (self.ancho + self.alto)


# Polimorfismo: misma interfaz, distinto comportamiento
figuras: list[Figura] = [Circulo(5), Rectangulo(4, 6)]
for figura in figuras:
    print(figura.describir())
```

## Encapsulamiento

Proteger los datos internos de un objeto y controlar cómo se accede o modifica su estado es la función del encapsulamiento. La idea central es que el estado interno no debería ser accesible directamente desde afuera: cada objeto es el único responsable de mantener su propia consistencia.

Ambos conceptos son complementarios pero no equivalentes. Mientras la abstracción define *qué* puede hacer un objeto (su contrato externo), el encapsulamiento protege *cómo* lo hace (su implementación interna).

![El encapsulamiento oculta los detalles de implementación de un objeto](../img/encapsulamiento.png)

### Convenciones en Python

A diferencia de lenguajes como Java o C++, Python no tiene modificadores de acceso explícitos como `private` o `protected`. En cambio, sigue convenciones que todos los desarrolladores respetan:

| Convención | Ejemplo | Significado |
| --- | --- | --- |
| Sin prefijo | `self.nombre` | Público: accesible desde cualquier lugar |
| Un guión bajo | `self._saldo` | "Protegido": convención de que es interno, no parte de la API pública |
| Dos guiones bajos | `self.__saldo` | "Muy privado": Python aplica *name mangling* para dificultar el acceso externo |

En la práctica, `_un_guion` es suficiente para la mayoría de los casos. `__dos_guiones` se reserva para cuando realmente querés evitar que subclases o código externo accedan directamente al atributo.

```python
class CuentaBancaria:
    """Ejemplo de encapsulamiento: el saldo no es accesible directamente."""

    def __init__(self, titular: str, saldo_inicial: float = 0) -> None:
        self._titular = titular
        self.__saldo = saldo_inicial  # __ = name mangling, muy privado

    @property
    def saldo(self) -> float:
        """El saldo es de solo lectura desde afuera."""
        return self.__saldo

    @property
    def titular(self) -> str:
        return self._titular

    def depositar(self, monto: float) -> None:
        if monto <= 0:
            raise ValueError("El monto debe ser positivo")
        self.__saldo += monto

    def retirar(self, monto: float) -> None:
        if monto <= 0:
            raise ValueError("El monto debe ser positivo")
        if monto > self.__saldo:
            raise ValueError("Saldo insuficiente")
        self.__saldo -= monto

    def __repr__(self) -> str:
        return f"CuentaBancaria({self._titular!r}, saldo=${self.__saldo:.2f})"


cuenta = CuentaBancaria("Ana García", 1000)
cuenta.depositar(500)
print(cuenta.saldo)    # ✅ acceso controlado
# cuenta.__saldo = 999999  # ❌ no funciona (name mangling)
```

## Modularidad

Todo diseño orientado a objetos enfrenta una tensión inherente: el deseo de encapsular abstracciones versus la necesidad de que ciertas abstracciones sean visibles para otros módulos. La modularidad resuelve esa tensión al descomponer el sistema en un conjunto de módulos **cohesivos** y **débilmente acoplados**.

![La modularidad empaqueta abstracciones en unidades discretas e independientes](../img/modularidad.png)

En Python, cada archivo `.py` es naturalmente un módulo. Dado que abstracción, encapsulación y modularidad son principios sinérgicos, la modularidad opera como el contenedor que los organiza y delimita.

A continuación, el ejemplo muestra tres clases que en un sistema real vivirían en archivos separados (`productos.py`, `inventario.py`, `reporte.py`). Cada módulo tiene una única responsabilidad y sus dependencias son mínimas:

```python
# --- módulo: productos.py ---
class Producto:
    """Encapsula los datos y reglas de negocio de un producto."""

    def __init__(self, nombre: str, precio: float) -> None:
        if precio < 0:
            raise ValueError("El precio no puede ser negativo")
        self.nombre = nombre
        self._precio = precio

    @property
    def precio(self) -> float:
        return self._precio

    def __repr__(self) -> str:
        return f"Producto({self.nombre!r}, ${self._precio:.2f})"


# --- módulo: inventario.py ---
class Inventario:
    """Gestiona una colección de productos. Solo depende de Producto."""

    def __init__(self) -> None:
        self._items: list[Producto] = []

    def agregar(self, producto: Producto) -> None:
        self._items.append(producto)

    def buscar(self, nombre: str) -> Producto | None:
        return next((p for p in self._items if p.nombre == nombre), None)

    def total_stock(self) -> int:
        return len(self._items)


# --- módulo: reporte.py ---
class ReporteInventario:
    """Genera reportes. Cohesivo: solo sabe de reportes, nada más."""

    def generar(self, inventario: Inventario) -> str:
        lineas = [f"=== Reporte de Inventario ===",
                  f"Total de productos: {inventario.total_stock()}"]
        return "\n".join(lineas)


# Uso
inv = Inventario()
inv.agregar(Producto("Laptop", 1500))
inv.agregar(Producto("Monitor", 400))

reporte = ReporteInventario()
print(reporte.generar(inv))
```

## Jerarquía

Cualquier sistema real involucra decenas de abstracciones que necesitan relacionarse entre sí. La jerarquía es la forma de organizar esas relaciones de manera que el sistema siga siendo comprensible.

Las dos jerarquías fundamentales en POO son:

- **Estructura de clases** (jerarquía "es-un"): una subclase *es un* tipo específico de su clase padre. Se implementa con **herencia**.
- **Estructura de objetos** (jerarquía "tiene-un"): un objeto *contiene* o *usa* a otro. Se implementa con **composición**.

Elegir mal entre estas dos relaciones es uno de los errores más comunes en el diseño orientado a objetos. La regla es directa: si podés decir con naturalidad que A *es un* B, usá herencia. Si la relación es A *tiene un* B o A *usa un* B, usá composición. En la práctica, el abuso de herencia genera jerarquías rígidas y difíciles de cambiar — la composición suele ser la opción más flexible.

```python
# ── Jerarquía "es-un" (herencia) ────────────────────────────────────────────

class Vehiculo:
    """Un Vehiculo es la abstracción base."""

    def __init__(self, marca: str) -> None:
        self.marca = marca

    def describir(self) -> str:
        return f"Vehículo: {self.marca}"


class Moto(Vehiculo):
    """Una Moto ES UN Vehículo. ✅ herencia correcta."""

    def __init__(self, marca: str, cilindrada: int) -> None:
        super().__init__(marca)
        self.cilindrada = cilindrada

    def describir(self) -> str:
        return f"Moto: {self.marca} ({self.cilindrada}cc)"


# ── Jerarquía "tiene-un" (composición) ──────────────────────────────────────

class Motor:
    """Componente reutilizable e independiente."""

    def __init__(self, cilindros: int) -> None:
        self.cilindros = cilindros

    def encender(self) -> str:
        return f"Motor de {self.cilindros} cilindros encendido"


class Auto:
    """Un Auto TIENE UN Motor. ✅ composición correcta.
    Un Auto no 'es un' Motor, por eso no hereda de él.
    """

    def __init__(self, marca: str, cilindros: int) -> None:
        self.marca = marca
        self._motor = Motor(cilindros)  # composición

    def arrancar(self) -> str:
        return f"{self.marca}: {self._motor.encender()}"


# Demostración
moto = Moto("Honda", 600)
auto = Auto("Toyota", 4)

print(moto.describir())   # herencia: Moto es un Vehículo
print(auto.arrancar())    # composición: Auto tiene un Motor
```

---
