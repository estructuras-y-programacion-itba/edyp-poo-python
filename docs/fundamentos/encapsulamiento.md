# Encapsulamiento

Con clases, métodos e indicaciones de tipo en mano, surge una pregunta de diseño crítica: ¿quién puede leer y modificar los atributos de un objeto? El encapsulamiento responde esa pregunta — y es uno de los cuatro pilares fundamentales de la POO según Booch.

Proteger los datos internos de un objeto y controlar cómo se accede o modifica su estado es la función del encapsulamiento. La idea central es que el estado interno no debería ser accesible directamente desde afuera: cada objeto es el único responsable de mantener su propia consistencia.

Ambos conceptos —abstracción y encapsulamiento— son complementarios pero no equivalentes. Mientras la abstracción define *qué* puede hacer un objeto (su contrato externo), el encapsulamiento protege *cómo* lo hace (su implementación interna).

![El encapsulamiento oculta los detalles de implementación de un objeto](../img/encapsulamiento.png)

## Convenciones en Python

A diferencia de lenguajes como Java o C++, Python no tiene modificadores de acceso explícitos como `private` o `protected`. En cambio, sigue convenciones que todos los desarrolladores respetan:

| Convención | Ejemplo | Significado |
| --- | --- | --- |
| Sin prefijo | `self.nombre` | Público: accesible desde cualquier lugar |
| Un guión bajo | `self._estado` | "Protegido": convención de que es interno, no parte de la API pública |
| Dos guiones bajos | `self.__contador` | "Muy privado": Python aplica *name mangling* para dificultar el acceso externo |

En la práctica, `_un_guion` es suficiente para la mayoría de los casos. `__dos_guiones` se reserva para cuando realmente querés evitar que subclases o código externo accedan directamente al atributo.

```python
class Maquina:
    """Ejemplo de encapsulamiento: los contadores internos no son accesibles directamente."""

    def __init__(self, nombre: str) -> None:
        self.nombre = nombre                  # público
        self._estado = "inactiva"             # protegido
        self.__contador_piezas = 0            # muy privado (name mangling)
        self.__sensor_temperatura = 20.0      # muy privado

    @property
    def contador_piezas(self) -> int:
        """El contador de piezas es de solo lectura desde afuera."""
        return self.__contador_piezas

    @property
    def temperatura(self) -> float:
        """Temperatura actual del sensor interno (solo lectura)."""
        return self.__sensor_temperatura

    def iniciar_ciclo(self) -> None:
        """Ejecuta un ciclo de producción incrementando el contador interno."""
        if self._estado != "activa":
            raise RuntimeError(f"La máquina '{self.nombre}' no está activa")
        self.__contador_piezas += 1

    def activar(self) -> None:
        self._estado = "activa"

    def obtener_reporte(self) -> dict:
        """API pública para consultar el estado."""
        return {
            "nombre": self.nombre,
            "estado": self._estado,
            "piezas": self.__contador_piezas,
        }

    def __repr__(self) -> str:
        return f"Maquina({self.nombre!r}, estado={self._estado!r})"


maquina = Maquina("CNC-01")
maquina.activar()
maquina.iniciar_ciclo()
print(maquina.contador_piezas)    # ✅ acceso controlado
# maquina.__contador_piezas = 999  # ❌ no funciona (name mangling)
```

## Por qué importa el encapsulamiento

Sin encapsulamiento, cualquier parte del programa puede modificar el estado de cualquier objeto sin pasar por sus reglas de validación. Esto lleva a estados inconsistentes que son difíciles de rastrear.

```python
# ❌ Sin encapsulamiento: el contador puede quedar en un estado inválido
class MaquinaSinEncapsulamiento:
    def __init__(self, nombre: str) -> None:
        self.nombre = nombre
        self.contador_piezas = 0  # público, cualquiera puede modificarlo

maquina_rota = MaquinaSinEncapsulamiento("Torno-01")
maquina_rota.contador_piezas = -999  # Python no se queja, pero el estado es inválido
print(maquina_rota.contador_piezas)  # -999 ← estado inconsistente
```

