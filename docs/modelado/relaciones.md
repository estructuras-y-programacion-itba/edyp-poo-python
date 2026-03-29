# Relaciones entre Clases

!!! info "Antes de continuar"
    Esta sección asume que sabés definir clases con atributos y métodos. Si necesitás repasar, empezá por [Clases y Objetos](../fundamentos/clases_y_objetos.md).

Hasta acá diseñamos clases de forma individual. En cualquier sistema real, las clases no existen aisladas: colaboran, se agrupan y se especializan. Esta sección introduce la taxonomía de relaciones — el vocabulario que te permite leer y diseñar diagramas de clases con precisión.

Un sistema real no se compone de clases aisladas: las clases se relacionan entre sí para colaborar. Entender qué tipo de relación existe entre dos clases determina cómo las vas a implementar en código y cuán flexibles o rígidas van a ser esas decisiones a futuro.

Existen cuatro tipos de relaciones fundamentales en el diseño orientado a objetos. Las tres primeras (asociación, agregación, composición) responden a la pregunta "¿tiene-un / usa-un?"; la cuarta (herencia) responde a "¿es-un?".

## Asociación

La **asociación** es la relación más general: un objeto *conoce* o *usa* a otro, pero sin que ninguno sea dueño del otro. Los objetos asociados pueden existir de forma completamente independiente.

**Test:** ¿Puede cada objeto existir sin el otro? Si la respuesta es sí para ambos, es asociación.

```python
class Operario:
    """
    Operario de planta que puede ser asignado a distintas estaciones.

    Args:
        nombre: Nombre completo del operario.
        legajo: Número de legajo.
    """

    def __init__(self, nombre: str, legajo: str) -> None:
        self.nombre = nombre
        self.legajo = legajo

    def __repr__(self) -> str:
        return f"Operario({self.nombre!r}, legajo={self.legajo!r})"


class EstacionTrabajo:
    """
    Estación de trabajo con operario asignado.

    Args:
        nombre: Nombre de la estación.
    """

    def __init__(self, nombre: str) -> None:
        self.nombre = nombre
        self._operario_asignado: Operario | None = None

    def asignar_operario(self, operario: Operario) -> None:
        """Asigna un operario a esta estación."""
        self._operario_asignado = operario

    def liberar_operario(self) -> None:
        """Libera al operario de esta estación."""
        self._operario_asignado = None

    def operario_actual(self) -> str:
        if self._operario_asignado:
            return self._operario_asignado.nombre
        return "Sin asignar"


# El Operario puede existir sin la Estación y viceversa → asociación
juan = Operario("Juan Pérez", "OP-001")
maria = Operario("María López", "OP-002")

corte = EstacionTrabajo("Corte CNC")
corte.asignar_operario(juan)
print(corte.operario_actual())  # Juan Pérez

# El operario puede reasignarse a otra estación
corte.liberar_operario()
soldadura = EstacionTrabajo("Soldadura MIG")
soldadura.asignar_operario(juan)
print(soldadura.operario_actual())  # Juan Pérez
# Si la estación se cierra, Juan sigue existiendo
```

## Agregación

La **agregación** es una asociación con matiz: un objeto *contiene* una colección de otros, pero los objetos contenidos pueden existir de forma independiente. El contenedor "agrupa" elementos que tienen vida propia.

**Test:** Si destruís el contenedor, ¿los elementos siguen existiendo? Si sí, es agregación.

```python
class EstacionTrabajo:
    """
    Estación de trabajo de la planta.

    Args:
        nombre: Nombre de la estación.
        tiempo_ciclo: Tiempo de ciclo en segundos.
    """

    def __init__(self, nombre: str, tiempo_ciclo: float) -> None:
        self.nombre = nombre
        self.tiempo_ciclo = tiempo_ciclo

    def __repr__(self) -> str:
        return f"EstacionTrabajo({self.nombre!r})"


class LineaDeMontaje:
    """
    Línea de montaje. Agrega estaciones que existen independientemente.

    Args:
        nombre: Nombre de la línea.
    """

    def __init__(self, nombre: str) -> None:
        self.nombre = nombre
        self._estaciones: list[EstacionTrabajo] = []

    def agregar_estacion(self, estacion: EstacionTrabajo) -> None:
        """Agrega una estación a la línea."""
        self._estaciones.append(estacion)

    def total_estaciones(self) -> int:
        """Retorna la cantidad de estaciones en la línea."""
        return len(self._estaciones)


corte = EstacionTrabajo("Corte CNC", tiempo_ciclo=12.0)
soldadura = EstacionTrabajo("Soldadura MIG", tiempo_ciclo=30.0)

linea_a = LineaDeMontaje("Línea A")
linea_a.agregar_estacion(corte)
linea_a.agregar_estacion(soldadura)

# Si la Línea A se cierra, las estaciones pueden pasar a otra línea
# Una estación no es propiedad exclusiva de una línea → no es composición
```

## Composición

La **composición** es la relación más fuerte de propiedad: el objeto *compuesto* crea y posee a sus partes, y esas partes no tienen sentido ni existencia fuera del todo. Si el compuesto se destruye, sus partes se destruyen con él.

**Test:** ¿Tiene sentido la parte sin el todo? Si no, es composición.

