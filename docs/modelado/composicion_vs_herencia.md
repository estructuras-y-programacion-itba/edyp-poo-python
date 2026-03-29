# Composición vs. Herencia

Ya conocemos las relaciones "es-un" (herencia) y "tiene-un" (composición), y sabemos que la abstracción permite definir contratos formales. Ahora viene la pregunta práctica que aparece en casi cada diseño: ¿cuándo uso herencia y cuándo uso composición? Esta sección da el criterio para responderla.

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

Una `EstacionCorte` *es una* `EstacionTrabajo` → herencia.
Una `Maquina` *tiene un* `RegistradorEventos` → composición.
Un `SistemaControl` *usa una* `EstacionTrabajo` → asociación.

## El problema de la clase base frágil

La herencia crea un acoplamiento muy fuerte entre padre e hijo. El hijo depende de los detalles internos del padre: si el padre cambia, el hijo puede romperse aunque nadie lo haya tocado. Este problema se llama **Fragile Base Class Problem**.

```python
# ── Ejemplo del problema ──────────────────────────────────────────────────────

class EstacionBase:
    """Clase base que registra piezas procesadas."""

    def __init__(self) -> None:
        self._piezas: list = []
        self._contador = 0  # cuántas piezas se procesaron en total

    def agregar_pieza(self, pieza) -> None:
        self._piezas.append(pieza)
        self._contador += 1

    def agregar_lote(self, piezas: list) -> None:
        for pieza in piezas:
            self.agregar_pieza(pieza)  # llama a agregar_pieza() para mantener el contador


class EstacionConRegistro(EstacionBase):
    """Quiere contar cuántas veces se agrega una pieza."""

    def __init__(self) -> None:
        super().__init__()
        self._veces_llamado = 0

    def agregar_pieza(self, pieza) -> None:
        self._veces_llamado += 1
        super().agregar_pieza(pieza)


est = EstacionConRegistro()
est.agregar_pieza("P-0001")
est.agregar_pieza("P-0002")
est.agregar_lote(["P-0003", "P-0004", "P-0005"])

print(est._veces_llamado)  # ¿Cuánto esperabas? ¿2? ¿5?
# Es 5 — porque agregar_lote() llama a agregar_pieza() en un loop,
# y agregar_pieza() está sobreescrito en la subclase.
# Pequeños cambios en la clase padre producen efectos inesperados en los hijos.
```

Este tipo de bug es difícil de detectar porque el código "parece correcto" pero el comportamiento depende de detalles de implementación del padre que podrían cambiar en cualquier momento.

## El mismo problema resuelto con composición

```python
class RegistroPiezas:
    """
    Gestiona una colección de piezas.
    Clase autónoma, no hereda de nada especial.
    """

    def __init__(self) -> None:
        self._piezas: list = []

    def agregar(self, pieza) -> None:
        self._piezas.append(pieza)

    def agregar_lote(self, piezas: list) -> None:
        self._piezas.extend(piezas)

    def __len__(self) -> int:
        return len(self._piezas)


class EstacionConRegistroComposicion:
    """
    Estación con auditoría. USA un RegistroPiezas, no lo extiende.

    Al no heredar, no hay riesgo de que cambios en RegistroPiezas
    rompan el comportamiento de la auditoría.
    """

    def __init__(self) -> None:
        self._registro = RegistroPiezas()  # composición
        self._veces_llamado = 0

    def agregar_pieza(self, pieza) -> None:
        """Agrega una pieza y registra la operación."""
        self._registro.agregar(pieza)
        self._veces_llamado += 1

    def agregar_lote(self, piezas: list) -> None:
        """Agrega varias piezas registrando cada una."""
        for pieza in piezas:
            self.agregar_pieza(pieza)  # pasa por la auditoría correctamente

    def total_operaciones(self) -> int:
        """Retorna el total de operaciones de agregado."""
        return self._veces_llamado

    def __len__(self) -> int:
        return len(self._registro)


est = EstacionConRegistroComposicion()
est.agregar_pieza("P-0001")
est.agregar_pieza("P-0002")
est.agregar_lote(["P-0003", "P-0004", "P-0005"])

print(est.total_operaciones())  # 5 — predecible, sin sorpresas
print(len(est))                 # 5
```

## Un caso real: RegistradorEventos inyectable por composición

Uno de los ejemplos más clásicos de composición sobre herencia es un RegistradorEventos que se puede "enchufar" a cualquier clase. Con herencia, estarías forzando a todas las clases a ser subclase del registrador. Con composición, cualquier clase puede tener uno.

