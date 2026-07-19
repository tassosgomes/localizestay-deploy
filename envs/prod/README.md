# `prod` — LocalizeStay

**Status: não existe ainda. Bloqueado por decisão arquitetural (ADR-0003).**

Este diretório é um placeholder intencionalmente vazio de stack files.

## Por que `prod` não roda no `mcad-server`

ADR-0003 é explícito: o `mcad-server` é um servidor Swarm de nó único
**compartilhado** com a operação corporativa e outros projetos internos.
Produção do piloto da LocalizeStay envolve dados pessoais e financeiros
reais de clientes (LGPD), e o ADR estabelece como **critério de bloqueio**:

> "Nenhuma transação real de cliente ocorre no servidor compartilhado."

Motivos registrados no ADR:

- Isolamento de dados pessoais e financeiros do restante da operação
  corporativa (LGPD).
- Raio de explosão de incidentes independente da operação corporativa.
- Ciclo de manutenção próprio (janelas de manutenção, patching, upgrades)
  sem coordenação com outros times/projetos no mesmo host.
- Ponto único de falha: o `mcad-server` é Swarm de nó único — inaceitável
  como único host de produção de um sistema com transações reais.

## Pré-condição para este diretório ganhar conteúdo

1. **VPS dedicada provisionada** (verba prevista para quando o produto
   avançar, conforme ADR-0003) — nó Swarm próprio, não compartilhado.
2. Checklist de go-live do piloto cumprido, incluindo (ver ADR-0005):
   - RPO/RTO de backup reavaliados explicitamente para dados reais (o RPO
     de ~1h aceito em `dev`/`homolog` **não** é aceitável sem revisão).
   - Exercício de restauração de backup documentado e validado.
   - Revisão de segurança do firewall/portas da VPS (item já anotado no
     ADR-0003 a respeito de más práticas observadas no `mcad-server`, como
     dashboard do Traefik inseguro e portas publicadas fora do Traefik —
     não repetir na VPS dedicada).
3. Domínio definitivo do produto decidido (hosts `lstay-*.tasso.dev.br` são
   exclusivos de dev/homolog).
4. Provedor de e-mail transacional real definido e configurado (Mailpit não
   é aceitável em produção).

## O que entra aqui quando for construído

Estrutura análoga a `envs/dev/`: `localizestay.stack.yml` próprio (stack
Swarm da VPS dedicada), com tags de imagem imutáveis promovidas via CI,
Traefik + Cloudflare (ADR-0004) apontando para o domínio definitivo, e
segredos exclusivos do ambiente `prod` no Infisical. Nenhum serviço publica
porta no host; nenhuma exceção às regras já aplicadas em `dev`.
