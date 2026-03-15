# Programación Orientada a Objetos en Python

> Material de referencia para la materia Estructuras de Datos y Programación — ITBA
> Prerequisitos: conocimientos básicos de Python (variables, funciones, estructuras de control).

---

## ¿Qué es un paradigma de programación?

Un paradigma de programación es una manera de organizar y estructurar el código para resolver problemas. Define qué abstracciones tenés disponibles, cómo pensás el diseño y qué estilo de soluciones producís. No es solo una cuestión de sintaxis: es una forma de razonar sobre los problemas.

En la práctica, cuando los alumnos llegan a esta materia ya programan de forma estructurada sin saberlo. Cada vez que escribiste una función en Python para resolver un problema, estabas usando el paradigma estructurado. El objetivo de esta materia es agregar una nueva forma de pensar a esa base que ya tenés.

Python es multiparadigma: soporta el paradigma estructurado, el orientado a objetos y el funcional en el mismo lenguaje. Eso es una ventaja enorme — podés elegir el paradigma que mejor se adapta al problema que estás resolviendo, y en proyectos reales solemos combinarlos.

> **Prerequisitos:** Esta guía asume que ya sabés Python básico — variables, funciones, `if`, `for` y estructuras de datos como listas y diccionarios. No se requiere experiencia previa con POO.

---

## El Paradigma Estructurado

En la materia Informática General, usamos Python para desarrollar nuestros progamas aplicando el paradigma estructurado. Vamos a definirlo para entener que es lo que hicimos hasta ahora y luego como se diferencia del paradigma orientado a objetos.

La programación estructurada es un paradigma orientado a mejorar la claridad, calidad y tiempo de desarrollo de un programa, recurriendo únicamente a funciones y a tres estructuras de control básicas: secuencia, selección (`if` y `switch`) e iteración (bucles `for` y `while`).

![Estructuras de control fundamentales: secuencia, selección e iteración](img/espagueti.png)

Se considera innecesario y contraproducente el uso de la transferencia incondicional (`break` y `continue` abusivos); esta instrucción suele acabar generando el llamado código espagueti, mucho más difícil de seguir y mantener.

En la programación estructurada, los bloques fundamentales de construcción son **algoritmos** que definimos en funciones:

```python
def factorial(n: int) -> int | str:
    if n < 0:
        return "El factorial no está definido para números negativos."
    elif n == 0:
        return 1
    else:
        result = 1
        for i in range(1, n + 1):
            result *= i
        return result

# Ejemplo de uso:
print(f"El factorial de 5 es: {factorial(5)}")
print(f"El factorial de 0 es: {factorial(0)}")
print(f"El factorial de -3 es: {factorial(-3)}")
```

### Código Espagueti

Es un término peyorativo para los programas que tienen una estructura de control de flujo compleja e incomprensible. Su nombre deriva del hecho de que este tipo de código parece asemejarse a un plato de espaguetis: un montón de hilos intrincados y anudados.

```python
# ❌ Ejemplo de código espagueti: validación con flags anidados
def procesar_pedido(usuario, items, descuento):
    ok = False
    total = 0
    if usuario:
        if len(items) > 0:
            for item in items:
                if item['stock'] > 0:
                    total += item['precio']
                    ok = True
                else:
                    ok = False
                    break
            if ok:
                if descuento > 0:
                    total = total - (total * descuento / 100)
    return total if ok else -1
```

---

## Acoplamiento y Cohesión

Antes de entrar de lleno en el paradigma orientado a objetos, es fundamental entender dos conceptos que van a guiar todas tus decisiones de diseño: **acoplamiento** y **cohesión**. No son exclusivos de la POO — aplican a cualquier código — pero en cuanto empezás a diseñar clases, se vuelven el criterio más práctico para evaluar si tu diseño es bueno o no.

¿Por qué estudiarlos ahora, antes de ver clases en detalle? Porque el paradigma estructurado ya te enseña a escribir funciones, y con funciones también podés tener alto o bajo acoplamiento, alta o baja cohesión. Entender el problema en un contexto que ya conocés te va a hacer mucho más fácil aplicarlo después al diseño orientado a objetos. En la práctica, la mayoría de los problemas de diseño que vemos en los trabajos prácticos se reducen a uno de estos dos conceptos mal aplicados.

### Acoplamiento

