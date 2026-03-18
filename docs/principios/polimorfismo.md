# Polimorfismo

El polimorfismo es la capacidad de tratar objetos de distintos tipos de manera uniforme, siempre que compartan una interfaz común. La misma operación —la misma llamada de método— produce resultados distintos dependiendo del tipo real del objeto.

La palabra viene del griego: *poly* (muchas) + *morphē* (formas). Un mismo mensaje, muchas respuestas posibles.

## Duck typing: el polimorfismo de Python

Python es un lenguaje de tipado dinámico y su forma más natural de polimorfismo es el **duck typing**: si un objeto tiene el método que necesitás llamar, funciona — independientemente de su tipo o de si hereda de alguna clase en particular.

El nombre viene de la frase atribuida a James Whitcomb Riley:

> *"Si camina como un pato y grazna como un pato, entonces probablemente sea un pato."*

En código: si el objeto tiene el método `procesar()`, podés llamarlo. No importa de qué clase es, no importa si hereda de `EstacionTrabajo` o no.

```python
class EstacionCorte:
    """Estación de corte CNC."""

    def __init__(self, velocidad: float) -> None:
        self._velocidad = velocidad

    def procesar(self, pieza: str) -> str:
        return f"[CORTE] {pieza} a {self._velocidad} mm/min"


class EstacionSoldadura:
    """Estación de soldadura MIG."""

    def __init__(self, temperatura: float) -> None:
        self._temperatura = temperatura

    def procesar(self, pieza: str) -> str:
        return f"[SOLDADURA] {pieza} a {self._temperatura}°C"


class EstacionPintura:
    """Estación de pintura industrial."""

    def __init__(self, color: str) -> None:
        self._color = color

    def procesar(self, pieza: str) -> str:
        return f"[PINTURA] {pieza} en color {self._color}"


# Duck typing: ninguna de estas clases hereda de la misma base
# pero todas tienen procesar() → polimorfismo
def procesar_en_todas(estaciones: list, pieza: str) -> list[str]:
    """Procesa una pieza en todas las estaciones disponibles."""
    return [est.procesar(pieza) for est in estaciones]


estaciones = [
    EstacionCorte(200.0),
    EstacionSoldadura(350.0),
    EstacionPintura("rojo"),
]
resultados = procesar_en_todas(estaciones, "P-0001")
for r in resultados:
    print(r)
```

## Polimorfismo con herencia y method overriding

La forma más clásica de polimorfismo es a través de la herencia: una subclase redefine un método de la clase padre, y al llamarlo, Python ejecuta la versión de la subclase.

```python
from abc import ABC, abstractmethod


class EstacionTrabajo(ABC):
    """
    Abstracción base para estaciones de la línea de montaje.
    Define el contrato: toda estación debe poder procesar piezas y generar reportes.
    """

    def __init__(self, nombre: str) -> None:
        self.nombre = nombre
        self._piezas_procesadas: int = 0

    @abstractmethod
    def procesar(self, pieza: str) -> str:
        """Procesa una pieza y retorna el resultado."""
        ...

    @abstractmethod
    def obtener_reporte(self) -> str:
        """Retorna un reporte del estado de la estación."""
        ...

    def describir(self) -> str:
        """Descripción con reporte. Mismo código, distinto resultado."""
        return f"{self.nombre}: {self.obtener_reporte()}"


class EstacionCorte(EstacionTrabajo):
    """
    Estación de corte CNC.

    Args:
        nombre: Nombre de la estación.
        velocidad_corte: Velocidad de corte en mm/min.
    """

    def __init__(self, nombre: str, velocidad_corte: float) -> None:
        super().__init__(nombre)
        self._velocidad = velocidad_corte

    def procesar(self, pieza: str) -> str:
        self._piezas_procesadas += 1
        return f"[CORTE] {pieza} a {self._velocidad} mm/min"

    def obtener_reporte(self) -> str:
        return f"Corte a {self._velocidad} mm/min, {self._piezas_procesadas} piezas"


class EstacionSoldadura(EstacionTrabajo):
    """
    Estación de soldadura MIG.

    Args:
        nombre: Nombre de la estación.
        temperatura_soldadura: Temperatura de soldadura en °C.
    """

    def __init__(self, nombre: str, temperatura_soldadura: float) -> None:
        super().__init__(nombre)
        self._temperatura = temperatura_soldadura

    def procesar(self, pieza: str) -> str:
        self._piezas_procesadas += 1
        return f"[SOLDADURA] {pieza} a {self._temperatura}°C"

    def obtener_reporte(self) -> str:
        return f"Soldadura a {self._temperatura}°C, {self._piezas_procesadas} piezas"


class EstacionPintura(EstacionTrabajo):
    """
    Estación de pintura industrial.

    Args:
        nombre: Nombre de la estación.
        color: Color aplicado en esta estación.
    """

    def __init__(self, nombre: str, color: str) -> None:
        super().__init__(nombre)
        self._color = color

    def procesar(self, pieza: str) -> str:
        self._piezas_procesadas += 1
        return f"[PINTURA] {pieza} en {self._color}"

    def obtener_reporte(self) -> str:
        return f"Pintura ({self._color}), {self._piezas_procesadas} piezas"


# Polimorfismo: misma interfaz, distintos resultados
estaciones: list[EstacionTrabajo] = [
    EstacionCorte("CNC-01", 200.0),
    EstacionSoldadura("MIG-01", 350.0),
    EstacionPintura("PAINT-01", "rojo"),
]

for est in estaciones:
    print(est.procesar("P-0001"))
    print(est.describir())
    print()

print(f"Total piezas procesadas: {sum(e._piezas_procesadas for e in estaciones)}")
```

