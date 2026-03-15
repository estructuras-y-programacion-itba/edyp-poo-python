# Programación Orientada a Objetos en Python

Material de referencia para la materia de Programación Orientada a Objetos, orientado a estudiantes universitarios de carreras no informáticas con conocimientos previos de Python.

## Prerequisitos

- Python 3.12+
- Conocimientos básicos de Python (variables, funciones, estructuras de control)
- Sin experiencia previa en POO requerida

## Contenido

El material principal está en [`programacion_orientada_a_objetos.md`](programacion_orientada_a_objetos.md) e incluye:

1. Paradigmas de programación (estructurado vs. orientado a objetos)
2. Problemas de diseño: acoplamiento y cohesión
3. Código limpio y principios SOLID
4. El paradigma orientado a objetos (Booch, Meyer, Kay, Fowler, Martin)
5. Los cuatro pilares: abstracción, encapsulamiento, modularidad, jerarquía
6. Excepciones
7. Testing de clases con `pytest`

## Estructura del repositorio

```text
.
├── docs/                                  # Fuentes del sitio MkDocs
│   ├── index.md                           # Página de inicio (este README)
│   ├── programacion_orientada_a_objetos.md # Material principal
│   └── img/                               # Imágenes del material
├── examples/                              # Snippets ejecutables por capítulo
│   ├── 01_paradigma_estructurado.py
│   ├── 02_acoplamiento_cohesion.py
│   ├── 03_principios_solid.py
│   ├── 04_abstraccion.py
│   ├── 05_encapsulamiento.py
│   ├── 06_modularidad_jerarquia.py
│   ├── 07_excepciones.py
│   └── 08_testing_pytest.py
├── mkdocs.yml                             # Configuración de MkDocs
├── .github/workflows/deploy.yml           # GitHub Action para gh-pages
├── pyproject.toml                         # Dependencias del proyecto (uv)
├── uv.lock                                # Lockfile generado por uv
└── CLAUDE.md                              # Instrucciones para el agente IA
```

---

## Instalación del entorno con `uv`

Este proyecto usa [`uv`](https://docs.astral.sh/uv/) para gestionar dependencias y el entorno virtual de Python. `uv` es una herramienta moderna que reemplaza el flujo tradicional de `pip` + `venv`: es significativamente más rápida y maneja el entorno virtual de forma automática.

### 1. Instalar `uv`

```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows (PowerShell)
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Verificá la instalación:

```bash
uv self version
```

### 2. Clonar el repositorio e instalar dependencias

```bash
git clone <url-del-repo>
cd edyp-poo-python

# Crea el entorno virtual (.venv/) e instala todo lo declarado en pyproject.toml
uv sync
```

Eso es todo. `uv` crea automáticamente un entorno virtual en `.venv/` dentro del proyecto. No necesitás activarlo manualmente para correr comandos.

### ¿Por qué `uv` en lugar de `pip`?

| | `pip` + `venv` | `uv` |
| --- | --- | --- |
| Crear entorno virtual | `python -m venv .venv` | automático con `uv sync` |
| Instalar paquete | `pip install pytest` | `uv add pytest` |
| Instalar desde lockfile | `pip install -r requirements.txt` | `uv sync` |
| Velocidad | normal | 10–100× más rápido |
| Reproducibilidad | depende de `requirements.txt` manual | `uv.lock` generado automáticamente |

> **Nota:** `uv` guarda las dependencias en `pyproject.toml` y genera un `uv.lock` con las versiones exactas de cada paquete. Esto garantiza que todos los estudiantes instalen exactamente las mismas versiones.

### Agregar una dependencia nueva

```bash
uv add nombre-del-paquete
```

Esto actualiza `pyproject.toml` y `uv.lock` automáticamente.

---

## Cómo usar los ejemplos

Cada archivo en `examples/` corresponde a un capítulo del material y es ejecutable de forma independiente.

### Correr un ejemplo directamente

```bash
uv run python examples/01_paradigma_estructurado.py
uv run python examples/05_encapsulamiento.py
# etc.
```

`uv run` se encarga de usar el entorno virtual del proyecto sin que necesites activarlo.

### Correr los tests

El archivo `examples/08_testing_pytest.py` contiene tests unitarios diseñados para ejecutarse con `pytest`:

```bash
# Todos los tests
uv run pytest examples/08_testing_pytest.py -v

# Con output más detallado y traceback corto en caso de falla
uv run pytest examples/08_testing_pytest.py -v --tb=short
```

### Correr todos los ejemplos de una vez

```bash
for f in examples/0*.py; do
    echo "=== $f ==="; uv run python "$f"; echo
done
```

---

## Referencias

- Grady Booch — *Object-Oriented Analysis and Design with Applications*
- Bertrand Meyer — *Object-Oriented Software Construction*
- Alan Kay — Diseñador del lenguaje Smalltalk, padre del término "OOP"
- Martin Fowler — *Refactoring: Improving the Design of Existing Code*
- Robert C. Martin — *Clean Code* / *Agile Software Development (SOLID)*