El acoplamiento es el grado en que las clases de un programa dependen unas de otras. Si para hacer cambios en una clase es necesario hacer cambios en otra, existe acoplamiento entre ambas.

En POO, si una clase `X` usa una clase `Y`, se dice que `X` depende de `Y`: no puede realizar su trabajo sin `Y`. El acoplamiento es direccional: puede haber acoplamiento de `X` con `Y` sin que exista en sentido inverso.

```python
# ❌ ALTO ACOPLAMIENTO: OrderProcessor crea y controla FileManager

class FileManager:
    def __init__(self) -> None:
        self.file_path = "./order_log.txt"

    def save_data(self, data: str) -> bool:
        with open(self.file_path, 'a') as f:
            f.write(f"{data}\n")
        return True


class OrderProcessor:
    def __init__(self) -> None:
        self.file_manager = FileManager()  # ❌ dependencia hardcodeada

    def process_order(self, order_details: str) -> bool:
        return self.file_manager.save_data(f"Pedido: {order_details}")


# ✅ BAJO ACOPLAMIENTO: FileManager se inyecta desde afuera

class OrderProcessorV2:
    def __init__(self, file_manager: FileManager) -> None:
        self.file_manager = file_manager  # ✅ inyección de dependencia

    def process_order(self, order_details: str) -> bool:
        return self.file_manager.save_data(f"Pedido: {order_details}")

# Ahora es fácil cambiar el gestor de archivos sin tocar OrderProcessorV2
```

### Cohesión

Cohesión es lo contrario a acoplamiento. Algo está cohesionado si tiene sentido y una dirección común.

En ingeniería del software, algo tiene **alta cohesión** si tiene un alcance definido, límites claros y contenido delimitado y perfectamente ubicado.

En POO, una clase tendrá alta cohesión si sus métodos están relacionados entre sí, tienen contenido claro y temática común. Todo bien encerrado dentro de la clase y perfectamente delimitado.

```python
# ❌ BAJA COHESIÓN: UserManager hace demasiado
class UserManager:
    def create_user(self, name: str): ...
    def send_email(self, user, msg: str): ...    # ¿por qué gestión de usuarios envía emails?
    def generate_report(self): ...              # ¿y también genera reportes?
    def connect_to_db(self): ...                # ¿y maneja la conexión a BD?


# ✅ ALTA COHESIÓN: cada clase tiene una responsabilidad clara
class UserRepository:
    def create(self, name: str): ...
    def find_by_id(self, user_id: int): ...

class EmailService:
    def send(self, recipient: str, message: str): ...

class ReportGenerator:
    def generate_user_report(self, users: list): ...
```

Un código altamente cohesionado tiende a ser más autocontenido, con menos dependencias y más fácil de mantener.

---

### ¿Qué es lo ideal?

Teniendo en cuenta todo lo expuesto, hay que buscar el balance que garantiza mayor simplicidad y coherencia. La tendencia que deberíamos buscar es movernos siempre hacia **bajo acoplamiento y alta cohesión**.

![Cuadrante de acoplamiento y cohesión: el objetivo es bajo acoplamiento y alta cohesión](img/acoplamiento_cohesion_cuadrante.png)

Un bajo acoplamiento nos garantiza:

- Mejorar la mantenibilidad de los módulos del software, facilitar los cambios sin tener que revisar todos los módulos dependientes.
- Mejorar la reutilización de las unidades del software.
- Facilitar las pruebas de cada módulo, al ser más independientes.

Por otro lado, una alta cohesión nos permite:

- Tener un código más entendible, legible y coherente.
- Mejorar la reutilización, al tener todo lo relacionado con una cosa, en esa cosa.
- Mejorar el mantenimiento del software, ya que todo está perfectamente localizado.
- Facilitar las pruebas de caja negra.

---

## El Paradigma Orientado a Objetos

### Programación Orientada a Objetos

Grady Booch, uno de los referentes del campo, la define como *"una forma de programación en la cual los programas son organizados como una colección de objetos que cooperan, cada uno de los cuales es una instancia de alguna clase, y las clases son miembros de una jerarquía de clases relacionadas por relaciones de herencia."*

