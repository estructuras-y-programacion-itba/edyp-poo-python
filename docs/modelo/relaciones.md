# Relaciones entre Clases

Un sistema real no se compone de clases aisladas: las clases se relacionan entre sí para colaborar. Entender qué tipo de relación existe entre dos clases determina cómo las vas a implementar en código y cuán flexibles o rígidas van a ser esas decisiones a futuro.

Existen cuatro tipos de relaciones fundamentales en el diseño orientado a objetos. Las tres primeras (asociación, agregación, composición) responden a la pregunta "¿tiene-un / usa-un?"; la cuarta (herencia) responde a "¿es-un?".

## Asociación

La **asociación** es la relación más general: un objeto *conoce* o *usa* a otro, pero sin que ninguno sea dueño del otro. Los objetos asociados pueden existir de forma completamente independiente.

**Test:** ¿Puede cada objeto existir sin el otro? Si la respuesta es sí para ambos, es asociación.

```python
class Estudiante:
    """
    Estudiante universitario.

    Args:
        nombre: Nombre completo del estudiante.
        legajo: Número de legajo.
    """

    def __init__(self, nombre: str, legajo: str) -> None:
        self.nombre = nombre
        self.legajo = legajo

    def __repr__(self) -> str:
        return f"Estudiante({self.nombre!r}, legajo={self.legajo!r})"


class Curso:
    """
    Curso universitario con lista de estudiantes inscriptos.

    Args:
        nombre: Nombre del curso.
    """

    def __init__(self, nombre: str) -> None:
        self.nombre = nombre
        self._inscriptos: list[Estudiante] = []

    def inscribir(self, estudiante: Estudiante) -> None:
        """Inscribe un estudiante en el curso."""
        self._inscriptos.append(estudiante)

    def listar(self) -> list[str]:
        """Retorna los nombres de los inscriptos."""
        return [e.nombre for e in self._inscriptos]


# El Estudiante puede existir sin el Curso y viceversa → asociación
ana = Estudiante("Ana García", "12345")
carlos = Estudiante("Carlos López", "67890")

poo = Curso("Programación Orientada a Objetos")
poo.inscribir(ana)
poo.inscribir(carlos)

print(poo.listar())  # ['Ana García', 'Carlos López']
# Si el curso desaparece, Ana y Carlos siguen existiendo
```

## Agregación

La **agregación** es una asociación con matiz: un objeto *contiene* una colección de otros, pero los objetos contenidos pueden existir de forma independiente. El contenedor "agrupa" elementos que tienen vida propia.

**Test:** Si destruís el contenedor, ¿los elementos siguen existiendo? Si sí, es agregación.

```python
class Jugador:
    """
    Jugador de un equipo deportivo.

    Args:
        nombre: Nombre del jugador.
        numero: Número de camiseta.
    """

    def __init__(self, nombre: str, numero: int) -> None:
        self.nombre = nombre
        self.numero = numero

    def __repr__(self) -> str:
        return f"Jugador({self.nombre!r}, #{self.numero})"


class Equipo:
    """
    Equipo deportivo. Agrega jugadores que existen independientemente.

    Args:
        nombre: Nombre del equipo.
    """

    def __init__(self, nombre: str) -> None:
        self.nombre = nombre
        self._jugadores: list[Jugador] = []

    def agregar_jugador(self, jugador: Jugador) -> None:
        """Agrega un jugador al equipo."""
        self._jugadores.append(jugador)

    def total_jugadores(self) -> int:
        """Retorna la cantidad de jugadores en el equipo."""
        return len(self._jugadores)


messi = Jugador("Lionel Messi", 10)
di_maria = Jugador("Ángel Di María", 11)

argentina = Equipo("Argentina")
argentina.agregar_jugador(messi)
argentina.agregar_jugador(di_maria)

# Si el equipo Argentina se disuelve, los jugadores siguen existiendo
# Un jugador puede pasar a otro equipo → no es propiedad exclusiva
```

## Composición

La **composición** es la relación más fuerte de propiedad: el objeto *compuesto* crea y posee a sus partes, y esas partes no tienen sentido ni existencia fuera del todo. Si el compuesto se destruye, sus partes se destruyen con él.

**Test:** ¿Tiene sentido la parte sin el todo? Si no, es composición.

