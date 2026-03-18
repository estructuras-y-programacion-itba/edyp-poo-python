# Clases y Objetos

La distinción entre clase y objeto es la primera y más fundamental de toda la POO. Confundirlos es el error conceptual más común en los primeros pasos.

## La clase: el molde, no la cosa

Una **clase** es una plantilla o molde que describe cómo serán los objetos que se creen a partir de ella. Define qué datos van a tener (atributos) y qué comportamiento van a exhibir (métodos). Por sí sola, una clase no ocupa espacio de datos útil ni ejecuta nada: es solo la *descripción*.

La analogía clásica es la del plano de arquitectura: el plano de una casa no es una casa. Es la descripción de cómo construir una. Podés construir diez casas idénticas a partir del mismo plano, y cada una va a ser independiente de las demás.

En Python, una clase se define con la palabra reservada `class` seguida del nombre en **PascalCase** (también llamado CamelCase: cada palabra empieza con mayúscula):

```python
class Pieza:
    pass  # clase vacía por ahora
```

## El objeto: la instancia concreta

Un **objeto** es una instancia concreta de una clase, creada en tiempo de ejecución. Mientras la clase existe en el código fuente, el objeto existe en la memoria de la computadora mientras el programa se ejecuta.

Se crea invocando la clase como si fuera una función, con el operador `()`:

```python
pieza_a = Pieza()  # se crea una instancia de Pieza
pieza_b = Pieza()  # una instancia diferente, independiente
```

`pieza_a` y `pieza_b` son dos objetos distintos, cada uno con su propia identidad en memoria.

## El constructor: `__init__`

El método `__init__` es el **constructor** de la clase. Python lo llama automáticamente cada vez que se crea un nuevo objeto. Su propósito es inicializar el estado del objeto: asignar valores a los atributos que va a tener.

```python
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
```

Cuando escribís `Pieza("P-0001", "acero", 2.3)`, Python crea un objeto vacío y llama a `__init__` pasando ese objeto como primer argumento (que dentro del método se llama `self`).

## `self`: la referencia al objeto actual

`self` es el primer parámetro de cualquier método de instancia y representa al objeto sobre el que se está operando. Python lo pasa automáticamente: cuando llamás `pieza.describir()`, Python llama internamente a `Pieza.describir(pieza)`.

No es una palabra reservada (podría llamarse diferente), pero es una convención tan universal que nunca deberías cambiarla.

```python
class Pieza:
    """
    Pieza industrial con número de serie, material y peso.

    Args:
        numero_serie: Identificador único de la pieza.
        material: Tipo de material (acero, aluminio, etc.).
        peso: Peso en kilogramos.
    """

    def __init__(self, numero_serie: str, material: str, peso: float) -> None:
        self.numero_serie = numero_serie   # atributo de instancia
        self.material = material           # atributo de instancia
        self.peso = peso                   # atributo de instancia

    def describir(self) -> str:
        """Retorna una descripción legible de la pieza."""
        return f"Pieza {self.numero_serie}: {self.material}, {self.peso:.2f} kg"
```

## Atributos de instancia: el estado de cada objeto

Los **atributos de instancia** son variables que pertenecen a cada objeto individualmente. Se crean asignándolos a través de `self` en `__init__`. Cada objeto tiene su propio espacio de memoria para estos atributos: modificar el atributo de un objeto no afecta al otro.

## Identidad, estado y comportamiento

Todo objeto tiene tres dimensiones:

- **Identidad**: lo que lo hace único en memoria (su dirección). En Python, `id(objeto)` la retorna.
- **Estado**: los valores actuales de sus atributos.
- **Comportamiento**: las operaciones que puede realizar (sus métodos).

```python
class Pieza:
    """
    Pieza industrial con número de serie, material, peso y grosor.

    Args:
        numero_serie: Identificador único de la pieza.
        material: Tipo de material (acero, aluminio, etc.).
        peso: Peso en kilogramos.
        grosor: Grosor en milímetros.
    """

    def __init__(
        self, numero_serie: str, material: str, peso: float, grosor: float = 1.0
    ) -> None:
        self.numero_serie = numero_serie
        self.material = material
        self.peso = peso
        self.grosor = grosor

    def describir(self) -> str:
        """Retorna una descripción legible de la pieza."""
        return f"Pieza {self.numero_serie}: {self.material}, {self.peso:.2f} kg"

    def aplicar_desgaste(self, porcentaje: float) -> None:
        """Reduce el grosor por desgaste durante el procesamiento.

        Args:
            porcentaje: Porcentaje de reducción del grosor (0-100).
        """
        if not 0 < porcentaje <= 100:
            raise ValueError("El porcentaje debe estar entre 0 y 100")
        self.grosor *= (1 - porcentaje / 100)

    def __repr__(self) -> str:
        return f"Pieza({self.numero_serie!r}, {self.material!r}, {self.peso:.2f}kg)"


# Instanciación: se crean dos objetos independientes
pieza_1 = Pieza("P-0001", "acero", 2.3, grosor=5.0)
pieza_2 = Pieza("P-0002", "aluminio", 0.8, grosor=3.0)

# Identidad: son distintos objetos en memoria
print(id(pieza_1) == id(pieza_2))   # False

# Estado: cada objeto tiene su propio estado
print(pieza_1.describir())  # Pieza P-0001: acero, 2.30 kg
print(pieza_2.describir())  # Pieza P-0002: aluminio, 0.80 kg

# Independencia: modificar uno no afecta al otro
pieza_1.aplicar_desgaste(10)
print(pieza_1.grosor)  # 4.5
print(pieza_2.grosor)  # 3.0  ← sin cambios
```

## Las instancias son independientes entre sí

Este punto merece énfasis: cada objeto es su propio mundo. Crear cien instancias de `Pieza` produce cien objetos independientes. El estado de uno no interfiere con el de los demás.

```python
# Tres piezas, tres estados independientes
piezas = [
    Pieza("P-0010", "acero", 1.5, grosor=4.0),
    Pieza("P-0011", "aluminio", 0.9, grosor=2.5),
    Pieza("P-0012", "acero", 2.1, grosor=6.0),
]

# Aplicar desgaste solo a la primera
piezas[0].aplicar_desgaste(20)

for p in piezas:
    print(p)
# Pieza('P-0010', 'acero', 1.50kg)   ← modificada
# Pieza('P-0011', 'aluminio', 0.90kg) ← sin cambios
# Pieza('P-0012', 'acero', 2.10kg)   ← sin cambios
```

> **En la práctica:** un error clásico de quienes empiezan es usar atributos de *clase* (definidos directamente en el cuerpo de la clase, fuera de `__init__`) pensando que son atributos de instancia. Los atributos de clase son compartidos por *todas* las instancias, lo cual raramente es lo que se quiere. Siempre asigná el estado inicial del objeto dentro de `__init__` usando `self`.

---
