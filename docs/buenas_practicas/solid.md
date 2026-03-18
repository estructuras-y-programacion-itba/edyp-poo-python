# Principios SOLID

## S — Single Responsibility Principle (Principio de Responsabilidad Única)

Una clase debe tener una, y solo una, razón para cambiar.

Tomá el caso de una clase `Maquina` que gestiona su estado operativo, guarda eventos en base de datos y además genera reportes de turno: está haciendo demasiado. Separar responsabilidades conduce a un diseño más mantenible: `Maquina` (estado y ciclos), `RegistradorEventos` (persistencia), `GeneradorReportes` (presentación).

```python
# ❌ Viola SRP: la clase hace demasiado
class MaquinaConTodo:
    def iniciar_ciclo(self): ...
    def guardar_en_db(self): ...       # responsabilidad de persistencia
    def imprimir_reporte(self): ...    # responsabilidad de presentación

# ✅ Respeta SRP: cada clase tiene una razón para cambiar
class Maquina:
    def iniciar_ciclo(self) -> None: ...
    def obtener_estado(self) -> dict: ...

class RegistradorEventos:
    def guardar(self, maquina: Maquina) -> None: ...

class GeneradorReportes:
    def imprimir(self, maquina: Maquina) -> None: ...
```

## O — Open/Closed Principle (Principio de Abierto/Cerrado)

Las clases deben estar abiertas para su extensión, pero cerradas para su modificación.

Agregar nueva funcionalidad no debería requerir modificar código que ya funciona —y arriesgarse a romperlo. La solución es el polimorfismo: extendé el comportamiento agregando nuevas clases en lugar de acumular `if/else`.

```python
from abc import ABC, abstractmethod

# ❌ Viola OCP: hay que modificar la clase para agregar una nueva estación
class ProcesadorPieza:
    def procesar(self, pieza: str, tipo_estacion: str) -> str:
        if tipo_estacion == "corte":
            return f"Cortando {pieza}"
        elif tipo_estacion == "soldadura":
            return f"Soldando {pieza}"
        # Cada nueva estación requiere modificar esta clase


# ✅ Respeta OCP: se extiende sin modificar
class ProcesadorBase(ABC):
    @abstractmethod
    def procesar(self, pieza: str) -> str: ...

class ProcesadorCorte(ProcesadorBase):
    def procesar(self, pieza: str) -> str:
        return f"Cortando {pieza}"

class ProcesadorSoldadura(ProcesadorBase):
    def procesar(self, pieza: str) -> str:
        return f"Soldando {pieza}"

class ProcesadorPintura(ProcesadorBase):
    def procesar(self, pieza: str) -> str:
        return f"Pintando {pieza}"
```

## L — Liskov Substitution Principle (Principio de Sustitución de Liskov)

Las clases derivadas deben poder sustituirse por sus clases base sin alterar el comportamiento correcto del programa.

Cuando una clase hija viola el contrato establecido por la clase base —por ejemplo, `EstacionMantenimiento` lanzando una excepción en `procesar()` cuando la base promete que ese método es válido—, el principio queda roto. La herencia debe tener sentido tanto semántico como funcional.

```python
from abc import ABC, abstractmethod


class EstacionTrabajo(ABC):
    """Contrato base: toda estación puede procesar una pieza."""

    @abstractmethod
    def procesar(self, pieza: str) -> str: ...


# ❌ Viola LSP: EstacionDesactivada no respeta el contrato de la base
class EstacionDesactivadaMal(EstacionTrabajo):
    def procesar(self, pieza: str) -> str:
        raise NotImplementedError("¡Esta estación está fuera de servicio!")
        # ❌ Quien use EstacionTrabajo espera poder llamar procesar() sin excepciones


# ✅ Respeta LSP: la jerarquía refleja capacidades reales
class EstacionCorte(EstacionTrabajo):
    """Estación operativa que puede procesar piezas."""

    def procesar(self, pieza: str) -> str:
        return f"[CORTE] Procesando {pieza}"


class EstacionSoldadura(EstacionTrabajo):
    """Estación operativa que puede procesar piezas."""

    def procesar(self, pieza: str) -> str:
        return f"[SOLDADURA] Procesando {pieza}"


# Ahora podemos substituir EstacionTrabajo por cualquier subclase sin sorpresas
estaciones: list[EstacionTrabajo] = [EstacionCorte(), EstacionSoldadura()]
for est in estaciones:
    print(est.procesar("P-0001"))  # ✅ funciona para todas
```

