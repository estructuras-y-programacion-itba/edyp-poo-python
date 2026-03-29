# Abstracción

Identificar las características esenciales de un objeto e ignorar los detalles irrelevantes para el contexto es, precisamente, la definición de abstracción. Según Booch, *"una abstracción denota las características esenciales de un objeto que lo distinguen de todos los demás tipos de objetos y, por lo tanto, proporcionan límites conceptuales claramente definidos en relación con la perspectiva del observador."*

![La abstracción se centra en las características esenciales de un objeto según la perspectiva del observador](../img/abstraccion.png)

Para ilustrar esto, pensá en cómo operás una máquina industrial de la planta: presionás el botón de inicio, configurás la velocidad y monitoreás el indicador de temperatura — sin necesitar saber nada sobre el circuito de control, los actuadores hidráulicos ni el PLC interno. La máquina expone una interfaz simple y oculta la complejidad del mecanismo. En POO hacemos exactamente lo mismo con las clases: exponemos lo necesario y ocultamos lo demás.

## Abstracción en el diseño de clases

La abstracción no es solo para clases especiales — es una decisión de diseño que tomás al escribir *cualquier* clase. Cada vez que definís qué atributos y métodos va a tener una clase, estás eligiendo cuál es el modelo mental relevante para el problema y descartando todo lo demás.

Tomemos el ejemplo de una estación de trabajo en una línea de montaje. Una `EstacionCorte` real tiene decenas de parámetros internos: voltaje del motor, temperatura del aceite, estado de los sensores, historial de mantenimiento. Pero para el sistema de producción que gestiona la línea, solo importan dos cosas: procesar una pieza y reportar su estado. Eso es la abstracción en acción.

```python
class EstacionCorte:
    """
    Abstracción de una estación de corte CNC.

    El sistema de producción solo necesita saber que puede
    procesar piezas y reportar su estado. Los detalles del
    mecanismo de corte son irrelevantes para ese contexto.
    """

    def __init__(self, velocidad_corte: float) -> None:
        self._velocidad = velocidad_corte  # detalle interno, no expuesto
        self._piezas = 0                   # estado interno

    def procesar(self, pieza: str) -> str:
        """Procesa una pieza. Interfaz pública."""
        self._piezas += 1
        return f"[CORTE] {pieza} cortado a {self._velocidad} mm/min"

    def obtener_reporte(self) -> str:
        """Reporta el estado. Interfaz pública."""
        return f"Corte: {self._piezas} piezas procesadas"
```

La decisión de diseño fue: ¿qué necesita saber el sistema de producción sobre esta estación? Solo `procesar()` y `obtener_reporte()`. El resto es un detalle interno que la clase gestiona sola.

Esta misma decisión la tomás al modelar cualquier cosa: un `Cliente`, un `Pedido`, un `Sensor`. Antes de escribir una sola línea de código, la pregunta es: *¿qué rol cumple este objeto en el sistema y qué necesita exponer para cumplirlo?*

## Abstracción y encapsulamiento

Abstracción y encapsulamiento son conceptos relacionados pero distintos:

- **Abstracción** responde a *¿qué* modelo del objeto es relevante para este problema?
- **Encapsulamiento** responde a *¿cómo* protejo ese modelo de accesos no controlados?

Primero abstraés (decidís qué exponer), después encapsulás (decidís cómo proteger lo que no exponés). En la práctica van de la mano, pero conceptualmente son dos decisiones separadas.

## Clases abstractas en Python

Cuando trabajás con jerarquías de clases, la abstracción toma una forma específica: definir un contrato que todas las subclases deben respetar. Una clase abstracta declara *qué* operaciones debe ofrecer cualquier objeto de esa familia, sin especificar *cómo* las implementa cada subclase concreta.

En Python esto se implementa con el módulo `abc`:

