# Herencia

La herencia es el mecanismo por el cual una clase (la **subclase** o clase hija) adquiere los atributos y métodos de otra clase (la **superclase** o clase padre). La relación que establece es "es-un": un `Auto` *es un* `Vehiculo`, un `Perro` *es un* `Animal`.

Además de reutilizar código, la herencia permite tratar a objetos de distintas subclases de manera uniforme a través de la interfaz común de su clase padre — lo que habilita el polimorfismo.

## Sintaxis básica

```python
class Vehiculo:
    """Clase base para cualquier vehículo."""

    def __init__(self, marca: str, modelo: str) -> None:
        self.marca = marca
        self.modelo = modelo

    def describir(self) -> str:
        """Descripción genérica del vehículo."""
        return f"{self.marca} {self.modelo}"


class Auto(Vehiculo):  # Auto hereda de Vehiculo
    """Auto de pasajeros. ES UN Vehículo."""

    def __init__(self, marca: str, modelo: str, puertas: int) -> None:
        super().__init__(marca, modelo)  # llama al __init__ del padre
        self.puertas = puertas

    def describir(self) -> str:  # override del método del padre
        return f"Auto {self.marca} {self.modelo} ({self.puertas} puertas)"
```

## `super()`: por qué y cómo usarlo

`super()` retorna un proxy que permite llamar a métodos de la clase padre desde la clase hija. En el constructor, su uso más frecuente es delegar la inicialización de los atributos heredados al `__init__` del padre.

¿Por qué es importante? Porque si no llamás a `super().__init__()`, los atributos definidos en el padre nunca se inicializan en la instancia hija. El objeto queda incompleto.

```python
class Moto(Vehiculo):
    """
    Moto. ES UN Vehículo.

    Args:
        marca: Marca de la moto.
        modelo: Modelo de la moto.
        cilindrada: Cilindrada del motor en cc.
    """

    def __init__(self, marca: str, modelo: str, cilindrada: int) -> None:
        super().__init__(marca, modelo)  # ✅ inicializa marca y modelo
        self.cilindrada = cilindrada

    def describir(self) -> str:
        base = super().describir()  # reutiliza el método del padre
        return f"Moto {base} ({self.cilindrada}cc)"

    def __repr__(self) -> str:
        return f"Moto({self.marca!r}, {self.modelo!r}, {self.cilindrada}cc)"


moto = Moto("Honda", "CB500", 500)
print(moto.describir())    # Moto Honda CB500 (500cc)
print(moto.marca)          # Honda — heredado de Vehiculo
```

## Method overriding: redefinir el comportamiento heredado

Una subclase puede **redefinir** (override) cualquier método de la clase padre. Python siempre busca primero el método en la clase del objeto y recién entonces sube por la jerarquía. Esto permite que cada subclase tenga su propio comportamiento específico manteniendo la misma firma.

```python
class Vehiculo:
    """Vehículo base con método de movimiento genérico."""

    def __init__(self, marca: str) -> None:
        self.marca = marca

    def mover(self) -> str:
        """Comportamiento genérico de movimiento."""
        return f"{self.marca} se mueve"


class Auto(Vehiculo):
    """Auto con movimiento específico."""

    def mover(self) -> str:  # override
        return f"{self.marca} rueda sobre el asfalto"


class Barco(Vehiculo):
    """Barco con movimiento específico."""

    def mover(self) -> str:  # override
        return f"{self.marca} navega por el agua"


class Avion(Vehiculo):
    """Avión con movimiento específico."""

    def mover(self) -> str:  # override
        return f"{self.marca} vuela por el aire"


# Polimorfismo: misma llamada, distinto resultado
vehiculos: list[Vehiculo] = [
    Auto("Toyota"),
    Barco("Yamaha"),
    Avion("Boeing"),
]
for v in vehiculos:
    print(v.mover())
# Toyota rueda sobre el asfalto
# Yamaha navega por el agua
# Boeing vuela por el aire
```

## El test "es-un"

Antes de usar herencia, aplicá este test: ¿Podés decir con naturalidad que "B es un A" *en todos los contextos posibles*?

- `Auto` es un `Vehiculo` → ✅
- `Moto` es un `Vehiculo` → ✅
- `Empleado` es una `Persona` → ✅ (generalmente)
- `Stack` es una `Lista` → ❌ (técnicamente hereda, pero viola el contrato de la lista)
- `Logger` es una `BaseDeDatos` → ❌ (herencia por conveniencia, no por relación real)

Si el test falla o generás dudas, usá composición.

## Cuándo NO usar herencia

La herencia crea un acoplamiento muy fuerte entre padre e hijo. Cambiar la clase padre puede romper todas las subclases. Martin Fowler lo llama el "problema de la clase base frágil".

Preferí composición cuando:

- La relación es "usa-un" o "tiene-un", no "es-un".
- Querés reutilizar comportamiento de múltiples fuentes (Python permite herencia múltiple, pero puede ser confuso).
- El padre y el hijo evolucionan de forma independiente.

