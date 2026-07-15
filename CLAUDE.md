# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## O que é este projeto

Landing page estática da Daxton (`daxton.com.br` / `www.daxton.com.br`), usada em campanha do Google Ads. Não há framework, build step ou dependências — é HTML/CSS/JS puro servido diretamente pelo Vercel (Framework Preset: "Other").

## Estrutura

- `index.html` — home institucional da Daxton: `<head>` com meta tags e a tag do Google Ads, `<style>` inline com todo o CSS, corpo com nav/hero/seção "Quem representamos" (cards das marcas)/diferenciais/setores/contato, e múltiplos blocos `<script>` inline (contador do hero, tracking de conversão, player do vídeo)
- `real-escadas.html` (servida em `/real-escadas`) e `csm.html` (servida em `/csm`) — páginas dedicadas de cada marca representada, ver "Padrão de página por marca" abaixo
- `links.html` — página estilo linktree, servida em `/links` via rewrite no `vercel.json`
- `catalogo-csm-2026.pdf`, `catalogo-real-escadas-2026.pdf`, `video-daxton.mp4`/`.vtt` — assets estáticos referenciados direto no HTML
- `produtos/` — fotos de produto usadas em `real-escadas.html`
- `marcas/` — fotos de produto usadas em `csm.html` e nos cards de marca da home (`index.html`)

Não há processo de build: editar os `.html` diretamente é o fluxo normal.

## Padrão de página por marca representada

Cada marca que a Daxton representa (hoje: Real Escadas, CSM) tem direito a:
1. Um card na seção "Quem representamos" da home (`index.html`, `id="marcas"`), com foto de produto real (não logo) e link direto pra página própria — nunca `onclick="activateTab(...)"` fazendo scroll na própria home.
2. Uma página dedicada (`<marca>.html`) espelhando a estrutura do `real-escadas.html`: nav com logo/botão "Home" voltando pra `/`, hero (headline + `hero-img-col` com imagem grande de portfólio), tira de diferenciais rápidos, seção de canais em abas (`produtos-tabs` + `tab-panel`, controlada por `activateTab(id)` no JS), seção "Por que a marca", cobertura em SC, processo de venda, formulário de lead (`#contato`) e footer.
3. Uma entrada em `vercel.json` (`rewrites`) mapeando `/marca` → `/marca.html`.

Ao copiar `real-escadas.html` como base pra uma marca nova, usar edição por PowerShell/linha (não o Edit tool direto) pra qualquer linha com imagem embutida em base64 — ver seção de gotchas técnicos.

### Fotos de produto

- Fonte principal: pasta compartilhada do Google Drive `!_PRODUTOS_GERAL` (pedir o link ao Daxton se não tiver — cada categoria de produto tem sua própria subpasta, ex: `BETONEIRAS`, `GERADORES`, `ARGAMASSADEIRA`). Navegar, abrir a foto, baixar (vai pra `C:\Users\Carlos\Downloads`), depois redimensionar.
- Padrão de redimensionamento: PowerShell + `System.Drawing`, resize pra ~700px de largura, salvar como JPEG qualidade 85, destino `marcas/` ou `produtos/` com nome descritivo (`csm-<produto>.jpg`). Preferir fotos com fundo branco/neutro já cortado (a maioria dos arquivos do Drive já vem assim).
- Card pequeno (~150px) não aguenta foto com fundo complexo/lifestyle — nesses casos usar o corte de catálogo, não a foto de contexto.

### Copy de produto

Ao escrever/revisar texto de produto CSM, conferir contra o site oficial **csmequipamentos.com.br** (menu Categorias) antes de publicar — usar `get_page_text` na página do produto específico pra extrair as características reais em vez de inventar specs. Já aconteceu de eu supor um diferencial errado (ex: atribuir "tambor fundo triplo" à IZI Max quando na verdade é da Rental Force, e o diferencial real da IZI Max é o motor removível antifurto).

## Deploy

- Repositório GitHub: `Daxton-2026/daxton-site`, branch de produção `main`
- Projeto Vercel: `carlos-sguario-s-projects/daxton-deploy`
- Fluxo normal: commit + `git push origin main` → Vercel builda automaticamente e promove para o domínio customizado sozinho

**Gotcha já resolvido (2026-06-30):** o projeto tinha `autoAssignCustomDomains: false` nas configurações do Vercel — todo push gerava um deploy "Ready" mas o domínio customizado não era promovido automaticamente, ficando preso numa versão antiga até alguém rodar `vercel alias set <deployment-url> daxton.com.br` manualmente. Já foi corrigido (ativado via API). Se voltar a acontecer (domínio servindo versão desatualizada mesmo após push), primeiro suspeitar dessa configuração antes de qualquer outra coisa.

## Tag do Google Ads

- Tag base: `AW-17062055869`, presente no `<head>` de `index.html`, `real-escadas.html` e `csm.html`
- Em `index.html`: dois eventos de conversão via `onclick` em links (não há `<form>` na página): `gtag_report_conversion(url)` para cliques em WhatsApp/PDF/Drive, `gtag_report_conversion_lead()` para cliques em `mailto:`
- Em `real-escadas.html`/`csm.html`: além do mesmo padrão de `onclick`, existe um `<form id="leadForm">` cujo `submit` monta uma mensagem de WhatsApp com os dados preenchidos e chama `gtag_report_conversion(url)` — ao duplicar a página pra uma marca nova, atualizar o texto da mensagem e as opções do `<select id="f-canal">` pros canais daquela marca
- Conversion labels específicos: clique/WhatsApp = `AW-17062055869/YnaVCL78v8QaEL2f6cc_`, lead (e-mail) = `AW-17062055869/-EeFCK_X_sMaEL2f6cc_`
- **Cuidado ao editar esse bloco de script:** já rolou um caso onde o fechamento foi escrito como `<\/script>` (barra invertida) para "evitar duplicata" — isso quebra o parser HTML do navegador, que não reconhece isso como fechamento de tag e passa a tratar todo o resto do documento como texto de script (página fica em branco). Sempre fechar com `</script>` puro, sem escape.

## Gotchas técnicos

- **Linhas com imagem base64 embutida** (comuns em `index.html`) travam o Read/Edit tool padrão por excesso de tokens. Usar PowerShell (`[System.IO.File]::ReadAllLines`/`WriteAllLines`, `UTF8Encoding($false)`) pra editar por número de linha sem carregar o conteúdo da linha gigante no contexto; ou `Grep` com `-A`/`-B` pequenos pra inspecionar a vizinhança sem tocar na linha em si.
- **Teste local**: `npx --yes serve -l 8532 .` na raiz do repo, depois `taskkill //F //IM node.exe` pra derrubar. As páginas com abas (`produtos-tabs`) e animações `fade-up` só ficam com opacidade cheia depois de entrar no viewport — se um screenshot vier "apagado", rolar de novo ou dar `location.reload(true)` e aguardar.
- **Mobile**: pra testar breakpoints sem depender do `resize_window` (que não afeta o viewport de captura nesse ambiente), injetar um `<iframe>` de largura fixa via `javascript_tool` apontando pra `localhost:8532` e screenshotar o iframe.
