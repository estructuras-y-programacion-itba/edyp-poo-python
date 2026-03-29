# Los Cuatro Elementos del Modelo

A lo largo del capítulo de Fundamentos fuiste construyendo un kit de herramientas: constructores, métodos de instancia, type hints, encapsulamiento, enums. Esta sección da un paso atrás para integrar esas piezas en un marco conceptual sólido: los cuatro elementos que Booch identificó como pilares de todo modelo orientado a objetos bien formado.

Grady Booch identificó cuatro elementos que todo modelo orientado a objetos bien formado debe contemplar. No son características opcionales ni capas que se agregan al final: son los pilares sobre los que se construye el diseño. Entender qué es cada uno —y en qué se diferencia de los demás— es el primer paso para tomar decisiones de diseño fundamentadas.

Los cuatro elementos son:

1. **Abstracción** — qué características esenciales expone un objeto
2. **Encapsulamiento** — cómo se protege y controla el estado interno
3. **Modularidad** — cómo se organiza el sistema en unidades cohesivas e independientes
4. **Jerarquía** — cómo se relacionan y ordenan las abstracciones entre sí

## Abstracción

La abstracción consiste en identificar las características esenciales de un objeto e ignorar los detalles irrelevantes para el contexto. Un objeto expone una **interfaz** — lo que puede hacer — y oculta su **implementación** — cómo lo hace. Quien usa el objeto no necesita saber nada de sus entrañas.

En Python, la abstracción formal se implementa con la librería `abc`. Una clase abstracta define el *contrato* que todas sus subclases deben cumplir, sin especificar cómo lo cumplen.

> Ver en detalle: [Abstracción](../modelado/abstraccion.md)

## Encapsulamiento

El encapsulamiento protege el estado interno de un objeto y controla cómo se accede o modifica desde afuera. La idea central es que cada objeto es el único responsable de mantener su propia consistencia: nadie externo debería poder poner un objeto en un estado inválido.

Abstracción y encapsulamiento son complementarios pero distintos: la abstracción define *qué* puede hacer un objeto; el encapsulamiento protege *cómo* lo hace.

> Ver en detalle: [Encapsulamiento](encapsulamiento.md)

## Modularidad

Todo diseño orientado a objetos enfrenta una tensión inherente: el deseo de encapsular abstracciones versus la necesidad de que ciertas abstracciones sean visibles para otros módulos. La modularidad resuelve esa tensión al descomponer el sistema en un conjunto de módulos **cohesivos** y **débilmente acoplados**.

En Python, cada archivo `.py` es naturalmente un módulo. Dado que abstracción, encapsulación y modularidad son principios sinérgicos, la modularidad opera como el contenedor que los organiza y delimita.

A continuación, el ejemplo muestra tres clases que en un sistema real vivirían en archivos separados (`maquina.py`, `linea_montaje.py`, `reporte.py`). Cada módulo tiene una única responsabilidad y sus dependencias son mínimas:

```python
# --- módulo: maquina.py ---
class Maquina:
    """Encapsula los datos y reglas de negocio de una máquina industrial."""

    def __init__(self, nombre: str, capacidad: int) -> None:
        if capacidad <= 0:
            raise ValueError("La capacidad debe ser positiva")
        self.nombre = nombre
        self._capacidad = capacidad
        self._estado = "inactiva"

    @property
    def capacidad(self) -> int:
        return self._capacidad

    @property
    def estado(self) -> str:
        return self._estado

    def activar(self) -> None:
        self._estado = "activa"

    def __repr__(self) -> str:
        return f"Maquina({self.nombre!r}, estado={self._estado!r})"


# --- módulo: linea_montaje.py ---
from maquina import Maquina


class LineaDeMontaje:
    """Gestiona una colección de máquinas. Solo depende de Maquina."""

    def __init__(self) -> None:
        self._maquinas: list[Maquina] = []

    def agregar(self, maquina: Maquina) -> None:
        self._maquinas.append(maquina)

    def buscar(self, nombre: str) -> Maquina | None:
        for m in self._maquinas:
            if m.nombre == nombre:
                return m
        return None

    def total_maquinas(self) -> int:
        return len(self._maquinas)


# --- módulo: reporte.py ---
from linea_montaje import LineaDeMontaje


class ReportePlanta:
    """Genera reportes. Cohesivo: solo sabe de reportes, nada más."""

    def generar(self, linea: LineaDeMontaje) -> str:
        lineas = [
            "=== Reporte de Planta ===",
            f"Total de máquinas: {linea.total_maquinas()}",
        ]
        return "\n".join(lineas)


# Uso
linea = LineaDeMontaje()
linea.agregar(Maquina("CNC-01", capacidad=100))
linea.agregar(Maquina("Soldadora-02", capacidad=50))

reporte = ReportePlanta()
print(reporte.generar(linea))
```

### Cohesión y acoplamiento

Dos métricas guían el diseño modular: **alta cohesión** (cada módulo tiene una única responsabilidad bien definida) y **bajo acoplamiento** (los módulos dependen de pocos otros y solo de sus interfaces públicas). Las definiciones completas, con ejemplos, están en [Acoplamiento y Cohesión](../introduccion/saberes_previos.md#acoplamiento-y-cohesión).

> En la práctica, cuando un cambio en una parte del sistema irradia modificaciones hacia muchos otros archivos, es una señal clara de alto acoplamiento. La solución casi siempre implica identificar qué responsabilidades están mal asignadas y redistribuirlas en módulos más cohesivos.

## Jerarquía

Cualquier sistema real involucra decenas de abstracciones que necesitan relacionarse entre sí. La jerarquía organiza esas relaciones de manera que el sistema siga siendo comprensible. Las dos formas fundamentales son:

- **Jerarquía "es-un"** (herencia): una subclase *es un* tipo específico de su clase padre. `EstacionCorte` es una `EstacionTrabajo`.
- **Jerarquía "tiene-un"** (composición): un objeto *contiene* o *usa* a otro. `Maquina` tiene un `Sensor`.

Elegir entre estas dos es una de las decisiones de diseño más frecuentes y más importantes. La regla práctica: si podés decir con naturalidad que A *es un* B en todos los contextos, usá herencia. Si la relación es que A *tiene un* B o A *usa un* B, usá composición. En la práctica, el abuso de herencia genera jerarquías rígidas y difíciles de cambiar — la composición suele ser la opción más flexible.

> Ver en detalle: [Herencia](../modelado/herencia.md) · [Composición vs. Herencia](../modelado/composicion_vs_herencia.md)

## Ver también

Estos cuatro elementos son el marco para todas las decisiones de diseño que siguen. Los próximos capítulos los aplican en profundidad:

- [Métodos de Clase y Estáticos](../metodos/clase_y_estaticos.md) — más tipos de métodos para completar el comportamiento de una clase
- [Relaciones entre Clases](../modelado/relaciones.md) — cómo se implementa la jerarquía en un sistema con múltiples clases
- [Abstracción](../modelado/abstraccion.md) — la implementación formal del primer elemento con clases abstractas

---
