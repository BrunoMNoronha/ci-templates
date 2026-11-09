# Como contribuir

Workflows reutilizáveis de CI/CD. Mudanças passam pelo autoteste (`selftest.yml`) e por revisão. Siga as regras abaixo — elas controlam custo e segurança dos Actions em todos os repositórios consumidores.

## Regras dos workflows

1. **Runner** `ubuntu-24.04` fixo, nunca `ubuntu-latest`. Windows (1,7×) ou macOS (10×) só com justificativa escrita no próprio arquivo.
2. **`timeout-minutes` em todo job** (CI ≤ 15, integração ≤ 25, deploy ≤ 30). O default de 360 minutos é proibido.
3. **`concurrency` em todo workflow não reutilizável**, no formato `${{ github.workflow }}-${{ github.event.pull_request.number || github.sha }}`, com `cancel-in-progress` ligado só em `pull_request`.
4. **Triggers**: `pull_request` + `push` só em `main` (nunca `push` sem filtro de branch). `paths-ignore` para `**.md` e `docs/**`.
5. **Ações fixadas por SHA completo** com comentário `# vX.Y.Z`. O Dependabot (`.github/dependabot.yml`) atualiza os SHAs.
6. **`persist-credentials: false`** em todo checkout. **`permissions: contents: read`** no topo; eleve só no job que precisar.
7. **Versão de runtime vem do repositório**: Node por `node-version-file`, pnpm pelo `packageManager`. Não repita a versão no YAML.
8. **Cache sempre**: store do pnpm, `.next/cache`, NuGet, e `type=gha` no Docker.
9. **Um build por SHA**: o deploy consome o artefato/validação da CI, não roda a CI de novo.
10. **`retention-days` explícito** em artefatos (CI 7; evidências e backups 30).

## Antes de abrir PR

- Rode o actionlint:
  ```bash
  docker run --rm -v "$PWD:/repo" -w /repo rhysd/actionlint:1.7.7 -color
  ```
- O `selftest.yml` exercita `node-ci` e `docker-build` contra `tests/`. Ajuste as fixtures se mudar a interface.

## Versionamento

Consumidores fixam a major por tag flutuante: `@v1`. Toda mudança compatível move a tag `v1` para o novo commit. Mudança incompatível cria `v2`. Crie `v1` só depois do primeiro merge na `main` com o autoteste verde:

```bash
git tag -a v1 -m "v1: node-ci, vercel-deploy, docker-build, dotnet-ci" <sha-da-main>
git push origin v1
```