```python
from abc import ABC, abstractmethod


class RegistradorEventos(ABC):
    """Abstracción de un registrador de eventos. Define el contrato."""

    @abstractmethod
    def log(self, mensaje: str) -> None:
        """Registra un mensaje."""
        ...


class RegistradorConsola(RegistradorEventos):
    """Registrador que imprime en consola."""

    def log(self, mensaje: str) -> None:
        print(f"[LOG] {mensaje}")


class RegistradorArchivo(RegistradorEventos):
    """Registrador que simula escritura en archivo."""

    def __init__(self, ruta: str) -> None:
        self._ruta = ruta

    def log(self, mensaje: str) -> None:
        # En producción, abriría el archivo y escribiría
        print(f"[ARCHIVO:{self._ruta}] {mensaje}")


class RegistradorSilencioso(RegistradorEventos):
    """Registrador que no hace nada. Útil para tests."""

    def log(self, mensaje: str) -> None:
        pass  # no hace nada — Null Object pattern


# ── Herencia: el problema ────────────────────────────────────────────────────

class MaquinaMala(RegistradorConsola):
    """❌ Hereda RegistradorConsola solo para tener logging. Relación forzada."""

    def iniciar_ciclo(self) -> str:
        self.log("Iniciando ciclo...")  # usa el método heredado
        return "Ciclo ejecutado"

# MaquinaMala ES UN RegistradorConsola → ¿tiene sentido esa relación?
# ¿Qué pasa si querés cambiar al RegistradorArchivo? Tenés que cambiar la herencia.


# ── Composición: la solución ─────────────────────────────────────────────────

class Maquina:
    """
    ✅ Recibe un RegistradorEventos por composición. Flexible y sin acoplamiento.

    Args:
        nombre: Nombre de la máquina.
        registrador: Implementación del registrador a usar.
    """

    def __init__(self, nombre: str, registrador: RegistradorEventos) -> None:
        self.nombre = nombre
        self._registrador = registrador  # composición: tiene-un RegistradorEventos

    def iniciar_ciclo(self) -> str:
        """Inicia un ciclo de producción y lo registra."""
        self._registrador.log(f"[{self.nombre}] Iniciando ciclo de producción")
        return "Ciclo ejecutado"


# La misma clase, tres comportamientos distintos, sin cambiar Maquina
m1 = Maquina("CNC-01", RegistradorConsola())
m2 = Maquina("MIG-01", RegistradorArchivo("planta.log"))
m3 = Maquina("PAINT-01", RegistradorSilencioso())

m1.iniciar_ciclo()  # [LOG] [CNC-01] Iniciando ciclo de producción
m2.iniciar_ciclo()  # [ARCHIVO:planta.log] [MIG-01] Iniciando ciclo de producción
m3.iniciar_ciclo()  # (sin output)
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


class EstacionTrabajo(SerializableMixin):
    """
    Estación de trabajo con serialización gracias al mixin.

    Args:
        nombre: Nombre de la estación.
        tiempo_ciclo: Tiempo de ciclo en segundos.
    """

    def __init__(self, nombre: str, tiempo_ciclo: float) -> None:
        self.nombre = nombre
        self.tiempo_ciclo = tiempo_ciclo


class Pieza(SerializableMixin):
    """
    Pieza con serialización gracias al mismo mixin.

    Args:
        numero_serie: Número de serie de la pieza.
        material: Material de la pieza.
    """

    def __init__(self, numero_serie: str, material: str) -> None:
        self.numero_serie = numero_serie
        self.material = material


est = EstacionTrabajo("CNC-01", 15.0)
pieza = Pieza("P-0001", "acero")

print(est.a_dict())    # {'nombre': 'CNC-01', 'tiempo_ciclo': 15.0}
print(pieza.a_dict())  # {'numero_serie': 'P-0001', 'material': 'acero'}
print(est)             # nombre='CNC-01', tiempo_ciclo=15.0
```

Los mixins son herencia, pero con un propósito muy acotado: agregan un comportamiento transversal sin establecer una jerarquía de dominio. La convención en Python es nombrarlos con el sufijo `Mixin` para dejar claro que no son clases base completas.

> **En la práctica:** la regla que más me funcionó para elegir es la siguiente: si tenés que explicar por qué `B` hereda de `A` (porque no es obvio que "B es un A"), entonces no deberías usar herencia. El diseño debería comunicarse solo. Si la relación necesita justificación, es una señal de que estás forzando la herencia donde corresponde composición.

## Ver también

Con las herramientas de diseño cubiertas, el paso que cierra el capítulo de modelado es aprender el proceso: cómo pasar de un enunciado a un conjunto de clases bien diseñadas:

- [Análisis y Diseño OO](analisis_y_diseno.md) — del problema en lenguaje natural a clases con responsabilidades claras
- [Principios SOLID](../avanzado/solid.md) — especialmente SRP y OCP, que refuerzan las reglas de composición vistas acá

---