## I — Interface Segregation Principle (Principio de Segregación de Interfaz)

Ninguna clase debería verse forzada a depender de métodos que no utiliza.

Aunque Python no tiene "interfaces" explícitas, este principio cobra especial relevancia al diseñar Clases Base Abstractas (ABCs) o Protocolos. Crear una clase base gigante que obligue a sus subclases a implementar métodos sin sentido para ellas es exactamente lo que se busca evitar: mejor definir contratos pequeños y específicos.

```python
from abc import ABC, abstractmethod

# ❌ Viola ISP: una ABC enorme que fuerza a EstacionSimple a implementar
#    métodos que no tiene sentido para ella
class IEstacionCompleta(ABC):
    @abstractmethod
    def procesar(self, pieza: str) -> str: ...

    @abstractmethod
    def calibrar(self) -> None: ...

    @abstractmethod
    def enviar_telemetria(self) -> dict: ...

    @abstractmethod
    def actualizar_firmware(self, version: str) -> None: ...


# ✅ Respeta ISP: interfaces pequeñas y específicas
class IMonitoreable(ABC):
    @abstractmethod
    def enviar_telemetria(self) -> dict: ...


class IMantenible(ABC):
    @abstractmethod
    def calibrar(self) -> None: ...


class IConfigurable(ABC):
    @abstractmethod
    def actualizar_firmware(self, version: str) -> None: ...


class EstacionCorte(IMonitoreable, IMantenible):
    """Implementa solo lo que necesita. ✅"""

    def procesar(self, pieza: str) -> str:
        return f"Cortando {pieza}"

    def calibrar(self) -> None:
        print("Calibrando velocidad de corte")

    def enviar_telemetria(self) -> dict:
        return {"velocidad": 200, "temperatura": 45}


class SensorBasico(IMonitoreable):
    """Solo monitorea — no necesita calibración ni firmware. ✅"""

    def enviar_telemetria(self) -> dict:
        return {"lectura": 23.5}
```

## D — Dependency Inversion Principle (Principio de Inversión de Dependencias)

Los módulos de alto nivel no deben depender de módulos de bajo nivel. Ambos deben depender de abstracciones.

Cuando una clase de alto nivel instancia directamente sus dependencias concretas, queda acoplada a una implementación específica —difícil de testear, difícil de cambiar. La inversión de dependencias rompe ese acoplamiento: en lugar de crear la dependencia, la clase la recibe desde afuera (inyección de dependencia), dependiendo de una abstracción que puede tomar cualquier forma concreta.

```python
from abc import ABC, abstractmethod

# ❌ Viola DIP: SistemaControl depende de EstacionCorte concreta
class SistemaControlMal:
    def __init__(self) -> None:
        self.estacion = EstacionCorte()  # dependencia de bajo nivel hardcodeada

    def ejecutar_ciclo(self, pieza: str) -> str:
        return self.estacion.procesar(pieza)


# ✅ Respeta DIP: SistemaControl depende de abstracción
class EstacionTrabajo(ABC):
    @abstractmethod
    def procesar(self, pieza: str) -> str: ...

class SistemaControl:
    def __init__(self, estacion: EstacionTrabajo) -> None:
        self._estacion = estacion  # inyección de dependencia

    def ejecutar_ciclo(self, pieza: str) -> str:
        return self._estacion.procesar(pieza)


# El mismo SistemaControl funciona con cualquier estación
class EstacionCorte(EstacionTrabajo):
    def procesar(self, pieza: str) -> str:
        return f"[CORTE] {pieza}"

class EstacionSoldadura(EstacionTrabajo):
    def procesar(self, pieza: str) -> str:
        return f"[SOLDADURA] {pieza}"

control_corte = SistemaControl(EstacionCorte())
control_soldadura = SistemaControl(EstacionSoldadura())

print(control_corte.ejecutar_ciclo("P-0001"))     # [CORTE] P-0001
print(control_soldadura.ejecutar_ciclo("P-0001")) # [SOLDADURA] P-0001
```

---
