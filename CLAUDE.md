# Projeto — Faróis Como Novos (Garagem Detail)

Landing page de vendas (low ticket, R$19,90) para um curso de restauração de faróis,
com ângulo de **renda extra / faturamento**. Página única em `index.html`
(HTML + CSS inline, fonte Sora, sem build).

## Conta Cloudflare — SEMPRE usar o profile `info-mecanica`

> **Regra deste projeto:** todo deploy/Wrangler aqui usa o profile Cloudflare **`info-mecanica`**.
> Não usar `matheus-consultoria`, `ventture-leads`, `fluxxo-pet` nem `bruno-leads`.

- Profile registrado via os helpers `cf-*` (definidos em `~/.bashrc`).
- Conta: "Matheusercolani.blake@gmail.com's Account" — ID `e4a039cfcd4e8160e4a12883e63b36c5`.
  (Obs.: é a mesma conta de outros profiles; `info-mecanica` é o rótulo dedicado a este projeto.)

### Antes de qualquer comando Wrangler / deploy, ative o profile:

```bash
cf-on info-mecanica
npx --yes wrangler whoami   # confere a conta
```

### Detalhe do ambiente (Windows + Git Bash)

As variáveis `CF_CONFIG`/`CF_TOKENS` **não vêm exportadas** na shell do snapshot do Claude Code.
Se os comandos `cf-*` reclamarem de caminho vazio, prefixe a chamada com:

```bash
export CF_CONFIG="$HOME/.config/cloudflare"; export CF_TOKENS="$CF_CONFIG/tokens"; source ~/.bashrc
```

## Deploy (Cloudflare Pages, site estático)

- Sem etapa de build. Diretório de saída = raiz do projeto (o próprio `index.html`).
- Deploy direto via Wrangler:

```bash
cf-on info-mecanica
npx --yes wrangler pages deploy . --project-name farois-como-novos
```

- Alternativa: conectar o repositório GitHub no painel do Cloudflare Pages
  (build command vazio, output dir = `/`).
