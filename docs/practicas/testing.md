# Testing de Clases con pytest

Un programa sin tests es un programa que no sabés si funciona. En POO, los tests tienen un rol especial: **verifican el contrato público de una clase**. Si cambiás la implementación interna sin romper los tests, sabés que el comportamiento externo se preservó.

## ¿Qué es un test unitario?

Un **test unitario** verifica **una sola unidad** de código en aislamiento — en POO, esa unidad es una clase. El objetivo es confirmar que sus métodos se comportan correctamente dado un estado inicial y unos inputs determinados.

Lo que **no** hace un test unitario:

- No testea cómo dos clases interactúan entre sí (eso es un test de integración).
- No testea la base de datos, la red ni el sistema de archivos.
- No testea lógica de otras clases relacionadas.

Si tu clase `Inventario` usa una clase `Producto`, en el test de `Inventario` vas a **simular** el comportamiento de `Producto` con un *mock*, sin usar la clase real.

## pytest: El Framework Estándar

`pytest` es el framework de testing más usado en el ecosistema Python: más conciso que `unittest` y con descubrimiento automático de tests.

**Instalación:**

```bash
pip install pytest pytest-mock
```

**Estructura de archivos recomendada:**

```text
mi_proyecto/
├── src/
│   ├── cuenta_bancaria.py
│   └── producto.py
└── tests/
    ├── test_cuenta_bancaria.py
    └── test_producto.py
```

Los archivos de test deben llamarse `test_*.py` o `*_test.py`, y las funciones (o métodos de clase) deben comenzar con `test_`. Dada esa convención, pytest los descubre y ejecuta automáticamente.

**Cómo correr los tests:**

```bash
# Correr todos los tests del proyecto
pytest

# Correr un archivo específico
pytest tests/test_cuenta_bancaria.py

# Ver output detallado
pytest -v

# Mostrar solo los tests que fallan, con traceback corto
pytest -v --tb=short
```

**Assertions en pytest:**

pytest usa el `assert` nativo de Python; cuando una aserción falla, muestra exactamente qué valores tenía cada lado:

```python
assert cuenta.saldo == 1500    # si falla: AssertionError: assert 1200 == 1500
assert resultado is not None
assert "error" in mensaje.lower()
```

## Tests básicos con clases

```python
# src/cuenta_bancaria.py
class CuentaBancaria:
    """Cuenta bancaria con saldo controlado."""

    def __init__(self, titular: str, saldo_inicial: float = 0.0) -> None:
        if saldo_inicial < 0:
            raise ValueError("El saldo inicial no puede ser negativo")
        self._titular = titular
        self._saldo = saldo_inicial

    @property
    def saldo(self) -> float:
        return self._saldo

    @property
    def titular(self) -> str:
        return self._titular

    def depositar(self, monto: float) -> None:
        if monto <= 0:
            raise ValueError("El monto debe ser positivo")
        self._saldo += monto

    def retirar(self, monto: float) -> None:
        if monto <= 0:
            raise ValueError("El monto debe ser positivo")
        if monto > self._saldo:
            raise ValueError("Saldo insuficiente")
        self._saldo -= monto


# tests/test_cuenta_bancaria.py
import pytest


class TestCuentaBancaria:
    """Tests unitarios para CuentaBancaria."""

    def test_saldo_inicial_por_defecto_es_cero(self) -> None:
        cuenta = CuentaBancaria("Carlos")
        assert cuenta.saldo == 0.0

    def test_saldo_inicial_se_asigna_correctamente(self) -> None:
        cuenta = CuentaBancaria("Carlos", 1000.0)
        assert cuenta.saldo == 1000.0

    def test_saldo_inicial_negativo_lanza_value_error(self) -> None:
        with pytest.raises(ValueError):
            CuentaBancaria("Carlos", -100.0)

    def test_depositar_incrementa_el_saldo(self) -> None:
        cuenta = CuentaBancaria("Carlos", 1000.0)
        cuenta.depositar(500.0)
        assert cuenta.saldo == 1500.0

    def test_depositar_monto_negativo_lanza_value_error(self) -> None:
        cuenta = CuentaBancaria("Carlos", 1000.0)
        with pytest.raises(ValueError, match="debe ser positivo"):
            cuenta.depositar(-50.0)

    def test_retirar_reduce_el_saldo(self) -> None:
        cuenta = CuentaBancaria("Carlos", 1000.0)
        cuenta.retirar(300.0)
        assert cuenta.saldo == 700.0

    def test_retirar_exactamente_el_saldo_disponible(self) -> None:
        """Caso borde: retirar exactamente lo que hay disponible."""
        cuenta = CuentaBancaria("Carlos", 500.0)
        cuenta.retirar(500.0)
        assert cuenta.saldo == 0.0

    def test_retirar_mas_del_saldo_lanza_value_error(self) -> None:
        cuenta = CuentaBancaria("Carlos", 100.0)
        with pytest.raises(ValueError, match="Saldo insuficiente"):
            cuenta.retirar(200.0)


# Fixtures: setup compartido entre tests
@pytest.fixture
def cuenta_con_saldo() -> CuentaBancaria:
    """Cuenta con saldo de $1000 lista para usar en los tests."""
    return CuentaBancaria("Test User", 1000.0)


class TestCuentaBancariaConFixture:
    """Mismos tests usando fixtures para no repetir el setup en cada uno."""

    def test_depositar_con_fixture(self, cuenta_con_saldo: CuentaBancaria) -> None:
        cuenta_con_saldo.depositar(200.0)
        assert cuenta_con_saldo.saldo == 1200.0

    def test_retirar_con_fixture(self, cuenta_con_saldo: CuentaBancaria) -> None:
        cuenta_con_saldo.retirar(400.0)
        assert cuenta_con_saldo.saldo == 600.0
```

