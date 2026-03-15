# Excepciones

Las excepciones son el mecanismo que usa Python para manejar situaciones anómalas durante la ejecución. En el paradigma orientado a objetos, las excepciones son **objetos**: instancias de clases que forman una jerarquía de herencia.

Lejos de ser un recurso de "último momento", el manejo de excepciones es parte integral del **diseño**. Bertrand Meyer lo formalizó en el concepto de *Design by Contract*: cada método tiene precondiciones (qué espera recibir) y postcondiciones (qué garantiza producir). Las excepciones son el instrumento para comunicar que ese contrato fue violado.

> **En la práctica:** en proyectos reales, las excepciones mal diseñadas son una fuente constante de bugs difíciles de rastrear. Una excepción genérica como `Exception("algo salió mal")` no le dice nada a quien la captura. Una excepción bien diseñada como `StockInsuficienteError` es autodocumentada y le permite a quien llama tomar decisiones informadas.

## Excepciones Built-in Principales

Python incluye una jerarquía de excepciones. Todas heredan de `BaseException`, pero en código de aplicación trabajás casi siempre con subclases de `Exception`:

```text
BaseException
 ├── SystemExit           → sys.exit() lo lanza, no lo capturés
 ├── KeyboardInterrupt    → Ctrl+C, tampoco lo capturés
 └── Exception
      ├── ValueError       → tipo correcto, valor inválido (ej: precio negativo)
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

El bloque `else` es útil para separar "la operación exitosa" del bloque `try`, dejando este último solo para el manejo de errores. El bloque `finally` es ideal para cerrar archivos, conexiones a bases de datos, etc.

## Uso de excepciones built-in en una clase de dominio

```python
class CuentaBancaria:
    """
    Cuenta bancaria con saldo protegido.

    Attributes:
        titular: Nombre del titular de la cuenta.
    """

    def __init__(self, titular: str, saldo_inicial: float = 0.0) -> None:
        if not isinstance(titular, str) or not titular.strip():
            raise TypeError("El titular debe ser un string no vacío")
        if saldo_inicial < 0:
            raise ValueError(f"El saldo inicial no puede ser negativo: {saldo_inicial}")
        self._titular = titular
        self._saldo = saldo_inicial

    @property
    def saldo(self) -> float:
        return self._saldo

    @property
    def titular(self) -> str:
        return self._titular

    def depositar(self, monto: float) -> None:
        """Incrementa el saldo. Lanza ValueError si el monto no es positivo."""
        if monto <= 0:
            raise ValueError(f"El monto a depositar debe ser positivo, recibí: {monto}")
        self._saldo += monto

    def retirar(self, monto: float) -> None:
        """Decrementa el saldo. Lanza ValueError si el monto es inválido o el saldo insuficiente."""
        if monto <= 0:
            raise ValueError(f"El monto a retirar debe ser positivo, recibí: {monto}")
        if monto > self._saldo:
            raise ValueError(
                f"Saldo insuficiente: tenés ${self._saldo:.2f}, intentás retirar ${monto:.2f}"
            )
        self._saldo -= monto

    def __repr__(self) -> str:
        return f"CuentaBancaria({self._titular!r}, saldo=${self._saldo:.2f})"


# Patrones de manejo
cuenta = CuentaBancaria("Ana García", 1000.0)

# try / except / else / finally
try:
    cuenta.retirar(200.0)
    saldo_actual = cuenta.saldo      # solo se ejecuta si no hubo excepción
except ValueError as e:
    print(f"Error de negocio: {e}")
else:
    print(f"Retiro exitoso. Saldo actual: ${saldo_actual:.2f}")   # ✅ se ejecuta
finally:
    print("Operación registrada.")                                 # siempre se ejecuta
```

## Excepciones Personalizadas

Las excepciones built-in son suficientes para errores de programación: un `TypeError` o `ValueError` son claros. Pero para **reglas de negocio** propias de tu dominio, lo correcto es definir excepciones propias.

**¿Por qué crear excepciones propias?**

- **Semántica**: `StockInsuficienteError` comunica el problema sin necesidad de leer el mensaje.
- **Captura selectiva**: quien llama puede hacer `except StockInsuficienteError` para manejar ese caso específico.
- **Información adicional**: podés agregar atributos con el contexto relevante.

**Patrón recomendado:** definí una excepción base por módulo o dominio, y excepciones específicas que hereden de ella:

```text
Exception
 └── InventarioError                ← excepción base del dominio
      ├── StockInsuficienteError    ← error específico con atributos propios
      └── ProductoNoEncontradoError ← otro error específico
```

```python
class InventarioError(Exception):
    """Excepción base para todos los errores del módulo de inventario."""
    pass


class StockInsuficienteError(InventarioError):
    """Se lanza cuando no hay suficiente stock para completar una operación."""

    def __init__(self, producto: str, disponible: int, solicitado: int) -> None:
        self.producto = producto
        self.disponible = disponible
        self.solicitado = solicitado
        super().__init__(
            f"Stock insuficiente para '{producto}': "
            f"disponible={disponible}, solicitado={solicitado}"
        )


class ProductoNoEncontradoError(InventarioError):
    """Se lanza cuando se busca un producto que no existe en el inventario."""

    def __init__(self, nombre: str) -> None:
        self.nombre = nombre
        super().__init__(f"Producto no encontrado: '{nombre}'")


class Producto:
    """
    Representa un producto con nombre, precio y stock disponible.

    Args:
        nombre: Nombre del producto.
        precio: Precio unitario (debe ser positivo).
        stock: Cantidad disponible (debe ser no negativa).
    """

    def __init__(self, nombre: str, precio: float, stock: int) -> None:
        self.nombre = nombre
        self._precio = precio
        self._stock = stock

    def vender(self, cantidad: int) -> float:
        """
        Descuenta stock y retorna el total de la venta.

        Raises:
            ValueError: si la cantidad no es positiva.
            StockInsuficienteError: si no hay suficiente stock.
        """
        if cantidad <= 0:
            raise ValueError("La cantidad debe ser positiva")
        if cantidad > self._stock:
            raise StockInsuficienteError(self.nombre, self._stock, cantidad)
        self._stock -= cantidad
        return cantidad * self._precio


# Uso con captura selectiva
laptop = Producto("Laptop", 1500.0, 3)

try:
    total = laptop.vender(10)
except StockInsuficienteError as e:
    print(f"No podemos completar el pedido: {e}")
    print(f"Te ofrecemos {e.disponible} unidades en su lugar.")
except InventarioError as e:
    print(f"Error de inventario: {e}")
```

---
