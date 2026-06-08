# Projeto — Faróis Como Novos (Garagem Detail)

Landing page de vendas (low ticket, R$19,90) para um curso de restauração de faróis,
com ângulo de **renda extra / faturamento**. Página única em `index.html`
(HTML + CSS inline, fonte Sora, sem build).

## Stack de tracking (server-side) — integrada do krob-tracking-stack

Funil: anúncio → `index.html` (gera `trk`, grava `checkout_sessions`) → checkout
**Kiwify** (`trk` vai no parâmetro `sck`) → webhook `/webhook/kiwify/<slug>` →
eventos de conversão server-side pra Meta CAPI / GA4 / Google Ads + dashboard `/dash`.

- Backend em `functions/` (Cloudflare Pages Functions), schema D1 em `migrations/`,
  config por produto em `config/products.js`, dashboard em `dash/index.html`.
- Skills do Claude Code em `.claude/skills/` (`deploy-stack`, `verify-tracking`,
  `add-page`, `add-sales-platform`). Docs de referência em `docs/`.
- A `index.html` já tem: Meta Pixel + GA4 + PageView no `<head>`, e o wiring de
  atribuição antes do `</body>` (botão de compra = `#checkout-btn`, plataforma
  `kiwify`). **Falta editar:** `META_PIXEL_ID`, `G-XXXXXXXXXX` e `CHECKOUT_URL`.

### O que falta para o tracking funcionar (precisa de você / dashboard)

1. **Criar o D1** e aplicar migrations: ver skill `.claude/skills/deploy-stack`.
   `cp wrangler.toml.example wrangler.toml` e preencher name/db. `wrangler.toml`
   é gitignored (Pages com Git não o lê — serve só pro wrangler local/migrations).
2. **Bindar o D1** ao Pages project no dashboard (binding DEVE se chamar `DB`).
3. **Variáveis de ambiente** no dashboard do Pages (Settings → Environment
   variables): `META_PIXEL_ID`, `META_ACCESS_TOKEN`, `GA4_MEASUREMENT_ID`,
   `GA4_API_SECRET`, `DASH_KEY`, `KIWIFY_WEBHOOK_SLUG` (UUID v4). Detalhes e
   opcionais (Google Ads, ad-spend sync) em `wrangler.toml.example` e `docs/`.
4. **Webhook na Kiwify**: apontar para `https://infor-produto.pages.dev/webhook/kiwify/<KIWIFY_WEBHOOK_SLUG>`, evento `order_approved`.
5. Conferir tudo com a skill `verify-tracking`.

> Regras duras do stack (não violar): nunca commitar segredos; SQL sempre com
> `.bind()`; PII com hash SHA-256 antes de ir pra Meta; PageView não grava no
> `event_log`. Resumo completo no topo de cada arquivo e em `docs/architecture.md`.

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

## Deploy (Cloudflare Pages — conectado ao GitHub)

- **GitHub:** https://github.com/matheusercolani/infor-produto (branch `main`, público).
- **Cloudflare Pages project:** `infor-produto` → **https://infor-produto.pages.dev**
- **Deploy automático:** cada `git push` na `main` dispara um build/deploy no Pages.
  Sem etapa de build (site estático, output = raiz, o próprio `index.html`).

### Fluxo de atualização

```bash
# editar index.html, depois:
git add -A && git commit -m "..."
git push                      # Cloudflare publica sozinho em segundos
```

> Push: a credencial do GitHub está salva no Windows Credential Manager.
> O push pela git bash falha (`/dev/tty`); rodar o `git push` pelo **PowerShell**.

### Deploy manual (fallback, sem depender do GitHub)

```bash
cf-on info-mecanica
npx --yes wrangler pages deploy . --project-name infor-produto
```

### Comandos úteis (sempre com o profile ativo)

```bash
cf-on info-mecanica
npx --yes wrangler pages deployment list --project-name infor-produto
```
