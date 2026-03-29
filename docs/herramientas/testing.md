# Testing de Clases con pytest

!!! info "Antes de continuar"
    Esta sección da por sentado que sabés definir clases con `@property` y lanzar excepciones con mensajes descriptivos. Revisá [Encapsulamiento](../fundamentos/encapsulamiento.md) y [Excepciones](excepciones.md) si necesitás repasar.

Diseñamos clases con encapsulamiento, contratos claros y excepciones bien definidas. Los tests son la forma de verificar que ese diseño se comporta como esperamos — y de garantizar que siga haciéndolo después de cualquier cambio futuro. En POO, un test unitario verifica el contrato público de una clase, no sus detalles internos.

Un programa sin tests es un programa que no sabés si funciona. En POO, los tests tienen un rol especial: **verifican el contrato público de una clase**. Si cambiás la implementación interna sin romper los tests, sabés que el comportamiento externo se preservó.

## ¿Qué es un test unitario?

Un **test unitario** verifica **una sola unidad** de código en aislamiento — en POO, esa unidad es una clase. El objetivo es confirmar que sus métodos se comportan correctamente dado un estado inicial y unos inputs determinados.

Lo que **no** hace un test unitario:

- No testea cómo dos clases interactúan entre sí (eso es un test de integración).
- No testea la base de datos, la red ni el sistema de archivos.
- No testea lógica de otras clases relacionadas.

Si tu clase `LineaDeMontaje` usa una clase `EstacionTrabajo`, en el test de `LineaDeMontaje` vas a **simular** el comportamiento de `EstacionTrabajo` con un *mock*, sin usar la clase real.

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
│   ├── estacion_trabajo.py
│   └── linea_montaje.py
└── tests/
    ├── test_estacion_corte.py
    └── test_linea_montaje.py
```

Los archivos de test deben llamarse `test_*.py` o `*_test.py`, y las funciones (o métodos de clase) deben comenzar con `test_`. Dada esa convención, pytest los descubre y ejecuta automáticamente.

**Cómo correr los tests:**

```bash
# Correr todos los tests del proyecto
pytest

# Correr un archivo específico
pytest tests/test_estacion_corte.py

# Ver output detallado
pytest -v

# Mostrar solo los tests que fallan, con traceback corto
pytest -v --tb=short
```

**Assertions en pytest:**

pytest usa el `assert` nativo de Python; cuando una aserción falla, muestra exactamente qué valores tenía cada lado:

```python
assert estacion.piezas_procesadas() == 5    # si falla: AssertionError: assert 3 == 5
assert resultado is not None
assert "falla" in mensaje.lower()
```

## Tests básicos con clases

```python
# src/estacion_corte.py
class EstacionCorte:
    """Estación de corte CNC."""

    def __init__(self, nombre: str, velocidad_corte: float) -> None:
        if velocidad_corte <= 0:
            raise ValueError("La velocidad de corte debe ser positiva")
        self.nombre = nombre
        self._velocidad = velocidad_corte
        self._piezas_procesadas: int = 0
        self._bloqueada: bool = False

    @property
    def piezas_procesadas(self) -> int:
        return self._piezas_procesadas

    @property
    def esta_bloqueada(self) -> bool:
        return self._bloqueada

    def bloquear(self) -> None:
        self._bloqueada = True

    def desbloquear(self) -> None:
        self._bloqueada = False

    def procesar(self, pieza: str) -> str:
        if self._bloqueada:
            raise RuntimeError(f"La estación '{self.nombre}' está bloqueada")
        self._piezas_procesadas += 1
        return f"[{self.nombre}] {pieza} cortado a {self._velocidad} mm/min"


# tests/test_estacion_corte.py
import pytest