```python
class Direccion:
    """
    Dirección de envío. Solo existe en el contexto de un pedido.

    Args:
        calle: Nombre de la calle y número.
        ciudad: Ciudad de destino.
        codigo_postal: Código postal.
    """

    def __init__(self, calle: str, ciudad: str, codigo_postal: str) -> None:
        self.calle = calle
        self.ciudad = ciudad
        self.codigo_postal = codigo_postal

    def __repr__(self) -> str:
        return f"{self.calle}, {self.ciudad} ({self.codigo_postal})"


class Pedido:
    """
    Pedido de compra con dirección de entrega.
    La dirección es parte del pedido — no tiene existencia propia.

    Args:
        numero: Número único de pedido.
        calle: Calle y número de la dirección.
        ciudad: Ciudad de entrega.
        codigo_postal: Código postal.
    """

    def __init__(
        self, numero: str, calle: str, ciudad: str, codigo_postal: str
    ) -> None:
        self.numero = numero
        # La dirección se crea DENTRO del pedido → composición
        self._direccion = Direccion(calle, ciudad, codigo_postal)

    def info_envio(self) -> str:
        """Retorna la información de envío del pedido."""
        return f"Pedido #{self.numero} → {self._direccion}"

    def __repr__(self) -> str:
        return f"Pedido({self.numero!r})"


pedido = Pedido("0042", "Av. Corrientes 1234", "Buenos Aires", "C1043")
print(pedido.info_envio())
# Pedido #0042 → Av. Corrientes 1234, Buenos Aires (C1043)

# La Direccion no existe por sí sola: se crea y vive dentro del Pedido
```

## Herencia

La **herencia** establece una relación "es-un" entre clases: la subclase es una versión especializada de la clase padre. Hereda todos sus atributos y métodos, y puede agregar los propios o redefinir los heredados.

**Test:** ¿Podés decir con naturalidad que "B es un A"? Solo entonces usá herencia.

```python
class Vehiculo:
    """
    Abstracción base para cualquier vehículo.

    Args:
        marca: Marca del vehículo.
        velocidad_max: Velocidad máxima en km/h.
    """

    def __init__(self, marca: str, velocidad_max: int) -> None:
        self.marca = marca
        self.velocidad_max = velocidad_max

    def describir(self) -> str:
        """Descripción genérica del vehículo."""
        return f"{self.marca} (máx. {self.velocidad_max} km/h)"


class Auto(Vehiculo):
    """
    Auto: ES UN Vehículo con número de puertas.

    Args:
        marca: Marca del auto.
        velocidad_max: Velocidad máxima en km/h.
        puertas: Número de puertas.
    """

    def __init__(self, marca: str, velocidad_max: int, puertas: int) -> None:
        super().__init__(marca, velocidad_max)
        self.puertas = puertas

    def describir(self) -> str:
        return f"Auto {self.marca}, {self.puertas} puertas (máx. {self.velocidad_max} km/h)"


auto = Auto("Toyota", 180, 4)
print(auto.describir())  # Auto Toyota, 4 puertas (máx. 180 km/h)
print(isinstance(auto, Vehiculo))  # True — un Auto ES UN Vehículo
```

## Multiplicidad

Más allá del tipo de relación, importa cuántos objetos participan de cada lado:

| Multiplicidad | Implementación en Python | Ejemplo |
| --- | --- | --- |
| Uno a uno (1:1) | Atributo simple | Un `Pedido` tiene una `Direccion` |
| Uno a muchos (1:N) | `list[Tipo]` | Un `Carrito` tiene muchos `Producto` |
| Muchos a muchos (N:M) | `list[Tipo]` en ambos lados | `Estudiante` ↔ `Curso` |

```python
# Uno a muchos: una Factura tiene muchas LineaDeFactura
class LineaDeFactura:
    """
    Línea individual en una factura.

    Args:
        descripcion: Descripción del ítem.
        importe: Importe de la línea.
    """

    def __init__(self, descripcion: str, importe: float) -> None:
        self.descripcion = descripcion
        self.importe = importe


class Factura:
    """
    Factura con múltiples líneas.

    Args:
        numero: Número de factura.
    """

    def __init__(self, numero: str) -> None:
        self.numero = numero
        self._lineas: list[LineaDeFactura] = []   # 1:N

    def agregar_linea(self, descripcion: str, importe: float) -> None:
        """Agrega una línea a la factura."""
        self._lineas.append(LineaDeFactura(descripcion, importe))

    def total(self) -> float:
        """Calcula el total de la factura."""
        return sum(linea.importe for linea in self._lineas)


factura = Factura("F-0001")
factura.agregar_linea("Consultoría (10 hs)", 5000.0)
factura.agregar_linea("Gastos de traslado", 350.0)
print(f"Total: ${factura.total():.2f}")  # Total: $5350.00
```

## Regla práctica

En la mayoría de los sistemas reales, las relaciones más frecuentes son:

- **Composición**: cuando un objeto crea y posee a sus partes (fuerte ownership).
- **Asociación**: cuando un objeto recibe una referencia a otro y lo usa sin poseerlo.

La herencia es menos frecuente de lo que parece al principio. Antes de usarla, aplicá el test "es-un". Si el test falla o generás dudas, la composición suele ser la alternativa más flexible y mantenible.

> **En la práctica:** uno de los errores más costosos en proyectos reales es modelar como herencia lo que en realidad es composición. Heredar por conveniencia (para reutilizar un método) en lugar de por relación "es-un" produce jerarquías frágiles que se rompen cuando los requerimientos cambian. El tema se desarrolla en profundidad en [Composición vs. Herencia](../diseno/composicion_vs_herencia.md).

---
