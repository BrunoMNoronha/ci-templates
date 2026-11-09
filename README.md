# ci-templates

Workflows reutilizáveis de CI/CD dos projetos de BrunoMNoronha. Padronizam custo, cache e segurança dos GitHub Actions. As regras estão em [CONTRIBUTING.md](CONTRIBUTING.md).

Consumidores fixam a major: `@v1`. Enquanto a `v1` não existir, use `@main`.

## Workflows

### `node-ci.yml` — CI de projeto Node.js (pnpm)

Install com lockfile congelado, Prisma opcional, PostgreSQL efêmero opcional, format/typecheck/lint/testes/build e scripts extras. Runner `ubuntu-24.04`, `timeout-minutes` 15, cache do store do pnpm e do `.next/cache`. Sem acesso a secrets. Exemplo: [`examples/ci.yml`](examples/ci.yml).

Principais inputs: `node-version-file` (recomendado, ex.: `package.json`), `pnpm-version` (vazio = `packageManager`), `postgres` (+`postgres-image`, +`postgres-db`), `prisma`, `format-check`, `lint`/`typecheck`/`test`/`build`, `extra-scripts` (um script por linha), `next-cache`, `working-directory`, `env`.

### `vercel-deploy.yml` — deploy na Vercel

`vercel pull/build/deploy --prebuilt` consumindo um SHA já validado, com migração opcional. Não reexecuta a CI. Exemplos: [`examples/deploy-preview.yml`](examples/deploy-preview.yml), [`examples/deploy-production.yml`](examples/deploy-production.yml).

Inputs: `environment`, `ref`, `prod`, `run-migrations` (+`migrate-command`), `vercel-version`, `node-version-file`. Secrets: `VERCEL_TOKEN`, `VERCEL_ORG_ID`, `VERCEL_PROJECT_ID`, `MIGRATION_DATABASE_URL` (opcional). Saída: `url`.

### `docker-build.yml` — build de imagem com cache

Buildx com cache de camadas `type=gha`. Sem push por padrão (valida que a imagem compila). Exemplo: [`examples/docker.yml`](examples/docker.yml).

Inputs: `context`, `dockerfile`, `target`, `image`, `tags`, `build-args`, `platforms`, `push`, `registry`. Secrets (só com `push: true`): `REGISTRY_USERNAME`, `REGISTRY_PASSWORD`.

### `dotnet-ci.yml` — CI de projeto .NET

Restore/build/testes em `ubuntu-24.04`, SDK fixado por `global.json`, cache de NuGet opcional. Windows só com justificativa (`runs-on: windows-2025`).

Inputs: `global-json-file`, `dotnet-version`, `solution`, `cache`, `test`, `shell`, `runs-on`.

## Nome do check ao migrar

Como reusable workflow, o check aparece como `<job do chamador> / <job-name>` (ex.: `ci / Lint, typecheck, testes e build`). Antes do merge, atualize os rulesets que exigem o check pelo nome.

## Autoteste

[`selftest.yml`](.github/workflows/selftest.yml) exercita `node-ci` e `docker-build` contra as fixtures em [`tests/`](tests) a cada PR.
