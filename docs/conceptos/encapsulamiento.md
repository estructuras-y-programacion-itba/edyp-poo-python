# Encapsulamiento

Proteger los datos internos de un objeto y controlar cómo se accede o modifica su estado es la función del encapsulamiento. La idea central es que el estado interno no debería ser accesible directamente desde afuera: cada objeto es el único responsable de mantener su propia consistencia.

Ambos conceptos —abstracción y encapsulamiento— son complementarios pero no equivalentes. Mientras la abstracción define *qué* puede hacer un objeto (su contrato externo), el encapsulamiento protege *cómo* lo hace (su implementación interna).

![El encapsulamiento oculta los detalles de implementación de un objeto](../img/encapsulamiento.png)

## Convenciones en Python

A diferencia de lenguajes como Java o C++, Python no tiene modificadores de acceso explícitos como `private` o `protected`. En cambio, sigue convenciones que todos los desarrolladores respetan:

| Convención | Ejemplo | Significado |
| --- | --- | --- |
| Sin prefijo | `self.nombre` | Público: accesible desde cualquier lugar |
| Un guión bajo | `self._saldo` | "Protegido": convención de que es interno, no parte de la API pública |
| Dos guiones bajos | `self.__saldo` | "Muy privado": Python aplica *name mangling* para dificultar el acceso externo |

En la práctica, `_un_guion` es suficiente para la mayoría de los casos. `__dos_guiones` se reserva para cuando realmente querés evitar que subclases o código externo accedan directamente al atributo.

```python
class CuentaBancaria:
    """Ejemplo de encapsulamiento: el saldo no es accesible directamente."""

    def __init__(self, titular: str, saldo_inicial: float = 0) -> None:
        self._titular = titular
        self.__saldo = saldo_inicial  # __ = name mangling, muy privado

    @property
    def saldo(self) -> float:
        """El saldo es de solo lectura desde afuera."""
        return self.__saldo

    @property
    def titular(self) -> str:
        return self._titular

    def depositar(self, monto: float) -> None:
        if monto <= 0:
            raise ValueError("El monto debe ser positivo")
        self.__saldo += monto

    def retirar(self, monto: float) -> None:
        if monto <= 0:
            raise ValueError("El monto debe ser positivo")
        if monto > self.__saldo:
            raise ValueError("Saldo insuficiente")
        self.__saldo -= monto

    def __repr__(self) -> str:
        return f"CuentaBancaria({self._titular!r}, saldo=${self.__saldo:.2f})"


cuenta = CuentaBancaria("Ana García", 1000)
cuenta.depositar(500)
print(cuenta.saldo)    # ✅ acceso controlado
# cuenta.__saldo = 999999  # ❌ no funciona (name mangling)
```

## Por qué importa el encapsulamiento

Sin encapsulamiento, cualquier parte del programa puede modificar el estado de cualquier objeto sin pasar por sus reglas de validación. Esto lleva a estados inconsistentes que son difíciles de rastrear.

```python
# ❌ Sin encapsulamiento: el saldo puede quedar negativo sin control
class CuentaSinEncapsulamiento:
    def __init__(self, titular: str) -> None:
        self.saldo = 0.0  # público, cualquiera puede modificarlo

cuenta_rota = CuentaSinEncapsulamiento("Carlos")
cuenta_rota.saldo = -99999.0  # Python no se queja, pero el estado es inválido
print(cuenta_rota.saldo)      # -99999.0 ← estado inconsistente
```

