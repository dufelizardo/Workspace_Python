# Workspace Python

Monorepo Python moderno baseado em **uv workspaces**. Cada projeto vive em `projetos/<nome>/` com suas próprias dependências, compartilhando um único `.venv` e `uv.lock` na raiz.

## Pré-requisitos

| Ferramenta | Versão mínima | Instalação |
|---|---|---|
| Python | 3.12 | [python.org](https://python.org) |
| uv | 0.11+ | `pip install uv` |
| Git | qualquer | [git-scm.com](https://git-scm.com) |

## Setup inicial

```bash
# clonar e entrar na pasta
git clone <url> "Workspace Python"
cd "Workspace Python"

# criar o venv e instalar todas as dependências
uv sync

# instalar os hooks de qualidade
pre-commit install
```

## Estrutura

```
Workspace Python/
├── pyproject.toml              ← workspace root: membros + configs compartilhadas
├── uv.lock                     ← lock único (commitar sempre)
├── .venv/                      ← venv compartilhado (não commitar)
├── .pre-commit-config.yaml     ← hooks de qualidade
├── .env.example                ← template de variáveis de ambiente
├── projetos/
│   ├── .gitkeep                ← mantém o diretório no git
│   └── <nome-projeto>/         ← NÃO commitado (ciclo próprio de versionamento)
│       ├── pyproject.toml
│       ├── src/
│       │   └── <nome>/         ← código-fonte (src layout)
│       └── tests/
└── .vscode/
    ├── settings.json
    └── extensions.json
```

> **Versionamento:** o diretório `projetos/` é rastreado pelo git do workspace, mas seu conteúdo não. Cada projeto dentro de `projetos/<nome>/` tem seu próprio repositório git independente.

## Comandos principais

### Workspace (rodar na raiz)

```bash
# sincronizar venv com todos os projetos
uv sync

# adicionar dependência a um projeto específico
uv add <pacote> --package <nome-projeto>

# remover dependência de um projeto
uv remove <pacote> --package <nome-projeto>

# rodar qualquer comando no contexto do workspace
uv run <comando>
```

### Por projeto (rodar dentro de `projetos/<nome>/`)

```bash
# rodar testes com cobertura
uv run pytest

# lint
uv run ruff check .

# formatar
uv run ruff format .

# verificação de tipos
basedpyright .
```

### Qualidade (hooks manuais)

```bash
# rodar todos os hooks em todos os arquivos
pre-commit run --all-files
```

## Adicionando um novo projeto

```bash
# 1. criar estrutura
mkdir projetos\meu-projeto\src\meu_projeto
mkdir projetos\meu-projeto\tests

# 2. criar projetos\meu-projeto\pyproject.toml
#    (ver template mínimo abaixo)

# 3. sincronizar o workspace
uv sync

# 4. (opcional) inicializar git próprio do projeto
cd projetos\meu-projeto
git init
```

**Template mínimo de `pyproject.toml`:**

```toml
[project]
name = "meu-projeto"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = []

[dependency-groups]
dev = ["pytest", "pytest-cov"]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/meu_projeto"]

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "--cov=src --cov-report=term-missing --cov-report=html"

[tool.coverage.run]
source = ["src"]
omit = ["tests/*"]
```

## VS Code

Abra a pasta raiz `Workspace Python/` no VS Code. Na primeira abertura, aceite a instalação das extensões recomendadas (`.vscode/extensions.json`). O interpretador Python já está apontado para `.venv/Scripts/python.exe`.

---

**Autor:** Eduardo Felizardo Cândido

**Cargo:** Senior QA Automation Engineer | AI-driven Testing | Robot
