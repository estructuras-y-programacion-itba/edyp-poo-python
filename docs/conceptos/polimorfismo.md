# Polimorfismo

El polimorfismo es la capacidad de tratar objetos de distintos tipos de manera uniforme, siempre que compartan una interfaz común. La misma operación —la misma llamada de método— produce resultados distintos dependiendo del tipo real del objeto.

La palabra viene del griego: *poly* (muchas) + *morphē* (formas). Un mismo mensaje, muchas respuestas posibles.

## Duck typing: el polimorfismo de Python

Python es un lenguaje de tipado dinámico y su forma más natural de polimorfismo es el **duck typing**: si un objeto tiene el método que necesitás llamar, funciona — independientemente de su tipo o de si hereda de alguna clase en particular.

El nombre viene de la frase atribuida a James Whitcomb Riley:

> *"Si camina como un pato y grazna como un pato, entonces probablemente sea un pato."*

En código: si el objeto tiene el método `area()`, podés llamarlo. No importa de qué clase es, no importa si hereda de `Figura` o no.

```python
import math


class Circulo:
    """Figura circular."""

    def __init__(self, radio: float) -> None:
        self.radio = radio

    def area(self) -> float:
        return math.pi * self.radio ** 2


class Cuadrado:
    """Figura cuadrada."""

    def __init__(self, lado: float) -> None:
        self.lado = lado

    def area(self) -> float:
        return self.lado ** 2


class Triangulo:
    """Triángulo rectángulo."""

    def __init__(self, base: float, altura: float) -> None:
        self.base = base
        self.altura = altura

    def area(self) -> float:
        return self.base * self.altura / 2


# Duck typing: ninguna de estas clases hereda de la misma base
# pero todas tienen area() → polimorfismo
def calcular_area_total(figuras: list) -> float:
    """Calcula el área total de una colección de figuras."""
    return sum(figura.area() for figura in figuras)


figuras = [Circulo(5), Cuadrado(4), Triangulo(6, 3)]
print(f"Área total: {calcular_area_total(figuras):.2f}")  # Área total: 102.54
```

## Polimorfismo con herencia y method overriding

La forma más clásica de polimorfismo es a través de la herencia: una subclase redefine un método de la clase padre, y al llamarlo, Python ejecuta la versión de la subclase.

```python
import math
from abc import ABC, abstractmethod


class Figura(ABC):
    """
    Abstracción base para figuras geométricas.
    Define el contrato: toda figura debe poder calcular su área y perímetro.
    """

    @abstractmethod
    def area(self) -> float:
        """Calcula el área de la figura."""
        ...

    @abstractmethod
    def perimetro(self) -> float:
        """Calcula el perímetro de la figura."""
        ...

    def describir(self) -> str:
        """Descripción con área y perímetro. Mismo código, distinto resultado."""
        nombre = self.__class__.__name__
        return f"{nombre}: área={self.area():.2f}, perímetro={self.perimetro():.2f}"


class Circulo(Figura):
    """
    Figura circular.

    Args:
        radio: Radio del círculo.
    """

    def __init__(self, radio: float) -> None:
        self.radio = radio

    def area(self) -> float:
        return math.pi * self.radio ** 2

    def perimetro(self) -> float:
        return 2 * math.pi * self.radio


class Rectangulo(Figura):
    """
    Figura rectangular.

    Args:
        ancho: Ancho del rectángulo.
        alto: Alto del rectángulo.
    """

    def __init__(self, ancho: float, alto: float) -> None:
        self.ancho = ancho
        self.alto = alto

    def area(self) -> float:
        return self.ancho * self.alto

    def perimetro(self) -> float:
        return 2 * (self.ancho + self.alto)


class Triangulo(Figura):
    """
    Triángulo con tres lados.

    Args:
        a: Lado a.
        b: Lado b.
        c: Lado c.
    """

    def __init__(self, a: float, b: float, c: float) -> None:
        if a + b <= c or a + c <= b or b + c <= a:
            raise ValueError("Los lados no forman un triángulo válido")
        self.a = a
        self.b = b
        self.c = c

    def area(self) -> float:
        # Fórmula de Herón
        s = (self.a + self.b + self.c) / 2
        return math.sqrt(s * (s - self.a) * (s - self.b) * (s - self.c))

    def perimetro(self) -> float:
        return self.a + self.b + self.c


# Polimorfismo: misma interfaz, distintos resultados
figuras: list[Figura] = [
    Circulo(5),
    Rectangulo(4, 6),
    Triangulo(3, 4, 5),
]

for figura in figuras:
    print(figura.describir())

print(f"\nÁrea total: {sum(f.area() for f in figuras):.2f}")
```

