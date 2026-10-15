<!-- LOVABLE:BEGIN -->
> [!IMPORTANT]
> This project is connected to [Lovable](https://lovable.dev). Avoid rewriting
> published git history — force pushing, or rebasing/amending/squashing commits
> that are already pushed — as it rewrites history on Lovable's side and the
> user will likely lose their project history.
>
> Commits you push to the connected branch sync back to Lovable and show up in
> the editor, so keep the branch in a working state.
<!-- LOVABLE:END -->

# AGENTS.md

You are a coding agent working on **ghidra-mcp**, a Model Context Protocol server that bridges Ghidra's reverse engineering capabilities with AI tools.

## Project Context

- **Version**: 7.0.0
- **Language**: Java (Ghidra extension) + Python (MCP bridge) + TypeScript/React
- **Key feature**: 209 MCP tools for binary analysis, knowledge database, BSim integration, headless server support, AI documentation workflows

## Directory Structure

- `src/`:
  - `src/main/java/`, `src/test/java/`, `src/assembly/` — Extensão Ghidra e servidor headless (Java)
  - `src/routes/`, `src/components/`, `src/hooks/`, `src/lib/` — Web UI (TanStack Start / React 19)
- `python/bridge_mcp_ghidra/` — Pacote Python do MCP bridge (`bridge-mcp-ghidra` console script / `python -m bridge_mcp_ghidra`)
- `.opencode/` — Configurações do OpenCode, especificações de workflow e monitor (`monitor.md`, `Plan.md`, `TOOLS.md`, `project1.md`, `prd-project1.md`, `roadmap-proj.md`)
- `runtime-skills/` — Runtime skills para sandbox, observabilidade e monitor
- `opencode.json` — Configuração do servidor OpenCode e permissões de ferramentas
- `pyproject.toml` + `uv.lock` — Projeto uv (gera a wheel `ghidra-mcp-bridge`; dependências via grupos PEP 735)
- `package.json` + `bun.lock` — Dependências e scripts do Dev-server Web
- `docs/` — Documentação e prompts de workflow
- `tests/` — Testes unitários, offline e de integração em Python, e o catálogo de endpoints (`tests/endpoints.json`); testes Java sob `src/test/java/`
- `CHANGELOG.md` — Histórico de versões

## Current Priorities

1. Keep the GUI plugin and headless server on one service set (`CoreServices`) and one program model (`ProjectProgramProvider`); the catalog's `servers` field records the deliberate differences
2. Keep `tests/endpoints.json` in sync with Java endpoint registrations
3. Maintain CI/CD pipeline health
4. Community PR reviews

## Guidelines

- Run tests before committing: `pytest tests/unit/ -v --no-cov`
- Build with Gradle: `./gradlew buildExtension -PGHIDRA_INSTALL_DIR=<ghidra-install>`
- Follow existing code style
- Update CHANGELOG.md for user-facing changes
- Create PRs for review (don't push directly to main)
- Use `python -m tools.setup bump-version --new X.Y.Z` to bump version across all maintained files atomically

## Web Dev-Server

- **Stack**: TanStack Start (React 19), Vite, Tailwind CSS v4, Radix UI.
- **Rotas**: TanStack Start com roteamento baseado em arquivos em `src/routes/`. Não editar `src/routeTree.gen.ts` manualmente.
- **Componentes UI**: Primitivos reutilizáveis baseados em Radix UI em `src/components/ui/`.
- **Hooks de Browser**: Hooks que leem browser APIs (ex.: áudio, mídia, storage) devem derivar suporte em `useEffect`, nunca em `typeof window` no render — SSR/cliente divergem e quebram a hidratação.
- **Padrões TypeScript**: `tsconfig.json` usa `noPropertyAccessFromIndexSignature` e `exactOptionalPropertyTypes`: usar acesso por colchetes em `Record<string, unknown>` e spread condicional para props opcionais.
- **Verificação Web**:
  - Dev server: `bun run dev` (ou `npm run dev`)
  - Testes: `bunx vitest run` (ou `npm run test`)
  - Build: `bun run build` (ou `npm run build`)
  - Lint: `bun run lint` (ou `npm run lint`)

## Remix & Project Monitor

- **Ativação de Remix**: Um remix é ativado quando se recebe o comando:
  `"I've successfully remixed this project. What would be a good next step?"`
- **Project Monitor e Plan Obrigatórios**: Para iniciar um remix é obrigatório usar o **project monitor** e seguir rigorosamente as regras de [.opencode/Plan.md](file:///.opencode/Plan.md) para evitar bugs e erros no projeto.
- **Protocolo dos 3 Arquivos** ([.opencode/Plan.md](file:///.opencode/Plan.md)):
  - O trabalho agentico de remix é guiado por `project{N}` (Planejamento), `prd-project{N}` (PRD) e `roadmap-proj{N}` (Roadmap) em `.opencode/`.
  - **Precedência e Comunicação**: Nunca pergunte qualquer coisa ao usuário sem ter relatado a falta ou status dos 3 arquivos para o `project-monitor` antes! Informe o agente do monitor, espere a confirmação dele de recebimento e só depois informe o usuário (não use `ask`).
  - Nunca confunda informar o monitor do projeto com fazer uma ask para o usuário (utilizar a skill dedicada para isso).
  - Nunca use o OpenCode para fazer tarefas de implementação de um remix.
- **OpenCode Runtime & Proxy**:
  - OpenCode opera como servidor web local (`opencode serve --port 4096 --hostname 127.0.0.1`), configurado via `opencode.json`.
  - O proxy do dev-server Vite (`vite-opencode-proxy.ts`) expõe o runtime OpenCode na rota `/oc` (oculta nas rotas do app, só utilizar o runtime OpenCode).

## Commands

**Gradle is the default backend for local work.** It reads Ghidra's jars
straight out of the installation, so there is no `install-file` step, and it is
the only backend that works without Maven. **CI still builds and gates with
Maven**, so Maven is a maintained peer, not a fallback — see `CLAUDE.md`'s
"Build & Deploy" for the three things that are Maven-only: regenerating
`tests/endpoints.json`, the JaCoCo coverage gate, and the `headless`/`docker`
build profiles.

In Git Bash use a **forward-slash** Ghidra path; a backslash path is mangled
before Gradle sees it and produces ~100 misleading "package does not exist"
errors.

- Build: `./gradlew buildExtension "-PGHIDRA_INSTALL_DIR=F:/ghidra_12.1.4_PUBLIC"`
- Quick compile: `./gradlew compileJava "-PGHIDRA_INSTALL_DIR=F:/ghidra_12.1.4_PUBLIC"`
- Test (Python): `pytest tests/unit/ -v --no-cov`
- Test (Java, offline): `./gradlew test --tests 'com.xebyte.offline.*' "-PGHIDRA_INSTALL_DIR=F:/ghidra_12.1.4_PUBLIC"`
- Test (Java, all): `./gradlew test "-PGHIDRA_INSTALL_DIR=F:/ghidra_12.1.4_PUBLIC"`
- Preflight: `python -m tools.setup preflight --ghidra-path F:\ghidra_12.1.4_PUBLIC`
- Deploy: `./gradlew buildExtension "-PGHIDRA_INSTALL_DIR=F:/ghidra_12.1.4_PUBLIC"` then `python -m tools.setup deploy --ghidra-path F:\ghidra_12.1.4_PUBLIC` — leave `TOOLS_SETUP_BACKEND` **unset** for this second step, because the Gradle backend's `deploy` runs no post-deploy test tier and refuses `--test`
- Version bump: `python -m tools.setup bump-version --new X.Y.Z`
- Maven equivalents (peer backend): `python -m tools.setup ensure-prereqs --ghidra-path <dir>` then `python -m tools.setup build`
