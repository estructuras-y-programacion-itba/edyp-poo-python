# Métodos de Clase y Estáticos

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
class Temperatura:
    """
    Temperatura con múltiples constructores para distintas unidades.

    Args:
        celsius: Temperatura en grados Celsius.
    """

    CERO_ABSOLUTO_CELSIUS = -273.15

    def __init__(self, celsius: float) -> None:
        if celsius < self.CERO_ABSOLUTO_CELSIUS:
            raise ValueError(
                f"Temperatura {celsius}°C es menor al cero absoluto"
            )
        self._celsius = celsius

    @classmethod
    def desde_fahrenheit(cls, fahrenheit: float) -> "Temperatura":
        """
        Constructor alternativo: crea una Temperatura desde grados Fahrenheit.

        Args:
            fahrenheit: Temperatura en grados Fahrenheit.

        Returns:
            Nueva instancia de Temperatura.
        """
        celsius = (fahrenheit - 32) * 5 / 9
        return cls(celsius)  # cls, no Temperatura — funciona con subclases también

    @classmethod
    def desde_kelvin(cls, kelvin: float) -> "Temperatura":
        """
        Constructor alternativo: crea una Temperatura desde Kelvin.

        Args:
            kelvin: Temperatura en Kelvin (debe ser no negativo).

        Returns:
            Nueva instancia de Temperatura.

        Raises:
            ValueError: Si el valor en Kelvin es negativo.
        """
        if kelvin < 0:
            raise ValueError("La temperatura en Kelvin no puede ser negativa")
        celsius = kelvin + cls.CERO_ABSOLUTO_CELSIUS
        return cls(celsius)

    @property
    def celsius(self) -> float:
        """Temperatura en grados Celsius."""
        return self._celsius

    @property
    def fahrenheit(self) -> float:
        """Temperatura en grados Fahrenheit."""
        return self._celsius * 9 / 5 + 32

    @property
    def kelvin(self) -> float:
        """Temperatura en Kelvin."""
        return self._celsius - self.CERO_ABSOLUTO_CELSIUS

    def __repr__(self) -> str:
        return f"Temperatura({self._celsius:.2f}°C)"


# Tres formas de crear el mismo objeto
t1 = Temperatura(100.0)                     # desde Celsius (constructor principal)
t2 = Temperatura.desde_fahrenheit(212.0)    # desde Fahrenheit
t3 = Temperatura.desde_kelvin(373.15)       # desde Kelvin

print(t1)  # Temperatura(100.00°C)
print(t2)  # Temperatura(100.00°C)
print(t3)  # Temperatura(100.00°C)
```

### Por qué usar `cls` en lugar de repetir el nombre de la clase

Fijate que en `desde_fahrenheit` usamos `cls(celsius)` en lugar de `Temperatura(celsius)`. Esto importa cuando hay subclases:

```python
class TemperaturaIndustrial(Temperatura):
    """Temperatura con mayor precisión para uso industrial."""
    pass

# Con cls, el método heredado retorna la subclase correcta
ti = TemperaturaIndustrial.desde_fahrenheit(212.0)
print(type(ti))  # <class 'TemperaturaIndustrial'> ✅

# Si hubiéramos escrito Temperatura(celsius) en lugar de cls(celsius):
# type(ti) sería <class 'Temperatura'> ❌ — perdemos el tipo correcto
```

## `@staticmethod`: funciones utilitarias que pertenecen a la clase

Un método estático no recibe ni `self` ni `cls`. Es una función regular que, por organización y cohesión, vive dentro de la clase porque está relacionada con su dominio, pero no necesita acceder al estado de ningún objeto ni a la clase misma.

```python
class Temperatura:
    # ... (misma clase de arriba, extendida)

    CERO_ABSOLUTO_CELSIUS = -273.15

    def __init__(self, celsius: float) -> None:
        if celsius < self.CERO_ABSOLUTO_CELSIUS:
            raise ValueError(f"Temperatura {celsius}°C es menor al cero absoluto")
        self._celsius = celsius

    @classmethod
    def desde_fahrenheit(cls, fahrenheit: float) -> "Temperatura":
        celsius = (fahrenheit - 32) * 5 / 9
        return cls(celsius)

    @classmethod
    def desde_kelvin(cls, kelvin: float) -> "Temperatura":
        if kelvin < 0:
            raise ValueError("La temperatura en Kelvin no puede ser negativa")
        celsius = kelvin + cls.CERO_ABSOLUTO_CELSIUS
        return cls(celsius)

    @staticmethod
    def es_temperatura_valida(valor_celsius: float) -> bool:
        """
        Verifica si un valor en Celsius es físicamente válido.

        No necesita acceder a ningún objeto ni a la clase — es pura lógica
        de dominio que convive con la clase por cohesión.

        Args:
            valor_celsius: Valor a verificar.

        Returns:
            True si el valor es mayor al cero absoluto, False si no.
        """
        return valor_celsius >= -273.15

    @staticmethod
    def convertir_celsius_a_fahrenheit(celsius: float) -> float:
        """
        Convierte Celsius a Fahrenheit sin crear un objeto.

        Args:
            celsius: Temperatura en grados Celsius.

        Returns:
            Temperatura equivalente en grados Fahrenheit.
        """
        return celsius * 9 / 5 + 32

    @property
    def celsius(self) -> float:
        return self._celsius

    def __repr__(self) -> str:
        return f"Temperatura({self._celsius:.2f}°C)"


# Los métodos estáticos se pueden llamar sin instancia
print(Temperatura.es_temperatura_valida(25.0))     # True
print(Temperatura.es_temperatura_valida(-300.0))   # False
print(Temperatura.convertir_celsius_a_fahrenheit(0.0))   # 32.0
print(Temperatura.convertir_celsius_a_fahrenheit(100.0)) # 212.0

# También se pueden llamar desde una instancia (aunque es menos idiomático)
t = Temperatura(20.0)
print(t.es_temperatura_valida(-280.0))  # False
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
class Animal:
    sonido = "..."

    @staticmethod
    def crear_generico_mal() -> "Animal":
        return Animal()  # ❌ hardcodeado — las subclases no pueden sobreescribir bien esto

    @classmethod
    def crear_generico_bien(cls) -> "Animal":
        return cls()  # ✅ crea instancia de la clase que recibe (subclase incluida)


class Perro(Animal):
    sonido = "guau"


# Con @staticmethod: siempre retorna Animal
perro_mal = Perro.crear_generico_mal()
print(type(perro_mal))   # <class 'Animal'> ❌ — esperábamos Perro

# Con @classmethod: retorna Perro porque cls = Perro
perro_bien = Perro.crear_generico_bien()
print(type(perro_bien))  # <class 'Perro'> ✅
```

> **En la práctica:** los `@classmethod` son especialmente útiles para leer objetos desde distintos formatos (JSON, CSV, diccionarios), implementando el patrón *Factory*. En lugar de llenar el `__init__` con parámetros opcionales para cada formato posible, creás un método de clase por formato: `Producto.desde_dict(data)`, `Producto.desde_csv(linea)`, `Producto.desde_json(texto)`. El `__init__` queda limpio y cada constructor alternativo tiene su propósito claro.

---
