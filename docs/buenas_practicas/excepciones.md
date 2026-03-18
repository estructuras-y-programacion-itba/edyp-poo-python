# Excepciones

Python maneja las situaciones anómalas durante la ejecución mediante excepciones. En el paradigma orientado a objetos, estas son **objetos**: instancias de clases que forman una jerarquía de herencia.

Lejos de ser un recurso de "último momento", el manejo de excepciones es parte integral del **diseño**. Bertrand Meyer formalizó esta idea en el concepto de *Design by Contract*: cada método tiene precondiciones (qué espera recibir) y postcondiciones (qué garantiza producir); las excepciones son el instrumento para comunicar que ese contrato fue violado.

> **En la práctica:** en proyectos reales, las excepciones mal diseñadas son una fuente constante de bugs difíciles de rastrear. Una excepción genérica como `Exception("algo salió mal")` nada le dice a quien la captura; en cambio, una como `MaquinaFalloError` es autodocumentada y le permite a quien llama tomar decisiones informadas.

## Excepciones Built-in Principales

Python incluye una jerarquía de excepciones integrada. Todas heredan de `BaseException`, aunque en código de aplicación trabajás casi siempre con subclases de `Exception`:

```text
BaseException
 ├── SystemExit           → sys.exit() lo lanza, no lo capturés
 ├── KeyboardInterrupt    → Ctrl+C, tampoco lo capturés
 └── Exception
      ├── ValueError       → tipo correcto, valor inválido (ej: temperatura negativa)
      ├── TypeError        → tipo incorrecto (ej: recibís str, esperabas int)
      ├── AttributeError   → el atributo no existe en el objeto
      ├── KeyError         → la clave no existe en el diccionario
      ├── IndexError       → índice fuera de rango en una lista
      ├── FileNotFoundError → el archivo no existe
      ├── NotImplementedError → método abstracto sin implementar
      └── RuntimeError     → error genérico en tiempo de ejecución
```

| Excepción | Cuándo usarla |
| --- | --- |
| `ValueError` | El argumento tiene el tipo correcto pero un valor semánticamente inválido |
| `TypeError` | El argumento tiene el tipo incorrecto |
| `AttributeError` | Se accede a un atributo que no existe en el objeto |
| `KeyError` | Se busca una clave inexistente en un `dict` |
| `IndexError` | Se accede a un índice fuera de rango en una lista |
| `NotImplementedError` | Método que las subclases deben implementar (alternativa a ABC) |

## Estructura del manejo de excepciones

```python
try:
    # código que puede fallar
except TipoDeError as e:
    # qué hacer si ocurre ese error específico
except OtroTipoDeError as e:
    # qué hacer si ocurre este otro error
else:
    # se ejecuta solo si NO hubo ninguna excepción en el try
finally:
    # se ejecuta siempre, con o sin excepción (ideal para liberar recursos)
```

El bloque `else` separa con claridad "la operación exitosa" del bloque `try`, que queda reservado exclusivamente para el manejo de errores. Por su parte, `finally` es ideal para liberar recursos: cerrar archivos, conexiones a bases de datos, etc.

## Uso de excepciones built-in en una clase de dominio

```python
class Maquina:
    """
    Máquina de producción con validación en cada operación.

    Attributes:
        nombre: Nombre o código de la máquina.
    """

    def __init__(self, nombre: str, temp_max: float = 80.0) -> None:
        if not isinstance(nombre, str) or not nombre.strip():
            raise TypeError("El nombre debe ser un string no vacío")
        if temp_max <= 0:
            raise ValueError(f"La temperatura máxima debe ser positiva: {temp_max}")
        self.nombre = nombre
        self._temp_max = temp_max
        self._estado = "inactiva"
        self._ciclos = 0

    @property
    def ciclos(self) -> int:
        return self._ciclos

    def activar(self) -> None:
        self._estado = "activa"

    def iniciar_ciclo(self) -> None:
        """Ejecuta un ciclo. Lanza RuntimeError si la máquina no está activa."""
        if self._estado != "activa":
            raise RuntimeError(f"La máquina '{self.nombre}' debe estar activa")
        self._ciclos += 1

    def __repr__(self) -> str:
        return f"Maquina({self.nombre!r}, ciclos={self._ciclos})"


# Patrones de manejo
maquina = Maquina("CNC-01", temp_max=75.0)
maquina.activar()

# try / except / else / finally
try:
    maquina.iniciar_ciclo()
    ciclos_actuales = maquina.ciclos   # solo se ejecuta si no hubo excepción
except RuntimeError as e:
    print(f"Error operativo: {e}")
else:
    print(f"Ciclo exitoso. Total ciclos: {ciclos_actuales}")   # ✅ se ejecuta
finally:
    print("Operación registrada en bitácora.")                  # siempre se ejecuta
```

