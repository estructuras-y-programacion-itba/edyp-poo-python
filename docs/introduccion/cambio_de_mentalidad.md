# El Cambio de Mentalidad

Ya sabemos qué es la POO y cuáles son sus cuatro características fundamentales. Pero conocer la teoría no alcanza: el error más frecuente al aprender POO es seguir pensando de forma procedural y simplemente "envolver" ese código en una clase. Esta sección trabaja exactamente ese error — y cómo evitarlo.

Aprender la sintaxis de las clases en Python es la parte fácil. El verdadero desafío al aprender POO es otro: cambiar la manera en que pensás un problema antes de escribir una sola línea de código.

## De verbos a sustantivos

En programación estructurada, el diseño empieza con una pregunta: **¿qué pasos necesito para resolver esto?** La respuesta son funciones (verbos: calcular, guardar, mostrar, procesar).

En POO, la pregunta inicial es diferente: **¿qué entidades existen en este dominio y cómo se relacionan entre sí?** La respuesta son objetos (sustantivos: pieza, estación de trabajo, línea de montaje, operario).

Este cambio de verbos a sustantivos como unidad de diseño fue descripto por Alan Kay —inventor de Smalltalk y uno de los padres de la POO— quien enfatizaba que los objetos son entidades que *reciben mensajes* y *responden* a ellos. Las acciones existen, pero son responsabilidad de los objetos que las llevan a cabo.

## El error más común: código procedural dentro de clases

Cuando alguien aprende POO por primera vez y viene del paradigma procedural, es muy frecuente escribir algo así:

```python
# ❌ Esto NO es POO — es código procedural disfrazado de clase
class ControladorLinea:
    def procesar_linea(self, materiales, estaciones, operarios, cantidad):
        # 80 líneas de lógica mezclada...
        piezas_ok = 0
        for material in materiales:
            if material["disponible"] > 0:
                for estacion in estaciones:
                    if estacion["activa"]:
                        piezas_ok += 1
        # validar temperaturas, registrar fallas, calcular eficiencia...
        return piezas_ok
```

Este código tiene una clase, pero no tiene *objetos*. Es una función grande con un nombre de clase como disfraz. No hay estado encapsulado, no hay responsabilidades distribuidas, no hay abstracción real.

El diseño orientado a objetos correcto distribuye responsabilidades entre múltiples objetos, cada uno con un propósito claro:

```python
# ✅ Esto SÍ es POO — responsabilidades distribuidas entre objetos
from enum import Enum


class EstadoPieza(Enum):
    EN_ESPERA = "en_espera"
    EN_PROCESO = "en_proceso"
    COMPLETADA = "completada"
    RECHAZADA = "rechazada"


class Pieza:
    """Conoce su propio material, peso y estado en la línea."""

    def __init__(self, numero_serie: str, material: str, peso: float) -> None:
        self.numero_serie = numero_serie
        self.material = material
        self.peso = peso
        self._estado = EstadoPieza.EN_ESPERA

    @property
    def estado(self) -> EstadoPieza:
        return self._estado

    def marcar_en_proceso(self) -> None:
        self._estado = EstadoPieza.EN_PROCESO

    def marcar_completada(self) -> None:
        self._estado = EstadoPieza.COMPLETADA

    def marcar_rechazada(self) -> None:
        self._estado = EstadoPieza.RECHAZADA


class EstacionTrabajo:
    """Procesa piezas. No sabe de operarios ni de reportes."""

    def __init__(self, nombre: str, tiempo_ciclo: float) -> None:
        self.nombre = nombre
        self.tiempo_ciclo = tiempo_ciclo
        self._piezas_procesadas: int = 0

    def procesar(self, pieza: Pieza) -> None:
        pieza.marcar_en_proceso()
        self._piezas_procesadas += 1
        pieza.marcar_completada()

    def piezas_procesadas(self) -> int:
        return self._piezas_procesadas


class LineaDeMontaje:
    """Orquesta estaciones. No sabe de materiales ni de operarios."""

    def __init__(self, nombre: str) -> None:
        self.nombre = nombre
        self._estaciones: list[EstacionTrabajo] = []

    def agregar_estacion(self, estacion: EstacionTrabajo) -> None:
        self._estaciones.append(estacion)

    def procesar_pieza(self, pieza: Pieza) -> None:
        for estacion in self._estaciones:
            estacion.procesar(pieza)
```

> **En la práctica:** cuando revisamos trabajos prácticos, el síntoma más frecuente de "mentalidad procedural" es una única clase con un único método enorme que hace todo. Si ese método tiene más de 20 líneas, casi siempre hay objetos que todavía no fueron identificados.

## Técnica: identificar sustantivos y verbos

Una técnica práctica para diseñar un sistema orientado a objetos desde una descripción en lenguaje natural es subrayar los **sustantivos** (candidatos a clases o atributos) y los **verbos** (candidatos a métodos).

Tomemos este enunciado:

> *"Una línea de montaje contiene estaciones de trabajo. Cada estación procesa piezas que llegan con un material y un peso. Una pieza puede ser completada o rechazada según el resultado del proceso."*

Aplicando la técnica:

