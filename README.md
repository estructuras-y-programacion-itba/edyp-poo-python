# Programación Orientada a Objetos en Python

Material de referencia para la materia Estructuras de Datos y Programación (EDyP) — ITBA.
Orientado a estudiantes de carreras no informáticas con conocimientos básicos de Python.

## Estructura del repositorio

```text
.
├── docs/                   # Fuentes del sitio MkDocs (Markdown)
│   ├── index.md
│   ├── introduccion/
│   ├── conceptos_basicos/
│   ├── principios/
│   ├── metodos/
│   ├── diseno/
│   └── buenas_practicas/
├── mkdocs.yml              # Configuración del sitio
├── pyproject.toml          # Dependencias del proyecto (uv)
├── uv.lock                 # Lockfile generado por uv
└── CLAUDE.md               # Instrucciones para el agente IA
```

---

## Instalación del entorno

Este proyecto usa [`uv`](https://docs.astral.sh/uv/) para gestionar dependencias.

### 1. Instalar `uv`

```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows (PowerShell)
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

### 2. Instalar dependencias

```bash
uv sync
```

Esto crea el entorno virtual en `.venv/` e instala todo lo declarado en `pyproject.toml`.

---

## Desarrollo

### Previsualizar el sitio localmente

```bash
uv run mkdocs serve
```

El sitio queda disponible en `http://127.0.0.1:8000`. Los cambios en `docs/` se
reflejan automáticamente sin necesidad de reiniciar.

### Construir el sitio estático

```bash
uv run mkdocs build
```

El sitio se genera en `site/`. Esta carpeta no se versiona (está en `.gitignore`).

### Verificar formato de los archivos Markdown

```bash
# Un archivo específico
npx markdownlint-cli2 docs/introduccion/primer_programa.md

# Todos los archivos
npx markdownlint-cli2 "docs/**/*.md"
```

### Agregar una dependencia nueva

```bash
uv add nombre-del-paquete
```

---

## Deploy

El sitio se despliega automáticamente a GitHub Pages mediante GitHub Actions en cada
push a `main`. El workflow está en `.github/workflows/deploy.yml`.

Para un deploy manual:

```bash
uv run mkdocs gh-deploy --force
```
