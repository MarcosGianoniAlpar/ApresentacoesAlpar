# Claude — Apresentações Alpar

You are working on a repository of one-page HTML presentations for Alpar's clients, published via GitHub Pages.

**Live:** https://marcosgianonialpar.github.io/ApresentacoesAlpar/

## Workflow para nova apresentação

1. **Coletar do usuário:** nome do cliente, dados (números, processos, datas, ROI, payback)
2. **Criar pasta** `<slug-cliente>/` (kebab-case) na branch `main`
3. **Copiar** `_template/index.html` para `<slug-cliente>/index.html`
4. **Substituir conteúdo** (cliente, números, descrições) seguindo padrões abaixo
5. **Pushar via GitHub MCP API** (NÃO use `git push` — proxy nega com 403)
6. **Tirar screenshots** via Playwright e enviar para revisão do usuário
7. **Atualizar `README.md`** listando a nova apresentação
8. **Aguardar aprovação** antes de iterar

## Padrões obrigatórios

### Marca
- **Logo:** `alpar-logo.png` na raiz do repo, fundo transparente
- **CSS no header:** `background: url('../alpar-logo.png') center/130% no-repeat;` (do subdiretório da apresentação para a raiz)
- **Texto brand:** `<b>Alpar + {{CLIENTE}}</b>` + `<span>{{TEMA}}</span>`
- **Paleta:**
  - `--bg: #070E1F` (navy escuro)
  - `--accent: #3FB8FF` (cyan ServiceNow)
  - `--ink: #E8EEFA` (branco)
  - `--ink-2: #A9B6CF` (cinza claro)
- **Fonts:** Inter (body), Manrope (headers/números)

### Conteúdo — NUNCA usar
- **Em-dash `—` ou hífen `-` como conector textual** → substituir por vírgula, `:` ou reformular
  - Exceções: title `<head>` e footer institucional (`Alpar — ServiceNow Global Partner`, `— Mês Ano`)
- E-mail pessoal no footer
- CTA "Talk to Alpar" no final

### Números (padrão Brasileiro)

| Tipo | Formato | Exemplo |
|---|---|---|
| Milhar | ponto | `13.200`, `1.223` |
| Decimal | vírgula | `6,6`, `R$ 1,03 M` |
| >1.000 com decimal | descartar decimal | `1.000,00` → `1.000` |
| Horas | só horas, sem minutos | `168h30` → `168h` |

JS counter `formatNumber`: `n.toLocaleString('pt-BR')` para milhar, `.replace('.', ',')` para decimal.

### KPIs (seção Executive Summary)
- Grid 5 colunas (responsive 3/2/1)
- `grid-template-rows: 1fr 1fr` para alinhar números verticalmente entre cards
- Número branco `clamp(30px, 2.8vw, 40px)`, `white-space: nowrap`, `text-align: center`
- Unidade ciano (accent), `margin-left: 8px`
- **SEM** prefixo `~` aproximação

### Hero
- `padding-bottom: 140px` mínimo (scroll hint não pode sobrepor chips)
- Card lateral com textos curtos: só `FIEP`, só `Alpar` (não use `FIEP — Sistema Indústria`)
- Stat: `Freed` (não `Freed for strategy`)

## Git workflow

### ❌ NÃO funciona

```bash
git push origin <branch>    # proxy nega: HTTP 403
```

### ✅ Use SEMPRE MCP tools

| Operação | Tool |
|---|---|
| Criar/atualizar arquivo | `mcp__github__create_or_update_file` (precisa `sha` para update) |
| Deletar arquivo | `mcp__github__delete_file` |
| Ler conteúdo + SHA | `mcp__github__get_file_contents` |
| Listar pasta | `mcp__github__get_file_contents` com `path` da pasta |
| Criar branch | `mcp__github__create_branch` |

**Importante:** para `update`, sempre faça `get_file_contents` primeiro para pegar o SHA atual.

### Convenção
- **`main`** = branch publicada no GitHub Pages
- Cada apresentação = uma **pasta** em `main` (não branch)
- Branches `apresentacao/<slug>` só para drafts/revisão antes de mergear

### Se MCP retornar 403
Peça ao usuário:
1. github.com/settings/installations → Claude App
2. Repository access: marcar `ApresentacoesAlpar`
3. Permissions → Contents: **Read and write**

## Deploy GitHub Pages

- Settings → Pages → Source: branch `main`, folder `/ (root)`
- URL: `https://marcosgianonialpar.github.io/ApresentacoesAlpar/<slug>/`
- Rebuild automático ~30-60s após push

## Verificação visual antes de entregar

1. Servidor local em background: `python3 -m http.server 8765 --bind 127.0.0.1`
2. Chromium: `/opt/pw-browsers/chromium-1194/chrome-linux/chrome`
3. Playwright: `/opt/node22/lib/node_modules/playwright/node_modules/playwright-core`
4. **Stub IntersectionObserver** para forçar reveals/counters:
   ```js
   await page.addInitScript(() => {
     window.IntersectionObserver = class {
       constructor(cb){this.cb=cb}
       observe(el){this.cb([{target:el,isIntersecting:true}])}
       unobserve(){} disconnect(){}
     };
   });
   ```
5. Screenshot de cada seção em `viewport: { width: 1440, height: 900 }, deviceScaleFactor: 1.5`
6. Enviar via `SendUserFile` para revisão do usuário

## Referência canônica

`_template/index.html` é a cópia funcional completa da apresentação **bvA FIEP** (primeira do repo).
Para padrões de animação, side dots, cards, workload chart, processes detail: inspecione esse arquivo.

## Princípios de interação

- Conversar em **português**
- **Listar todos os ajustes** antes de aplicar em lote → "pode ajustar?"
- **Confirmar com "pode subir?"** antes do push final
- **Verificar deploy** com WebFetch ou Playwright após push
- Usar `AskUserQuestion` para escolhas ambíguas
- **Não criar arquivos .md** além de CLAUDE.md/README.md a menos que o usuário peça