## El beneficio práctico: el SistemaControl

El polimorfismo permite escribir clases que operan sobre la interfaz abstracta sin importar el tipo concreto. Esto hace que el código sea extensible: podés agregar nuevas subclases sin modificar las clases que las usan.

```python
class SistemaControl:
    """
    Sistema de control que ejecuta el ciclo de producción.
    Depende de la abstracción EstacionTrabajo, no de las concretas.
    """

    def ejecutar_ciclo(
        self, estaciones: list[EstacionTrabajo], pieza: str
    ) -> list[str]:
        """
        Ejecuta el ciclo de producción pasando la pieza por todas las estaciones.

        Args:
            estaciones: Lista de estaciones a ejecutar.
            pieza: Identificador de la pieza a procesar.

        Returns:
            Lista de resultados de cada estación.
        """
        return [est.procesar(pieza) for est in estaciones]


control = SistemaControl()
resultados = control.ejecutar_ciclo(estaciones, "P-0002")
for r in resultados:
    print(r)
# Funciona con cualquier EstacionTrabajo, incluso con subclases que todavía no existen
```

## `isinstance()` y `type()`: cuándo usarlos (y cuándo no)

`isinstance(objeto, Clase)` verifica si un objeto es instancia de una clase o de cualquiera de sus subclases. `type(objeto)` retorna el tipo exacto.

El problema es que abusar de estas verificaciones es un **código de olor** que indica que el polimorfismo no está siendo aprovechado. Si te encontrás escribiendo muchos `if isinstance(...)`, probablemente hay un método que debería estar en la clase en lugar de en quien la llama.

```python
# ❌ Esto es polimorfismo manual — antipatrón
def procesar_mal(estacion, pieza: str) -> str:
    if isinstance(estacion, EstacionCorte):
        return f"Cortando {pieza}"
    elif isinstance(estacion, EstacionSoldadura):
        return f"Soldando {pieza}"
    elif isinstance(estacion, EstacionPintura):
        return f"Pintando {pieza}"
    else:
        raise TypeError(f"Tipo no soportado: {type(estacion)}")

# Cada vez que agregás una nueva estación, tenés que modificar esta función → viola OCP
```

```python
# ✅ Polimorfismo real: cada clase sabe procesar su propia pieza
def procesar_bien(estacion: EstacionTrabajo, pieza: str) -> str:
    return estacion.procesar(pieza)

# Si agregás una EstacionEnsambladoRobotico mañana, esta función no cambia
```

`isinstance()` tiene usos válidos, principalmente para hacer validaciones de tipo en puntos de entrada (constructores, funciones públicas) o para verificar que un objeto cumple con una interfaz antes de usarlo. Lo que hay que evitar es usarlo como sustituto de los métodos polimórficos.

```python
# ✅ Uso legítimo de isinstance: validar en punto de entrada
def ejecutar_estacion(estacion: object, pieza: str) -> str:
    if not isinstance(estacion, EstacionTrabajo):
        raise TypeError(
            f"Se esperaba una EstacionTrabajo, recibí: {type(estacion).__name__}"
        )
    return estacion.procesar(pieza)
```

> **En la práctica:** cuando revisamos código de alumnos, el indicador más frecuente de "polimorfismo no aprovechado" es una cadena de `if/elif` que verifica el tipo del objeto para decidir qué hacer. Cada vez que te tentés a escribir `if isinstance(x, EstacionCorte): ... elif isinstance(x, EstacionSoldadura): ...`, preguntate: ¿debería este comportamiento estar en un método de `EstacionCorte` y `EstacionSoldadura` directamente?

---
