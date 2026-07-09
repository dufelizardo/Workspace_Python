# Requirements

Documentação de todas as ferramentas e dependências do workspace, organizadas por camada.

---

## Ferramentas globais (via `uv tool`)

Instaladas uma vez na máquina. Disponíveis em qualquer terminal, fora do `.venv`.

| Ferramenta | Versão | Papel |
|---|---|---|
| **uv** | 0.11.28 | Gerenciador de venvs, pacotes e workspaces. Substitui pip + virtualenv + pip-tools |
| **Ruff** | 0.15.20 | Linter e formatter ultrarápido. Substitui flake8, isort e black |
| **BasedPyright** | 1.39.9 | Análise estática de tipos. Fork do Pyright com diagnósticos mais rigorosos |
| **pre-commit** | 4.6.0 | Executor de hooks Git. Garante qualidade antes de cada commit |
| **IPython** | 9.15.0 | REPL interativo com autocomplete, syntax highlighting e magic commands |

---

## Dependências de produção (projeto `exemplo`)

Adicionadas via `uv add <pacote> --package exemplo`.

| Pacote | Versão | Papel |
|---|---|---|
| **httpx** | 0.28.1 | Cliente HTTP moderno com suporte a async, HTTP/2 e type hints nativos |
| **pydantic** | 2.13.4 | Validação e serialização de dados com type hints. Base para modelos e schemas |
| **python-dotenv** | 1.2.2 | Carrega variáveis de ambiente de arquivos `.env` |
| **rich** | 15.0.0 | Saídas formatadas no terminal: tabelas, progress bars, syntax highlighting |

---

## Dependências de desenvolvimento (projeto `exemplo`)

Adicionadas via `uv add --dev <pacote> --package exemplo`.

| Pacote | Versão | Papel |
|---|---|---|
| **pytest** | 9.1.1 | Framework de testes. Suporte a fixtures, parametrize e plugins |
| **pytest-cov** | 7.1.0 | Plugin de cobertura de código integrado ao pytest |

---

## Hooks pre-commit

Configurados em `.pre-commit-config.yaml`. Executados automaticamente a cada `git commit`.

| Hook | Origem | Ação |
|---|---|---|
| `ruff` | astral-sh/ruff-pre-commit | Lint com correção automática |
| `ruff-format` | astral-sh/ruff-pre-commit | Formatação de código |
| `trailing-whitespace` | pre-commit/pre-commit-hooks | Remove espaços no final das linhas |
| `end-of-file-fixer` | pre-commit/pre-commit-hooks | Garante newline no final dos arquivos |
| `check-yaml` | pre-commit/pre-commit-hooks | Valida sintaxe de arquivos YAML |
| `check-toml` | pre-commit/pre-commit-hooks | Valida sintaxe de arquivos TOML |
| `check-merge-conflict` | pre-commit/pre-commit-hooks | Detecta marcadores de conflito não resolvidos |
| `debug-statements` | pre-commit/pre-commit-hooks | Bloqueia `breakpoint()`, `pdb`, `ipdb` commitados |

---

## Dependências transitivas

Gerenciadas automaticamente pelo uv. Não editar manualmente. Consultar `uv.lock` para versões exatas.

| Pacote | Motivo |
|---|---|
| annotated-types | requerido por pydantic |
| anyio | requerido por httpx |
| certifi | requerido por httpx e httpcore |
| colorama | requerido por pytest (Windows) |
| coverage | requerido por pytest-cov |
| h11 | requerido por httpcore |
| httpcore | requerido por httpx |
| idna | requerido por anyio e httpx |
| iniconfig | requerido por pytest |
| markdown-it-py | requerido por rich |
| mdurl | requerido por markdown-it-py |
| packaging | requerido por pytest |
| pluggy | requerido por pytest e pytest-cov |
| pydantic-core | requerido por pydantic |
| pygments | requerido por rich e pytest |
| typing-extensions | requerido por pydantic, anyio |
| typing-inspection | requerido por pydantic |
