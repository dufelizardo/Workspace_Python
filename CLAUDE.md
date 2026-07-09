# CLAUDE.md — Workspace Python

Instruções para o Claude Code trabalhar neste workspace.

## O que é este workspace

Monorepo Python com **uv workspaces**. Cada projeto fica em `projetos/<nome>/` com seu próprio `pyproject.toml`. Um único `.venv` e `uv.lock` na raiz servem todos os projetos.

Não é um projeto de Data Science, IA ou Robot Framework — é um workspace genérico para APIs, CLIs, automações e integrações Python.

## Estrutura

```
Workspace Python/
├── pyproject.toml          ← workspace root (sem código próprio)
├── uv.lock                 ← não editar manualmente
├── .venv/                  ← nunca instalar pacotes diretamente aqui
├── projetos/
│   └── <nome>/
│       ├── pyproject.toml
│       ├── src/<nome>/     ← src layout obrigatório
│       └── tests/
├── .pre-commit-config.yaml
└── .vscode/
```

## Comandos essenciais

```bash
# instalar / sincronizar todas as dependências
uv sync

# adicionar pacote a um projeto específico
uv add <pacote> --package <nome-projeto>

# rodar testes de um projeto (executar dentro de projetos/<nome>/)
uv run pytest

# lint
uv run ruff check .

# formatar
uv run ruff format .

# regenerar requirements.txt após mudança de dependências
uv export --format requirements-txt --no-hashes > requirements.txt
```

## Regras obrigatórias

### Gerenciador de pacotes
- Usar **somente `uv`**. Nunca `pip install` dentro do projeto.
- `uv sync` na raiz para instalar tudo. `uv add` para adicionar pacotes.

### Estrutura de cada projeto
- Todo projeto usa **src layout**: código em `src/<nome_pacote>/`, não na raiz.
- Build backend: **hatchling** declarado em `[build-system]`.
- Cada projeto tem seu próprio `pyproject.toml` com `name`, `dependencies` e configurações de pytest/coverage.

### Qualidade de código
- **Ruff** é o único linter e formatter. Não sugerir black, flake8 ou isort.
- **BasedPyright** para análise de tipos. Não sugerir mypy.
- Type hints obrigatórios em funções públicas.
- Aspas duplas, 88 chars por linha, imports ordenados (regras E, F, I, UP, B, SIM ativas).

### Testes
- Framework: **pytest** com **pytest-cov**.
- Testes em `tests/`, nomeados `test_<funcionalidade>.py`.
- Fixtures compartilhadas em `tests/conftest.py`.
- Nunca mockar onde um teste real é viável.

### Commits
- Padrão Conventional Commits: `tipo: descrição`.
- Os hooks de pre-commit rodam automaticamente — não usar `--no-verify`.
- Se o hook corrigir arquivos, re-staged e re-commitar.

## Como criar um novo projeto

```bash
# 1. criar estrutura
mkdir projetos\meu-projeto\src\meu_projeto
mkdir projetos\meu-projeto\tests

# 2. criar projetos\meu-projeto\pyproject.toml
#    (basear em projetos\exemplo\pyproject.toml, ajustar name e deps)

# 3. sincronizar
uv sync
```

O `pyproject.toml` de cada projeto deve conter:
- `[project]` com `name`, `version`, `requires-python = ">=3.12"`, `dependencies`
- `[dependency-groups] dev` com pytest e pytest-cov
- `[build-system]` com hatchling
- `[tool.hatch.build.targets.wheel] packages = ["src/<nome>"]`
- `[tool.pytest.ini_options]` com `testpaths = ["tests"]` e addopts de cobertura
- `[tool.coverage.run]` com `source = ["src"]`

## Configs compartilhadas (raiz)

O `pyproject.toml` da raiz contém as configs de Ruff e BasedPyright que valem para todos os projetos. Não duplicar essas seções nos `pyproject.toml` dos projetos membros, a menos que seja uma sobrescrita intencional.

## Versionamento de projetos

O diretório `projetos/` é versionado (contém `.gitkeep`), mas **seu conteúdo não**.
Cada projeto dentro de `projetos/<nome>/` é ignorado pelo `.gitignore` via padrão:

```
projetos/*
!projetos/.gitkeep
```

Isso significa que cada projeto tem seu próprio ciclo de versionamento independente
(repositório Git próprio, se necessário). O workspace apenas organiza e orquestra os
projetos localmente via uv workspaces.

Nunca adicionar arquivos de projetos individuais ao git do workspace.

## O que não fazer

- Não criar `requirements.txt` manualmente — gerado por `uv export`.
- Não colocar código-fonte na raiz do workspace — somente dentro de `projetos/<nome>/src/`.
- Não criar ambientes virtuais separados por projeto — o `.venv` da raiz é compartilhado.
- Não usar `typer` como dependência de todos os projetos — adicionar somente nos projetos que criam CLIs.
- Não commitar `.env`, `.venv/`, `htmlcov/`, `.coverage`, `__pycache__/` — todos no `.gitignore`.
- Não commitar o conteúdo de `projetos/` — apenas o diretório em si (`.gitkeep`).