## Mocking: Aislar la Clase Bajo Test

Cuando una clase depende de otra, no querés que los tests de la primera fallen por culpa de la segunda. La solución es **mockear** la dependencia: reemplazarla con un objeto simulado.

**¿Cuándo mockear?**

- Cuando tu clase usa otra clase de dominio (para aislar la unidad bajo test).
- Cuando la dependencia conecta con recursos externos: bases de datos, APIs, sistema de archivos.
- Cuando necesitás simular condiciones difíciles de reproducir (ej: una API que devuelve error 500).

| Necesidad | Herramienta |
| --- | --- |
| Reemplazar una clase entera con un objeto falso | `MagicMock(spec=MiClase)` |
| Controlar qué retorna un método | `mock.metodo.return_value = valor` |
| Simular que un método lanza una excepción | `mock.metodo.side_effect = MiError(...)` |
| Verificar que un método fue llamado | `mock.metodo.assert_called_once_with(args)` |
| Verificar que un método NO fue llamado | `mock.metodo.assert_not_called()` |

> **Nota sobre `spec=`**: usar `MagicMock(spec=MiClase)` restringe el mock a los métodos que `MiClase` realmente tiene. Así se evita que los mocks "aprueben" código que llama a métodos con nombres incorrectos.

```python
# src/gateway_pago.py
class GatewayPago:
    """Servicio externo de pago (ej: Mercado Pago, Stripe)."""

    def procesar(self, monto: float, tarjeta: str) -> bool:
        """Llama a la API externa. En tests NO queremos que esto suceda."""
        raise NotImplementedError("Esto conectaría con la API real")


class ProcesadorPedido:
    """
    Procesa un pedido usando un gateway de pago externo.

    Attributes:
        gateway: Servicio de pago inyectado por el constructor.
        comision: Porcentaje de comisión sobre el total (0.0 a 1.0).
    """

    def __init__(self, gateway: GatewayPago, comision: float = 0.05) -> None:
        self._gateway = gateway
        self._comision = comision

    def procesar_pedido(self, monto: float, tarjeta: str) -> dict:
        """
        Procesa el pago de un pedido.

        Returns:
            dict con 'aprobado' (bool) y 'total_cobrado' (float).

        Raises:
            ValueError: si el monto no es positivo.
        """
        if monto <= 0:
            raise ValueError(f"El monto debe ser positivo: {monto}")
        total = monto * (1 + self._comision)
        aprobado = self._gateway.procesar(total, tarjeta)
        return {"aprobado": aprobado, "total_cobrado": total if aprobado else 0.0}


# tests/test_procesador_pedido.py
import pytest
from unittest.mock import MagicMock


class TestProcesadorPedido:
    """
    Tests unitarios de ProcesadorPedido.
    GatewayPago es mockeado: nunca se llama a la API real.
    """

    def test_pedido_aprobado_retorna_total_con_comision(self) -> None:
        gateway_mock = MagicMock(spec=GatewayPago)
        gateway_mock.procesar.return_value = True

        procesador = ProcesadorPedido(gateway_mock, comision=0.10)
        resultado = procesador.procesar_pedido(100.0, "4111-1111-1111-1111")

        assert resultado["aprobado"] is True
        assert resultado["total_cobrado"] == pytest.approx(110.0)

    def test_pedido_rechazado_retorna_cero(self) -> None:
        gateway_mock = MagicMock(spec=GatewayPago)
        gateway_mock.procesar.return_value = False

        procesador = ProcesadorPedido(gateway_mock)
        resultado = procesador.procesar_pedido(200.0, "tarjeta-invalida")

        assert resultado["aprobado"] is False
        assert resultado["total_cobrado"] == 0.0

    def test_monto_negativo_lanza_error_sin_llamar_al_gateway(self) -> None:
        gateway_mock = MagicMock(spec=GatewayPago)
        procesador = ProcesadorPedido(gateway_mock)

        with pytest.raises(ValueError):
            procesador.procesar_pedido(-50.0, "tarjeta")

        # El gateway nunca debe ser llamado si el monto es inválido
        gateway_mock.procesar.assert_not_called()
```

---
