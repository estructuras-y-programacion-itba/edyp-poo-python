# Abstracción

Identificar las características esenciales de un objeto e ignorar los detalles irrelevantes para el contexto es, precisamente, la definición de abstracción. Según Booch, *"una abstracción denota las características esenciales de un objeto que lo distinguen de todos los demás tipos de objetos y, por lo tanto, proporcionan límites conceptuales claramente definidos en relación con la perspectiva del observador."*

![La abstracción se centra en las características esenciales de un objeto según la perspectiva del observador](../img/abstraccion.png)

Para ilustrar esto, pensá en cómo operás una máquina industrial de la planta: presionás el botón de inicio, configurás la velocidad y monitoreás el indicador de temperatura — sin necesitar saber nada sobre el circuito de control, los actuadores hidráulicos ni el PLC interno. La máquina expone una interfaz simple y oculta la complejidad del mecanismo. En POO hacemos exactamente lo mismo con las clases: exponemos lo necesario y ocultamos lo demás.

Llevado al código Python, la abstracción formal se implementa con clases abstractas del módulo `abc`. Una clase abstracta define *qué* operaciones debe ofrecer un objeto, sin especificar *cómo* las implementa cada subclase concreta.

```python
from abc import ABC, abstractmethod


class EstacionTrabajo(ABC):
    """Abstracción de una estación de trabajo en la línea de montaje."""

    @abstractmethod
    def procesar(self, pieza: str) -> str:
        """Procesa una pieza y retorna el resultado."""
        ...

    @abstractmethod
    def obtener_reporte(self) -> str:
        """Retorna un reporte del estado de la estación."""
        ...

    def describir(self) -> str:
        """Descripción general con el reporte incluido."""
        return f"Estación activa — {self.obtener_reporte()}"


class EstacionCorte(EstacionTrabajo):
    """
    Estación de corte CNC.

    Args:
        velocidad_corte: Velocidad de corte en mm/min.
    """

    def __init__(self, velocidad_corte: float) -> None:
        self._velocidad = velocidad_corte
        self._piezas = 0

    def procesar(self, pieza: str) -> str:
        self._piezas += 1
        return f"[CORTE] {pieza} cortado a {self._velocidad} mm/min"

    def obtener_reporte(self) -> str:
        return f"Corte: {self._piezas} piezas procesadas"


class EstacionSoldadura(EstacionTrabajo):
    """
    Estación de soldadura MIG.

    Args:
        temperatura_soldadura: Temperatura de soldadura en °C.
    """

    def __init__(self, temperatura_soldadura: float) -> None:
        self._temperatura = temperatura_soldadura
        self._piezas = 0

    def procesar(self, pieza: str) -> str:
        self._piezas += 1
        return f"[SOLDADURA] {pieza} soldado a {self._temperatura}°C"

    def obtener_reporte(self) -> str:
        return f"Soldadura: {self._piezas} piezas procesadas"


class EstacionPintura(EstacionTrabajo):
    """
    Estación de pintura industrial.

    Args:
        color: Color aplicado en esta estación.
    """

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

## La abstracción como contrato

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

## Niveles de abstracción

La abstracción no es solo cosa de ABCs. Existe en múltiples niveles:

- **Abstracción de datos**: una clase oculta cómo almacena internamente su información.
- **Abstracción de comportamiento**: una clase abstracta o interfaz define qué hace sin decir cómo.
- **Abstracción de subsistema**: un módulo o paquete expone una API simple que oculta su complejidad interna.

```python
from abc import ABC, abstractmethod


class SistemaAlerta(ABC):
    """
    Abstracción de un sistema de alertas de planta.
    Define el contrato sin especificar el canal de notificación.
    """

    @abstractmethod
    def enviar(self, destinatario: str, mensaje: str) -> bool:
        """
        Envía una alerta.

        Args:
            destinatario: Identificador del destinatario (nombre, email, etc.).
            mensaje: Texto de la alerta.

        Returns:
            True si el envío fue exitoso, False en caso contrario.
        """
        ...


class NotificadorOperario(SistemaAlerta):
    """Notifica al operario mediante pantalla de la planta."""

    def enviar(self, destinatario: str, mensaje: str) -> bool:
        print(f"[PANTALLA PLANTA] Para: {destinatario} | {mensaje}")
        return True


class NotificadorSupervisor(SistemaAlerta):
    """Notifica al supervisor mediante email."""

    def enviar(self, destinatario: str, mensaje: str) -> bool:
        print(f"[EMAIL SUPERVISOR] Para: {destinatario} | {mensaje}")
        return True


def alertar_todos(
    sistemas: list[SistemaAlerta],
    destinatario: str,
    mensaje: str,
) -> None:
    """Envía una alerta por todos los sistemas disponibles."""
    for sistema in sistemas:
        sistema.enviar(destinatario, mensaje)


canales: list[SistemaAlerta] = [NotificadorOperario(), NotificadorSupervisor()]
alertar_todos(canales, "Juan Pérez", "Temperatura fuera de rango en CNC-01")
```

## En la práctica: ABC vs duck typing

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
print(robot.describir())          # ❌ describir() no existe en duck typing — error en runtime
```

> **En la práctica:** en proyectos de equipo o en código que otros van a consumir, preferí ABC. El contrato explícito evita bugs silenciosos: si olvidás implementar un método, el error aparece al instanciar la clase, no horas después en producción cuando ese método es llamado. Para código propio y proyectos pequeños, duck typing es perfectamente válido y más ágil.

---
