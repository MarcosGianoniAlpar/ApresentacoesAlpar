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

## Página protegida por senha (login client-side criptografado)

Para páginas com **dado sensível** num link **público** (ex.: report de horas/billabilidade). Um login só em JS (esconder/mostrar) NÃO protege: os dados ficam no HTML e qualquer um lê no "ver código-fonte". A solução real: **criptografar o payload com a senha** e descriptografar no navegador só após login.

### Como funciona
- Payload sensível (dados + Excel embutido) vira **`{data, xlsx}` em JSON**, criptografado com **AES-256-GCM**, chave derivada da senha via **PBKDF2 (SHA-256, 200.000 iterações)**.
- No HTML fica só o cifrado (`const ENC={s:salt, i:iv, n:iters, c:ciphertext}`, tudo base64). Sem a senha = lixo cifrado, nem no source-code aparece nada.
- Tela de login (overlay `#lock`) → senha certa descriptografa via Web Crypto e chama `boot(payload)` que popula as variáveis e roda `render()`. Senha errada → o decrypt lança erro.
- 100% estático/offline (GitHub Pages). **Requer secure context**: https (Pages), localhost ou file:// — `crypto.subtle` não roda em http puro.

### Regras de implementação
- Trocar `const RAW={...};const PEOPLE=...` por `let RAW={},PEOPLE=[],ROLES=[],MONTHS=[],DATA=[];` + `const ENC={...}`.
- O Excel embutido (`XLSX_FILE_B64`) também entra no payload cifrado → vira `let XLSX_FILE_B64="";` e o botão de download só funciona após login.
- Remover o `render()` automático do fim; ele passa a ser chamado dentro de `boot()`.
- **Senha compartilhada** (mesma p/ todos). Trocar a senha = **re-criptografar** o payload (gera novo ENC). Pode ter mais de uma senha gerando múltiplos ENC.
- Verificar no deploy: `grep -c "<algum nome/dado>" pagina.html` deve dar **0** (nada em claro).
- Compartilhar a senha por canal separado do link.

### JS de descriptografia/login (reusável)
```js
async function decryptPayload(pw){const d=s=>Uint8Array.from(atob(s),c=>c.charCodeAt(0));
  const km=await crypto.subtle.importKey("raw",new TextEncoder().encode(pw),"PBKDF2",false,["deriveKey"]);
  const key=await crypto.subtle.deriveKey({name:"PBKDF2",salt:d(ENC.s),iterations:ENC.n,hash:"SHA-256"},km,{name:"AES-GCM",length:256},false,["decrypt"]);
  const pt=await crypto.subtle.decrypt({name:"AES-GCM",iv:d(ENC.i)},key,d(ENC.c));return JSON.parse(new TextDecoder().decode(pt));}
// no submit: try{ boot(await decryptPayload(senha)); document.getElementById('lock').remove(); }catch{ erro(); }
```

### Build em Python (cripto compatível com Web Crypto)
A lib `cryptography` está quebrada no ambiente → usar **`hashlib.pbkdf2_hmac`** (nativo) + **`pycryptodome`** (`pip install pycryptodome`):
```python
import hashlib, os, base64; from Crypto.Cipher import AES
salt=os.urandom(16); iv=os.urandom(12)
key=hashlib.pbkdf2_hmac('sha256', PW.encode(), salt, 200000, 32)
ct,tag=AES.new(key,AES.MODE_GCM,nonce=iv).encrypt_and_digest(payload_json_bytes)
# ENC.c = base64(ct+tag)  (Web Crypto AES-GCM espera ciphertext||tag de 16 bytes)
```

### Referência funcional
`report-horas/index.html` (PT) e `report-hours/index.html` (EN) usam exatamente essa solução. Login atual: usuário `Alpar`, senha `Alpar@2026`.

## Referência canônica

`_template/index.html` é a cópia funcional completa da apresentação **bvA FIEP** (primeira do repo).
Para padrões de animação, side dots, cards, workload chart, processes detail: inspecione esse arquivo.

## Princípios de interação

- Conversar em **português**
- **No começo da conversa**, perguntar se a página vai ter **login/senha** (solução criptografada acima) — para já planejar.
- **Logo após publicar** uma página (deploy no Pages), **confirmar** com o usuário se ele quer **incluir senha no link**; se sim, aplicar o gate criptografado e republicar.
- **Listar todos os ajustes** antes de aplicar em lote → "pode ajustar?"
- **Confirmar com "pode subir?"** antes do push final
- **Verificar deploy** com WebFetch ou Playwright após push
- Usar `AskUserQuestion` para escolhas ambíguas
- **Não criar arquivos .md** além de CLAUDE.md/README.md a menos que o usuário peça
