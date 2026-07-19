# `homolog` — LocalizeStay

**Status: ainda não implantado.** Este diretório é um placeholder.

Conforme ADR-0003 e ADR-0009, `homolog` roda no mesmo `mcad-server`
compartilhado usado por `dev`, como uma stack Swarm irmã (nome de stack
próprio, ex.: `localizestay-homolog`, para não colidir com `dev`).

## O que muda em relação a `dev`

`homolog` **replica a topologia de `envs/dev/localizestay.stack.yml`**
(mesmos serviços: `api`, `traveler`, `postgres`, `logto`, `logto-postgres`,
`mailpit`, `backup-app-db`, `backup-logto-db`), com estas diferenças:

- **Imagens promovidas**, não `:dev` mutável: as tags de `api` e `traveler`
  passam a ser a tag imutável de release testada em `dev` (`sha-<hash>` ou
  tag semântica promovida pelo pipeline de CI — mecanismo exato definido na
  implementação do pipeline, ADR-0009), nunca `:dev` flutuante.
- **Hosts/DNS próprios** (subdomínio distinto do de `dev`, a decidir — ex.
  `lstay-*-homolog.tasso.dev.br` ou domínio próprio, respeitando o limite de
  nível único do certificado Universal da Cloudflare enquanto não houver
  domínio de produto definitivo).
- **Volumes, redes e nome de stack isolados dos de `dev`** — nenhum dado
  ou tráfego compartilhado entre os dois ambientes no mesmo host.
- **Segredos próprios no Infisical** (ambiente `homolog` separado de `dev`
  no mesmo projeto `localizestay`), nunca reaproveitados de `dev`.
- **Provedor de e-mail transacional real** (não Mailpit) — pendente de
  decisão conforme `docs/setup-logto.md` (candidatos: Amazon SES, Resend,
  Brevo). Até essa decisão, `homolog` pode manter Mailpit como em `dev`.

## Pré-condição para existir de fato

Nenhuma — `homolog` pode nascer no servidor compartilhado assim que houver
necessidade de um ambiente de validação separado de `dev` (ao contrário de
`prod`, que depende da VPS dedicada, ver `../prod/README.md`). Quando for
implementado, copiar `envs/dev/localizestay.stack.yml` como ponto de
partida, ajustar tags de imagem, hosts, nome de stack, redes/volumes e
variáveis de ambiente do Infisical.