El cambio fundamental respecto al paradigma estructurado es dónde ponés el foco: en lugar de pensar en **verbos** (funciones, acciones), pensás en **sustantivos** (objetos, entidades). En lugar de preguntarte *"¿qué pasos necesito para resolver esto?"*, te preguntás *"¿qué entidades existen en este dominio y cómo se relacionan entre sí?"*

En la práctica, este cambio de mentalidad es el mayor desafío del curso. El código procedural te da resultados rápido; el diseño orientado a objetos te da código que escala, que es mantenible y que otros pueden entender sin leer cada línea.

| | Paradigma Estructurado | Paradigma OOP |
| --- | --- | --- |
| Unidad fundamental | Función/Procedimiento | Objeto |
| Dato y comportamiento | Separados | Unidos en la misma entidad |
| Reuso | Copy-paste, funciones genéricas | Herencia, Composición |
| Complejidad | Crece con el tamaño del programa | Se gestiona con encapsulación |

### Análisis Orientado a Objetos (AOO)

El análisis orientado a objetos es un método de análisis que examina los requisitos desde la perspectiva de las clases y los objetos que se encuentran en el vocabulario del dominio del problema. Consiste en analizar los requisitos observando el problema en términos de objetos.

Estos objetos representan entidades físicas o abstractas del mundo real relevantes para el dominio. El objetivo consiste en identificar los objetos, sus atributos, comportamientos y relaciones, sin enfocarse en cómo serán implementados.

### Técnica práctica: User Story Mapping

El AOO puede sonar abstracto al principio: ¿cómo sabés qué objetos identificar en un sistema que todavía no existe? Una técnica accesible para empezar es el **User Story Mapping**, creada por Jeff Patton. No requiere conocimientos técnicos previos y es útil precisamente porque te obliga a pensar desde el usuario, no desde el código.

Un **user story** (historia de usuario) describe una funcionalidad desde la perspectiva de quien la usa, con el formato:

> *"Como [tipo de usuario], quiero [acción], para [objetivo o beneficio]."*

Por ejemplo: *"Como cliente, quiero agregar productos al carrito, para poder comprarlos juntos."*

El **User Story Mapping** organiza esas historias en un mapa bidimensional:

- **Eje horizontal (izquierda a derecha):** el flujo de uso del sistema, ordenado como lo haría un usuario real. Primero busca productos, después los agrega al carrito, después paga, etc.
- **Eje vertical (arriba a abajo):** el nivel de detalle y prioridad. Las historias más importantes van arriba; las variantes y casos especiales, abajo.

#### Pasos para construir un User Story Map

1. **Identificar a los actores:** ¿Quiénes usan el sistema? (cliente, administrador, vendedor…). Cada actor tiene sus propias necesidades.
2. **Describir las actividades principales:** ¿Qué tareas grandes hace cada actor? Por ejemplo: *buscar*, *comprar*, *gestionar stock*. Estas van en la fila superior del mapa.
3. **Desglosar en historias concretas:** Para cada actividad, escribir las acciones específicas que la componen. *"Buscar por categoría"*, *"filtrar por precio"*, *"ver detalle del producto"* son historias que componen la actividad *buscar*.
4. **Priorizar:** Marcar cuáles historias forman el flujo mínimo que necesitás para que el sistema funcione (el llamado *walking skeleton*). Esto define qué construir primero.
5. **Identificar los objetos:** Revisá todas las historias y subrayá los **sustantivos** (cliente, carrito, producto, pedido). Esos son los candidatos a clases de tu modelo.

#### ¿Por qué usar esta técnica?

El valor del User Story Mapping en este contexto no es que sea la técnica definitiva de análisis —en proyectos reales se usa mucho más formal— sino que te da una forma concreta de pasar de *"tengo un problema"* a *"tengo una lista de objetos candidatos"* sin necesitar experiencia previa en diseño de software. En la práctica, hacer este ejercicio antes de escribir cualquier clase evita el error más común: modelar objetos que nadie usa o, peor, no modelar los que realmente importan.

### Diseño Orientado a Objetos (DOO)

El diseño orientado a objetos es un método de diseño de software que abarca el proceso de descomposición orientada a objetos y una notación para representar modelos lógicos y físicos del sistema en diseño.

Durante el DOO se toma el modelo de análisis obtenido en el AOO y se transforma en un modelo detallado de implementación.

![La pirámide del diseño orientado a objetos: desde el diseño de subsistemas hasta las responsabilidades de cada clase](img/poo_diagrama.png)

