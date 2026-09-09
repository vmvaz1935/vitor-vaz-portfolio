# Design System — vitorvaz.com

Referência dos tokens que governam o `index.html`. A regra é simples:
**componente não escreve cor, raio, sombra, curva ou fonte literal — ele lê um token.**

Todos os tokens vivem em um único bloco no topo do `<style>` do `index.html`.

---

## 1. Cor

### Canais RGB (primitivos)

Cores de marca são declaradas como canais crus, sem `rgb()`, para permitir
alpha arbitrário no ponto de uso:

```css
--accent-rgb: 96 165 250;        /* azul primário   */
--accent-deep-rgb: 59 130 246;   /* azul saturado   */
--accent-soft-rgb: 147 197 253;  /* azul claro      */
--accent-2-rgb: 45 212 191;      /* teal secundário */
--fg-rgb: 255 255 255;           /* neutro sobre fundo escuro */
```

Uso:

```css
border-color: rgb(var(--accent-rgb) / 0.35);
background: rgb(var(--fg-rgb) / 0.04);
```

Nunca escreva `rgba(96, 165, 250, 0.35)` — isso quebra o theming por escopo (§2).

### Tokens derivados

`--accent`, `--accent-2`, `--accent-soft`, `--accent-glow`, `--line-accent`,
`--text-accent`, `--glow-accent`, `--glow-accent-lg` e `--focus-ring` são
calculados a partir dos canais.

> **Por que eles são declarados em `:root, [data-accent]` e não só em `:root`?**
> Uma custom property resolve seus `var()` no elemento onde é **declarada**.
> Se `--accent` fosse derivado só em `:root`, ele computaria azul ali e
> herdaria azul para dentro de um escopo dourado. Redeclarar no seletor
> compartilhado faz cada escopo recomputar a partir do próprio canal.

### Superfícies, linhas e texto

| Token | Uso |
|---|---|
| `--bg-0` / `--bg-1` / `--bg-2` | fundos da página, do card e do card elevado |
| `--surface-1/2/3` | vidro sobre fundo (0.03 / 0.05 / 0.08 de branco) |
| `--surface-inset` | fundo de badge sobre imagem |
| `--line` / `--line-strong` / `--line-accent` | bordas neutra, forte e de accent |
| `--text-1` … `--text-4` | branco → cinza de apoio |
| `--text-accent` | texto em cima do accent do escopo |

---

## 2. Escopos de accent

Qualquer bloco marcado com `data-accent` recebe uma paleta própria — e **todo
componente dentro dele muda junto**, sem CSS extra, porque os componentes leem
tokens.

```html
<article class="project-card" data-accent="gold"> … </article>
```

Escopos disponíveis:

| Escopo | Paleta | Onde é usado |
|---|---|---|
| *(padrão)* | azul `#60a5fa` + teal `#2dd4bf` | site inteiro |
| `gold` | ouro `#d4af37`, ouro velho, champanhe, ouro rosé | projeto **E-commerce de Joias** |
| `teal` | teal como primário, azul como secundário | disponível, sem uso no momento |

Para criar um escopo novo, basta redefinir os quatro canais:

```css
[data-accent="emerald"] {
  --accent-rgb: 52 211 153;
  --accent-deep-rgb: 16 185 129;
  --accent-soft-rgb: 167 243 208;
  --accent-2-rgb: 96 165 250;
}
```

Nenhuma regra de componente precisa ser tocada.

---

## 3. Forma

```css
--r-xs: 4px;   --r-sm: 8px;    --r-md: 12px;   --r-lg: 20px;
--r-xl: 22px;  --r-2xl: 28px;  --r-3xl: 32px;  --r-full: 9999px;
```

Sombras em escala, mais dois glows que acompanham o accent do escopo:

```css
--shadow-sm | --shadow-md | --shadow-lg | --shadow-xl
--glow-accent | --glow-accent-lg
```

---

## 4. Tipografia

| Token | Valor |
|---|---|
| `--font-display` | Manrope — títulos |
| `--font-body` | Inter — corpo |
| `--font-mono` | Geist Mono — readouts, chips, labels técnicos |

Tamanhos fixos (`--fs-micro` … `--fs-base`) para UI técnica, e **escalas
fluidas com `clamp()`** para títulos de componente:

```css
--fs-title: clamp(2rem, 1.4rem + 2.4vw, 2.75rem);
--fs-subtitle: clamp(1.1rem, 0.98rem + 0.5vw, 1.35rem);
```

Títulos de card (`.project-title`, `.expertise-card-title`, `.step-card-title`)
escalam por `clamp()` em vez de overrides por breakpoint — não há mais
`@media` só para trocar `font-size`.

---

## 5. Movimento

```css
--ease-out-expo: cubic-bezier(0.16, 1, 0.3, 1);   /* entradas, reveal */
--ease-standard: cubic-bezier(0.4, 0, 0.2, 1);    /* hover, estado    */
--dur-fast: 0.2s | --dur-base: 0.3s | --dur-slow: 0.6s | --dur-slower: 1s
```

### `prefers-reduced-motion`

O site é fortemente animado (scan HUD, órbita, god rays, aurora). Com movimento
reduzido:

- todo loop infinito para e as transições ficam instantâneas;
- estados de entrada (`.reveal`, `.anim-fade-up`, `.step-card`) assumem a forma
  final — **nada some por ficar preso em `opacity: 0`**;
- decoração puramente cinética (scan line, retículo, EKG, partículas, god rays)
  é removida;
- o marquee vira lista estática que quebra em linhas — o conteúdo é
  informativo, então ele não é escondido, só parado;
- molduras cônicas animadas viram borda estática de accent.

---

## 6. Acessibilidade

- `--focus-ring` dá anel de foco visível a todo elemento interativo via
  `:focus-visible`, e acompanha o accent do escopo.
- `.skip-link` no topo do `<body>` leva direto ao conteúdo pelo teclado.
- Camadas em tokens (`--z-bg`, `--z-content`, `--z-menu`, `--z-nav`) para o
  empilhamento não virar números mágicos.

---

## 7. Como adicionar um projeto

1. Duplique um `<article class="project-card reveal">` na seção `#projetos`.
2. Alterne o lado da imagem (`lg:order-1` / `lg:order-2`) para manter o zigue-zague.
3. Atualize `/ NN` em `.project-image-num` e o número fantasma da seção.
4. Se o projeto tem identidade própria, marque `data-accent="…"` no `<article>`.