## El beneficio práctico: código que trabaja con cualquier subtipo

El polimorfismo permite escribir funciones que operan sobre la interfaz abstracta sin importar el tipo concreto. Esto hace que el código sea extensible: podés agregar nuevas subclases sin modificar las funciones que las usan.

```python
def figura_mas_grande(figuras: list[Figura]) -> Figura:
    """
    Retorna la figura con mayor área.

    Args:
        figuras: Lista de figuras a comparar.

    Returns:
        La figura con el área más grande.
    """
    return max(figuras, key=lambda f: f.area())


ganadora = figura_mas_grande(figuras)
print(f"La figura más grande es: {ganadora.describir()}")
# Funciona con cualquier Figura, incluso con subclases que todavía no existen
```

## `isinstance()` y `type()`: cuándo usarlos (y cuándo no)

`isinstance(objeto, Clase)` verifica si un objeto es instancia de una clase o de cualquiera de sus subclases. `type(objeto)` retorna el tipo exacto.

El problema es que abusar de estas verificaciones es un **código de olor** que indica que el polimorfismo no está siendo aprovechado. Si te encontrás escribiendo muchos `if isinstance(...)`, probablemente hay un método que debería estar en la clase en lugar de en quien la llama.

```python
# ❌ Esto es polimorfismo manual — antipatrón
def calcular_area_mal(figura) -> float:
    if type(figura) == Circulo:
        return math.pi * figura.radio ** 2
    elif type(figura) == Rectangulo:
        return figura.ancho * figura.alto
    elif type(figura) == Triangulo:
        s = (figura.a + figura.b + figura.c) / 2
        return math.sqrt(s * (s - figura.a) * (s - figura.b) * (s - figura.c))
    else:
        raise TypeError(f"Tipo no soportado: {type(figura)}")

# Cada vez que agregás una nueva figura, tenés que modificar esta función → viola OCP
```

```python
# ✅ Polimorfismo real: cada clase sabe calcular su propia área
def calcular_area_bien(figura: Figura) -> float:
    return figura.area()

# Si agregás un Hexagono mañana, esta función no cambia
```

`isinstance()` tiene usos válidos, principalmente para hacer validaciones de tipo en puntos de entrada (constructores, funciones públicas) o para verificar que un objeto cumple con una interfaz antes de usarlo. Lo que hay que evitar es usarlo como sustituto de los métodos polimórficos.

```python
# ✅ Uso legítimo de isinstance: validar en punto de entrada
def procesar_figura(figura: object) -> None:
    if not isinstance(figura, Figura):
        raise TypeError(f"Se esperaba una Figura, recibí: {type(figura).__name__}")
    print(figura.describir())
```

> **En la práctica:** cuando revisamos código de alumnos, el indicador más frecuente de "polimorfismo no aprovechado" es una cadena de `if/elif` que verifica el tipo del objeto para decidir qué hacer. Cada vez que te tentés a escribir `if isinstance(x, TipoA): ... elif isinstance(x, TipoB): ...`, preguntate: ¿debería este comportamiento estar en un método de `TipoA` y `TipoB` directamente?

---