```python
# ✅ Con encapsulamiento: las reglas de negocio se aplican siempre
class CuentaSegura:
    """
    Cuenta bancaria con saldo protegido por encapsulamiento.

    Args:
        titular: Nombre del titular.
        saldo_inicial: Saldo inicial de la cuenta.
    """

    def __init__(self, titular: str, saldo_inicial: float = 0.0) -> None:
        if saldo_inicial < 0:
            raise ValueError("El saldo inicial no puede ser negativo")
        self._titular = titular
        self._saldo = saldo_inicial

    @property
    def saldo(self) -> float:
        """Saldo actual (solo lectura)."""
        return self._saldo

    def depositar(self, monto: float) -> None:
        """Acredita monto en la cuenta."""
        if monto <= 0:
            raise ValueError(f"El monto a depositar debe ser positivo, recibí: {monto}")
        self._saldo += monto

    def retirar(self, monto: float) -> None:
        """Debita monto de la cuenta."""
        if monto <= 0:
            raise ValueError(f"El monto a retirar debe ser positivo, recibí: {monto}")
        if monto > self._saldo:
            raise ValueError(
                f"Saldo insuficiente: tenés ${self._saldo:.2f}, intentás retirar ${monto:.2f}"
            )
        self._saldo -= monto

    def __repr__(self) -> str:
        return f"CuentaSegura({self._titular!r}, saldo=${self._saldo:.2f})"
```

## Propiedades con `@property`

El decorador `@property` es la forma idiomática de Python para implementar getters y setters con validación, manteniendo la sintaxis de acceso a atributos (`objeto.atributo`) sin exponer el estado directamente.

La gran ventaja es que podés empezar con un atributo público simple y, si más adelante necesitás agregar validación, convertirlo en una propiedad sin cambiar el código que lo consume.

```python
class Temperatura:
    """
    Temperatura en grados Celsius con validación de rango.

    Args:
        celsius: Temperatura inicial en grados Celsius.
    """

    CERO_ABSOLUTO = -273.15  # constante de clase

    def __init__(self, celsius: float) -> None:
        # Usamos el setter desde el inicio para validar desde la construcción
        self.celsius = celsius

    @property
    def celsius(self) -> float:
        """Temperatura en grados Celsius."""
        return self._celsius

    @celsius.setter
    def celsius(self, valor: float) -> None:
        """Establece la temperatura con validación de cero absoluto."""
        if valor < self.CERO_ABSOLUTO:
            raise ValueError(
                f"Temperatura inválida: {valor}°C es menor al cero absoluto "
                f"({self.CERO_ABSOLUTO}°C)"
            )
        self._celsius = valor

    @property
    def fahrenheit(self) -> float:
        """Temperatura en grados Fahrenheit (derivada, solo lectura)."""
        return self._celsius * 9 / 5 + 32

    @property
    def kelvin(self) -> float:
        """Temperatura en Kelvin (derivada, solo lectura)."""
        return self._celsius - self.CERO_ABSOLUTO

    def __repr__(self) -> str:
        return f"Temperatura({self._celsius}°C)"


# Uso
temp = Temperatura(100.0)
print(temp.celsius)     # 100.0
print(temp.fahrenheit)  # 212.0
print(temp.kelvin)      # 373.15

temp.celsius = 25.0     # ✅ setter con validación
print(temp)             # Temperatura(25.0°C)

try:
    temp.celsius = -300.0  # ❌ ValueError: menor al cero absoluto
except ValueError as e:
    print(e)

# Una propiedad derivada es de solo lectura (no tiene setter)
try:
    temp.fahrenheit = 100.0  # ❌ AttributeError
except AttributeError as e:
    print(e)
```

### Cuándo usar `@property` vs atributo público

| Situación | Recomendación |
| --- | --- |
| El atributo no necesita validación ni transformación | Atributo público directo |
| El atributo necesita validación al ser asignado | `@property` con setter |
| El valor se deriva de otros atributos | `@property` sin setter (solo lectura) |
| El atributo es interno y no forma parte de la API | Prefijo `_` sin property |

> **En la práctica:** un error frecuente es crear getters y setters para *todos* los atributos por defecto, imitando el estilo de Java. En Python, si no necesitás validación ni lógica adicional, un atributo público es perfectamente aceptable. Los `@property` agregan valor cuando hay algo concreto que controlar o calcular, no como burocracia.

---
