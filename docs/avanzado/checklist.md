# Checklist

Esta es la última sección del material. Todos los conceptos — clases, métodos, encapsulamiento, relaciones, herencia, polimorfismo, abstracción, composición, excepciones, testing y principios SOLID — están ahora disponibles como herramientas. Esta checklist te ayuda a verificar que los estás aplicando correctamente antes de entregar cualquier trabajo práctico.

## Checklist: Antes de Entregar

Un buen diseño orientado a objetos no implica únicamente que el código funcione — implica que sea mantenible, legible y correcto por diseño. Usá esta lista como guía antes de entregar cualquier trabajo práctico.

### Código Limpio

Antes de revisar ítem por ítem, conviene tener presentes los criterios generales de código limpio que atraviesan todo el trabajo:

- **Resulta obvio para otros programadores.** Los nombres de variables pobremente elegidos, clases y métodos extensos, y números mágicos hacen del código algo desprolijo y difícil de comprender.
- **No contiene líneas duplicadas.** Cada vez que hacés un cambio en código duplicado tenés que repetirlo en cada copia. Esto induce a errores y dificulta la mantenibilidad.
- **Usa la menor cantidad de clases necesarias.** Menos código facilita el mantenimiento y reduce la cantidad de bugs. El diseño debe ser el mínimo para el propósito.
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
