# Paradigmas de Programación

## ¿Qué es un paradigma de programación?

Un paradigma de programación es una manera de organizar y estructurar el código para resolver problemas. Define qué abstracciones tenés disponibles, cómo pensás el diseño y qué estilo de soluciones producís. No es solo una cuestión de sintaxis: es una forma de razonar sobre los problemas.

Quienes llegan a esta materia ya programan de forma estructurada sin saberlo. Cada vez que escribiste una función en Python para resolver un problema, estabas aplicando el paradigma estructurado. El objetivo es agregar una nueva forma de pensar a esa base que ya tenés.

Python es multiparadigma: soporta el estructurado, el orientado a objetos y el funcional dentro del mismo lenguaje. Esa flexibilidad es una ventaja concreta — podés elegir el paradigma que mejor se adapta al problema y, en proyectos reales, solemos combinarlos.

---

## El Paradigma Estructurado

En la materia Informática General, usamos Python para desarrollar nuestros programas aplicando el paradigma estructurado. Conviene definirlo para entender qué hicimos hasta ahora y, a partir de eso, contrastar con el paradigma orientado a objetos.

La programación estructurada busca mejorar la claridad, calidad y tiempo de desarrollo de un programa, recurriendo únicamente a funciones y a tres estructuras de control básicas: secuencia, selección (`if` y `switch`) e iteración (bucles `for` y `while`).

![Estructuras de control fundamentales: secuencia, selección e iteración](../img/espagueti.png)

Dentro de este modelo, se considera innecesario y contraproducente el uso de transferencias incondicionales (`break` y `continue` abusivos); este tipo de instrucciones tiende a generar el llamado código espagueti, mucho más difícil de seguir y mantener.

El bloque fundamental de construcción son **algoritmos** encapsulados en funciones:

```python
def calcular_eficiencia(piezas_producidas: int, tiempo_total: float) -> float:
    """Calcula la eficiencia de una línea como piezas por hora."""
    if tiempo_total <= 0:
        return 0.0
    return piezas_producidas / tiempo_total * 60  # piezas por minuto × 60

# Ejemplo de uso:
print(f"Eficiencia: {calcular_eficiencia(120, 45):.2f} piezas/hora")
print(f"Eficiencia: {calcular_eficiencia(0, 10):.2f} piezas/hora")
```

### Código Espagueti

Término peyorativo para los programas con una estructura de control de flujo compleja e incomprensible. Su nombre alude a la similitud visual con un plato de fideos: una masa de hilos intrincados y anudados sin dirección aparente.

```python
# ❌ Ejemplo de código espagueti: control de planta con flags anidados
def procesar_turno(estaciones, piezas, temperatura):
    ok = False
    total = 0
    if estaciones:
        if len(piezas) > 0:
            for pieza in piezas:
                if pieza['material'] != 'defectuoso':
                    total += 1
                    ok = True
                else:
                    ok = False
                    break
            if ok:
                if temperatura > 0:
                    total = total - (total * temperatura / 100)
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
# ❌ ALTO ACOPLAMIENTO: ControladorLinea crea y controla EstacionCorteV1
class EstacionCorteV1:
    def __init__(self) -> None:
        self.velocidad = 100  # mm/min fija

    def cortar(self, pieza: str) -> str:
        return f"Cortando {pieza} a {self.velocidad} mm/min"


class ControladorLineaAcoplado:
    def __init__(self) -> None:
        self.estacion = EstacionCorteV1()  # ❌ dependencia hardcodeada

    def ejecutar(self, pieza: str) -> str:
        return self.estacion.cortar(pieza)


# ✅ BAJO ACOPLAMIENTO: la estación se inyecta desde afuera
class EstacionCorteV2:
    def __init__(self, velocidad: float) -> None:
        self.velocidad = velocidad

    def cortar(self, pieza: str) -> str:
        return f"Cortando {pieza} a {self.velocidad} mm/min"


class ControladorLineaDesacoplado:
    def __init__(self, estacion: EstacionCorteV2) -> None:
        self.estacion = estacion  # ✅ inyección de dependencia

    def ejecutar(self, pieza: str) -> str:
        return self.estacion.cortar(pieza)

# Ahora es fácil cambiar la estación sin tocar ControladorLineaDesacoplado
```

### Cohesión

Mientras el acoplamiento mide la dependencia *entre* módulos, la cohesión mide qué tan relacionados están los elementos *dentro* de un módulo. Algo tiene **alta cohesión** si su alcance está bien definido, sus límites son claros y su contenido responde a una única responsabilidad.

En POO, una clase tendrá alta cohesión si sus métodos están relacionados entre sí, tienen contenido claro y temática común — todo perfectamente delimitado dentro de la clase.

```python
# ❌ BAJA COHESIÓN: GestorPlanta hace demasiado
class GestorPlanta:
    def registrar_maquina(self, nombre: str): ...
    def enviar_alerta(self, operario, msg: str): ...    # ¿por qué gestión de planta envía alertas?
    def generar_reporte(self): ...                     # ¿y también genera reportes?
    def conectar_sensor(self, sensor_id: str): ...     # ¿y maneja sensores?


# ✅ ALTA COHESIÓN: cada clase tiene una responsabilidad clara
class RepositorioMaquinas:
    def registrar(self, nombre: str): ...
    def buscar_por_id(self, maquina_id: str): ...

class ServicioMantenimiento:
    def enviar_alerta(self, operario: str, mensaje: str): ...

class GeneradorReportes:
    def generar_reporte_turno(self, maquinas: list): ...
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
