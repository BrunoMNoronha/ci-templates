# ci-templates

Workflows reutilizáveis de CI dos projetos de BrunoMNoronha.

## `node-ci.yml`

CI para projetos Node.js com **pnpm** (npm não é usado): install com lockfile congelado, Prisma opcional, PostgreSQL efêmero opcional, typecheck, lint, testes e build.

Recursos:

- runner fixado (`ubuntu-24.04`) e `timeout-minutes` (padrão 15);
- cache do store do pnpm e do `.next/cache`;
- scripts ausentes no `package.json` são ignorados (`pnpm run --if-present`);
- sem acesso a secrets; o banco é um container de serviço descartável.

Exemplo completo em [`examples/ci.yml`](examples/ci.yml).

### Inputs

| Input | Padrão | Descrição |
|---|---|---|
| `job-name` | `Lint, typecheck, testes e build` | Nome exibido do job |
| `runs-on` | `ubuntu-24.04` | Runner |
| `timeout-minutes` | `15` | Tempo máximo do job |
| `working-directory` | `.` | Diretório do projeto |
| `node-version` | `24` | Versão do Node.js |
| `pnpm-version` | vazio | Vazio = campo `packageManager` do `package.json` |
| `postgres` | `false` | Sobe PostgreSQL efêmero e exporta `DATABASE_URL` |
| `postgres-image` | `postgres:16-alpine` | Imagem do PostgreSQL |
| `prisma` | `false` | `prisma validate` e, com `postgres`, `prisma migrate deploy` |
| `next-cache` | `true` | Cache do `.next/cache` |
| `lint` / `typecheck` / `test` / `build` | `true` | Liga/desliga cada etapa |
| `env` | vazio | Variáveis **não sensíveis** da CI, `CHAVE=valor` por linha |

## Migração de um repositório

O check passa a se chamar `<job do chamador> / <job-name>` (ex.: `ci / Lint, typecheck, testes e build`). Antes do merge, atualize:

1. os rulesets que exigem o check pelo nome;
2. qualquer workflow que consulte o check pelo nome (ex.: `migracoes.yml` da `upa-do-tenis`).

## Versionamento

Enquanto em validação, os consumidores usam `@main`. Depois de validado, criar a tag `v1` e trocar para `@v1`.