class TestEstacionCorte:
    """Tests unitarios para EstacionCorte."""

    def test_velocidad_invalida_lanza_value_error(self) -> None:
        with pytest.raises(ValueError):
            EstacionCorte("CNC-01", velocidad_corte=-50.0)

    def test_velocidad_cero_lanza_value_error(self) -> None:
        with pytest.raises(ValueError):
            EstacionCorte("CNC-01", velocidad_corte=0.0)

    def test_piezas_procesadas_inician_en_cero(self) -> None:
        estacion = EstacionCorte("CNC-01", velocidad_corte=200.0)
        assert estacion.piezas_procesadas == 0

    def test_procesar_incrementa_contador(self) -> None:
        estacion = EstacionCorte("CNC-01", velocidad_corte=200.0)
        estacion.procesar("P-0001")
        assert estacion.piezas_procesadas == 1

    def test_procesar_multiples_piezas(self) -> None:
        estacion = EstacionCorte("CNC-01", velocidad_corte=200.0)
        estacion.procesar("P-0001")
        estacion.procesar("P-0002")
        estacion.procesar("P-0003")
        assert estacion.piezas_procesadas == 3

    def test_procesar_retorna_string_con_nombre_pieza(self) -> None:
        estacion = EstacionCorte("CNC-01", velocidad_corte=200.0)
        resultado = estacion.procesar("P-0001")
        assert "P-0001" in resultado

    def test_estacion_bloqueada_lanza_runtime_error(self) -> None:
        estacion = EstacionCorte("CNC-01", velocidad_corte=200.0)
        estacion.bloquear()
        with pytest.raises(RuntimeError, match="bloqueada"):
            estacion.procesar("P-0001")

    def test_pieza_rechazada_no_incrementa_contador_si_bloqueada(self) -> None:
        estacion = EstacionCorte("CNC-01", velocidad_corte=200.0)
        estacion.bloquear()
        try:
            estacion.procesar("P-0001")
        except RuntimeError:
            pass
        assert estacion.piezas_procesadas == 0

    def test_desbloquear_permite_procesar_de_nuevo(self) -> None:
        estacion = EstacionCorte("CNC-01", velocidad_corte=200.0)
        estacion.bloquear()
        estacion.desbloquear()
        resultado = estacion.procesar("P-0001")
        assert "P-0001" in resultado
        assert estacion.piezas_procesadas == 1


# Fixtures: setup compartido entre tests
@pytest.fixture
def estacion_corte_vacia() -> EstacionCorte:
    """EstacionCorte lista para usar, sin piezas procesadas."""
    return EstacionCorte("CNC-TEST", velocidad_corte=200.0)


@pytest.fixture
def linea_con_dos_estaciones():
    """Línea con dos estaciones preconfiguradas."""
    linea = LineaDeMontaje("Línea TEST")
    linea.agregar_estacion(EstacionCorte("CNC-01", velocidad_corte=200.0))
    linea.agregar_estacion(EstacionCorte("CNC-02", velocidad_corte=150.0))
    return linea


class TestEstacionCorteConFixture:
    """Tests usando fixtures para no repetir el setup en cada uno."""

    def test_procesar_con_fixture(self, estacion_corte_vacia: EstacionCorte) -> None:
        estacion_corte_vacia.procesar("P-0001")
        assert estacion_corte_vacia.piezas_procesadas == 1

    def test_bloquear_con_fixture(self, estacion_corte_vacia: EstacionCorte) -> None:
        estacion_corte_vacia.bloquear()
        assert estacion_corte_vacia.esta_bloqueada is True
