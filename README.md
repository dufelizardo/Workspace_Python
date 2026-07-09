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
│   └── <nome-projeto>/
│       ├── pyproject.toml      ← dependências e configs do projeto
│       ├── src/
│       │   └── <nome>/         ← código-fonte (src layout)
│       └── tests/
└── .vscode/
    ├── settings.json
    └── extensions.json
```

## Projetos

| Projeto | Descrição | Localização |
|---|---|---|
| exemplo | Projeto template | `projetos/exemplo/` |

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
#    (copiar de projetos\exemplo\ e ajustar name e dependencies)

# 3. sincronizar
uv sync
```

## VS Code

Abra a pasta raiz `Workspace Python/` no VS Code. Na primeira abertura, aceite a instalação das extensões recomendadas (`.vscode/extensions.json`). O interpretador Python já está apontado para `.venv/Scripts/python.exe`.
