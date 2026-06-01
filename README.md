# Apresentações Alpar

Apresentações HTML one-page da Alpar, publicadas via GitHub Pages.

🔗 **Portal:** https://marcosgianonialpar.github.io/ApresentacoesAlpar/

## Apresentações

| Cliente / Projeto | URL direta |
|---|---|
| **Assaí** — Alpar AI Framework (Phase 1 roadmap, Top Offenders, BVA) | [Abrir](https://marcosgianonialpar.github.io/ApresentacoesAlpar/assai-ai-framework/) |
| **FIEP** — Backoffice Process Optimization & Automation | [Abrir](https://marcosgianonialpar.github.io/ApresentacoesAlpar/bva-fiep/) |
| **Afya** — Digital Admission | [Abrir](https://marcosgianonialpar.github.io/ApresentacoesAlpar/afya-digital-admission/) |
| **Afya** — Access Management (BVA · 3 meses pós Go-Live) | [Abrir](https://marcosgianonialpar.github.io/ApresentacoesAlpar/afya-access-management/) |

## Estrutura

```
ApresentacoesAlpar/
├── CLAUDE.md            # Instruções para Claude (IA)
├── README.md            # Este arquivo
├── index.html           # Portal raiz (lista as apresentações)
├── alpar-logo.png       # Logo oficial Alpar (asset compartilhado)
├── _template/
│   └── index.html       # Template base (cópia funcional da bvA FIEP)
└── <slug>/
    ├── index.html       # Uma pasta por apresentação
    └── README.md
```

## Criar nova apresentação

Abra uma sessão Claude Code neste repo e diga:

> "Criar uma apresentação para o cliente **<NOME>** sobre **<TEMA>**"

O Claude segue automaticamente as instruções em `CLAUDE.md` (paleta, formato de números, fluxo Git, deploy).

---

Alpar — ServiceNow Global Partner