```

## Mocking: Aislar la Clase Bajo Test

Cuando una clase depende de otra, no querés que los tests de la primera fallen por culpa de la segunda. La solución es **mockear** la dependencia: reemplazarla con un objeto simulado.

**¿Cuándo mockear?**

- Cuando tu clase usa otra clase de dominio (para aislar la unidad bajo test).
- Cuando la dependencia conecta con recursos externos: bases de datos, APIs, sistema de archivos.
- Cuando necesitás simular condiciones difíciles de reproducir (ej: un sensor que devuelve valores fuera de rango).

| Necesidad | Herramienta |
| --- | --- |
| Reemplazar una clase entera con un objeto falso | `MagicMock(spec=MiClase)` |
| Controlar qué retorna un método | `mock.metodo.return_value = valor` |
| Simular que un método lanza una excepción | `mock.metodo.side_effect = MiError(...)` |
| Verificar que un método fue llamado | `mock.metodo.assert_called_once_with(args)` |
| Verificar que un método NO fue llamado | `mock.metodo.assert_not_called()` |

> **Nota sobre `spec=`**: usar `MagicMock(spec=MiClase)` restringe el mock a los métodos que `MiClase` realmente tiene. Así se evita que los mocks "aprueben" código que llama a métodos con nombres incorrectos.

```python
# src/sensor_externo.py
class SensorExterno:
    """Sensor IoT externo que consulta una API de telemetría."""

    def leer_temperatura(self) -> float:
        """Conecta con la API IoT y retorna la temperatura actual."""
        raise NotImplementedError("Esto conectaría con la API real de sensores")

    def esta_activo(self) -> bool:
        """Verifica si el sensor está activo."""
        raise NotImplementedError("Esto consultaría el estado real del sensor")


class ControladorMaquina:
    """
    Controlador que monitorea la máquina usando un sensor IoT externo.

    Attributes:
        sensor: Sensor externo inyectado por el constructor.
        temp_critica: Temperatura a partir de la cual se activa la alerta.
    """

    def __init__(self, sensor: SensorExterno, temp_critica: float = 80.0) -> None:
        self._sensor = sensor
        self._temp_critica = temp_critica
        self._alertas: int = 0

    def monitorear(self) -> dict:
        """
        Lee el sensor y determina el estado de la máquina.

        Returns:
            dict con 'temperatura', 'alerta' (bool) y 'total_alertas'.

        Raises:
            RuntimeError: Si el sensor no está activo.
        """
        if not self._sensor.esta_activo():
            raise RuntimeError("El sensor externo no está activo")
        temperatura = self._sensor.leer_temperatura()
        alerta = temperatura >= self._temp_critica
        if alerta:
            self._alertas += 1
        return {
            "temperatura": temperatura,
            "alerta": alerta,
            "total_alertas": self._alertas,
        }


# tests/test_controlador_maquina.py
import pytest
from unittest.mock import MagicMock


class TestControladorMaquina:
    """
    Tests unitarios de ControladorMaquina.
    SensorExterno es mockeado: nunca se llama a la API real.
    """

    def test_temperatura_normal_no_genera_alerta(self) -> None:
        sensor_mock = MagicMock(spec=SensorExterno)
        sensor_mock.esta_activo.return_value = True
        sensor_mock.leer_temperatura.return_value = 45.0

        controlador = ControladorMaquina(sensor_mock, temp_critica=80.0)
        resultado = controlador.monitorear()

        assert resultado["temperatura"] == 45.0
        assert resultado["alerta"] is False
        assert resultado["total_alertas"] == 0

    def test_temperatura_critica_genera_alerta(self) -> None:
        sensor_mock = MagicMock(spec=SensorExterno)
        sensor_mock.esta_activo.return_value = True
        sensor_mock.leer_temperatura.return_value = 85.0

        controlador = ControladorMaquina(sensor_mock, temp_critica=80.0)
        resultado = controlador.monitorear()

        assert resultado["alerta"] is True
        assert resultado["total_alertas"] == 1

    def test_sensor_inactivo_lanza_runtime_error(self) -> None:
        sensor_mock = MagicMock(spec=SensorExterno)
        sensor_mock.esta_activo.return_value = False

        controlador = ControladorMaquina(sensor_mock)

        with pytest.raises(RuntimeError):
            controlador.monitorear()

        # El sensor de temperatura nunca debe ser consultado si está inactivo
        sensor_mock.leer_temperatura.assert_not_called()
```

## Ver también

Con tests en mano, estás listo para aplicar principios de diseño de mayor nivel que hacen que el código sea más fácil de extender y mantener:

- [Principios SOLID](../avanzado/solid.md) — cinco principios para diseños mantenibles; los tests son la red de seguridad que los hace aplicables
- [Checklist](../avanzado/checklist.md) — lista de verificación antes de entregar cualquier trabajo práctico

---
