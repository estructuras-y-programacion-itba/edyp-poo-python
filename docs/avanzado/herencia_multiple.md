# Herencia Múltiple

!!! info "Antes de continuar"
    Esta sección extiende los conceptos de [Herencia](../modelado/herencia.md), [Composición vs. Herencia](../modelado/composicion_vs_herencia.md) y [Principios SOLID](solid.md). Es material de profundización — asegurate de tener esas bases antes de seguir.

Aplicando los principios SOLID — especialmente el Principio de Sustitución de Liskov — la herencia múltiple requiere disciplina adicional: cuando una clase tiene varios padres, el orden en que Python resuelve los métodos puede producir comportamientos inesperados. Esta sección cubre la mecánica completa y el patrón mixin, que es la forma correcta de usar herencia múltiple en Python.

En el artículo sobre [herencia](../modelado/herencia.md) vimos que una clase hija adquiere los atributos y métodos de una clase padre: la relación "es-un". En [composición vs. herencia](../modelado/composicion_vs_herencia.md) discutimos cuándo esa relación se vuelve forzada y conviene delegar con composición. Ahí también aparecieron los **mixins** como un punto intermedio: clases pequeñas que inyectan comportamiento transversal sin establecer una jerarquía de dominio.

Lo que todavía no abordamos es la mecánica completa: ¿qué pasa cuando una clase hereda de **más de un padre simultáneamente**? ¿Cómo decide Python cuál método ejecutar si varios padres definen el mismo nombre? A eso se le llama **herencia múltiple** y es la herramienta que hace posible el patrón mixin.

## Qué es la herencia múltiple

Python permite que una clase herede de más de una clase base a la vez. Sintácticamente, se listan las clases padre separadas por comas:

```python
class ClaseHija(PadreA, PadreB, PadreC):
    ...
```

Cada padre aporta sus atributos y métodos a la hija. Esto es distinto a una cadena de herencia simple (A → B → C), donde cada eslabón tiene **un solo padre directo**. Acá, `ClaseHija` tiene tres padres directos al mismo nivel.

La ventaja es la flexibilidad: podés combinar capacidades de orígenes distintos sin duplicar código. El riesgo es la ambigüedad: si `PadreA` y `PadreB` definen un método con el mismo nombre, ¿cuál se ejecuta?

## MRO: cómo Python resuelve la ambigüedad

El **Method Resolution Order (MRO)** es el algoritmo que Python usa para decidir, ante una llamada a método, en qué orden recorrer la jerarquía de clases hasta encontrar la implementación correcta.

### El algoritmo C3 Linearization

Python usa **C3 Linearization** (introducido en Python 2.3) para calcular el MRO. El algoritmo garantiza tres propiedades:

1. **Consistencia**: la clase hija siempre aparece antes que sus padres.
2. **Preservación del orden local**: si declarás `class H(A, B)`, el MRO respeta que `A` va antes que `B`.
3. **Monotonicidad**: si una clase `X` aparece antes que `Y` en el MRO de un ancestro, lo mismo se mantiene en todos los descendientes.

Si estas reglas no pueden satisfacerse —por ejemplo, dos padres contradicen el orden—, Python lanza un `TypeError` al definir la clase. Esto significa que el MRO siempre es determinista y libre de ambigüedades.

### Consultar el MRO

Podés inspeccionar el MRO de cualquier clase con el atributo `__mro__` o el método `mro()`:

```python
class A:
    pass

class B(A):
    pass

class C(A):
    pass

class D(B, C):
    pass

print(D.__mro__)
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)
```

El orden se lee de izquierda a derecha: cuando `D` busca un método, primero revisa en `D`, después en `B`, después en `C`, Luego en `A` y finalmente en `object`.

### Ejemplo concreto de resolución

```python
class EstacionTrabajo:
    def describir(self) -> str:
        return "EstacionTrabajo"

class EstacionCorte(EstacionTrabajo):
    def describir(self) -> str:
        return "EstacionCorte"

class EstacionRobotica(EstacionTrabajo):
    def describir(self) -> str:
        return "EstacionRobotica"

class EstacionRoboticaCorte(EstacionCorte, EstacionRobotica):
    pass  # no define describir()


er = EstacionRoboticaCorte()
print(er.describir())  # "EstacionCorte"

# ¿Por qué? Porque el MRO es:
# EstacionRoboticaCorte → EstacionCorte → EstacionRobotica → EstacionTrabajo → object
# Python encuentra describir() en EstacionCorte primero y deja de buscar.
print(EstacionRoboticaCorte.__mro__)
```

Si necesitaras que `EstacionRobotica` tenga prioridad, bastaría con invertir el orden en la declaración: `class EstacionRoboticaCorte(EstacionRobotica, EstacionCorte)`.

## Ejemplo de uso de herencia multiple: El patrón Mixin

Un **mixin** es una clase diseñada para ser combinada con otras a través de herencia múltiple, pero que **no tiene sentido instanciar por sí sola**. No establece una relación "es-un" de dominio; simplemente agrega un comportamiento auxiliar. La convención en Python es nombrar estos mixins con el sufijo `Mixin`.