### Beneficios del ADOO

- Aumenta la modularidad y la facilidad de mantenimiento del software al fomentar la creación de componentes pequeños y reutilizables.
- Proporciona una representación abstracta de alto nivel del sistema.
- Promueve la reutilización de objetos, lo que reduce el código necesario y mejora la calidad.
- Facilita el trabajo en equipo con un lenguaje y método comunes.
- Ayuda a crear sistemas escalables que se adapten a las cambiantes necesidades.

---

## Elementos del Modelo de Objetos

Este modelo consta de cuatro elementos principales:

1. Abstracción
2. Encapsulación
3. Modularidad
4. Jerarquía

### Abstracción

La abstracción es la capacidad de identificar las características esenciales de un objeto e ignorar los detalles irrelevantes para el contexto. Según Booch, *"una abstracción denota las características esenciales de un objeto que lo distinguen de todos los demás tipos de objetos y, por lo tanto, proporcionan límites conceptuales claramente definidos en relación con la perspectiva del observador."*

![La abstracción se centra en las características esenciales de un objeto según la perspectiva del observador](img/abstraccion.png)

Un ejemplo cotidiano: cuando manejás un auto, usás el volante, el acelerador y los frenos sin necesitar saber nada sobre cómo funciona el motor internamente. El auto te expone una interfaz simple y oculta la complejidad del mecanismo. En POO hacemos exactamente lo mismo con las clases: exponemos lo necesario y ocultamos lo demás.

En Python, la abstracción formal se implementa con clases abstractas del módulo `abc`. Una clase abstracta define *qué* operaciones debe ofrecer un objeto, sin especificar *cómo* las implementa cada subclase concreta.

```python
from abc import ABC, abstractmethod

class Figura(ABC):
    """Abstracción de una figura geométrica."""

    @abstractmethod
    def area(self) -> float:
        """Calcula el área de la figura."""
        ...

    @abstractmethod
    def perimetro(self) -> float:
        """Calcula el perímetro de la figura."""
        ...

    def describir(self) -> str:
        return f"Área: {self.area():.2f}, Perímetro: {self.perimetro():.2f}"


class Circulo(Figura):
    def __init__(self, radio: float) -> None:
        self.radio = radio

    def area(self) -> float:
        import math
        return math.pi * self.radio ** 2

    def perimetro(self) -> float:
        import math
        return 2 * math.pi * self.radio


class Rectangulo(Figura):
    def __init__(self, ancho: float, alto: float) -> None:
        self.ancho = ancho
        self.alto = alto

    def area(self) -> float:
        return self.ancho * self.alto

    def perimetro(self) -> float:
        return 2 * (self.ancho + self.alto)


# Polimorfismo: misma interfaz, distinto comportamiento
figuras: list[Figura] = [Circulo(5), Rectangulo(4, 6)]
for figura in figuras:
    print(figura.describir())
```

### Encapsulamiento

El encapsulamiento es el mecanismo que permite proteger los datos internos de un objeto y controlar cómo se accede o modifica su estado. La idea central es que el estado interno de un objeto no debería ser accesible directamente desde afuera: el objeto es el único responsable de mantener su propia consistencia.

La abstracción y el encapsulamiento son conceptos complementarios. La abstracción define *qué* puede hacer un objeto (su contrato externo); el encapsulamiento protege *cómo* lo hace (su implementación interna).

![El encapsulamiento oculta los detalles de implementación de un objeto](img/encapsulamiento.png)

#### Convenciones en Python

Python no tiene modificadores de acceso como `private` o `protected`, pero tiene convenciones que todos los desarrolladores respetan:

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

### Modularidad

El desarrollador debe equilibrar dos preocupaciones técnicas contrapuestas: el deseo de encapsular abstracciones y la necesidad de que ciertas abstracciones sean visibles para otros módulos.

![La modularidad empaqueta abstracciones en unidades discretas e independientes](img/modularidad.png)

La modularidad es la propiedad de un sistema que se ha descompuesto en un conjunto de módulos **cohesivos** y **débilmente acoplados**. En Python, cada archivo `.py` es un módulo. Los principios de abstracción, encapsulación y modularidad son sinérgicos.

