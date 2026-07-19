# Segredos do ambiente `dev` — mecanismo Infisical

Este repositório **nunca** contém segredos reais (ADR-0009). Todo valor
`${VAR}` em `localizestay.stack.yml` é um placeholder de substituição de
variável de ambiente do Docker Compose/Stack, resolvido **fora** deste repo,
no momento do `docker stack deploy` executado pelo Portainer.

## Como os valores chegam à stack (padrão da casa)

1. **Infisical** é a fonte de verdade dos segredos. Existe (ou será criado)
   um projeto/ambiente `localizestay` → `dev` no Infisical contendo cada uma
   das chaves listadas abaixo.
2. O **Infisical Agent**, já operando no `mcad-server` como os demais
   segredos da casa, autentica via Universal Auth/machine identity e
   **renderiza um arquivo de ambiente** (template Go, ver skill
   `infisical-agent`) a partir dessas chaves — por exemplo em
   `/etc/infisical-agent/localizestay-dev.env`. O agente mantém esse arquivo
   atualizado (polling/renovação de token) sem intervenção manual.
3. **Portainer não lê arquivos arbitrários do host para stacks GitOps.** A
   ponte entre o arquivo renderizado pelo Infisical Agent e a stack é feita
   pela seção **"Environment variables"** da definição da stack no Portainer
   (chave/valor usados na substituição `${VAR}` ao rodar `docker stack
   deploy` sobre o compose deste repositório).
4. **O mecanismo exato de sincronização** (colar manualmente os valores
   renderizados na UI do Portainer vs. um script/`on-change` do Infisical
   Agent chamando a API do Portainer para atualizar as env vars da stack e
   disparar redeploy) **é uma pendência de implementação**, a ser resolvida
   na configuração operacional do servidor — não faz parte deste repositório
   de deploy. Até lá, a atualização de segredos é manual: copiar os valores
   do Infisical (UI ou `infisical export`) para a stack no Portainer.

## Variáveis esperadas (ambiente `dev`)

| Variável | Usada em | Conteúdo |
|---|---|---|
| `LOCALIZESTAY_DB_PASSWORD` | `postgres`, `api`, `backup-app-db` | Senha do usuário `localizestay` no Postgres da aplicação |
| `LOGTO_DB_PASSWORD` | `logto-postgres`, `logto`, `backup-logto-db` | Senha do usuário `logto` no Postgres do LogTo |
| `LOCALIZESTAY_API_LOGTO_M2M_CLIENT_ID` | `api` | Client ID da aplicação M2M `localizestay-backend-m2m` (Management API do LogTo) |
| `LOCALIZESTAY_API_LOGTO_M2M_CLIENT_SECRET` | `api` | Client secret correspondente |
| `LOCALIZESTAY_TRAVELER_LOGTO_CLIENT_ID` | `traveler` | Client ID da aplicação SPA `localizestay-app` no LogTo (não é segredo — fica exposto no browser — mas só existe após o setup do LogTo; mantido aqui junto das demais variáveis da stack) |
| `CLOUDFLARE_R2_ACCOUNT_ID` | `backup-app-db`, `backup-logto-db` | Account ID do Cloudflare R2 (compõe `S3_ENDPOINT`) |
| `R2_ACCESS_KEY_ID` | `backup-app-db`, `backup-logto-db` | Access Key do token R2 com permissão de escrita no bucket `localizestay-dev-backups` |
| `R2_SECRET_ACCESS_KEY` | `backup-app-db`, `backup-logto-db` | Secret Key correspondente |
| `LOCALIZESTAY_BACKUP_PASSPHRASE` | `backup-app-db` | Passphrase GPG (ADR-0005) para criptografar os dumps do banco da app |
| `LOGTO_BACKUP_PASSPHRASE` | `backup-logto-db` | Passphrase GPG para criptografar os dumps do banco do LogTo |

> Recomendação: usar passphrases distintas por banco (evita que o
> comprometimento de um dump exponha o outro).

## IPs do `ipAllowList` (Admin Console do LogTo e UI do Mailpit)

Os IPs no middleware `lstay-admin-ipallowlist` (labels do serviço `logto` e
do serviço `mailpit` em `localizestay.stack.yml`) **não são segredos** no
sentido do Infisical, mas também não devem ficar com o placeholder de
exemplo (`198.51.100.10/32`, bloco de documentação RFC 5737) em produção
deste arquivo. Substituir pelos IPs reais da equipe antes do primeiro deploy
— editar o label diretamente no compose (commit revisável, é exatamente o
tipo de mudança de infraestrutura que este repositório deve versionar).

Depende do `forwardedHeaders.trustedIPs` (ranges da Cloudflare) já
configurado globalmente na stack do Traefik do host (ADR-0004) — sem isso, o
`ipAllowList` avaliaria o IP da Cloudflare, não o do cliente real.

## Credenciais de registry (GHCR)

As imagens `ghcr.io/tassosgomes/localizestay-api` e
`ghcr.io/tassosgomes/localizestay-app` podem ser privadas. Se forem, o
Portainer precisa de uma credencial de registry GHCR configurada
(Settings → Registries) para conseguir puxar a imagem no redeploy. Isso é
configuração do Portainer, não deste repositório — registrar como pendência
de setup.
