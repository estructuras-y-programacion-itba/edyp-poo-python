# Enumeraciones (`Enum`)

En varios ejemplos de este material vas a encontrar código como `EstadoPieza.COMPLETADA` o `EstadoPieza.EN_PROCESO`. Esos valores vienen de una **enumeración**, una clase especial de Python que agrupa un conjunto fijo de constantes con nombre.

## ¿Para qué sirve un `Enum`?

Considerá este problema: el estado de una pieza puede ser `"en_espera"`, `"en_proceso"`, `"completada"` o `"rechazada"`. La forma más directa de representarlo es con strings:

```python
# ❌ Sin Enum: strings sueltos
class Pieza:
    def __init__(self, numero_serie: str) -> None:
        self.numero_serie = numero_serie
        self.estado = "en_espera"   # string literal

pieza = Pieza("P-0001")
pieza.estado = "completdaa"         # ❌ typo — Python no se queja
pieza.estado = "finalizada"         # ❌ valor inválido — Python tampoco se queja
```

El problema es que los strings literales no tienen ninguna restricción: cualquier valor pasa sin error. Un `Enum` soluciona eso definiendo de antemano los únicos valores válidos:

```python
from enum import Enum

# ✅ Con Enum: valores finitos y con nombre
class EstadoPieza(Enum):
    EN_ESPERA  = "en_espera"
    EN_PROCESO = "en_proceso"
    COMPLETADA = "completada"
    RECHAZADA  = "rechazada"

pieza_estado = EstadoPieza.EN_ESPERA
print(pieza_estado)         # EstadoPieza.EN_ESPERA
print(pieza_estado.value)   # 'en_espera'
print(pieza_estado.name)    # 'EN_ESPERA'
```

## Cómo se define un `Enum`

Un `Enum` es una clase que hereda de `Enum` del módulo estándar `enum`. Cada atributo de clase es un **miembro** de la enumeración:

```python
from enum import Enum


class EstadoMaquina(Enum):
    INACTIVA = "inactiva"
    ACTIVA   = "activa"
    FALLA    = "falla"
```

Cada miembro tiene dos atributos clave:

| Atributo | Qué retorna | Ejemplo |
| --- | --- | --- |
| `.name` | El nombre del miembro (string) | `"ACTIVA"` |
| `.value` | El valor asignado | `"activa"` |

## Comparación e identidad

Los miembros de un `Enum` se comparan con `==` o con `is`. Dos referencias al mismo miembro son idénticas:

```python
estado = EstadoMaquina.ACTIVA

print(estado == EstadoMaquina.ACTIVA)   # True
print(estado is EstadoMaquina.ACTIVA)   # True — mismo objeto en memoria
print(estado == EstadoMaquina.FALLA)    # False
```

## Iterar y listar miembros

Podés recorrer todos los miembros con un `for`:

```python
for estado in EstadoMaquina:
    print(f"{estado.name}: {estado.value}")
# INACTIVA: inactiva
# ACTIVA: activa
# FALLA: falla
```

## Uso en `if` y `match`

Los `Enum` funcionan naturalmente en estructuras de control:

```python
# Con if/elif
def describir_estado(estado: EstadoMaquina) -> str:
    if estado == EstadoMaquina.ACTIVA:
        return "La máquina está produciendo"
    elif estado == EstadoMaquina.FALLA:
        return "La máquina requiere mantenimiento"
    else:
        return "La máquina está detenida"

# Con match (Python 3.10+)
def describir_estado_match(estado: EstadoMaquina) -> str:
    match estado:
        case EstadoMaquina.ACTIVA:
            return "La máquina está produciendo"
        case EstadoMaquina.FALLA:
            return "La máquina requiere mantenimiento"
        case _:
            return "La máquina está detenida"
```

## Ejemplo integrado: `Pieza` con `EstadoPieza`

```python
from enum import Enum


class EstadoPieza(Enum):
    EN_ESPERA  = "en_espera"
    EN_PROCESO = "en_proceso"
    COMPLETADA = "completada"
    RECHAZADA  = "rechazada"


class Pieza:
    """
    Pieza industrial con ciclo de vida controlado por un Enum.

    Args:
        numero_serie: Identificador único de la pieza.
        material: Tipo de material.
    """

    def __init__(self, numero_serie: str, material: str) -> None:
        self.numero_serie = numero_serie
        self.material = material
        self._estado = EstadoPieza.EN_ESPERA

    @property
    def estado(self) -> EstadoPieza:
        return self._estado

    def marcar_en_proceso(self) -> None:
        """Inicia el procesamiento de la pieza."""
        if self._estado != EstadoPieza.EN_ESPERA:
            raise ValueError(
                f"No se puede iniciar: la pieza está en estado '{self._estado.value}'"
            )
        self._estado = EstadoPieza.EN_PROCESO

    def marcar_completada(self) -> None:
        """Finaliza el procesamiento con resultado exitoso."""
        self._estado = EstadoPieza.COMPLETADA

    def marcar_rechazada(self) -> None:
        """Finaliza el procesamiento con resultado de rechazo."""
        self._estado = EstadoPieza.RECHAZADA

    def esta_lista(self) -> bool:
        """Indica si la pieza completó el proceso correctamente."""
        return self._estado == EstadoPieza.COMPLETADA

    def __repr__(self) -> str:
        return f"Pieza({self.numero_serie!r}, estado={self._estado.value!r})"


# Demostración
pieza = Pieza("P-0001", "acero")
print(pieza)              # Pieza('P-0001', estado='en_espera')

pieza.marcar_en_proceso()
print(pieza.estado)       # EstadoPieza.EN_PROCESO
print(pieza.estado.value) # 'en_proceso'

pieza.marcar_completada()
print(pieza.esta_lista()) # True
print(pieza)              # Pieza('P-0001', estado='completada')

# Intentar retroceder el estado
try:
    pieza.marcar_en_proceso()   # ❌ ya no está en EN_ESPERA
except ValueError as e:
    print(e)
# No se puede iniciar: la pieza está en estado 'completada'
```

## `Enum` vs. constantes sueltas

Una alternativa común al `Enum` son las constantes como variables de módulo:

```python
# Sin Enum: constantes sueltas
ESTADO_ACTIVA   = "activa"
ESTADO_INACTIVA = "inactiva"
ESTADO_FALLA    = "falla"
```

El `Enum` tiene ventajas claras en comparación:

| Aspecto | Constantes sueltas | `Enum` |
| --- | --- | --- |
| Agrupación | Dispersas, sin relación explícita | Agrupadas en una clase con nombre |
| Type hints | Solo `str` | `EstadoMaquina` — el tipo dice exactamente qué valores son válidos |
| Autocompletado | Dependés de conocer el nombre exacto | El IDE muestra los miembros disponibles |
| Iteración | No posible directamente | `for estado in EstadoMaquina` |
| Comparación accidental con strings | `estado == "activa"` pasa sin error | `estado == EstadoMaquina.ACTIVA` es explícito |

> **En la práctica:** usá `Enum` siempre que tengas un conjunto cerrado y conocido de valores posibles para un atributo. Los casos más comunes en diseño OO son estados de un ciclo de vida (`EN_ESPERA`, `EN_PROCESO`, `COMPLETADA`), categorías fijas, y modos de operación. Si los valores pueden cambiar o agregarse en runtime, un `Enum` no es la herramienta correcta.

---