El siguiente ejemplo muestra tres clases que en un sistema real vivirían en archivos separados (`productos.py`, `inventario.py`, `reporte.py`). Cada módulo tiene una única responsabilidad y sus dependencias son mínimas:

```python
# --- módulo: productos.py ---
class Producto:
    """Encapsula los datos y reglas de negocio de un producto."""

    def __init__(self, nombre: str, precio: float) -> None:
        if precio < 0:
            raise ValueError("El precio no puede ser negativo")
        self.nombre = nombre
        self._precio = precio

    @property
    def precio(self) -> float:
        return self._precio

    def __repr__(self) -> str:
        return f"Producto({self.nombre!r}, ${self._precio:.2f})"


# --- módulo: inventario.py ---
class Inventario:
    """Gestiona una colección de productos. Solo depende de Producto."""

    def __init__(self) -> None:
        self._items: list[Producto] = []

    def agregar(self, producto: Producto) -> None:
        self._items.append(producto)

    def buscar(self, nombre: str) -> Producto | None:
        return next((p for p in self._items if p.nombre == nombre), None)

    def total_stock(self) -> int:
        return len(self._items)


# --- módulo: reporte.py ---
class ReporteInventario:
    """Genera reportes. Cohesivo: solo sabe de reportes, nada más."""

    def generar(self, inventario: Inventario) -> str:
        lineas = [f"=== Reporte de Inventario ===",
                  f"Total de productos: {inventario.total_stock()}"]
        return "\n".join(lineas)


# Uso
inv = Inventario()
inv.agregar(Producto("Laptop", 1500))
inv.agregar(Producto("Monitor", 400))

reporte = ReporteInventario()
print(reporte.generar(inv))
```

### Jerarquía

La abstracción es buena, pero en cualquier sistema real aparecen decenas de abstracciones que necesitan relacionarse entre sí. La jerarquía es la forma en que organizamos esas relaciones de manera que el sistema siga siendo comprensible.

Las dos jerarquías fundamentales en POO son:

- **Estructura de clases** (jerarquía "es-un"): una subclase *es un* tipo específico de su clase padre. Se implementa con **herencia**.
- **Estructura de objetos** (jerarquía "tiene-un"): un objeto *contiene* o *usa* a otro. Se implementa con **composición**.

Elegir mal entre estas dos relaciones es uno de los errores más comunes en el diseño orientado a objetos. La regla es directa: si podés decir con naturalidad que A *es un* B, usá herencia. Si la relación es A *tiene un* B o A *usa un* B, usá composición. En la práctica, el abuso de herencia genera jerarquías rígidas y difíciles de cambiar — la composición suele ser la opción más flexible.

```python
# ── Jerarquía "es-un" (herencia) ────────────────────────────────────────────

class Vehiculo:
    """Un Vehiculo es la abstracción base."""

    def __init__(self, marca: str) -> None:
        self.marca = marca

    def describir(self) -> str:
        return f"Vehículo: {self.marca}"


class Moto(Vehiculo):
    """Una Moto ES UN Vehículo. ✅ herencia correcta."""

    def __init__(self, marca: str, cilindrada: int) -> None:
        super().__init__(marca)
        self.cilindrada = cilindrada

    def describir(self) -> str:
        return f"Moto: {self.marca} ({self.cilindrada}cc)"


# ── Jerarquía "tiene-un" (composición) ──────────────────────────────────────

class Motor:
    """Componente reutilizable e independiente."""

    def __init__(self, cilindros: int) -> None:
        self.cilindros = cilindros

    def encender(self) -> str:
        return f"Motor de {self.cilindros} cilindros encendido"


class Auto:
    """Un Auto TIENE UN Motor. ✅ composición correcta.
    Un Auto no 'es un' Motor, por eso no hereda de él.
    """

    def __init__(self, marca: str, cilindros: int) -> None:
        self.marca = marca
        self._motor = Motor(cilindros)  # composición

    def arrancar(self) -> str:
        return f"{self.marca}: {self._motor.encender()}"


# Demostración
moto = Moto("Honda", 600)
auto = Auto("Toyota", 4)

print(moto.describir())   # herencia: Moto es un Vehículo
print(auto.arrancar())    # composición: Auto tiene un Motor
```

---

## Principios SOLID

### S — Single Responsibility Principle (Principio de Responsabilidad Única)

Una clase debe tener una, y solo una, razón para cambiar.

