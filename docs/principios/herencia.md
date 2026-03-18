# Herencia

La herencia es el mecanismo por el cual una clase (la **subclase** o clase hija) adquiere los atributos y métodos de otra clase (la **superclase** o clase padre). La relación que establece es "es-un": una `EstacionCorte` *es una* `EstacionTrabajo`, una `EstacionSoldadura` *es una* `EstacionTrabajo`.

Además de reutilizar código, la herencia permite tratar a objetos de distintas subclases de manera uniforme a través de la interfaz común de su clase padre — lo que habilita el polimorfismo.

## Sintaxis básica

```python
class EstacionTrabajo:
    """Clase base para cualquier estación de la línea de montaje."""

    def __init__(self, nombre: str, tiempo_ciclo: float) -> None:
        self.nombre = nombre
        self.tiempo_ciclo = tiempo_ciclo

    def describir(self) -> str:
        """Descripción genérica de la estación."""
        return f"{self.nombre} (ciclo: {self.tiempo_ciclo}s)"


class EstacionCorte(EstacionTrabajo):  # EstacionCorte hereda de EstacionTrabajo
    """Estación de corte CNC. ES UNA EstacionTrabajo."""

    def __init__(self, nombre: str, tiempo_ciclo: float, velocidad_corte: float) -> None:
        super().__init__(nombre, tiempo_ciclo)  # llama al __init__ del padre
        self.velocidad_corte = velocidad_corte

    def describir(self) -> str:  # override del método del padre
        return f"Corte {self.nombre} a {self.velocidad_corte} mm/min (ciclo: {self.tiempo_ciclo}s)"
```

## `super()`: por qué y cómo usarlo

`super()` retorna un proxy que permite llamar a métodos de la clase padre desde la clase hija. En el constructor, su uso más frecuente es delegar la inicialización de los atributos heredados al `__init__` del padre.

¿Por qué es importante? Porque si no llamás a `super().__init__()`, los atributos definidos en el padre nunca se inicializan en la instancia hija. El objeto queda incompleto.

```python
class EstacionSoldadura(EstacionTrabajo):
    """
    Estación de soldadura MIG. ES UNA EstacionTrabajo.

    Args:
        nombre: Nombre de la estación.
        tiempo_ciclo: Tiempo de ciclo en segundos.
        temperatura_soldadura: Temperatura de soldadura en °C.
    """

    def __init__(self, nombre: str, tiempo_ciclo: float, temperatura_soldadura: float) -> None:
        super().__init__(nombre, tiempo_ciclo)  # ✅ inicializa nombre y tiempo_ciclo
        self.temperatura_soldadura = temperatura_soldadura

    def describir(self) -> str:
        base = super().describir()  # reutiliza el método del padre
        return f"Soldadura {base} a {self.temperatura_soldadura}°C"

    def __repr__(self) -> str:
        return f"EstacionSoldadura({self.nombre!r}, {self.temperatura_soldadura}°C)"


soldadura = EstacionSoldadura("MIG-01", tiempo_ciclo=30.0, temperatura_soldadura=350.0)
print(soldadura.describir())    # Soldadura MIG-01 (ciclo: 30.0s) a 350.0°C
print(soldadura.nombre)         # MIG-01 — heredado de EstacionTrabajo
```

## Method overriding: redefinir el comportamiento heredado

Una subclase puede **redefinir** (override) cualquier método de la clase padre. Python siempre busca primero el método en la clase del objeto y recién entonces sube por la jerarquía. Esto permite que cada subclase tenga su propio comportamiento específico manteniendo la misma firma.

```python
class EstacionTrabajo:
    """Estación base con método de procesamiento genérico."""

    def __init__(self, nombre: str) -> None:
        self.nombre = nombre

    def procesar(self, pieza: str) -> str:
        """Comportamiento genérico de procesamiento."""
        return f"{self.nombre} procesa {pieza}"


class EstacionCorte(EstacionTrabajo):
    """Estación de corte con procesamiento específico."""

    def procesar(self, pieza: str) -> str:  # override
        return f"{self.nombre} corta {pieza} con láser CNC"


class EstacionSoldadura(EstacionTrabajo):
    """Estación de soldadura con procesamiento específico."""

    def procesar(self, pieza: str) -> str:  # override
        return f"{self.nombre} suelda {pieza} con arco MIG"


class EstacionPintura(EstacionTrabajo):
    """Estación de pintura con procesamiento específico."""

    def procesar(self, pieza: str) -> str:  # override
        return f"{self.nombre} pinta {pieza} con cabina electroestática"


# Polimorfismo: misma llamada, distinto resultado
estaciones: list[EstacionTrabajo] = [
    EstacionCorte("CNC-01"),
    EstacionSoldadura("MIG-01"),
    EstacionPintura("PAINT-01"),
]
for est in estaciones:
    print(est.procesar("P-0001"))
# CNC-01 corta P-0001 con láser CNC
# MIG-01 suelda P-0001 con arco MIG
# PAINT-01 pinta P-0001 con cabina electroestática
```

## El test "es-un"

Antes de usar herencia, aplicá este test: ¿Podés decir con naturalidad que "B es un A" *en todos los contextos posibles*?

- `EstacionCorte` es una `EstacionTrabajo` → ✅
- `EstacionSoldadura` es una `EstacionTrabajo` → ✅
- `LineaDeMontaje` es una `EstacionTrabajo` → ❌ (una línea tiene estaciones, no es una estación)
- `RegistradorEventos` es una `Maquina` → ❌ (herencia por conveniencia, no por relación real)

Si el test falla o generás dudas, usá composición.

## Cuándo NO usar herencia