```python
from abc import ABC, abstractmethod


class EstacionTrabajo(ABC):
    """
    Contrato para cualquier estación de la línea de montaje.

    Define qué operaciones debe ofrecer una estación, sin
    importar si es de corte, soldadura o pintura.
    """

    @abstractmethod
    def procesar(self, pieza: str) -> str:
        """Procesa una pieza y retorna el resultado."""
        ...

    @abstractmethod
    def obtener_reporte(self) -> str:
        """Retorna un reporte del estado de la estación."""
        ...

    def describir(self) -> str:
        """Descripción general — disponible para todas las subclases."""
        return f"Estación activa — {self.obtener_reporte()}"


class EstacionCorte(EstacionTrabajo):
    def __init__(self, velocidad_corte: float) -> None:
        self._velocidad = velocidad_corte
        self._piezas = 0

    def procesar(self, pieza: str) -> str:
        self._piezas += 1
        return f"[CORTE] {pieza} cortado a {self._velocidad} mm/min"

    def obtener_reporte(self) -> str:
        return f"Corte: {self._piezas} piezas procesadas"


class EstacionSoldadura(EstacionTrabajo):
    def __init__(self, temperatura_soldadura: float) -> None:
        self._temperatura = temperatura_soldadura
        self._piezas = 0

    def procesar(self, pieza: str) -> str:
        self._piezas += 1
        return f"[SOLDADURA] {pieza} soldado a {self._temperatura}°C"

    def obtener_reporte(self) -> str:
        return f"Soldadura: {self._piezas} piezas procesadas"


class EstacionPintura(EstacionTrabajo):
    def __init__(self, color: str) -> None:
        self._color = color
        self._piezas = 0

    def procesar(self, pieza: str) -> str:
        self._piezas += 1
        return f"[PINTURA] {pieza} pintado en {self._color}"

    def obtener_reporte(self) -> str:
        return f"Pintura ({self._color}): {self._piezas} piezas procesadas"


# Polimorfismo: misma interfaz, distinto comportamiento
estaciones: list[EstacionTrabajo] = [
    EstacionCorte(200.0),
    EstacionSoldadura(350.0),
    EstacionPintura("rojo"),
]
for est in estaciones:
    print(est.procesar("P-0001"))
    print(est.describir())
```

### La clase abstracta como contrato

Una clase abstracta no es solo una "clase que no se puede instanciar". Es un **contrato**: cualquier clase que herede de `EstacionTrabajo` y no implemente `procesar()` y `obtener_reporte()` lanzará un `TypeError` al intentar ser instanciada. Python impone ese contrato automáticamente.

```python
# Intentar instanciar directamente la clase abstracta falla
try:
    est = EstacionTrabajo()  # ❌ TypeError
except TypeError as e:
    print(e)
# Can't instantiate abstract class EstacionTrabajo without an implementation...

# Una subclase que no implementa todos los métodos también falla
class EstacionIncompleta(EstacionTrabajo):
    def procesar(self, pieza: str) -> str:
        return f"Procesando {pieza}"
    # Falta implementar obtener_reporte()

try:
    ei = EstacionIncompleta()  # ❌ TypeError
except TypeError as e:
    print(e)
```

La diferencia con duck typing es el momento en que aparece el error: con ABC el error aparece al instanciar la clase incompleta; con duck typing aparece al llamar el método ausente, que puede ser mucho más tarde en la ejecución.

!!! info "Antes de continuar"
    Esta sección usa herencia y polimorfismo. Si todavía no los leíste, revisá [Herencia](herencia.md) y [Polimorfismo](polimorfismo.md).

### ABC vs duck typing

Python ofrece dos caminos para lograr abstracción polimórfica:

1. **Clases abstractas (ABC)**: explícitas, imponen el contrato en tiempo de instanciación, ofrecen mejor soporte en IDEs y linters.
2. **Duck typing**: implícito, si el objeto tiene el método necesario, funciona. Sin herencia, sin declaración formal.

| Criterio | ABC | Duck typing |
| --- | --- | --- |
| Contrato explícito | Sí — `abstractmethod` lo impone | No — el error aparece al llamar el método |
| Flexibilidad | Requiere herencia de la ABC | Cualquier objeto con el método sirve |
| Soporte en IDEs | Excelente autocompletado y type checking | Depende de los type hints |
| Cuándo usarlo | APIs públicas, frameworks, código de biblioteca | Código interno, scripts, prototipos rápidos |

```python
# Duck typing: sin ABC, sin herencia formal
class EstacionRobotica:
    """Estación robótica. No hereda de EstacionTrabajo (duck typing)."""

    def __init__(self, modelo: str) -> None:
        self._modelo = modelo

    def procesar(self, pieza: str) -> str:
        return f"[ROBOT {self._modelo}] Procesando {pieza} automáticamente"

    def obtener_reporte(self) -> str:
        return f"Robot {self._modelo} operativo"


# Funciona igual porque tiene los métodos procesar() y obtener_reporte()
robot = EstacionRobotica("ABB-IRB6700")
print(robot.procesar("P-0005"))   # ✅
print(robot.describir())          # ❌ describir() no existe — error en runtime
```

> **En la práctica:** en proyectos de equipo o en código que otros van a consumir, preferí ABC. El contrato explícito evita bugs silenciosos: si olvidás implementar un método, el error aparece al instanciar la clase, no horas después en producción cuando ese método es llamado. Para código propio y proyectos pequeños, duck typing es perfectamente válido y más ágil.

## Ver también

Con abstracción e herencia estudiadas, el debate de diseño más frecuente en la práctica es cuándo usar cada una:

- [Composición vs. Herencia](composicion_vs_herencia.md) — la decisión de diseño más importante y más abusada en POO
- [Principios SOLID](../avanzado/solid.md) — especialmente el Principio de Sustitución de Liskov, que formaliza cuándo la herencia es correcta

---
