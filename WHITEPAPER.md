# Whitepaper — Workspace Python

**Versão:** 1.0
**Data:** 2026-07-09
**Autor:** Eduardo Felizardo Cândido
**Cargo:** Senior QA Automation Engineer | AI-driven Testing | Robot
**Repositório:** `C:\dev\Workspace Python`

---

## Sumário

1. [Visão Geral](#1-visão-geral)
2. [Problema e Motivação](#2-problema-e-motivação)
3. [Arquitetura do Workspace](#3-arquitetura-do-workspace)
4. [Stack Tecnológica](#4-stack-tecnológica)
5. [Padrões de Qualidade](#5-padrões-de-qualidade)
6. [Estrutura de Projetos](#6-estrutura-de-projetos)
7. [Fluxo de Trabalho](#7-fluxo-de-trabalho)
8. [Decisões de Design](#8-decisões-de-design)
9. [Evolução e Extensibilidade](#9-evolução-e-extensibilidade)

---

## 1. Visão Geral

O **Workspace Python** é um ambiente de desenvolvimento multi-projeto estruturado como monorepo, baseado na stack moderna do ecossistema Python. Ele foi projetado para ser o ponto de partida padronizado para qualquer tipo de projeto Python — APIs, CLIs, automações, scripts e integrações — sem foco em um domínio específico como Data Science ou Machine Learning.

O workspace utiliza **uv workspaces** como mecanismo de orquestração, garantindo que todos os projetos compartilhem um único ambiente virtual, um único arquivo de lock e configurações de qualidade unificadas, ao mesmo tempo que mantém dependências e configurações de testes isoladas por projeto.

---

## 2. Problema e Motivação

### 2.1 Cenário sem um workspace padronizado

Sem uma base estruturada, cada novo projeto Python tende a acumular inconsistências:

- Versões diferentes de Python entre projetos
- Ambientes virtuais criados de formas distintas (`venv`, `virtualenv`, `conda`, `pyenv`)
- Ferramentas de lint e formatação configuradas de forma diferente ou ausentes
- Ausência de verificação de tipos
- Testes sem cobertura mensurada
- Nenhuma automação de qualidade no ciclo de commit

O resultado é um conjunto de projetos difíceis de manter, com comportamentos imprevisíveis entre máquinas e colaboradores.

### 2.2 Motivação para o monorepo com uv

A abordagem de monorepo com uv workspaces resolve esses problemas de forma elegante:

| Problema | Solução adotada |
|---|---|
| Versões Python inconsistentes | Python 3.12 fixo, declarado em cada `pyproject.toml` |
| Múltiplos gerenciadores de ambiente | uv como único gerenciador |
| Lock files fragmentados | Um único `uv.lock` para todo o workspace |
| Ferramentas de qualidade ausentes | Ruff e BasedPyright configurados na raiz |
| Commits sem validação | pre-commit com hooks automáticos |
| Projetos com dependências conflitantes | Cada projeto declara suas próprias dependências |

---

## 3. Arquitetura do Workspace

### 3.1 Estrutura de diretórios

```
Workspace Python/                   ← repositório git do workspace
│
├── pyproject.toml              ← workspace root virtual (sem código próprio)
├── uv.lock                     ← lock file único e determinístico (commitado)
├── .venv/                      ← ambiente virtual compartilhado (não commitado)
│
├── projetos/                   ← diretório commitado
│   ├── .gitkeep                ← mantém o diretório rastreado pelo git
│   └── <nome-projeto>/         ← NÃO commitado no workspace
│       ├── pyproject.toml      ← metadados, dependências e configs do projeto
│       ├── src/
│       │   └── <nome>/         ← pacote Python (src layout)
│       │       └── __init__.py
│       └── tests/
│           └── test_<nome>.py
│
├── .pre-commit-config.yaml     ← hooks de qualidade compartilhados
├── .env.example                ← template de variáveis de ambiente
├── .gitignore
│
├── .vscode/
│   ├── settings.json           ← formatação automática, interpretador, type checker
│   └── extensions.json         ← extensões recomendadas
│
├── README.md
├── REQUIREMENTS.md
├── WHITEPAPER.md               ← este documento
└── CLAUDE.md                   ← instruções para o Claude Code
```

### 3.2 Modelo de responsabilidades

```
┌─────────────────────────────────────────────┐
│              pyproject.toml (raiz)           │
│  • declara membros do workspace              │
│  • configurações compartilhadas:             │
│    Ruff, BasedPyright                        │
└──────────────┬──────────────────────────────┘
               │ members = ["projetos/*"]
       ┌───────┴───────┐
       │               │
┌──────▼──────┐  ┌──────▼──────┐
│  projeto-A  │  │  projeto-B  │  ...
│             │  │             │
│ pyproject   │  │ pyproject   │
│ • name      │  │ • name      │
│ • deps      │  │ • deps      │
│ • pytest    │  │ • pytest    │
│ • coverage  │  │ • coverage  │
└─────────────┘  └─────────────┘
       │               │
       └───────┬───────┘
               │ resolve conjunto
        ┌──────▼──────┐
        │  uv.lock    │  ← versões exatas de todas as deps
        └──────┬──────┘
               │ instala em
        ┌──────▼──────┐
        │    .venv    │  ← ambiente virtual único
        └─────────────┘
```

---

## 4. Stack Tecnológica

### 4.1 Runtime

**Python 3.12**

A versão 3.12 foi escolhida por ser a versão de suporte ativo mais recente com o maior conjunto de melhorias de performance (até 5% mais rápido que 3.11) e qualidade de mensagens de erro. Todos os projetos declaram `requires-python = ">=3.12"` para garantir consistência.

---

### 4.2 Gerenciador de pacotes e ambientes

**uv** (Astral) — substitui pip + virtualenv + pip-tools

uv é escrito em Rust e resolve dependências em ordem de magnitude mais rápido que pip. É o único gerenciador de pacotes e ambientes utilizados neste workspace.

| Operação | pip + venv | uv |
|---|---|---|
| Criar venv | `python -m venv .venv` | `uv venv` (implícito no sync) |
| Instalar deps | `pip install -r requirements.txt` | `uv sync` |
| Adicionar pacote | editar requirements, pip install | `uv add <pkg>` |
| Lock file | pip-tools ou manualmente | automático via `uv.lock` |
| Workspaces | não suportado | `[tool.uv.workspace]` |

O mecanismo de **uv workspaces** é o núcleo desta arquitetura: um único `uv sync` na raiz resolve e instala as dependências de todos os projetos membros simultaneamente.

---

### 4.3 Linter e Formatter

**Ruff** — substitui flake8 + isort + black

Ruff implementa as mesmas regras de flake8, isort e black em um único binário escrito em Rust, sendo 10-100x mais rápido que as ferramentas originais.

Regras ativas neste workspace:

| Código | Grupo | Exemplos de verificações |
|---|---|---|
| `E` | pycodestyle errors | indentação, espaçamento, comprimento de linha |
| `F` | pyflakes | variáveis não usadas, imports não usados |
| `I` | isort | ordenação de imports |
| `UP` | pyupgrade | modernização de sintaxe Python |
| `B` | flake8-bugbear | padrões propensos a bugs |
| `SIM` | flake8-simplify | simplificação de código |

---

### 4.4 Verificação de tipos

**BasedPyright** — fork do Pyright com diagnósticos mais rigorosos

BasedPyright adiciona ao Pyright original: modo `strict` verdadeiro, detecção de código morto, e diagnósticos extras para tipos `Unknown`. Configurado em modo `standard` (equilibrado entre rigor e produtividade).

---

### 4.5 Testes

**pytest** + **pytest-cov**

pytest é o framework de testes padrão do ecossistema Python moderno. Cada projeto configura seus próprios `testpaths`, garantindo que `uv run pytest` dentro de `projetos/<nome>/` execute apenas os testes daquele projeto.

A cobertura é gerada automaticamente em cada execução:
- Terminal: `--cov-report=term-missing` (linhas não cobertas visíveis)
- HTML: `--cov-report=html` (relatório navegável em `htmlcov/`)

---

### 4.6 Automação de qualidade

**pre-commit**

Hooks executados automaticamente a cada `git commit`, impedindo que código que não passa nas validações entre no repositório.

```
git commit
     │
     ▼
┌─────────────────────────────┐
│  pre-commit hooks           │
│  1. ruff (lint + fix)       │
│  2. ruff-format             │
│  3. trailing-whitespace     │
│  4. end-of-file-fixer       │
│  5. check-yaml / check-toml │
│  6. check-merge-conflict    │
│  7. debug-statements        │
└──────────┬──────────────────┘
           │ todos passam
           ▼
        commit criado
```

---

### 4.7 Utilitários de desenvolvimento

| Ferramenta | Papel |
|---|---|
| **IPython** | REPL avançado para exploração interativa de código |
| **python-dotenv** | Carrega variáveis de ambiente de `.env` sem expô-las no código |
| **Rich** | Output formatado no terminal (tabelas, logs coloridos, progress bars) |
| **HTTPX** | Cliente HTTP moderno com suporte nativo a async e type hints |
| **Pydantic** | Validação de dados e definição de modelos com type hints |
| **Typer** | Criação de CLIs a partir de funções Python tipadas |

---

## 5. Padrões de Qualidade

### 5.1 Formatação

- Comprimento máximo de linha: **88 caracteres**
- Aspas: **duplas** (`"`)
- Indentação: **espaços** (4 por nível)
- Imports: ordenados por grupo (stdlib → terceiros → locais), separados por linha em branco

### 5.2 Tipagem

- Type hints obrigatórios em funções públicas
- Modo `standard` do BasedPyright (verifica anotações presentes, não exige anotação total)
- `Unknown` types são reportados como aviso, não erro

### 5.3 Testes

- Cobertura mínima recomendada: **80%** por projeto
- Testes nomeados como `test_<funcionalidade>.py`
- Fixtures compartilhadas em `tests/conftest.py`

### 5.4 Commits

- Mensagens no padrão **Conventional Commits**: `tipo: descrição`
- Tipos aceitos: `feat`, `fix`, `refactor`, `docs`, `chore`, `test`, `style`
- Nenhum commit passa sem os hooks de pre-commit

---

## 6. Estrutura de Projetos

### 6.1 Src layout

Todos os projetos adotam **src layout**: o código-fonte fica em `src/<nome_do_pacote>/`, não diretamente na raiz do projeto.

**Motivação:** O src layout evita que o diretório raiz do projeto seja implicitamente adicionado ao `sys.path`, forçando a instalação correta do pacote e evitando importações acidentais de código não instalado.

```
projetos/meu-projeto/
├── src/
│   └── meu_projeto/        ← pacote instalável
│       ├── __init__.py
│       └── ...
└── tests/
    └── test_meu_projeto.py ← importa de meu_projeto, não de ../src
```

### 6.2 Build backend

Cada projeto usa **hatchling** como build backend, declarado em `[build-system]`. Isso permite que uv instale o projeto em modo editável (`.venv`) sem depender de setuptools legado.

### 6.3 Dependências por projeto

Cada projeto declara apenas as dependências de que efetivamente precisa. Dependências compartilhadas entre projetos são resolvidas pelo uv no mesmo `.venv`, sem duplicação.

### 6.4 Versionamento dos projetos

O diretório `projetos/` é rastreado pelo git do workspace via `.gitkeep`, mas seu conteúdo é ignorado pelo `.gitignore`:

```
projetos/*
!projetos/.gitkeep
```

Cada projeto dentro de `projetos/<nome>/` tem ciclo de versionamento independente — pode ter seu próprio repositório git, sem qualquer acoplamento ao repositório do workspace. O workspace apenas orquestra os projetos localmente via uv workspaces.

---

## 7. Fluxo de Trabalho

### 7.1 Setup de máquina nova

```bash
# 1. instalar Python 3.12 e uv
pip install uv

# 2. instalar ferramentas globais
uv tool install ruff
uv tool install basedpyright
uv tool install pre-commit
uv tool install ipython

# 3. clonar e configurar o workspace
git clone <url> "Workspace Python"
cd "Workspace Python"
uv sync
pre-commit install
```

### 7.2 Criar um novo projeto

```bash
# 1. criar estrutura
mkdir projetos\meu-projeto\src\meu_projeto
mkdir projetos\meu-projeto\tests

# 2. criar projetos\meu-projeto\pyproject.toml
#    (usar o template em README.md como base)

# 3. sincronizar o workspace
uv sync

# 4. verificar que o projeto foi detectado
uv run python -c "import meu_projeto"

# 5. (opcional) inicializar git próprio do projeto
cd projetos\meu-projeto
git init
git add .
git commit -m "chore: initial project setup"
```

### 7.3 Ciclo de desenvolvimento diário

```bash
# adicionar dependência
uv add httpx --package meu-projeto

# rodar testes
cd projetos/meu-projeto
uv run pytest

# lint manual
uv run ruff check .

# commit (hooks rodam automaticamente)
git add .
git commit -m "feat: adicionar endpoint de autenticação"
```

### 7.4 Exportar dependências para pip

O workspace não mantém um `requirements.txt` commitado, pois os projetos em `projetos/` não são commitados e suas dependências variam por instalação local.

Para gerar um export pip-compatível das dependências instaladas localmente:

```bash
uv export --format requirements-txt --no-hashes > requirements.txt
```

Este arquivo é de uso local e não deve ser commitado no workspace.

---

## 8. Decisões de Design

### 8.1 Por que uv e não Poetry ou PDM?

| Critério | uv | Poetry | PDM |
|---|---|---|---|
| Velocidade de resolução | Muito rápida (Rust) | Lenta (Python) | Moderada |
| Suporte a workspaces | Nativo | Não nativo | Parcial |
| Compatibilidade PEP 517/518 | Total | Parcial | Total |
| Maturidade | Crescente (Astral) | Alta | Moderada |
| Ferramentas globais (`tool install`) | Sim | Não | Não |

uv foi escolhido por ser o único gerenciador que combina velocidade, suporte nativo a workspaces e gestão de ferramentas globais em um único binário.

### 8.2 Por que Ruff e não black + flake8 + isort?

Ruff implementa as mesmas regras das três ferramentas em um único binário, com performance muito superior. A migração é transparente: a configuração de Ruff aceita os mesmos parâmetros de black (`line-length`) e isort (`known-first-party`).

### 8.3 Por que BasedPyright e não mypy?

mypy tem suporte limitado a plugins, é mais lento e apresenta falsos negativos em cenários com Pydantic e FastAPI. BasedPyright, baseado no mesmo motor do Pylance (VS Code), oferece diagnósticos mais precisos e integração nativa com o editor.

### 8.4 Por que src layout?

O src layout é a recomendação da Python Packaging Authority (PyPA) desde 2020. Elimina a ambiguidade de importação durante os testes e garante que o pacote testado é sempre o instalado, não o código-fonte direto do disco.

### 8.5 Por que monorepo e não repositórios separados?

| Cenário | Monorepo | Repositórios separados |
|---|---|---|
| Projetos relacionados | Melhor | Fragmentado |
| Configs compartilhadas (ruff, pyright) | Um lugar | Duplicadas em cada repo |
| Lock file único | Sim | Um por repo |
| Refatoração cross-projeto | Simples | Coordenação entre repos |
| Projetos totalmente independentes | Overhead | Mais limpo |

Para projetos que evoluem juntos e compartilham padrões, o monorepo com uv workspaces é a abordagem mais eficiente.

---

## 9. Evolução e Extensibilidade

### 9.1 Adicionar domínio específico

Este workspace é genérico por design. Para projetos de domínios específicos, adicione as dependências no `pyproject.toml` do projeto correspondente:

| Domínio | Dependências sugeridas |
|---|---|
| APIs REST | `fastapi`, `uvicorn` |
| Banco de dados | `sqlalchemy`, `alembic` |
| CLI avançada | `typer`, `rich` (já incluídos) |
| Tarefas assíncronas | `celery`, `redis` |
| Testes de integração | `httpx` (já incluído), `pytest-asyncio` |

### 9.2 Adicionar mais hooks pre-commit

Edite `.pre-commit-config.yaml` na raiz para adicionar novos hooks sem afetar os projetos individualmente.

### 9.3 CI/CD

O `requirements.txt` gerado por `uv export` garante compatibilidade com pipelines que não suportam uv nativamente. Para pipelines com suporte a uv, use diretamente:

```yaml
- run: uv sync
- run: uv run pytest
```

### 9.4 Dependências compartilhadas entre projetos

Se dois projetos precisam compartilhar código, crie um terceiro projeto (ex: `projetos/shared/`) e referencie-o como dependência local:

```toml
# em projetos/meu-projeto/pyproject.toml
dependencies = [
    "shared",
]

[tool.uv.sources]
shared = { workspace = true }
```

---

*Documento gerado em 2026-07-09. Atualize a versão e a data a cada revisão significativa da arquitetura.*
