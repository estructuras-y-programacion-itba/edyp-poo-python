# ¿Qué es la Programación Orientada a Objetos?

La Programación Orientada a Objetos (POO) es un paradigma de diseño y desarrollo de software que organiza el código en torno a **objetos**: entidades que combinan estado (datos) y comportamiento (operaciones) en una sola unidad cohesiva.

Grady Booch, uno de sus principales referentes, la define así:

> *"La programación orientada a objetos es un método de implementación en el que los programas se organizan como colecciones cooperativas de objetos, cada uno de los cuales representa una instancia de alguna clase, y cuyas clases son todas miembros de una jerarquía de clases unidas mediante relaciones de herencia."*
>
> — Grady Booch, *Object-Oriented Analysis and Design with Applications*

## ¿Por qué nació la POO?

A medida que los sistemas de software crecieron en tamaño y complejidad durante las décadas de 1960 y 1970, el paradigma estructurado (procedural) empezó a mostrar sus límites. Programas de decenas de miles de líneas con funciones que modificaban variables globales se volvían imposibles de mantener: cambiar algo en un lugar rompía algo inesperado en otro. Este fenómeno se conoció como la **crisis del software**.

La POO surgió como respuesta a esa crisis. La idea central era simple pero poderosa: en lugar de pensar el programa como una secuencia de instrucciones que operan sobre datos separados, pensarlo como un conjunto de **entidades autónomas** que se envían mensajes entre sí. Cada entidad es responsable de sus propios datos y de las operaciones que los afectan.

## El concepto central: objeto como unidad de estado y comportamiento

Un objeto tiene dos dimensiones inseparables:

- **Estado**: los datos que el objeto conoce sobre sí mismo (sus atributos o campos).
- **Comportamiento**: las acciones que el objeto puede realizar o recibir (sus métodos).

Pensá en una máquina de la planta de producción. Tiene un estado (si está activa o en falla, cuántos ciclos completó, su historial de fallas) y un comportamiento (iniciar un ciclo, reportar una falla, consultar su estado). En un diseño procedural, esos datos serían variables dispersas y las funciones vivirían separadas. En POO, todo eso convive dentro de un único objeto `Maquina`.

```python
class Maquina:
    """
    Máquina de producción con estado y comportamiento encapsulados.

    Attributes:
        nombre: Nombre o código identificador de la máquina.
    """

    def __init__(self, nombre: str) -> None:
        self.nombre = nombre
        self._estado = "inactiva"
        self._ciclos_producidos = 0
        self._historial_fallas: list[str] = []

    def iniciar_ciclo(self) -> None:
        """Ejecuta un ciclo de producción si la máquina está activa."""
        if self._estado != "activa":
            raise RuntimeError(f"La máquina '{self.nombre}' no está activa")
        self._ciclos_producidos += 1

    def activar(self) -> None:
        """Pone la máquina en estado activo."""
        self._estado = "activa"

    def reportar_falla(self, descripcion: str) -> None:
        """Registra una falla y cambia el estado a 'falla'."""
        self._estado = "falla"
        self._historial_fallas.append(descripcion)

    def obtener_estado(self) -> dict:
        """Retorna un resumen del estado actual de la máquina."""
        return {
            "nombre": self.nombre,
            "estado": self._estado,
            "ciclos": self._ciclos_producidos,
            "fallas": len(self._historial_fallas),
        }

    def __repr__(self) -> str:
        return f"Maquina('{self.nombre}', estado='{self._estado}')"


# Estado y comportamiento juntos en un objeto
maquina = Maquina("CNC-01")
maquina.activar()
maquina.iniciar_ciclo()
maquina.iniciar_ciclo()
print(maquina.obtener_estado())
# {'nombre': 'CNC-01', 'estado': 'activa', 'ciclos': 2, 'fallas': 0}
print(maquina)
# Maquina('CNC-01', estado='activa')
```

## Las cuatro características fundamentales

La POO se apoya en cuatro pilares que se desarrollan en detalle a lo largo de este material. Acá va una vista rápida:

| Característica | Pregunta que responde | En una línea |
| --- | --- | --- |
| **Abstracción** | ¿Qué hace este objeto? | Exponer solo lo esencial, ocultar la complejidad interna |
| **Encapsulamiento** | ¿Cómo protejo el estado interno? | El objeto controla el acceso a sus propios datos |
| **Modularidad** | ¿Cómo organizo el código? | Dividir el sistema en unidades cohesivas e independientes |
| **Jerarquía** | ¿Cómo relaciono abstracciones? | Herencia ("es-un") y composición ("tiene-un") |

Estos cuatro principios no son independientes: se refuerzan mutuamente. Un buen diseño orientado a objetos los aplica todos al mismo tiempo.

## Paradigma estructurado vs. POO

| Aspecto | Paradigma estructurado | Programación Orientada a Objetos |
| --- | --- | --- |
| Unidad fundamental | Función / procedimiento | Objeto (datos + comportamiento) |
| Organización del código | Funciones que operan sobre datos separados | Objetos que encapsulan sus propios datos |
| Reutilización | Llamado a funciones | Herencia y composición de clases |
| Extensibilidad | Agregar más funciones | Extender o componer clases existentes |
| Gestión del estado | Variables (a menudo globales) | Atributos privados por instancia |
| Pregunta central de diseño | ¿Qué pasos necesito seguir? | ¿Qué entidades existen y cómo se relacionan? |

> **En la práctica:** el cambio más difícil no es aprender la sintaxis de las clases — es cambiar la manera de *pensar* el problema. La mayoría de los estudiantes que llegan de programación procedural tienden a escribir funciones dentro de clases sin aprovechar los objetos. Ese cambio de mentalidad es el tema de la próxima sección.

---
