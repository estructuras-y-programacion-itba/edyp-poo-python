# Paradigmas de Programación

## ¿Qué es un paradigma de programación?

Un paradigma de programación es una manera de organizar y estructurar el código para resolver problemas. Define qué abstracciones tenés disponibles, cómo pensás el diseño y qué estilo de soluciones producís. No es solo una cuestión de sintaxis: es una forma de razonar sobre los problemas.

Quienes llegan a esta materia ya programan de forma estructurada sin saberlo. Cada vez que escribiste una función en Python para resolver un problema, estabas aplicando el paradigma estructurado. El objetivo es agregar una nueva forma de pensar a esa base que ya tenés.

Python es multiparadigma: soporta el estructurado, el orientado a objetos y el funcional dentro del mismo lenguaje. Esa flexibilidad es una ventaja concreta — podés elegir el paradigma que mejor se adapta al problema y, en proyectos reales, solemos combinarlos.

> **Prerequisitos:** Esta guía asume que ya sabés Python básico — variables, funciones, `if`, `for` y estructuras de datos como listas y diccionarios. No se requiere experiencia previa con POO.

---

## El Paradigma Estructurado

En la materia Informática General, usamos Python para desarrollar nuestros programas aplicando el paradigma estructurado. Conviene definirlo para entender qué hicimos hasta ahora y, a partir de eso, contrastar con el paradigma orientado a objetos.

La programación estructurada busca mejorar la claridad, calidad y tiempo de desarrollo de un programa, recurriendo únicamente a funciones y a tres estructuras de control básicas: secuencia, selección (`if` y `switch`) e iteración (bucles `for` y `while`).

![Estructuras de control fundamentales: secuencia, selección e iteración](../img/espagueti.png)

Dentro de este modelo, se considera innecesario y contraproducente el uso de transferencias incondicionales (`break` y `continue` abusivos); este tipo de instrucciones tiende a generar el llamado código espagueti, mucho más difícil de seguir y mantener.

El bloque fundamental de construcción son **algoritmos** encapsulados en funciones:

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

Término peyorativo para los programas con una estructura de control de flujo compleja e incomprensible. Su nombre alude a la similitud visual con un plato de fideos: una masa de hilos intrincados y anudados sin dirección aparente.

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

Grado en que los módulos de un programa dependen entre sí. Si para hacer cambios en una clase es necesario modificar otra, existe acoplamiento entre ambas.

En POO, cuando una clase `X` usa una clase `Y`, se dice que `X` depende de `Y`: no puede realizar su trabajo sin ella. Cabe notar que el acoplamiento es direccional — puede existir dependencia de `X` hacia `Y` sin que se dé en sentido inverso.

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

Mientras el acoplamiento mide la dependencia *entre* módulos, la cohesión mide qué tan relacionados están los elementos *dentro* de un módulo. Algo tiene **alta cohesión** si su alcance está bien definido, sus límites son claros y su contenido responde a una única responsabilidad.

En POO, una clase tendrá alta cohesión si sus métodos están relacionados entre sí, tienen contenido claro y temática común — todo perfectamente delimitado dentro de la clase.

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

Un código altamente cohesionado tiende a ser más autocontenido, con menos dependencias externas y más fácil de mantener.

---

### ¿Qué es lo ideal?

Considerando todo lo expuesto, el objetivo es alcanzar el balance que garantiza mayor simplicidad y coherencia. La dirección correcta es siempre hacia **bajo acoplamiento y alta cohesión**.

![Cuadrante de acoplamiento y cohesión: el objetivo es bajo acoplamiento y alta cohesión](../img/acoplamiento_cohesion_cuadrante.png)

Apuntar al bajo acoplamiento garantiza:

- Mejorar la mantenibilidad: los cambios en un módulo no arrastran modificaciones en otros.
- Favorecer la reutilización de las unidades del software.
- Facilitar las pruebas de cada módulo, al ser más independientes entre sí.

Por su parte, la alta cohesión permite:

- Tener un código más entendible, legible y coherente.
- Mejorar la reutilización, al concentrar todo lo relacionado con una responsabilidad en el mismo lugar.
- Mejorar el mantenimiento del software, dado que todo está perfectamente localizado.
- Facilitar las pruebas de caja negra.

---
