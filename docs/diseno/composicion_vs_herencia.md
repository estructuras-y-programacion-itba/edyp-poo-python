# Composición vs. Herencia

Uno de los debates de diseño más frecuentes en POO es cuándo usar herencia y cuándo usar composición. Tanto Grady Booch como el "Gang of Four" (Gamma, Helm, Johnson, Vlissides en *Design Patterns*) son explícitos al respecto:

> *"Favorecé la composición sobre la herencia."*
>
> — Design Patterns: Elements of Reusable Object-Oriented Software

Esto no significa que la herencia sea mala. Significa que la composición suele ser la herramienta más flexible, y que la herencia debería reservarse para cuando la relación "es-un" sea genuina e inamovible.

## La pregunta fundamental

Antes de elegir, aplicá este test:

| Relación | Pregunta | Implementación |
| --- | --- | --- |
| **es-un** | ¿B es una versión especializada de A? | Herencia (`class B(A)`) |
| **tiene-un** | ¿A posee un B como parte de sí mismo? | Composición (atributo en `__init__`) |
| **usa-un** | ¿A necesita un B para funcionar, pero no lo posee? | Asociación (parámetro o atributo) |

Un `Auto` *es un* `Vehiculo` → herencia.
Un `Auto` *tiene un* `Motor` → composición.
Un `Logger` *usa una* `ConexionDB` → asociación.

## El problema de la clase base frágil

La herencia crea un acoplamiento muy fuerte entre padre e hijo. El hijo depende de los detalles internos del padre: si el padre cambia, el hijo puede romperse aunque nadie lo haya tocado. Este problema se llama **Fragile Base Class Problem**.

```python
# ── Ejemplo del problema ──────────────────────────────────────────────────────

class Contenedor:
    """Clase base que guarda elementos."""

    def __init__(self) -> None:
        self._items: list = []
        self._contador = 0  # cuántos elementos se agregaron en total

    def agregar(self, item) -> None:
        self._items.append(item)
        self._contador += 1

    def agregar_varios(self, items: list) -> None:
        for item in items:
            self.agregar(item)  # llama a agregar() para mantener el contador


class ContenedorAuditado(Contenedor):
    """Quiere contar cuántas veces se agrega un elemento."""

    def __init__(self) -> None:
        super().__init__()
        self._veces_llamado = 0

    def agregar(self, item) -> None:
        self._veces_llamado += 1
        super().agregar(item)


ca = ContenedorAuditado()
ca.agregar("a")
ca.agregar("b")
ca.agregar_varios(["c", "d", "e"])

print(ca._veces_llamado)  # ¿Cuánto esperabas? ¿2? ¿5?
# Es 5 — porque agregar_varios() llama a agregar() en un loop,
# y agregar() está sobreescrito en la subclase.
# Pequeños cambios en la clase padre producen efectos inesperados en los hijos.
```

Este tipo de bug es difícil de detectar porque el código "parece correcto" pero el comportamiento depende de detalles de implementación del padre que podrían cambiar en cualquier momento.

## El mismo problema resuelto con composición

```python
class RegistroDeItems:
    """
    Gestiona una colección de ítems.
    Clase autónoma, no hereda de nada especial.
    """

    def __init__(self) -> None:
        self._items: list = []

    def agregar(self, item) -> None:
        self._items.append(item)

    def agregar_varios(self, items: list) -> None:
        self._items.extend(items)

    def __len__(self) -> int:
        return len(self._items)


class ContenedorAuditadoComposicion:
    """
    Contenedor con auditoría. USA un RegistroDeItems, no lo extiende.

    Al no heredar, no hay riesgo de que cambios en RegistroDeItems
    rompan el comportamiento de la auditoría.
    """

    def __init__(self) -> None:
        self._registro = RegistroDeItems()  # composición
        self._veces_llamado = 0

    def agregar(self, item) -> None:
        """Agrega un ítem y registra la operación."""
        self._registro.agregar(item)
        self._veces_llamado += 1

    def agregar_varios(self, items: list) -> None:
        """Agrega varios ítems registrando cada uno."""
        for item in items:
            self.agregar(item)  # pasa por la auditoría correctamente

    def total_operaciones(self) -> int:
        """Retorna el total de operaciones de agregado."""
        return self._veces_llamado

    def __len__(self) -> int:
        return len(self._registro)


ca = ContenedorAuditadoComposicion()
ca.agregar("a")
ca.agregar("b")
ca.agregar_varios(["c", "d", "e"])

print(ca.total_operaciones())  # 5 — predecible, sin surpresas
print(len(ca))                 # 5
```

## Un caso real: Logger inyectable por composición

Uno de los ejemplos más clásicos de composición sobre herencia es un Logger que se puede "enchufar" a cualquier clase. Con herencia, estarías forzando a todas las clases a ser subclase de `Logger`. Con composición, cualquier clase puede tener un logger.