## Excepciones Personalizadas

Para errores de programación —un `TypeError` o un `ValueError`— las excepciones built-in son suficientes. Cuando se trata de **reglas de negocio** propias del dominio, sin embargo, lo correcto es definir excepciones propias.

**¿Por qué crear excepciones propias?**

- **Semántica**: `MaquinaFalloError` comunica el problema sin necesidad de leer el mensaje.
- **Captura selectiva**: quien llama puede hacer `except MaquinaFalloError` para manejar ese caso específico.
- **Información adicional**: podés agregar atributos con el contexto relevante.

**Patrón recomendado:** definí una excepción base por módulo o dominio, y excepciones específicas que hereden de ella:

```text
Exception
 └── PlantaError                    ← excepción base del dominio
      ├── MaquinaFalloError          ← error específico con atributos propios
      ├── MaterialInsuficienteError  ← otro error específico
      └── EstacionBloqueadaError     ← error de bloqueo de estación
```

```python
class PlantaError(Exception):
    """Excepción base para todos los errores del módulo de planta."""
    pass


class MaquinaFalloError(PlantaError):
    """Se lanza cuando una máquina detecta una condición de falla."""

    def __init__(self, maquina_id: str, codigo_error: str) -> None:
        self.maquina_id = maquina_id
        self.codigo_error = codigo_error
        super().__init__(
            f"Falla en máquina '{maquina_id}': código {codigo_error}"
        )


class MaterialInsuficienteError(PlantaError):
    """Se lanza cuando no hay suficiente material para completar una operación."""

    def __init__(self, material: str, disponible: float, requerido: float) -> None:
        self.material = material
        self.disponible = disponible
        self.requerido = requerido
        super().__init__(
            f"Material insuficiente para '{material}': "
            f"disponible={disponible:.1f}kg, requerido={requerido:.1f}kg"
        )


class EstacionBloqueadaError(PlantaError):
    """Se lanza cuando una estación está bloqueada y no puede procesar piezas."""

    def __init__(self, nombre: str) -> None:
        self.nombre = nombre
        super().__init__(f"Estación '{nombre}' está bloqueada")


class Maquina:
    """
    Máquina con detección de temperatura fuera de rango.

    Args:
        nombre: Nombre de la máquina.
        temp_max: Temperatura máxima operativa en °C.
    """

    def __init__(self, nombre: str, temp_max: float = 80.0) -> None:
        self.nombre = nombre
        self._temp_max = temp_max
        self._temperatura = 20.0
        self._estado = "activa"

    def actualizar_temperatura(self, temp: float) -> None:
        """
        Actualiza la temperatura del sensor.

        Raises:
            MaquinaFalloError: Si la temperatura supera el máximo operativo.
        """
        self._temperatura = temp
        if temp > self._temp_max:
            self._estado = "falla"
            raise MaquinaFalloError(
                maquina_id=self.nombre,
                codigo_error=f"TEMP_ALTA ({temp}°C > {self._temp_max}°C)"
            )


class LineaDeMontaje:
    """Línea de montaje con manejo de excepciones en el ciclo de producción."""

    def __init__(self, nombre: str) -> None:
        self.nombre = nombre
        self._maquinas: list[Maquina] = []

    def agregar_maquina(self, maquina: Maquina) -> None:
        self._maquinas.append(maquina)

    def ejecutar_ciclo(self, sensor_temp: float) -> str:
        """
        Ejecuta un ciclo de producción con manejo completo de excepciones.

        Args:
            sensor_temp: Temperatura registrada por el sensor externo.

        Returns:
            Mensaje indicando el resultado del ciclo.
        """
        resultado = "desconocido"
        try:
            for maquina in self._maquinas:
                maquina.actualizar_temperatura(sensor_temp)
            resultado = "exitoso"
        except MaquinaFalloError as e:
            resultado = f"fallido — {e}"
            print(f"[ALERTA] {e}")
            print(f"  Máquina afectada: {e.maquina_id}")
            print(f"  Código de error: {e.codigo_error}")
        except PlantaError as e:
            resultado = f"error general — {e}"
            print(f"[ERROR PLANTA] {e}")
        else:
            print(f"Ciclo completado sin incidentes. Temperatura: {sensor_temp}°C")
        finally:
            print(f"Ciclo de línea '{self.nombre}': {resultado}")
        return resultado


# Uso con captura selectiva
linea = LineaDeMontaje("Línea A")
linea.agregar_maquina(Maquina("CNC-01", temp_max=75.0))
linea.agregar_maquina(Maquina("MIG-01", temp_max=90.0))

linea.ejecutar_ciclo(60.0)   # temperatura normal → ciclo exitoso
linea.ejecutar_ciclo(85.0)   # temperatura alta → MaquinaFalloError en CNC-01
```

---
