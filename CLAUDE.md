# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## O que é este projeto

Landing page estática da Daxton (`daxton.com.br` / `www.daxton.com.br`), usada em campanha do Google Ads. Não há framework, build step ou dependências — é HTML/CSS/JS puro servido diretamente pelo Vercel (Framework Preset: "Other").

## Estrutura

- `index.html` — página única com tudo: `<head>` com meta tags e a tag do Google Ads, `<style>` inline com todo o CSS, corpo com nav/hero/seções de produto/contato, e múltiplos blocos `<script>` inline (contador do hero, tracking de conversão, player do vídeo)
- `links.html` — página estilo linktree, servida em `/links` via rewrite no `vercel.json`
- `catalogo-csm-2026.pdf`, `video-daxton.mp4`/`.vtt` — assets estáticos referenciados direto no HTML

Não há processo de build: editar `index.html`/`links.html` diretamente é o fluxo normal.

## Deploy

- Repositório GitHub: `Daxton-2026/daxton-site`, branch de produção `main`
- Projeto Vercel: `carlos-sguario-s-projects/daxton-deploy`
- Fluxo normal: commit + `git push origin main` → Vercel builda automaticamente e promove para o domínio customizado sozinho

**Gotcha já resolvido (2026-06-30):** o projeto tinha `autoAssignCustomDomains: false` nas configurações do Vercel — todo push gerava um deploy "Ready" mas o domínio customizado não era promovido automaticamente, ficando preso numa versão antiga até alguém rodar `vercel alias set <deployment-url> daxton.com.br` manualmente. Já foi corrigido (ativado via API). Se voltar a acontecer (domínio servindo versão desatualizada mesmo após push), primeiro suspeitar dessa configuração antes de qualquer outra coisa.

## Tag do Google Ads

- Tag base: `AW-17062055869`, no `<head>` do `index.html`
- Dois eventos de conversão via `onclick` em links (não há `<form>` na página): `gtag_report_conversion(url)` para cliques em WhatsApp/PDF/Drive, `gtag_report_conversion_lead()` para cliques em `mailto:`
- **Cuidado ao editar esse bloco de script:** já rolou um caso onde o fechamento foi escrito como `<\/script>` (barra invertida) para "evitar duplicata" — isso quebra o parser HTML do navegador, que não reconhece isso como fechamento de tag e passa a tratar todo o resto do documento como texto de script (página fica em branco). Sempre fechar com `</script>` puro, sem escape.