```python
from abc import ABC, abstractmethod


class Logger(ABC):
    """Abstracción de un logger. Define el contrato."""

    @abstractmethod
    def log(self, mensaje: str) -> None:
        """Registra un mensaje."""
        ...


class LoggerConsola(Logger):
    """Logger que imprime en consola."""

    def log(self, mensaje: str) -> None:
        print(f"[LOG] {mensaje}")


class LoggerArchivo(Logger):
    """Logger que simula escritura en archivo."""

    def __init__(self, ruta: str) -> None:
        self._ruta = ruta

    def log(self, mensaje: str) -> None:
        # En producción, abriría el archivo y escribiría
        print(f"[ARCHIVO:{self._ruta}] {mensaje}")


class LoggerSilencioso(Logger):
    """Logger que no hace nada. Útil para tests."""

    def log(self, mensaje: str) -> None:
        pass  # no hace nada — Null Object pattern


# ── Herencia: el problema ────────────────────────────────────────────────────

class ServicioMalo(LoggerConsola):
    """❌ Hereda Logger solo para tener logging. Relación forzada."""

    def procesar(self, dato: str) -> str:
        self.log(f"Procesando: {dato}")  # usa el método heredado
        return dato.upper()

# ServicioMalo ES UN LoggerConsola → ¿tiene sentido esa relación?
# ¿Qué pasa si querés cambiar al LoggerArchivo? Tenés que cambiar la herencia.


# ── Composición: la solución ─────────────────────────────────────────────────

class ServicioBueno:
    """
    ✅ Recibe un Logger por composición. Flexible y sin acoplamiento.

    Args:
        logger: Implementación del logger a usar.
    """

    def __init__(self, logger: Logger) -> None:
        self._logger = logger  # composición: tiene-un Logger

    def procesar(self, dato: str) -> str:
        """Procesa el dato y registra la operación."""
        self._logger.log(f"Procesando: {dato}")
        return dato.upper()


# La misma clase, tres comportamientos distintos, sin cambiar ServicioBueno
s1 = ServicioBueno(LoggerConsola())
s2 = ServicioBueno(LoggerArchivo("app.log"))
s3 = ServicioBueno(LoggerSilencioso())

s1.procesar("hola")  # [LOG] Procesando: hola
s2.procesar("hola")  # [ARCHIVO:app.log] Procesando: hola
s3.procesar("hola")  # (sin output)
```

Esto es el patrón **Strategy**: el comportamiento variable (logging) se encapsula en una clase separada y se inyecta en quien lo necesita. Es uno de los patrones más útiles que emerge naturalmente de preferir composición sobre herencia.

## Reglas prácticas de decisión

1. **Si B puede existir independientemente de A** → asociación o agregación (A tiene una referencia a B, pero no lo crea ni destruye).

2. **Si B no puede existir sin A** → composición (A crea B en su `__init__` y lo posee exclusivamente).

3. **Si A es genuinamente una versión especializada de B** y el test "es-un" se cumple en *todos* los contextos → herencia.

4. **Cuando tengás dudas** → composición. Siempre podés refactorizar de composición a herencia; el camino inverso es mucho más doloroso.

## Mixins: el punto intermedio

Cuando necesitás compartir comportamiento entre clases sin establecer una jerarquía "es-un", los **mixins** son una opción. Un mixin es una clase pequeña y enfocada que agrega un comportamiento específico y no está pensada para instanciarse sola.

```python
class SerializableMixin:
    """
    Mixin que agrega capacidad de serialización a cualquier clase.
    No se instancia sola — siempre se usa en combinación con otra clase.
    """

    def a_dict(self) -> dict:
        """Convierte el objeto a un diccionario con sus atributos públicos."""
        return {
            k: v for k, v in self.__dict__.items()
            if not k.startswith("_")
        }

    def __str__(self) -> str:
        datos = self.a_dict()
        return ", ".join(f"{k}={v!r}" for k, v in datos.items())


class Producto(SerializableMixin):
    """
    Producto con serialización gracias al mixin.

    Args:
        codigo: Código del producto.
        nombre: Nombre del producto.
        precio: Precio unitario.
    """

    def __init__(self, codigo: str, nombre: str, precio: float) -> None:
        self.codigo = codigo
        self.nombre = nombre
        self.precio = precio


class Cliente(SerializableMixin):
    """
    Cliente con serialización gracias al mismo mixin.

    Args:
        nombre: Nombre completo del cliente.
        email: Dirección de correo.
    """

    def __init__(self, nombre: str, email: str) -> None:
        self.nombre = nombre
        self.email = email


p = Producto("LAP001", "Laptop", 1500.0)
c = Cliente("Ana García", "ana@ejemplo.com")

print(p.a_dict())  # {'codigo': 'LAP001', 'nombre': 'Laptop', 'precio': 1500.0}
print(c.a_dict())  # {'nombre': 'Ana García', 'email': 'ana@ejemplo.com'}
print(p)           # codigo='LAP001', nombre='Laptop', precio=1500.0
```

Los mixins son herencia, pero con un propósito muy acotado: agregan un comportamiento transversal sin establecer una jerarquía de dominio. La convención en Python es nombrarlos con el sufijo `Mixin` para dejar claro que no son clases base completas.

> **En la práctica:** la regla que más me funcionó para elegir es la siguiente: si tenés que explicar por qué `B` hereda de `A` (porque no es obvio que "B es un A"), entonces no deberías usar herencia. El diseño debería comunicarse solo. Si la relación necesita justificación, es una señal de que estás forzando la herencia donde corresponde composición.

---
