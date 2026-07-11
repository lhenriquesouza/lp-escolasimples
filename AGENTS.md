# AGENTS.md — escolasimples Landing Page

## Sobre o projeto

Landing page do escolasimples — sistema de gestao para escolas de idiomas.
Single-file HTML puro (~1200 linhas), sem build step, sem framework, sem dependencias.

- **Repo:** `lhenriquesouza/lp-escolasimples`
- **Deploy:** GitHub Pages (branch `master`, raiz)
- **Dominio:** aponta para `app.escolasimples.com` (sistema) e a LP fica no GitHub Pages ou dominio proprio

## Estrutura

```
.
├── index.html          # Landing page completa (HTML + CSS + JS inline)
├── brand-mark.svg      # Logo marca (quadrado)
├── brand-icon.svg      # Logo icone
└── mq7kjmfq-escolasimples-demo.gif  # GIF de navegacao do produto
```

## Stack

- HTML5 puro
- CSS inline (sem pre-processador) — design tokens com `oklch()` e custom properties no `:root`
- JavaScript vanilla (~70 linhas) — nav sticky + carrossel de depoimentos com touch swipe
- Zero dependencias externas (sem CDN, sem fonts do Google, font stack de sistema)
- Layout responsivo com CSS Grid, breakpoints em 900px e 600px
- `prefers-reduced-motion` respeitado

## Design tokens

Definidos como CSS custom properties no `:root`:

| Token | Uso |
|-------|-----|
| `--bg`, `--surface`, `--surface-subtle`, `--surface-inset` | Fundos |
| `--fg`, `--muted` | Texto |
| `--border` | Bordas |
| `--accent` (azul), `--accent-strong`, `--accent-2` (verde) | Cor de destaque |
| `--success`, `--warning`, `--danger`, `--info` | Estados |
| `--font-display` (Georgia), `--font-body` (system-ui), `--font-mono` | Tipografia |
| `--radius-1` (10px), `--radius-2` (16px), `--radius-3` (24px) | Cantos |
| `--shadow-1`, `--shadow-2` | Sombras |

Cores no espaco `oklch`. Sempre usar `color-mix(in oklab, ...)` para variacoes sem criar novos tokens.

## Seccoes da pagina

1. **Nav** — sticky com backdrop-blur, logo SVG inline, CTA "Ver planos"
2. **Hero** — duas colunas (texto + ilustracao SVG), gradiente radial decorativo, h1 + CTA buttons
3. **Dor** — 3 cards com problemas reais de dono de escola (Pagamento, Agenda, Previsibilidade)
4. **Solucao** — "Um lugar so" com mini dashboard mock (stats + tabela)
5. **Demo** — iframe simulado (chrome fake) com GIF animado do produto
6. **Modulos** — 6 cards (Alunos, Financeiro, Agenda, Professores, Folha, Previsibilidade)
7. **Depoimentos** — carrossel com 5 slides, dots, arrows, autoplay 6s, touch swipe
8. **Planos** — 3 cards (Essencial R$149, Profissional R$249 destacado, Avancado R$349), features comuns, contato enterprise
9. **Footer** — 4 colunas (logo, Produto, Empresa, Contato), bandeira do Brasil

## Convencoes

- **Idioma:** todo o conteudo em pt-BR
- **SVGs:** inline no HTML (logo da nav, ilustracao do hero, logo do footer, bandeira)
- **Nomes de classe:** BEM-like simplificado (`nav-inner`, `hero-text`, `pain-card`)
- **Classes de tipografia:** `t-display`, `t-display-xl/lg/md`, `t-kicker`, `t-body`, `t-meta`
- **GIF de demo:** arquivo estatico no repo, servido como `<img>`, sem lazy load (esta no meio da pagina)
- **Email de contato:** `contato@escolasimples.com` redireciona para `henriquedesouza.l@gmail.com`

## Como editar

1. Editar `index.html` diretamente — e um arquivo unico
2. Testar localmente: abrir o arquivo no navegador (nao precisa de servidor)
3. Imagens e SVGs sao todos inline ou arquivos locais — sem hotlink externo
4. Commit e push na `master` — GitHub Pages serve automaticamente da raiz

## Cuidados

- Nao adicionar dependencias externas (CDN, fonts, frameworks). A pagina e propositalmente zero-dependencia.
- Nao quebrar o single-file: CSS e JS ficam inline no HTML.
- Manter os depoimentos realistas — sao ficticios mas verossimeis, com nome, escola e cidade.
- Precos dos planos alinhados com o produto real (Essencial R$149, Profissional R$249, Avancado R$349).
- Respeitar `prefers-reduced-motion` em qualquer animacao nova.
- Testar nos breakpoints 900px e 600px apos cada mudanca visual.
