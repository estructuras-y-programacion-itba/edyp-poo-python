# Métodos de Clase y Estáticos

!!! info "Antes de continuar"
    Esta sección asume que ya conocés los métodos de instancia. Si no los repasaste todavía, arrancá por [Métodos de Instancia](../fundamentos/instancia.md).

Los métodos de instancia son el tipo más frecuente, pero no el único. Python ofrece dos tipos adicionales — `@classmethod` y `@staticmethod` — que responden a necesidades específicas de diseño: crear objetos desde distintos formatos o agrupar funciones de dominio que no necesitan acceder al estado.

Además de los métodos de instancia, Python ofrece dos tipos adicionales de métodos que responden a necesidades específicas de diseño. Conocerlos te permite elegir la herramienta correcta para cada situación.

## Los tres tipos de métodos: un resumen

| Tipo | Decorador | Primer parámetro | Accede a |
| --- | --- | --- | --- |
| De instancia | (ninguno) | `self` (el objeto) | Estado del objeto + clase |
| De clase | `@classmethod` | `cls` (la clase) | Solo la clase, no el objeto |
| Estático | `@staticmethod` | (ninguno) | Ni la clase ni el objeto |

## `@classmethod`: métodos que operan sobre la clase

Un método de clase recibe `cls` como primer argumento: una referencia a la **clase** (no a un objeto particular). Puede acceder y modificar atributos de clase, pero no conoce el estado de ninguna instancia.

El uso más común y elegante de `@classmethod` son los **constructores alternativos**: formas adicionales de crear objetos que complementan el `__init__` estándar.

```python
class Maquina:
    """
    Máquina de producción con múltiples constructores para distintas fuentes de datos.

    Args:
        nombre: Nombre o código de la máquina.
        capacidad: Capacidad de producción en piezas/hora.
        tiempo_ciclo: Tiempo de ciclo en segundos.
    """

    def __init__(self, nombre: str, capacidad: int, tiempo_ciclo: float) -> None:
        if capacidad <= 0:
            raise ValueError("La capacidad debe ser positiva")
        if tiempo_ciclo <= 0:
            raise ValueError("El tiempo de ciclo debe ser positivo")
        self.nombre = nombre
        self._capacidad = capacidad
        self._tiempo_ciclo = tiempo_ciclo
        self._estado = "inactiva"

    @classmethod
    def desde_configuracion(cls, config: dict) -> "Maquina":
        """
        Constructor alternativo: crea una Maquina desde un diccionario de configuración.

        Args:
            config: Diccionario con las claves 'nombre', 'capacidad', 'tiempo_ciclo'.

        Returns:
            Nueva instancia de Maquina.
        """
        return cls(
            nombre=config["nombre"],
            capacidad=config["capacidad"],
            tiempo_ciclo=config["tiempo_ciclo"],
        )  # cls, no Maquina — funciona con subclases también

    @classmethod
    def desde_plantilla(cls, tipo: str) -> "Maquina":
        """
        Constructor alternativo: crea una Maquina desde una plantilla predefinida.

        Args:
            tipo: Tipo de máquina ('cnc', 'soldadora', 'torno').

        Returns:
            Nueva instancia de Maquina con configuración estándar.

        Raises:
            ValueError: Si el tipo no está reconocido.
        """
        plantillas = {
            "cnc":       ("CNC-STD", 60, 15.0),
            "soldadora": ("SOL-STD", 30, 30.0),
            "torno":     ("TOR-STD", 45, 20.0),
        }
        if tipo not in plantillas:
            raise ValueError(f"Tipo de máquina desconocido: '{tipo}'")
        nombre, capacidad, tiempo_ciclo = plantillas[tipo]
        return cls(nombre, capacidad, tiempo_ciclo)

    def __repr__(self) -> str:
        return f"Maquina({self.nombre!r}, cap={self._capacidad})"


# Tres formas de crear el mismo tipo de objeto
m1 = Maquina("CNC-01", capacidad=60, tiempo_ciclo=15.0)  # constructor principal
m2 = Maquina.desde_configuracion({                        # desde dict
    "nombre": "CNC-02",
    "capacidad": 60,
    "tiempo_ciclo": 15.0,
})
m3 = Maquina.desde_plantilla("cnc")                       # desde plantilla

print(m1)  # Maquina('CNC-01', cap=60)
print(m2)  # Maquina('CNC-02', cap=60)
print(m3)  # Maquina('CNC-STD', cap=60)
```

Otro uso comun que haremos de los classmethods es el de **métodos de clase utilitarios** como por ejemplo: evitar que un atributo identificador de una maquina o una persona se repita, o llevar un conteo de cuántas máquinas se han creado. En esos casos, el método de clase no es un constructor alternativo sino una función que opera sobre la clase para mantener su estado o lógica relacionada.

### Por qué usar `cls` en lugar de repetir el nombre de la clase

Fijate que en `desde_plantilla` usamos `cls(nombre, capacidad, tiempo_ciclo)` en lugar de `Maquina(...)`. Esto importa cuando hay subclases:

```python
class MaquinaRobotica(Maquina):
    """Máquina con control robótico de alta precisión."""
    pass

# Con cls, el método heredado retorna la subclase correcta
mr = MaquinaRobotica.desde_plantilla("cnc")
print(type(mr))  # <class 'MaquinaRobotica'> ✅

# Si hubiéramos escrito Maquina(...) en lugar de cls(...):
# type(mr) sería <class 'Maquina'> ❌ — perdemos el tipo correcto
```