Si tenés una clase `Usuario` que gestiona datos del usuario, guarda en base de datos y además envía emails de bienvenida, está haciendo demasiado. Separar responsabilidades: `Usuario` (datos), `UsuarioRepository` (base de datos), `EmailService` (envíos).

```python
# ❌ Viola SRP: la clase hace demasiado
class Invoice:
    def calculate_total(self): ...
    def print_invoice(self): ...    # responsabilidad de presentación
    def save_to_db(self): ...       # responsabilidad de persistencia

# ✅ Respeta SRP: cada clase tiene una razón para cambiar
class Invoice:
    def calculate_total(self) -> float: ...

class InvoicePrinter:
    def print(self, invoice: Invoice) -> None: ...

class InvoiceRepository:
    def save(self, invoice: Invoice) -> None: ...
```

### O — Open/Closed Principle (Principio de Abierto/Cerrado)

Las clases deben estar abiertas para su extensión, pero cerradas para su modificación.

Si querés agregar una nueva funcionalidad, no deberías tener que entrar a modificar el código que ya funciona y arriesgarte a romperlo. Usá polimorfismo: añadí nuevas clases en lugar de agregar `if/else` infinitos.

```python
# ❌ Viola OCP: hay que modificar la clase para agregar un nuevo formato
class ReportExporter:
    def export(self, report, format: str):
        if format == "pdf":
            ...
        elif format == "csv":
            ...
        # Cada nuevo formato requiere modificar esta clase

# ✅ Respeta OCP: se extiende sin modificar
from abc import ABC, abstractmethod

class ReportExporter(ABC):
    @abstractmethod
    def export(self, report) -> None: ...

class PDFExporter(ReportExporter):
    def export(self, report) -> None: ...

class CSVExporter(ReportExporter):
    def export(self, report) -> None: ...
```

### L — Liskov Substitution Principle (Principio de Sustitución de Liskov)

Las clases derivadas deben poder sustituirse por sus clases base sin alterar el comportamiento correcto del programa.

Si tenés una clase `Pato` y una hija `PatoDeHule`, y el programa explota porque `PatoDeHule` no puede volar, estás violando este principio. Asegurate de que la herencia tenga sentido semántico y funcional.

```python
# ❌ Viola LSP: PatoDeHule no respeta el contrato de la clase base
class Animal:
    def volar(self) -> str:
        return "Volando..."

class Pato(Animal):
    def volar(self) -> str:
        return "El pato vuela"

class PatoDeHule(Animal):
    def volar(self) -> str:
        raise NotImplementedError("¡Un pato de hule no puede volar!")
        # ❌ Quien use Animal espera poder llamar volar() sin excepciones


# ✅ Respeta LSP: la jerarquía refleja capacidades reales
class Animal:
    """Clase base: solo comportamiento común a todos los animales."""

    def __init__(self, nombre: str) -> None:
        self.nombre = nombre

    def respirar(self) -> str:
        return f"{self.nombre} respira"


class AnimalVolador(Animal):
    """Extensión para animales que pueden volar."""

    def volar(self) -> str:
        return f"{self.nombre} vuela"


class Pato(AnimalVolador):
    def volar(self) -> str:
        return f"{self.nombre} vuela con sus alas"


class PatoDeHule(Animal):
    """No hereda de AnimalVolador porque no puede volar. ✅"""

    def chillar(self) -> str:
        return f"{self.nombre}: ¡Squeak!"


# Ahora podemos substituir Animal por cualquier subclase sin sorpresas
animales: list[Animal] = [Pato("Donald"), PatoDeHule("Rubber")]
for a in animales:
    print(a.respirar())  # ✅ funciona para todos
```

### I — Interface Segregation Principle (Principio de Segregación de Interfaz)

Ninguna clase debería verse forzada a depender de métodos que no utiliza.

Aunque Python no tiene "interfaces" explícitas, este principio es vital cuando diseñamos Clases Base Abstractas (ABCs) o definimos Protocolos. El objetivo es evitar crear una clase base gigante que obligue a sus hijas a implementar métodos que no tienen sentido para ellas.

