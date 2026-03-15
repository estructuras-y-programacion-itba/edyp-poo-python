# Abstracción

Identificar las características esenciales de un objeto e ignorar los detalles irrelevantes para el contexto es, precisamente, la definición de abstracción. Según Booch, *"una abstracción denota las características esenciales de un objeto que lo distinguen de todos los demás tipos de objetos y, por lo tanto, proporcionan límites conceptuales claramente definidos en relación con la perspectiva del observador."*

![La abstracción se centra en las características esenciales de un objeto según la perspectiva del observador](../img/abstraccion.png)

Para ilustrar esto, pensá en cómo manejás un auto: usás el volante, el acelerador y los frenos sin necesitar saber nada sobre cómo funciona el motor internamente. El auto expone una interfaz simple y oculta la complejidad del mecanismo. En POO hacemos exactamente lo mismo con las clases: exponemos lo necesario y ocultamos lo demás.

Llevado al código Python, la abstracción formal se implementa con clases abstractas del módulo `abc`. Una clase abstracta define *qué* operaciones debe ofrecer un objeto, sin especificar *cómo* las implementa cada subclase concreta.

```python
from abc import ABC, abstractmethod


class Figura(ABC):
    """Abstracción de una figura geométrica."""

    @abstractmethod
    def area(self) -> float:
        """Calcula el área de la figura."""
        ...

    @abstractmethod
    def perimetro(self) -> float:
        """Calcula el perímetro de la figura."""
        ...

    def describir(self) -> str:
        """Descripción textual con área y perímetro."""
        return f"Área: {self.area():.2f}, Perímetro: {self.perimetro():.2f}"


class Circulo(Figura):
    """
    Figura geométrica circular.

    Args:
        radio: Radio del círculo (debe ser positivo).
    """

    def __init__(self, radio: float) -> None:
        self.radio = radio

    def area(self) -> float:
        import math
        return math.pi * self.radio ** 2

    def perimetro(self) -> float:
        import math
        return 2 * math.pi * self.radio


class Rectangulo(Figura):
    """
    Figura geométrica rectangular.

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


# Polimorfismo: misma interfaz, distinto comportamiento
figuras: list[Figura] = [Circulo(5), Rectangulo(4, 6)]
for figura in figuras:
    print(figura.describir())
```

## La abstracción como contrato

Una clase abstracta no es solo una "clase que no se puede instanciar". Es un **contrato**: cualquier clase que herede de `Figura` y no implemente `area()` y `perimetro()` lanzará un `TypeError` al intentar ser instanciada. Python impone ese contrato automáticamente.

```python
# Intentar instanciar directamente la clase abstracta falla
try:
    f = Figura()  # ❌ TypeError
except TypeError as e:
    print(e)
# Can't instantiate abstract class Figura without an implementation for abstract methods...

# Una subclase que no implementa todos los métodos también falla
class FiguraIncompleta(Figura):
    def area(self) -> float:
        return 0.0
    # Falta implementar perimetro()

try:
    fi = FiguraIncompleta()  # ❌ TypeError
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


class Notificador(ABC):
    """
    Abstracción de un sistema de notificaciones.
    Define el contrato sin especificar el canal.
    """

    @abstractmethod
    def enviar(self, destinatario: str, mensaje: str) -> bool:
        """
        Envía una notificación.

        Args:
            destinatario: Identificador del destinatario (email, número, etc.).
            mensaje: Texto de la notificación.

        Returns:
            True si el envío fue exitoso, False en caso contrario.
        """
        ...


class NotificadorEmail(Notificador):
    """Notificador que envía emails."""

    def enviar(self, destinatario: str, mensaje: str) -> bool:
        # En producción, aquí iría la lógica de envío de email
        print(f"[EMAIL] Para: {destinatario} | {mensaje}")
        return True


class NotificadorSMS(Notificador):
    """Notificador que envía SMS."""

    def enviar(self, destinatario: str, mensaje: str) -> bool:
        # En producción, aquí iría la lógica de envío de SMS
        print(f"[SMS] Para: {destinatario} | {mensaje}")
        return True


def notificar_todos(
    notificadores: list[Notificador],
    destinatario: str,
    mensaje: str,
) -> None:
    """Envía una notificación por todos los canales disponibles."""
    for notificador in notificadores:
        notificador.enviar(destinatario, mensaje)


canales: list[Notificador] = [NotificadorEmail(), NotificadorSMS()]
notificar_todos(canales, "ana@ejemplo.com", "Tu pedido fue confirmado")
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
class NotificadorWhatsApp:
    """Notificador por WhatsApp. No hereda de Notificador (duck typing)."""

    def enviar(self, destinatario: str, mensaje: str) -> bool:
        print(f"[WHATSAPP] Para: {destinatario} | {mensaje}")
        return True


# Funciona igual porque tiene el método enviar()
wp = NotificadorWhatsApp()
wp.enviar("+54911...", "Tu pedido fue confirmado")  # ✅
```

> **En la práctica:** en proyectos de equipo o en código que otros van a consumir, preferí ABC. El contrato explícito evita bugs silenciosos: si olvidás implementar un método, el error aparece al instanciar la clase, no horas después en producción cuando ese método es llamado. Para código propio y proyectos pequeños, duck typing es perfectamente válido y más ágil.

---