## `@staticmethod`: funciones utilitarias que pertenecen a la clase

Un método estático no recibe ni `self` ni `cls`. Es una función regular que, por organización y cohesión, vive dentro de la clase porque está relacionada con su dominio, pero no necesita acceder al estado de ningún objeto ni a la clase misma.

```python
class Maquina:
    # ... (misma clase de arriba, extendida)

    def __init__(self, nombre: str, capacidad: int, tiempo_ciclo: float) -> None:
        if capacidad <= 0:
            raise ValueError("La capacidad debe ser positiva")
        self.nombre = nombre
        self._capacidad = capacidad
        self._tiempo_ciclo = tiempo_ciclo
        self._estado = "inactiva"

    @classmethod
    def desde_configuracion(cls, config: dict) -> "Maquina":
        return cls(
            nombre=config["nombre"],
            capacidad=config["capacidad"],
            tiempo_ciclo=config["tiempo_ciclo"],
        )

    @classmethod
    def desde_plantilla(cls, tipo: str) -> "Maquina":
        plantillas = {
            "cnc":       ("CNC-STD", 60, 15.0),
            "soldadora": ("SOL-STD", 30, 30.0),
        }
        if tipo not in plantillas:
            raise ValueError(f"Tipo desconocido: '{tipo}'")
        nombre, cap, ciclo = plantillas[tipo]
        return cls(nombre, cap, ciclo)

    @staticmethod
    def calcular_eficiencia(piezas_buenas: int, piezas_totales: int) -> float:
        """
        Calcula la eficiencia de producción como porcentaje.

        No necesita acceder a ningún objeto ni a la clase — es pura lógica
        de dominio que convive con la clase por cohesión.

        Args:
            piezas_buenas: Cantidad de piezas aprobadas.
            piezas_totales: Cantidad total de piezas producidas.

        Returns:
            Eficiencia como porcentaje (0.0 a 100.0).
        """
        if piezas_totales == 0:
            return 0.0
        return piezas_buenas / piezas_totales * 100

    @staticmethod
    def es_temperatura_operativa(temp: float) -> bool:
        """
        Verifica si una temperatura está dentro del rango operativo estándar.

        Args:
            temp: Temperatura en °C a verificar.

        Returns:
            True si la temperatura está entre 10°C y 80°C.
        """
        return 10.0 <= temp <= 80.0

    def __repr__(self) -> str:
        return f"Maquina({self.nombre!r}, cap={self._capacidad})"


# Los métodos estáticos se pueden llamar sin instancia
print(Maquina.calcular_eficiencia(95, 100))   # 95.0
print(Maquina.calcular_eficiencia(0, 0))       # 0.0
print(Maquina.es_temperatura_operativa(45.0)) # True
print(Maquina.es_temperatura_operativa(95.0)) # False

# También se pueden llamar desde una instancia (aunque es menos idiomático)
m = Maquina("CNC-01", 60, 15.0)
print(m.calcular_eficiencia(88, 100))  # 88.0
```

## Tabla de decisión: ¿cuál uso?

| Pregunta | Respuesta | Usá |
| --- | --- | --- |
| ¿Necesita acceder al estado de un objeto específico? | Sí | Método de **instancia** |
| ¿Es un constructor alternativo o necesita crear instancias de la clase? | Sí | `@classmethod` |
| ¿Necesita acceder o modificar atributos de clase (compartidos)? | Sí | `@classmethod` |
| ¿Es lógica utilitaria relacionada al dominio pero sin acceso a objetos ni clase? | Sí | `@staticmethod` |

## Error frecuente: confundir `@staticmethod` con `@classmethod`

El error más común es usar `@staticmethod` cuando en realidad se necesita `@classmethod`, perdiendo los beneficios de la herencia:

```python
class Maquina:
    tipo_default = "genérica"

    @staticmethod
    def crear_default_mal() -> "Maquina":
        return Maquina("DEFAULT", 10, 5.0)  # ❌ hardcodeado — las subclases no pueden sobreescribir bien esto

    @classmethod
    def crear_default_bien(cls) -> "Maquina":
        return cls("DEFAULT", 10, 5.0)  # ✅ crea instancia de la clase que recibe (subclase incluida)


class MaquinaRobotica(Maquina):
    tipo_default = "robótica"


# Con @staticmethod: siempre retorna Maquina
m_mal = MaquinaRobotica.crear_default_mal()
print(type(m_mal))   # <class 'Maquina'> ❌ — esperábamos MaquinaRobotica

# Con @classmethod: retorna MaquinaRobotica porque cls = MaquinaRobotica
m_bien = MaquinaRobotica.crear_default_bien()
print(type(m_bien))  # <class 'MaquinaRobotica'> ✅
```

> **En la práctica:** los `@classmethod` son especialmente útiles para leer objetos desde distintos formatos (JSON, CSV, diccionarios), implementando el patrón *Factory*. En lugar de llenar el `__init__` con parámetros opcionales para cada formato posible, creás un método de clase por formato: `Maquina.desde_configuracion(config)`, `Maquina.desde_csv(linea)`, `Maquina.desde_plantilla(tipo)`. El `__init__` queda limpio y cada constructor alternativo tiene su propósito claro.

## Ver también

Con los tres tipos de métodos cubiertos, el paso natural es explorar los métodos especiales que Python llama automáticamente en respuesta a operadores y funciones built-in:

- [Métodos Mágicos (Dunder)](magicos.md) — `__repr__`, `__eq__`, `__len__` y cómo integrar tus clases con el lenguaje

---