```python
from abc import ABC, abstractmethod

# ❌ Viola ISP: una ABC enorme que fuerza a Robot a implementar
#    métodos que no tiene sentido para él
class Trabajador(ABC):
    @abstractmethod
    def trabajar(self) -> None: ...

    @abstractmethod
    def comer(self) -> None: ...

    @abstractmethod
    def dormir(self) -> None: ...


class Robot(Trabajador):
    def trabajar(self) -> None:
        print("Robot trabajando")

    def comer(self) -> None:
        raise NotImplementedError("Los robots no comen")  # ❌ forzado

    def dormir(self) -> None:
        raise NotImplementedError("Los robots no duermen")  # ❌ forzado


# ✅ Respeta ISP: interfaces pequeñas y específicas
class Trabajable(ABC):
    @abstractmethod
    def trabajar(self) -> None: ...


class Comedor(ABC):
    @abstractmethod
    def comer(self) -> None: ...


class Dormidor(ABC):
    @abstractmethod
    def dormir(self) -> None: ...


class Humano(Trabajable, Comedor, Dormidor):
    """Implementa todo porque un humano hace todo."""

    def trabajar(self) -> None:
        print("Humano trabajando")

    def comer(self) -> None:
        print("Humano comiendo")

    def dormir(self) -> None:
        print("Humano durmiendo")


class Robot(Trabajable):
    """Solo implementa lo que necesita. ✅"""

    def trabajar(self) -> None:
        print("Robot trabajando")
```

### D — Dependency Inversion Principle (Principio de Inversión de Dependencias)

Los módulos de alto nivel no deben depender de módulos de bajo nivel. Ambos deben depender de abstracciones.

```python
# ❌ Viola DIP: depende de implementación concreta
class Notification:
    def __init__(self):
        self.email = EmailSender()  # dependencia de bajo nivel

    def send(self, msg: str):
        self.email.send(msg)


# ✅ Respeta DIP: depende de abstracción
class MessageSender(ABC):
    @abstractmethod
    def send(self, message: str) -> None: ...

class Notification:
    def __init__(self, sender: MessageSender) -> None:
        self.sender = sender  # inyección de dependencia

    def send(self, message: str) -> None:
        self.sender.send(message)
```

---

## Excepciones

Las excepciones son el mecanismo que usa Python para manejar situaciones anómalas durante la ejecución. En el paradigma orientado a objetos, las excepciones son **objetos**: instancias de clases que forman una jerarquía de herencia.

Lejos de ser un recurso de "último momento", el manejo de excepciones es parte integral del **diseño**. Bertrand Meyer lo formalizó en el concepto de *Design by Contract*: cada método tiene precondiciones (qué espera recibir) y postcondiciones (qué garantiza producir). Las excepciones son el instrumento para comunicar que ese contrato fue violado.

> **En la práctica:** en proyectos reales, las excepciones mal diseñadas son una fuente constante de bugs difíciles de rastrear. Una excepción genérica como `Exception("algo salió mal")` no le dice nada a quien la captura. Una excepción bien diseñada como `StockInsuficienteError` es autodocumentada y le permite a quien llama tomar decisiones informadas.

### Excepciones Built-in Principales

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

### Estructura del manejo de excepciones

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

### Uso de excepciones built-in en una clase de dominio

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

### Excepciones Personalizadas

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

## Testing de Clases con pytest

Un programa sin tests es un programa que no sabés si funciona. En POO, los tests tienen un rol especial: **verifican el contrato público de una clase**. Si cambiás la implementación interna sin romper los tests, sabés que el comportamiento externo se preservó.

### ¿Qué es un test unitario?

Un **test unitario** testea **una sola unidad** de código en aislamiento. En POO, la unidad es una clase. El objetivo es verificar que sus métodos se comportan correctamente dado un estado inicial y unos inputs determinados.

Lo que **no** hace un test unitario:

- No testea cómo dos clases interactúan entre sí (eso es un test de integración).
- No testea la base de datos, la red ni el sistema de archivos.
- No testea lógica de otras clases relacionadas.

Si tu clase `Inventario` usa una clase `Producto`, en el test de `Inventario` vas a **simular** el comportamiento de `Producto` con un *mock*, sin usar la clase real.

### pytest: El Framework Estándar

`pytest` es el framework de testing más usado en el ecosistema Python. Es más conciso que `unittest` y tiene un sistema de descubrimiento automático de tests.

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

- Los archivos de test deben llamarse `test_*.py` o `*_test.py`.
- Las funciones (o métodos de clase) de test deben llamarse `test_*`.
- pytest los descubre y ejecuta automáticamente.

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