```python
# ✅ Con encapsulamiento: las reglas de negocio se aplican siempre
class MaquinaSegura:
    """
    Máquina con contadores protegidos por encapsulamiento.

    Args:
        nombre: Nombre o código de la máquina.
    """

    def __init__(self, nombre: str) -> None:
        self.nombre = nombre
        self._estado = "inactiva"
        self._contador_piezas: int = 0

    @property
    def contador_piezas(self) -> int:
        """Contador de piezas producidas (solo lectura)."""
        return self._contador_piezas

    def activar(self) -> None:
        """Pone la máquina en estado activo."""
        self._estado = "activa"

    def iniciar_ciclo(self) -> None:
        """Ejecuta un ciclo de producción. Valida el estado antes de ejecutar."""
        if self._estado != "activa":
            raise RuntimeError(f"La máquina '{self.nombre}' debe estar activa para producir")
        self._contador_piezas += 1

    def __repr__(self) -> str:
        return f"MaquinaSegura({self.nombre!r}, piezas={self._contador_piezas})"
```

## Propiedades con `@property`

El decorador `@property` es la forma idiomática de Python para implementar getters y setters con validación, manteniendo la sintaxis de acceso a atributos (`objeto.atributo`) sin exponer el estado directamente.

La gran ventaja es que podés empezar con un atributo público simple y, si más adelante necesitás agregar validación, convertirlo en una propiedad sin cambiar el código que lo consume.

```python
class Maquina:
    """
    Máquina de producción con sensor de temperatura validado.

    Args:
        nombre: Nombre o código de la máquina.
    """

    TEMP_MINIMA_OPERATIVA = 0.0    # constante de clase

    def __init__(self, nombre: str) -> None:
        self.nombre = nombre
        self._estado = "inactiva"
        self._contador_piezas: int = 0
        # Usamos el setter desde el inicio para validar desde la construcción
        self.temperatura = self.TEMP_MINIMA_OPERATIVA

    @property
    def temperatura(self) -> float:
        """Temperatura del sensor en °C."""
        return self.__sensor_temperatura

    @temperatura.setter
    def temperatura(self, valor: float) -> None:
        """Establece la temperatura con validación de rango operativo."""
        if valor < self.TEMP_MINIMA_OPERATIVA:
            raise ValueError(
                f"Temperatura inválida: {valor}°C es menor al mínimo operativo "
                f"({self.TEMP_MINIMA_OPERATIVA}°C)"
            )
        self.__sensor_temperatura = valor

    @property
    def contador_piezas(self) -> int:
        """Contador de piezas producidas (solo lectura)."""
        return self._contador_piezas

    def activar(self) -> None:
        self._estado = "activa"

    def iniciar_ciclo(self) -> None:
        if self._estado != "activa":
            raise RuntimeError(f"La máquina '{self.nombre}' no está activa")
        self._contador_piezas += 1

    def obtener_reporte(self) -> dict:
        return {
            "nombre": self.nombre,
            "estado": self._estado,
            "temperatura": self.__sensor_temperatura,
            "piezas": self._contador_piezas,
        }

    def __repr__(self) -> str:
        return f"Maquina({self.nombre!r})"


# Uso
maquina = Maquina("Soldadora-01")
maquina.temperatura = 45.0     # ✅ setter con validación
print(maquina.temperatura)     # 45.0

maquina.activar()
maquina.iniciar_ciclo()
print(maquina.obtener_reporte())
# {'nombre': 'Soldadora-01', 'estado': 'activa', 'temperatura': 45.0, 'piezas': 1}

try:
    maquina.temperatura = -10.0  # ❌ ValueError: menor al mínimo operativo
except ValueError as e:
    print(e)

# Una propiedad de solo lectura no tiene setter
try:
    maquina.contador_piezas = 100  # ❌ AttributeError
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

## Ver también

El encapsulamiento controla el acceso al estado. Pero a veces no alcanza con controlar el acceso: también queremos restringir los valores válidos que ese estado puede tomar:

- [Enumeraciones (Enum)](enums.md) — cómo definir conjuntos cerrados de valores válidos para un atributo
- [Los Cuatro Elementos del Modelo](elementos.md) — cómo el encapsulamiento encaja en el modelo completo de Booch

---