```python
class Sensor:
    """
    Sensor de temperatura interno de una máquina.
    No tiene sentido fuera de la máquina que lo contiene.

    Args:
        id_sensor: Identificador del sensor.
        rango_max: Temperatura máxima operativa en °C.
    """

    def __init__(self, id_sensor: str, rango_max: float) -> None:
        self._id = id_sensor
        self._rango_max = rango_max
        self._lectura_actual = 0.0

    def leer(self) -> float:
        """Retorna la lectura actual del sensor."""
        return self._lectura_actual

    def actualizar(self, temperatura: float) -> None:
        """Actualiza la lectura del sensor."""
        self._lectura_actual = temperatura

    def esta_en_rango(self) -> bool:
        """Indica si la temperatura está dentro del rango operativo."""
        return self._lectura_actual <= self._rango_max


class Maquina:
    """
    Máquina industrial con sensor de temperatura integrado.
    El sensor se crea y vive dentro de la máquina — composición.

    Args:
        nombre: Nombre de la máquina.
        temp_max: Temperatura máxima operativa en °C.
    """

    def __init__(self, nombre: str, temp_max: float = 80.0) -> None:
        self.nombre = nombre
        # El Sensor se crea DENTRO de la Maquina → composición
        self.__sensor = Sensor(f"SEN-{nombre}", rango_max=temp_max)

    @property
    def temperatura(self) -> float:
        """Temperatura actual según el sensor interno."""
        return self.__sensor.leer()

    def actualizar_temperatura(self, temp: float) -> None:
        """Actualiza la lectura del sensor interno."""
        self.__sensor.actualizar(temp)

    def esta_operativa(self) -> bool:
        """Indica si la temperatura está dentro del rango seguro."""
        return self.__sensor.esta_en_rango()


maquina = Maquina("Torno-01", temp_max=75.0)
maquina.actualizar_temperatura(60.0)
print(maquina.temperatura)     # 60.0
print(maquina.esta_operativa()) # True

# El Sensor no existe por sí solo: vive y muere con la Maquina
```

## Herencia

La **herencia** establece una relación "es-un" entre clases: la subclase es una versión especializada de la clase padre. Hereda todos sus atributos y métodos, y puede agregar los propios o redefinir los heredados.

**Test:** ¿Podés decir con naturalidad que "B es un A"? Solo entonces usá herencia.

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

    def describir(self) -> str:
        """Descripción genérica de la estación."""
        return f"{self.nombre} (ciclo: {self.tiempo_ciclo}s)"


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

    def describir(self) -> str:
        return f"Corte {self.nombre} a {self.velocidad_corte} mm/min (ciclo: {self.tiempo_ciclo}s)"


est = EstacionCorte("CNC-01", tiempo_ciclo=15.0, velocidad_corte=200.0)
print(est.describir())            # Corte CNC-01 a 200.0 mm/min (ciclo: 15.0s)
print(isinstance(est, EstacionTrabajo))  # True — una EstacionCorte ES UNA EstacionTrabajo
```

## Multiplicidad

Más allá del tipo de relación, importa cuántos objetos participan de cada lado:

| Multiplicidad | Implementación en Python | Ejemplo |
| --- | --- | --- |
| Uno a uno (1:1) | Atributo simple | Una `Maquina` tiene un `Sensor` |
| Uno a muchos (1:N) | `list[Tipo]` | Una `EstacionCorte` tiene N `Pieza` en cola |
| Muchos a muchos (N:M) | `list[Tipo]` en ambos lados | `Operario` ↔ `EstacionTrabajo` |

```python
class Pieza:
    """
    Pieza en proceso.

    Args:
        numero_serie: Número de serie de la pieza.
        material: Material de la pieza.
    """

    def __init__(self, numero_serie: str, material: str) -> None:
        self.numero_serie = numero_serie
        self.material = material

    def __repr__(self) -> str:
        return f"Pieza({self.numero_serie!r})"


class EstacionCorte:
    """
    Estación de corte con cola de piezas pendientes.

    Args:
        nombre: Nombre de la estación.
    """

    def __init__(self, nombre: str) -> None:
        self.nombre = nombre
        self._cola: list[Pieza] = []   # 1:N

    def encolar_pieza(self, pieza: Pieza) -> None:
        """Agrega una pieza a la cola de procesamiento."""
        self._cola.append(pieza)

    def piezas_en_cola(self) -> int:
        """Retorna la cantidad de piezas pendientes."""
        return len(self._cola)


estacion = EstacionCorte("CNC-02")
estacion.encolar_pieza(Pieza("P-0001", "acero"))
estacion.encolar_pieza(Pieza("P-0002", "aluminio"))
print(f"Piezas en cola: {estacion.piezas_en_cola()}")  # Piezas en cola: 2
```

## Regla práctica

En la mayoría de los sistemas reales, las relaciones más frecuentes son:

- **Composición**: cuando un objeto crea y posee a sus partes (fuerte ownership).
- **Asociación**: cuando un objeto recibe una referencia a otro y lo usa sin poseerlo.

La herencia es menos frecuente de lo que parece al principio. Antes de usarla, aplicá el test "es-un". Si el test falla o generás dudas, la composición suele ser la alternativa más flexible y mantenible.

> **En la práctica:** uno de los errores más costosos en proyectos reales es modelar como herencia lo que en realidad es composición. Heredar por conveniencia (para reutilizar un método) en lugar de por relación "es-un" produce jerarquías frágiles que se rompen cuando los requerimientos cambian. El tema se desarrolla en profundidad en [Composición vs. Herencia](composicion_vs_herencia.md).

## Ver también

Con el mapa de relaciones claro, los siguientes temas profundizan en las relaciones más importantes y sus implicancias de diseño:

- [Herencia](herencia.md) — la relación "es-un" en detalle: sintaxis, `super()` y sobreescritura de métodos
- [Composición vs. Herencia](composicion_vs_herencia.md) — cómo decidir entre las dos relaciones más frecuentes

---