La herencia crea un acoplamiento muy fuerte entre padre e hijo. Cambiar la clase padre puede romper todas las subclases. Martin Fowler lo llama el "problema de la clase base frágil".

Preferí composición cuando:

- La relación es "usa-un" o "tiene-un", no "es-un".
- Querés reutilizar comportamiento de múltiples fuentes (Python permite herencia múltiple, pero puede ser confuso).
- El padre y el hijo evolucionan de forma independiente.

```python
# ❌ Herencia incorrecta: un RegistradorEventos NO ES UNA Maquina
class Maquina:
    def iniciar_ciclo(self) -> None:
        print("Iniciando ciclo...")


class RegistradorMalo(Maquina):  # hereda solo por conveniencia → error de diseño
    def registrar(self, evento: str) -> None:
        self.iniciar_ciclo()  # llama al método del padre sin sentido semántico
        print(f"[LOG] {evento}")


# ✅ Composición correcta: el Registrador USA una referencia a la Maquina
class RegistradorEventos:
    """
    Registrador que delega operaciones a la máquina.

    Args:
        maquina: Instancia de la máquina a registrar.
    """

    def __init__(self, maquina: Maquina) -> None:
        self._maquina = maquina

    def registrar_ciclo(self, evento: str) -> None:
        """Registra el evento y ejecuta el ciclo correspondiente."""
        self._maquina.iniciar_ciclo()
        print(f"[LOG] {evento}")
```

## MRO: el orden en que Python busca métodos

Python usa el **Method Resolution Order (MRO)** para determinar en qué orden busca métodos a través de la jerarquía de clases. Podés consultarlo con el atributo `__mro__`:

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

class EstacionRoboticaCorte(EstacionCorte, EstacionRobotica):  # herencia múltiple
    pass

# MRO: EstacionRoboticaCorte → EstacionCorte → EstacionRobotica → EstacionTrabajo → object
print(EstacionRoboticaCorte.__mro__)

er = EstacionRoboticaCorte()
print(er.describir())  # "EstacionCorte" — Python encuentra describir() en EstacionCorte primero
```

El MRO sigue el algoritmo C3 Linearization, que garantiza un orden consistente y sin ambigüedades. En herencia simple (el caso más común), el orden es simplemente hijo → padre → abuelo → ... → `object`.

## Ejemplo completo: jerarquía EstacionTrabajo → subclases

```python
class EstacionTrabajo:
    """
    Abstracción base para cualquier estación de la línea de montaje.

    Args:
        nombre: Nombre de la estación.
        tiempo_ciclo: Tiempo de ciclo en segundos.
    """

    def __init__(self, nombre: str, tiempo_ciclo: float) -> None:
        self.nombre = nombre
        self.tiempo_ciclo = tiempo_ciclo
        self._piezas_procesadas: int = 0

    def procesar(self, pieza: str) -> str:
        """Procesa una pieza. Debe ser sobreescrito por subclases."""
        self._piezas_procesadas += 1
        return f"{self.nombre} procesa {pieza}"

    def info(self) -> str:
        """Retorna información básica de la estación."""
        return f"{self.nombre} — ciclo: {self.tiempo_ciclo}s, piezas: {self._piezas_procesadas}"

    def __repr__(self) -> str:
        return f"{self.__class__.__name__}({self.nombre!r})"


class EstacionCorte(EstacionTrabajo):
    """
    Estación de corte CNC. ES UNA EstacionTrabajo.

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
        return f"[CORTE] {pieza} a {self.velocidad_corte} mm/min"

    def info(self) -> str:
        return f"{super().info()} — velocidad: {self.velocidad_corte} mm/min"


class EstacionSoldadura(EstacionTrabajo):
    """
    Estación de soldadura MIG. ES UNA EstacionTrabajo.

    Args:
        nombre: Nombre de la estación.
        tiempo_ciclo: Tiempo de ciclo en segundos.
        temperatura_soldadura: Temperatura de soldadura en °C.
    """

    def __init__(self, nombre: str, tiempo_ciclo: float, temperatura_soldadura: float) -> None:
        super().__init__(nombre, tiempo_ciclo)
        self.temperatura_soldadura = temperatura_soldadura

    def procesar(self, pieza: str) -> str:
        self._piezas_procesadas += 1
        return f"[SOLDADURA] {pieza} a {self.temperatura_soldadura}°C"

    def info(self) -> str:
        return f"{super().info()} — temperatura: {self.temperatura_soldadura}°C"


# Demostración
estaciones: list[EstacionTrabajo] = [
    EstacionCorte("CNC-01", tiempo_ciclo=12.0, velocidad_corte=200.0),
    EstacionCorte("CNC-02", tiempo_ciclo=15.0, velocidad_corte=180.0),
    EstacionSoldadura("MIG-01", tiempo_ciclo=30.0, temperatura_soldadura=350.0),
]

for est in estaciones:
    print(est.procesar("P-0001"))
    print(est.info())
    print()

# Verificar relaciones
cnc = estaciones[0]
print(isinstance(cnc, EstacionCorte))    # True
print(isinstance(cnc, EstacionTrabajo)) # True — una EstacionCorte ES UNA EstacionTrabajo
print(type(cnc))                         # <class 'EstacionCorte'>
```

> **En la práctica:** la herencia es una de las herramientas más poderosas y, al mismo tiempo, más abusadas de la POO. En más de un proyecto vi jerarquías de 5 o 6 niveles de profundidad que nadie entendía y que nadie se animaba a tocar porque cualquier cambio rompía algo impredecible. La regla que más me funcionó: si necesitás más de 2 niveles de herencia, seguramente hay composición que todavía no fue identificada.

---
