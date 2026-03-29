# Métodos de Instancia

Ya sabemos que una clase define atributos (estado) y métodos (comportamiento). Los métodos de instancia son la forma más directa de implementar ese comportamiento: acceden al estado del objeto a través de `self` y son el tipo de método que más frecuentemente vas a escribir.

Los métodos de instancia son el tipo más común de métodos en Python. Son funciones definidas dentro de una clase que operan sobre un objeto específico: cada llamada a un método de instancia tiene acceso al estado particular de *ese* objeto.

## `self`: el nexo entre el método y el objeto

El primer parámetro de todo método de instancia se llama `self` por convención. Python lo pasa automáticamente cuando llamás el método sobre un objeto: `linea.agregar_estacion(est)` es equivalente a `LineaDeMontaje.agregar_estacion(linea, est)`.

A través de `self`, el método puede:

- Leer atributos del objeto: `self._estaciones`
- Modificar atributos del objeto: `self._total = 0`
- Llamar a otros métodos del mismo objeto: `self.obtener_reporte()`

```python
class LineaDeMontaje:
    def __init__(self) -> None:
        self._estaciones: list = []  # self da acceso al estado del objeto

    def agregar_estacion(self, estacion) -> None:
        self._estaciones.append(estacion)
        # self._estaciones es EL estado de ESTA línea, no de otra
```

## Métodos que leen vs. métodos que modifican el estado

Los métodos de instancia se dividen en dos grandes grupos según su efecto sobre el estado del objeto:

| Tipo | Efecto | Ejemplo |
| --- | --- | --- |
| **Consulta / getter** | Lee el estado, no lo modifica | `obtener_reporte()`, `esta_vacia()` |
| **Comando / mutador** | Modifica el estado del objeto | `agregar_estacion()`, `procesar_pieza()` |

Esta distinción importa para el diseño. Un principio útil es el **Command-Query Separation** de Bertrand Meyer: un método debería *o bien* modificar el estado (comando), *o bien* retornar información (consulta), pero no ambas cosas a la vez. Mezclarlas dificulta el razonamiento sobre el código.

## Convenciones de nomenclatura

- Nombres en **snake_case**: `procesar_pieza()`, `agregar_estacion()`.
- Verbos para métodos que realizan acciones: `agregar`, `procesar`, `activar`, `reportar`.
- Sustantivos o adjetivos para métodos que retornan información: `obtener_reporte()`, `esta_vacia()`, `total_estaciones()`.
- Prefijo `_` para métodos internos que no forman parte de la API pública: `_validar_estacion()`.

## Llamada a métodos

Cuando llamás `objeto.metodo()`, Python busca `metodo` en la clase del objeto y lo llama pasando el objeto como primer argumento (`self`). No tenés que pasarlo vos.

```python
linea = LineaDeMontaje("Línea A")
linea.agregar_estacion(est)  # Python pasa linea como self
```

## Ejemplo completo: clase `LineaDeMontaje`