| Elemento | Tipo | ¿Qué es en código? |
| --- | --- | --- |
| **línea de montaje** | Sustantivo | Clase `LineaDeMontaje` |
| **estación de trabajo** | Sustantivo | Clase `EstacionTrabajo` |
| **pieza** | Sustantivo | Clase `Pieza` |
| material, peso | Sustantivos (datos) | Atributos de `Pieza` |
| **contiene** | Verbo | `LineaDeMontaje` tiene una lista de `EstacionTrabajo` |
| **procesa** | Verbo | Método `procesar(pieza)` en `EstacionTrabajo` |
| **completada / rechazada** | Verbos | Estados en `EstadoPieza` y métodos en `Pieza` |

El diagrama de clases resultante sería:

```text
LineaDeMontaje  ──────────────>  EstacionTrabajo  ──────────────>  Pieza
               contiene (1..*)                    procesa (1..*)
```

```python
from enum import Enum


class EstadoPieza(Enum):
    EN_ESPERA = "en_espera"
    EN_PROCESO = "en_proceso"
    COMPLETADA = "completada"
    RECHAZADA = "rechazada"


class Pieza:
    """
    Pieza industrial con número de serie, material y peso.

    Args:
        numero_serie: Identificador único de la pieza.
        material: Tipo de material (acero, aluminio, etc.).
        peso: Peso en kilogramos.
    """

    def __init__(self, numero_serie: str, material: str, peso: float) -> None:
        if peso <= 0:
            raise ValueError("El peso debe ser positivo")
        self.numero_serie = numero_serie
        self.material = material
        self.peso = peso
        self._estado = EstadoPieza.EN_ESPERA

    @property
    def estado(self) -> EstadoPieza:
        return self._estado

    def marcar_en_proceso(self) -> None:
        if self._estado != EstadoPieza.EN_ESPERA:
            raise ValueError(f"No se puede iniciar: pieza en estado {self._estado.value}")
        self._estado = EstadoPieza.EN_PROCESO

    def marcar_completada(self) -> None:
        self._estado = EstadoPieza.COMPLETADA

    def marcar_rechazada(self) -> None:
        self._estado = EstadoPieza.RECHAZADA

    def __repr__(self) -> str:
        return f"Pieza({self.numero_serie!r}, {self.material!r}, estado={self._estado.value})"


class EstacionTrabajo:
    """
    Estación que procesa piezas en la línea de montaje.

    Args:
        nombre: Nombre identificador de la estación.
        tiempo_ciclo: Tiempo de ciclo en segundos.
    """

    def __init__(self, nombre: str, tiempo_ciclo: float) -> None:
        self.nombre = nombre
        self.tiempo_ciclo = tiempo_ciclo
        self._piezas_procesadas: int = 0

    def procesar(self, pieza: Pieza) -> None:
        """Procesa una pieza actualizando su estado."""
        pieza.marcar_en_proceso()
        self._piezas_procesadas += 1
        pieza.marcar_completada()

    def piezas_procesadas(self) -> int:
        return self._piezas_procesadas

    def __repr__(self) -> str:
        return f"EstacionTrabajo({self.nombre!r})"


class LineaDeMontaje:
    """
    Línea de montaje que agrupa estaciones de trabajo.

    Args:
        nombre: Nombre de la línea.
    """

    def __init__(self, nombre: str) -> None:
        self.nombre = nombre
        self._estaciones: list[EstacionTrabajo] = []

    def agregar_estacion(self, estacion: EstacionTrabajo) -> None:
        """Agrega una estación al final de la línea."""
        self._estaciones.append(estacion)

    def procesar_pieza(self, pieza: Pieza) -> None:
        """Pasa la pieza por todas las estaciones en orden."""
        for estacion in self._estaciones:
            estacion.procesar(pieza)

    def __repr__(self) -> str:
        return f"LineaDeMontaje({self.nombre!r}, estaciones={len(self._estaciones)})"


# Demostración
corte = EstacionTrabajo("Corte CNC", tiempo_ciclo=12.5)
soldadura = EstacionTrabajo("Soldadura MIG", tiempo_ciclo=30.0)

linea = LineaDeMontaje("Línea A")
linea.agregar_estacion(corte)
linea.agregar_estacion(soldadura)

pieza = Pieza("P-0001", "acero", 2.3)
linea.procesar_pieza(pieza)

print(pieza)         # Pieza('P-0001', 'acero', estado=completada)
print(linea)         # LineaDeMontaje('Línea A', estaciones=2)
```

## El cambio lleva tiempo

No te preocupés si al principio seguís pensando en "pasos". Es completamente normal. La mentalidad procedural está profundamente arraigada y el cambio al diseño orientado a objetos lleva semanas de práctica deliberada.

Una señal de progreso: cuando empezás a diseñar las clases *antes* de escribir el código, preguntándote qué entidades existen y cómo se relacionan, ya estás pensando orientado a objetos.

## Ver también

La técnica de sustantivos y verbos ya te da una hoja de ruta para diseñar. El siguiente paso es ver ese proceso aplicado en un programa completo y ejecutable, antes de descomponer cada concepto por separado:

- [Primer Programa Orientado a Objetos](primer_programa.md) — todos los conceptos juntos en un ejemplo concreto
- [Clases y Objetos](../fundamentos/clases_y_objetos.md) — formalización de la distinción clase / instancia

---