```python
# ❌ Herencia incorrecta: un Logger NO ES UNA BaseDeDatos
class BaseDeDatos:
    def guardar(self, dato: str) -> None:
        print(f"Guardando en BD: {dato}")


class LoggerMalo(BaseDeDatos):  # hereda solo por conveniencia → error de diseño
    def registrar(self, mensaje: str) -> None:
        self.guardar(f"[LOG] {mensaje}")


# ✅ Composición correcta: el Logger TIENE UNA referencia a la BD
class Logger:
    """
    Logger que delega el almacenamiento a una BD.

    Args:
        bd: Instancia de la base de datos donde guardar los logs.
    """

    def __init__(self, bd: BaseDeDatos) -> None:
        self._bd = bd

    def registrar(self, mensaje: str) -> None:
        """Registra un mensaje usando la base de datos provista."""
        self._bd.guardar(f"[LOG] {mensaje}")
```

## MRO: el orden en que Python busca métodos

Python usa el **Method Resolution Order (MRO)** para determinar en qué orden busca métodos a través de la jerarquía de clases. Podés consultarlo con el atributo `__mro__`:

```python
class A:
    def hablar(self) -> str:
        return "A"

class B(A):
    def hablar(self) -> str:
        return "B"

class C(A):
    def hablar(self) -> str:
        return "C"

class D(B, C):  # herencia múltiple
    pass

# MRO: D → B → C → A → object
print(D.__mro__)
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)

d = D()
print(d.hablar())  # "B" — Python encuentra hablar() en B primero
```

El MRO sigue el algoritmo C3 Linearization, que garantiza un orden consistente y sin ambigüedades. En herencia simple (el caso más común), el orden es simplemente hijo → padre → abuelo → ... → `object`.

## Ejemplo completo: jerarquía Vehiculo → Auto / Moto

```python
class Vehiculo:
    """
    Abstracción base para cualquier vehículo motorizado.

    Args:
        marca: Marca del vehículo.
        modelo: Modelo del vehículo.
        anio: Año de fabricación.
    """

    def __init__(self, marca: str, modelo: str, anio: int) -> None:
        self.marca = marca
        self.modelo = modelo
        self.anio = anio

    def arrancar(self) -> str:
        """Arranca el vehículo."""
        return f"{self.marca} {self.modelo}: motor encendido"

    def info(self) -> str:
        """Retorna información básica del vehículo."""
        return f"{self.anio} {self.marca} {self.modelo}"

    def __repr__(self) -> str:
        return f"{self.__class__.__name__}({self.marca!r}, {self.modelo!r}, {self.anio})"


class Auto(Vehiculo):
    """
    Auto de pasajeros. ES UN Vehículo.

    Args:
        marca: Marca del auto.
        modelo: Modelo del auto.
        anio: Año de fabricación.
        puertas: Número de puertas (2 o 4).
    """

    def __init__(self, marca: str, modelo: str, anio: int, puertas: int = 4) -> None:
        super().__init__(marca, modelo, anio)
        if puertas not in (2, 4):
            raise ValueError("Un auto debe tener 2 o 4 puertas")
        self.puertas = puertas

    def arrancar(self) -> str:
        base = super().arrancar()
        return f"{base} ({self.puertas} puertas)"

    def info(self) -> str:
        return f"{super().info()} — {self.puertas} puertas"


class Moto(Vehiculo):
    """
    Motocicleta. ES UN Vehículo.

    Args:
        marca: Marca de la moto.
        modelo: Modelo de la moto.
        anio: Año de fabricación.
        cilindrada: Cilindrada del motor en cc.
    """

    def __init__(self, marca: str, modelo: str, anio: int, cilindrada: int) -> None:
        super().__init__(marca, modelo, anio)
        self.cilindrada = cilindrada

    def arrancar(self) -> str:
        base = super().arrancar()
        return f"{base} ({self.cilindrada}cc)"

    def info(self) -> str:
        return f"{super().info()} — {self.cilindrada}cc"


# Demostración
vehiculos: list[Vehiculo] = [
    Auto("Toyota", "Corolla", 2022, 4),
    Auto("Ford", "Mustang", 2021, 2),
    Moto("Honda", "CB500F", 2023, 500),
    Moto("Kawasaki", "Ninja 400", 2022, 400),
]

for v in vehiculos:
    print(v.info())
    print(v.arrancar())
    print()

# Verificar relaciones
corolla = vehiculos[0]
print(isinstance(corolla, Auto))     # True
print(isinstance(corolla, Vehiculo)) # True — un Auto ES UN Vehículo
print(type(corolla))                 # <class 'Auto'>
```

> **En la práctica:** la herencia es una de las herramientas más poderosas y, al mismo tiempo, más abusadas de la POO. En más de un proyecto vi jerarquías de 5 o 6 niveles de profundidad que nadie entendía y que nadie se animaba a tocar porque cualquier cambio rompía algo impredecible. La regla que más me funcionó: si necesitás más de 2 niveles de herencia, seguramente hay composición que todavía no fue identificada.

---