pytest usa el `assert` nativo de Python. Cuando una aserción falla, pytest muestra exactamente qué valores tenía cada lado:

```python
assert cuenta.saldo == 1500    # si falla: AssertionError: assert 1200 == 1500
assert resultado is not None
assert "error" in mensaje.lower()
```

### Tests básicos con clases

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

### Mocking: Aislar la Clase Bajo Test

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

> **Nota sobre `spec=`**: usar `MagicMock(spec=MiClase)` hace que el mock solo permita llamar a los métodos que `MiClase` realmente tiene. Esto evita que los mocks "aprueben" código que llama a métodos con nombres incorrectos.

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

## Checklist: Antes de Entregar

Un buen diseño orientado a objetos no es solo que el código funcione — es que sea mantenible, legible y correcto por diseño. Usá esta lista como guía antes de entregar cualquier trabajo práctico.

### Código Limpio

Antes de revisar ítem por ítem, vale la pena recordar los criterios generales de código limpio que aplican a todo el trabajo:

- **Es obvio para otros programadores.** Los nombres de variables pobremente elegidos, clases y métodos extensos, y números mágicos hacen del código algo desprolijo y difícil de comprender.
- **No tiene líneas duplicadas.** Cada vez que hacés un cambio en código duplicado tenés que repetirlo en cada copia. Esto induce a errores y dificulta la mantenibilidad.
- **Tiene la menor cantidad de clases necesarias.** Menos código facilita el mantenimiento y reduce la cantidad de bugs. El diseño debe ser el mínimo para el propósito.
- **Pasa todos los tests.** Hay que probar nuestro código y asegurarse de que cada entrega satisface todos los tests.

### Diseño y Responsabilidad

- [ ] Cada clase tiene **una sola responsabilidad** clara (SRP)
- [ ] Ninguna clase "mete mano" en los datos internos de otra: si el Objeto A tiene una lista, el Objeto B llama a un método de A para modificarla, nunca hace `objeto_a.lista.append(...)` directamente
- [ ] Se usó **composición** en lugar de herencia cuando la relación no es "es-un" (ej. un `Auto` *tiene* un `Motor`, no *es* un `Motor`)
- [ ] La lógica de negocio está distribuida entre las clases que corresponden, no concentrada en una sola

### Código Python

- [ ] Todos los métodos y constructores tienen **type hints** explícitos
- [ ] Cada clase y método público tiene un **docstring** que explica qué hace
- [ ] Los atributos privados usan `_` (protegido por convención) o `__` (name mangling) según el nivel de restricción deseado
- [ ] No hay lógica duplicada — si el mismo cálculo aparece en dos lugares, es una señal de que falta un método o una clase
- [ ] Se usan funciones nativas de Python donde corresponde (`sum()`, `sorted()`, list comprehensions) en lugar de loops manuales para tareas comunes

### Validación y Excepciones

- [ ] Cada objeto valida sus propios datos de entrada en el `__init__`
- [ ] Los métodos que modifican estado validan las precondiciones antes de ejecutar
- [ ] Las excepciones lanzadas son **descriptivas**: el mensaje explica qué salió mal y con qué valor
- [ ] Se crearon excepciones personalizadas para errores propios del dominio (heredando de una excepción base del módulo)

### Testing

- [ ] Hay tests para el comportamiento normal (el "happy path")
- [ ] Hay tests para los casos de error (valores inválidos, estado incorrecto)
- [ ] Los tests verifican el **comportamiento observable** (qué devuelven los métodos), no la implementación interna
- [ ] Los tests son independientes entre sí — ninguno depende del resultado de otro

---

## Fuentes

- Booch, G. *Object Oriented Analysis and Design with Applications*, 3rd Edition.
- Martin, R. C. *Clean Code: A Handbook of Agile Software Craftsmanship*.
- <https://profile.es/blog/que-son-los-paradigmas-de-programacion/>
- <https://es.wikipedia.org/wiki/Programaci%C3%B3n_estructurada>
- <https://www.disrupciontecnologica.com/acoplamiento-y-cohesion/>
- <https://refactoring.guru/refactoring/what-is-refactoring>
- <https://www.geeksforgeeks.org/software-engineering/object-oriented-analysis-and-design/>
