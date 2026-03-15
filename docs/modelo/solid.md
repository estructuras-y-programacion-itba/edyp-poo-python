# Principios SOLID

## S — Single Responsibility Principle (Principio de Responsabilidad Única)

Una clase debe tener una, y solo una, razón para cambiar.

Tomá el caso de una clase `Usuario` que gestiona datos del usuario, guarda en base de datos y además envía emails de bienvenida: está haciendo demasiado. Separar responsabilidades conduce a un diseño más mantenible: `Usuario` (datos), `UsuarioRepository` (base de datos), `EmailService` (envíos).

```python
# ❌ Viola SRP: la clase hace demasiado
class Invoice:
    def calculate_total(self): ...
    def print_invoice(self): ...    # responsabilidad de presentación
    def save_to_db(self): ...       # responsabilidad de persistencia

# ✅ Respeta SRP: cada clase tiene una razón para cambiar
class Invoice:
    def calculate_total(self) -> float: ...

class InvoicePrinter:
    def print(self, invoice: Invoice) -> None: ...

class InvoiceRepository:
    def save(self, invoice: Invoice) -> None: ...
```

## O — Open/Closed Principle (Principio de Abierto/Cerrado)

Las clases deben estar abiertas para su extensión, pero cerradas para su modificación.

Agregar nueva funcionalidad no debería requerir modificar código que ya funciona —y arriesgarse a romperlo. La solución es el polimorfismo: extendé el comportamiento agregando nuevas clases en lugar de acumular `if/else`.

```python
# ❌ Viola OCP: hay que modificar la clase para agregar un nuevo formato
class ReportExporter:
    def export(self, report, format: str):
        if format == "pdf":
            ...
        elif format == "csv":
            ...
        # Cada nuevo formato requiere modificar esta clase

# ✅ Respeta OCP: se extiende sin modificar
from abc import ABC, abstractmethod

class ReportExporter(ABC):
    @abstractmethod
    def export(self, report) -> None: ...

class PDFExporter(ReportExporter):
    def export(self, report) -> None: ...

class CSVExporter(ReportExporter):
    def export(self, report) -> None: ...
```

## L — Liskov Substitution Principle (Principio de Sustitución de Liskov)

Las clases derivadas deben poder sustituirse por sus clases base sin alterar el comportamiento correcto del programa.

Cuando una clase hija viola el contrato establecido por la clase base —por ejemplo, `PatoDeHule` lanzando una excepción en `volar()` cuando la base promete que ese método es válido—, el principio queda roto. La herencia debe tener sentido tanto semántico como funcional.

```python
# ❌ Viola LSP: PatoDeHule no respeta el contrato de la clase base
class Animal:
    def volar(self) -> str:
        return "Volando..."

class Pato(Animal):
    def volar(self) -> str:
        return "El pato vuela"

class PatoDeHule(Animal):
    def volar(self) -> str:
        raise NotImplementedError("¡Un pato de hule no puede volar!")
        # ❌ Quien use Animal espera poder llamar volar() sin excepciones


# ✅ Respeta LSP: la jerarquía refleja capacidades reales
class Animal:
    """Clase base: solo comportamiento común a todos los animales."""

    def __init__(self, nombre: str) -> None:
        self.nombre = nombre

    def respirar(self) -> str:
        return f"{self.nombre} respira"


class AnimalVolador(Animal):
    """Extensión para animales que pueden volar."""

    def volar(self) -> str:
        return f"{self.nombre} vuela"


class Pato(AnimalVolador):
    def volar(self) -> str:
        return f"{self.nombre} vuela con sus alas"


class PatoDeHule(Animal):
    """No hereda de AnimalVolador porque no puede volar. ✅"""

    def chillar(self) -> str:
        return f"{self.nombre}: ¡Squeak!"


# Ahora podemos substituir Animal por cualquier subclase sin sorpresas
animales: list[Animal] = [Pato("Donald"), PatoDeHule("Rubber")]
for a in animales:
    print(a.respirar())  # ✅ funciona para todos
```

## I — Interface Segregation Principle (Principio de Segregación de Interfaz)

Ninguna clase debería verse forzada a depender de métodos que no utiliza.

Aunque Python no tiene "interfaces" explícitas, este principio cobra especial relevancia al diseñar Clases Base Abstractas (ABCs) o Protocolos. Crear una clase base gigante que obligue a sus subclases a implementar métodos sin sentido para ellas es exactamente lo que se busca evitar: mejor definir contratos pequeños y específicos.

```python
from abc import ABC, abstractmethod

# ❌ Viola ISP: una ABC enorme que fuerza a Robot a implementar
#    métodos que no tiene sentido para él
class Trabajador(ABC):
    @abstractmethod
    def trabajar(self) -> None: ...

    @abstractmethod
    def comer(self) -> None: ...

    @abstractmethod
    def dormir(self) -> None: ...


class Robot(Trabajador):
    def trabajar(self) -> None:
        print("Robot trabajando")

    def comer(self) -> None:
        raise NotImplementedError("Los robots no comen")  # ❌ forzado

    def dormir(self) -> None:
        raise NotImplementedError("Los robots no duermen")  # ❌ forzado


# ✅ Respeta ISP: interfaces pequeñas y específicas
class Trabajable(ABC):
    @abstractmethod
    def trabajar(self) -> None: ...


class Comedor(ABC):
    @abstractmethod
    def comer(self) -> None: ...


class Dormidor(ABC):
    @abstractmethod
    def dormir(self) -> None: ...


class Humano(Trabajable, Comedor, Dormidor):
    """Implementa todo porque un humano hace todo."""

    def trabajar(self) -> None:
        print("Humano trabajando")

    def comer(self) -> None:
        print("Humano comiendo")

    def dormir(self) -> None:
        print("Humano durmiendo")


class Robot(Trabajable):
    """Solo implementa lo que necesita. ✅"""

    def trabajar(self) -> None:
        print("Robot trabajando")
```

## D — Dependency Inversion Principle (Principio de Inversión de Dependencias)

Los módulos de alto nivel no deben depender de módulos de bajo nivel. Ambos deben depender de abstracciones.

Cuando una clase de alto nivel instancia directamente sus dependencias concretas, queda acoplada a una implementación específica —difícil de testear, difícil de cambiar. La inversión de dependencias rompe ese acoplamiento: en lugar de crear la dependencia, la clase la recibe desde afuera (inyección de dependencia), dependiendo de una abstracción que puede tomar cualquier forma concreta.

```python
# ❌ Viola DIP: depende de implementación concreta
class Notification:
    def __init__(self):
        self.email = EmailSender()  # dependencia de bajo nivel

    def send(self, msg: str):
        self.email.send(msg)


# ✅ Respeta DIP: depende de abstracción
class MessageSender(ABC):
    @abstractmethod
    def send(self, message: str) -> None: ...

class Notification:
    def __init__(self, sender: MessageSender) -> None:
        self.sender = sender  # inyección de dependencia

    def send(self, message: str) -> None:
        self.sender.send(message)
```

---