En [composición vs. herencia](../modelado/composicion_vs_herencia.md) vimos el ejemplo del `SerializableMixin`. Ahora vamos a desarrollar un caso más completo: una fábrica de estaciones de trabajo que combina capacidades mediante mixins.

### Ejemplo: fábrica de estaciones con mixins

Supongamos que las estaciones de nuestra planta pueden necesitar distintas combinaciones de capacidades transversales: serialización, registro de eventos y validación de piezas. Con herencia simple, tendríamos que crear una jerarquía enorme para cubrir todas las combinaciones. Los mixins resuelven esto con clases pequeñas y enfocadas que se combinan a demanda.

```python
from abc import ABC, abstractmethod


# ── Clase base del dominio ───────────────────────────────────────────────────

class EstacionTrabajo(ABC):
    """Abstracción base para cualquier estación de la línea de montaje."""

    def __init__(self, nombre: str, tiempo_ciclo: float) -> None:
        self.nombre = nombre
        self.tiempo_ciclo = tiempo_ciclo
        self._piezas_procesadas: int = 0

    @abstractmethod
    def procesar(self, pieza: str) -> str:
        """Procesa una pieza. Cada subclase define su lógica específica."""
        ...

    def __repr__(self) -> str:
        return f"{self.__class__.__name__}({self.nombre!r})"


# ── Mixins: comportamientos transversales ────────────────────────────────────

class SerializableMixin:
    """Agrega capacidad de serialización a diccionario."""

    def a_dict(self) -> dict:
        """Convierte los atributos públicos del objeto a un diccionario."""
        return {
            k: v for k, v in self.__dict__.items()
            if not k.startswith("_")
        }


class RegistroMixin:
    """Agrega registro de eventos en consola."""

    def log(self, mensaje: str) -> None:
        """Registra un mensaje con el nombre de la clase."""
        print(f"[{self.__class__.__name__}] {mensaje}")


class ValidacionMixin:
    """Agrega validación de piezas antes del procesamiento."""

    _piezas_validas: set[str] = set()

    def registrar_pieza_valida(self, pieza: str) -> None:
        """Registra una pieza como válida para procesamiento."""
        self._piezas_validas.add(pieza)

    def es_pieza_valida(self, pieza: str) -> bool:
        """Verifica si una pieza está registrada como válida."""
        if not self._piezas_validas:
            return True  # si no hay restricciones, todo es válido
        return pieza in self._piezas_validas
```

Cada mixin tiene una responsabilidad bien acotada. Ninguno tiene sentido por sí solo: `SerializableMixin` no sabe qué atributos serializar hasta que se combina con una clase que los defina.

### Combinando mixins en la fábrica

Ahora combinamos la clase base de dominio con los mixins que necesite cada tipo de estación:

```python
class EstacionCorte(SerializableMixin, RegistroMixin, EstacionTrabajo):
    """
    Estación de corte con serialización y registro.

    Args:
        nombre: Nombre de la estación.
        tiempo_ciclo: Tiempo de ciclo en segundos.
        velocidad_corte: Velocidad de corte en mm/min.
    """

    def __init__(self, nombre: str, tiempo_ciclo: float, velocidad_corte: float) -> None:
        super().__init__(nombre, tiempo_ciclo)
        self.velocidad_corte = velocidad_corte

    def procesar(self, pieza: str) -> str:
        self._piezas_procesadas += 1
        self.log(f"Cortando {pieza} a {self.velocidad_corte} mm/min")
        return f"[CORTE] {pieza}"


class EstacionSoldadura(SerializableMixin, ValidacionMixin, EstacionTrabajo):
    """
    Estación de soldadura con serialización y validación.

    Args:
        nombre: Nombre de la estación.
        tiempo_ciclo: Tiempo de ciclo en segundos.
        temperatura: Temperatura de soldadura en °C.
    """

    def __init__(self, nombre: str, tiempo_ciclo: float, temperatura: float) -> None:
        super().__init__(nombre, tiempo_ciclo)
        self.temperatura = temperatura

    def procesar(self, pieza: str) -> str:
        if not self.es_pieza_valida(pieza):
            return f"[SOLDADURA] {pieza} rechazada — no está en la lista de piezas válidas"
        self._piezas_procesadas += 1
        return f"[SOLDADURA] {pieza} a {self.temperatura}°C"
```

Observá el orden de herencia: los mixins van **antes** de la clase base del dominio. Esto sigue la convención de Python y tiene una razón técnica directa: el MRO se construye de izquierda a derecha. Colocar los mixins primero permite que sus métodos tengan prioridad cuando definen el mismo nombre que la clase base.

### Uso de la fábrica