```python
class EstacionNoDisponibleError(Exception):
    """Se lanza cuando se intenta usar una estación que no existe en la línea."""

    def __init__(self, nombre: str) -> None:
        self.nombre = nombre
        super().__init__(f"La estación '{nombre}' no está en la línea de montaje")


class EstacionTrabajo:
    """
    Estación de trabajo básica para los ejemplos.

    Args:
        nombre: Nombre de la estación.
        tiempo_ciclo: Tiempo de ciclo en segundos.
    """

    def __init__(self, nombre: str, tiempo_ciclo: float) -> None:
        self.nombre = nombre
        self.tiempo_ciclo = tiempo_ciclo
        self._piezas_procesadas: int = 0

    def procesar(self, pieza: str) -> str:
        self._piezas_procesadas += 1
        return f"[{self.nombre}] {pieza} procesada"

    def piezas_procesadas(self) -> int:
        return self._piezas_procesadas

    def __repr__(self) -> str:
        return f"EstacionTrabajo({self.nombre!r})"


class LineaDeMontaje:
    """
    Línea de montaje con estaciones de trabajo ordenadas.

    Una línea comienza vacía. Las estaciones se agregan en orden y las piezas
    las atraviesan en secuencia. El reporte se calcula a demanda.
    """

    def __init__(self, nombre: str) -> None:
        self._nombre = nombre
        self._estaciones: dict[str, EstacionTrabajo] = {}
        self._piezas_totales: int = 0

    # ── Comandos (modifican estado) ──────────────────────────────────────────

    def agregar_estacion(self, estacion: EstacionTrabajo) -> None:
        """
        Agrega una estación al final de la línea.

        Args:
            estacion: Estación de trabajo a agregar.

        Raises:
            ValueError: Si ya existe una estación con ese nombre.
        """
        if estacion.nombre in self._estaciones:
            raise ValueError(
                f"Ya existe una estación llamada '{estacion.nombre}' en esta línea"
            )
        self._estaciones[estacion.nombre] = estacion

    def remover_estacion(self, nombre: str) -> None:
        """
        Elimina una estación de la línea por nombre.

        Args:
            nombre: Nombre de la estación a remover.

        Raises:
            EstacionNoDisponibleError: Si la estación no existe.
        """
        if nombre not in self._estaciones:
            raise EstacionNoDisponibleError(nombre)
        del self._estaciones[nombre]

    def procesar_pieza(self, pieza: str) -> list[str]:
        """
        Pasa la pieza por todas las estaciones en orden.

        Args:
            pieza: Identificador de la pieza a procesar.

        Returns:
            Lista con el resultado de cada estación.

        Raises:
            RuntimeError: Si la línea no tiene estaciones.
        """
        if not self._estaciones:
            raise RuntimeError("La línea no tiene estaciones configuradas")
        self._piezas_totales += 1
        return [est.procesar(pieza) for est in self._estaciones.values()]

    # ── Consultas (retornan información, no modifican estado) ────────────────

    def obtener_reporte(self) -> str:
        """
        Genera un reporte del estado de la línea.

        Returns:
            Reporte en formato de texto.
        """
        lineas = [f"=== Línea de Montaje: {self._nombre} ==="]
        for est in self._estaciones.values():
            lineas.append(
                f"  {est.nombre}: {est.piezas_procesadas()} piezas (ciclo: {est.tiempo_ciclo}s)"
            )
        lineas.append(f"  Total piezas: {self._piezas_totales}")
        return "\n".join(lineas)

    def total_estaciones(self) -> int:
        """Retorna la cantidad de estaciones en la línea."""
        return len(self._estaciones)

    def esta_vacia(self) -> bool:
        """Indica si la línea no tiene estaciones."""
        return len(self._estaciones) == 0

    def listar_estaciones(self) -> list[str]:
        """
        Retorna una lista con los nombres de las estaciones.

        Returns:
            Lista de nombres de estaciones en orden.
        """
        return list(self._estaciones.keys())

    # ── Métodos mágicos ──────────────────────────────────────────────────────

    def __repr__(self) -> str:
        return f"LineaDeMontaje({self._nombre!r}, estaciones={self.total_estaciones()})"

    def __str__(self) -> str:
        if self.esta_vacia():
            return f"Línea '{self._nombre}' vacía"
        return self.obtener_reporte()

    def __len__(self) -> int:
        return len(self._estaciones)


# Demostración
linea = LineaDeMontaje("Línea A")

# Comandos: modifican el estado
corte = EstacionTrabajo("CNC-01", tiempo_ciclo=12.0)
soldadura = EstacionTrabajo("MIG-01", tiempo_ciclo=30.0)
pintura = EstacionTrabajo("PAINT-01", tiempo_ciclo=20.0)

linea.agregar_estacion(corte)
linea.agregar_estacion(soldadura)
linea.agregar_estacion(pintura)

resultados = linea.procesar_pieza("P-0001")
for r in resultados:
    print(r)
# [CNC-01] P-0001 procesada
# [MIG-01] P-0001 procesada
# [PAINT-01] P-0001 procesada

print(linea)
# === Línea de Montaje: Línea A ===
#   CNC-01: 1 piezas (ciclo: 12.0s)
#   MIG-01: 1 piezas (ciclo: 30.0s)
#   PAINT-01: 1 piezas (ciclo: 20.0s)
#   Total piezas: 1

# Consultas: leen sin modificar
print(f"Estaciones: {linea.total_estaciones()}")  # 3
print(f"Vacía: {linea.esta_vacia()}")              # False
print(f"len: {len(linea)}")                        # 3

# Remover una estación
linea.remover_estacion("PAINT-01")
print(linea.listar_estaciones())  # ['CNC-01', 'MIG-01']

# Manejo de error
try:
    linea.remover_estacion("SOLDADORA-99")  # no existe
except EstacionNoDisponibleError as e:
    print(e)  # La estación 'SOLDADORA-99' no está en la línea de montaje
```

## Los métodos de instancia y el encapsulamiento

Los métodos de instancia son la única vía legítima para interactuar con los atributos privados de un objeto. Esta es la esencia del encapsulamiento: el estado interno (`_estaciones`) no se expone directamente; en cambio, los métodos de instancia (`agregar_estacion`, `procesar_pieza`, `obtener_reporte`) definen la API a través de la cual el mundo exterior puede interactuar con la línea.

Si el estado fuera público, cualquier parte del código podría hacer `linea._estaciones["CNC-01"] = None` sin pasar por ninguna validación. Los métodos de instancia son los guardianes de la consistencia del estado.

> **En la práctica:** una señal de buen diseño es que podés cambiar completamente la representación interna del estado (por ejemplo, pasar de un `dict` a una `list` para las estaciones) sin que el código externo que usa la línea se entere. Eso es encapsulamiento real funcionando.

## Ver también

Los métodos de instancia son el núcleo del comportamiento de un objeto. Las siguientes secciones agregan capas que hacen ese comportamiento más robusto y expresivo:

- [Type Hinting](type_hinting.md) — cómo documentar los tipos de parámetros y retorno de tus métodos
- [Encapsulamiento](encapsulamiento.md) — cómo los métodos actúan como guardianes del estado interno
- [Métodos Mágicos](../metodos/magicos.md) — métodos especiales que integran tus clases con el lenguaje

---