```python
# Creación de estaciones con distintas combinaciones de capacidades
corte = EstacionCorte("CNC-01", tiempo_ciclo=12.0, velocidad_corte=200.0)
soldadura = EstacionSoldadura("MIG-01", tiempo_ciclo=30.0, temperatura=350.0)

# Serialización (viene del SerializableMixin)
print(corte.a_dict())
# {'nombre': 'CNC-01', 'tiempo_ciclo': 12.0, 'velocidad_corte': 200.0}

print(soldadura.a_dict())
# {'nombre': 'MIG-01', 'tiempo_ciclo': 30.0, 'temperatura': 350.0}

# Registro de eventos (viene del RegistroMixin, solo en EstacionCorte)
corte.procesar("P-0001")
# [EstacionCorte] Cortando P-0001 a 200.0 mm/min

# Validación (viene del ValidacionMixin, solo en EstacionSoldadura)
soldadura.registrar_pieza_valida("P-0001")
print(soldadura.procesar("P-0001"))   # [SOLDADURA] P-0001 a 350.0°C
print(soldadura.procesar("P-9999"))   # [SOLDADURA] P-9999 rechazada — ...

# Polimorfismo: ambas son EstacionTrabajo
estaciones: list[EstacionTrabajo] = [corte, soldadura]
for est in estaciones:
    print(est.procesar("P-0002"))

# Inspección del MRO
print(EstacionCorte.__mro__)
# EstacionCorte → SerializableMixin → RegistroMixin → EstacionTrabajo → ABC → object
```

### Cómo fluye `super()` en herencia múltiple

Un punto clave que suele generar confusión es cómo se comporta `super()` cuando hay herencia múltiple. En herencia simple, `super()` llama al padre. Pero en herencia múltiple, `super()` **sigue el MRO completo**, no solo al padre directo.

```python
class A:
    def metodo(self) -> str:
        return "A"

class B(A):
    def metodo(self) -> str:
        return "B → " + super().metodo()

class C(A):
    def metodo(self) -> str:
        return "C → " + super().metodo()

class D(B, C):
    def metodo(self) -> str:
        return "D → " + super().metodo()


d = D()
print(d.metodo())  # D → B → C → A
# super() en B no salta a A — salta a C, que es el siguiente en el MRO de D.
# MRO: D → B → C → A → object
```

Por eso es importante que cada mixin llame a `super()` cuando corresponda: permite que la cadena de cooperación funcione correctamente a lo largo de todo el MRO.

## Cuándo usar herencia múltiple (y cuándo no)

La herencia múltiple es una herramienta poderosa pero delicada. Estas son las reglas prácticas para usarla con criterio:

**Usá herencia múltiple cuando:**

- Necesitás combinar **comportamientos ortogonales** (serialización, logging, validación) que no comparten estado entre sí.
- Los mixins son **pequeños, enfocados y sin estado propio** (o con estado mínimo e independiente).
- La combinación tiene sentido semántico: cada mixin responde a una pregunta distinta ("¿se puede serializar?", "¿registra eventos?").

**Evitá herencia múltiple cuando:**

- Dos o más padres tienen **estado compartido o conflictivo** (el clásico "problema del diamante" con datos).
- La jerarquía se vuelve difícil de leer: si para entender una clase necesitás abrir 5 archivos, hay complejidad excesiva.
- Usás herencia múltiple para reutilizar código que podría resolverse con composición. Recordá: si la relación no es "es-un" ni "se comporta como", probablemente es "tiene-un".

## El problema del diamante

Uno de los problemas históricos de la herencia múltiple es el **diamante de la herencia**: cuando una clase hereda de dos padres que, a su vez, comparten un ancestro común.

```text
      A
     / \
    B   C
     \ /
      D
```

Python lo resuelve limpiamente gracias al MRO con C3 Linearization. Cada ancestro aparece **una sola vez** en el MRO, en un orden determinista:

```python
class A:
    def __init__(self) -> None:
        print("A.__init__")

class B(A):
    def __init__(self) -> None:
        print("B.__init__")
        super().__init__()

class C(A):
    def __init__(self) -> None:
        print("C.__init__")
        super().__init__()

class D(B, C):
    def __init__(self) -> None:
        print("D.__init__")
        super().__init__()


D()
# D.__init__
# B.__init__
# C.__init__
# A.__init__    ← se ejecuta una sola vez, no dos

print(D.__mro__)
# D → B → C → A → object
```

Si `B` y `C` no usaran `super()`, la cadena se rompería y `A.__init__` podría no ejecutarse o ejecutarse más de una vez. Por eso la regla: **en herencia múltiple, siempre usá `super()`** para delegar al siguiente en el MRO.

> **En la práctica:** la herencia múltiple en Python funciona bien cuando se usa con disciplina. Los mixins bien diseñados son una de las herramientas más elegantes del lenguaje —el propio framework Django los utiliza extensivamente en sus vistas basadas en clases. Pero en cuanto la jerarquía se complica, los mixins dejan de ser la solución y pasan a ser el problema. La regla que más me funcionó: si un mixin necesita conocer detalles de la clase con la que se combina, dejó de ser un mixin y pasó a ser acoplamiento disfrazado.

## Ver también

Con este tema cubierto, tenés el panorama completo del material. El último paso es usar la checklist para verificar la calidad de tus entregas:

- [Checklist](checklist.md) — criterios de calidad: diseño, código Python, validación y testing
- [Principios SOLID](solid.md) — especialmente LSP, que aplica directamente a jerarquías de herencia

---
