# Padrão de Replicação UX/UI Interativo

> Sistema de design extraído do projeto **Incubadora Platform Architecture**.
> Usado para criar apresentações técnicas interativas — diagramas C4, fluxos de
> usuário, specs — com visual dark, animado e profissional.

---

## 1. Stack Técnica

| Camada | Tecnologia |
|---|---|
| Framework | **Next.js 14** (App Router, `'use client'` em todos os componentes interativos) |
| UI | **Tailwind CSS 3** + CSS custom properties |
| Animação | **Framer Motion 11** (`motion`, `AnimatePresence`) |
| Ícones | **lucide-react** |
| Utils | **clsx** + **tailwind-merge** |
| Tipagem | **TypeScript 5** estrito |

### Setup mínimo `package.json`

```json
{
  "dependencies": {
    "next": "^14.2.0",
    "react": "^18.3.0",
    "react-dom": "^18.3.0",
    "framer-motion": "^11.0.0",
    "lucide-react": "^0.400.0",
    "clsx": "^2.1.0",
    "tailwind-merge": "^2.3.0"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "@types/react": "^18.3.0",
    "@types/react-dom": "^18.3.0",
    "autoprefixer": "^10.4.0",
    "postcss": "^8.4.0",
    "tailwindcss": "^3.4.0",
    "typescript": "^5.4.0"
  }
}
```

---

## 2. Design Tokens

### 2.1 Cores

```css
/* globals.css — base */
body {
  background: #0A0A12;     /* near-black com tint azul */
  color: #F1F5F9;          /* slate-100 — texto principal */
}

/* Tríade de acento principal */
--accent-emerald: #10B981;  /* primário — ações, destaque */
--accent-teal:     #14B8A6;  /* secundário — gradientes */
--accent-amber:    #F59E0B;  /* terciário — contrastes pontuais */

/* Cores por categoria (usadas em user flow, specs) */
--cat-purple: #8B5CF6;
--cat-blue:   #3B82F6;
--cat-emerald:#10B981;
--cat-amber:  #F59E0B;

/* Texto secundário */
--text-muted: slate-400 / slate-500 / slate-600;
```

### 2.2 Tipografia

```css
font-family: system-ui, -apple-system, sans-serif;
/* Tamanhos via Tailwind: text-sm, text-lg, text-3xl, text-5xl */
```

### 2.3 Glass Card (cartão base)

```css
.glass-card {
  background: rgba(255, 255, 255, 0.03);
  backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.08);
}
```

Uso Tailwind inline: `bg-white/[0.03] backdrop-blur-md border border-white/[0.08]`

### 2.4 Section Divider

```css
.section-divider {
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(16,185,129,0.3), transparent);
}
```

---

## 3. Sistema de Animação

### 3.1 Keyframes CSS (`globals.css`)

```css
@keyframes float {
  0%, 100% { transform: translateY(0px); }
  50%      { transform: translateY(-10px); }
}
@keyframes pulse-glow {
  0%, 100% { box-shadow: 0 0 20px rgba(16,185,129,0.15); }
  50%      { box-shadow: 0 0 40px rgba(16,185,129,0.3); }
}
@keyframes dash-flow {
  to { stroke-dashoffset: -20; }
}
@keyframes fade-in-up {
  from { opacity: 0; transform: translateY(30px); }
  to   { opacity: 1; transform: translateY(0); }
}

/* Utilities */
.animate-float       { animation: float 3s ease-in-out infinite; }
.animate-pulse-glow  { animation: pulse-glow 2s ease-in-out infinite; }
.animate-dash        { animation: dash-flow 1s linear infinite; }
.animate-fade-in-up  { animation: fade-in-up 0.6s ease-out forwards; }
```

### 3.2 Framer Motion — Variants padrão

Repetidos em **cada** componente que precisa de stagger:

```tsx
// Container orquestra o stagger dos filhos
const containerVariants = {
  hidden:  { opacity: 0 },
  visible: { opacity: 1, transition: { staggerChildren: 0.05 } },
};

// Cada item dentro do container
const itemVariants = {
  hidden:  { opacity: 0, y: 10 },
  visible: { opacity: 1, y: 0 },
};
```

Uso:

```tsx
<motion.ul variants={containerVariants} initial="hidden" whileInView="visible" viewport={{ once: true }}>
  {items.map((item, i) => (
    <motion.li key={i} variants={itemVariants}>…</motion.li>
  ))}
</motion.ul>
```

### 3.3 Entrada de seção (hero, headers)

```tsx
<motion.div
  initial={{ opacity: 0, y: 40 }}
  animate={{ opacity: 1, y: 0 }}   // ou whileInView para scroll-triggered
  transition={{ duration: 0.8 }}
>
```

### 3.4 Conditional render animado (detail panels)

```tsx
<AnimatePresence mode="wait">
  {activeNode && (
    <motion.div
      initial={{ opacity: 0, scale: 0.95 }}
      animate={{ opacity: 1, scale: 1 }}
      exit={{ opacity: 0, scale: 0.95 }}
      transition={{ duration: 0.2 }}
    >
      …
    </motion.div>
  )}
</AnimatePresence>
```

---

## 4. Componentes (Padrões)

### 4.1 Page Shell (`page.tsx`)

Estrutura canónica de uma página completa:

```tsx
'use client';

import Navigation from '@/components/arch/Navigation';
import HeroSection from '@/components/arch/HeroSection';
// … imports
import Footer from '@/components/arch/Footer';

export default function Page() {
  return (
    <div className="min-h-screen bg-[#0A0A12]">
      <Navigation />
      <HeroSection />

      <div className="section-divider" />
      {/* Cada secção precedida por divider */}

      <Footer />
    </div>
  );
}
```

### 4.2 Navigation — scroll-spy + mobile

```tsx
'use client';
import { useState, useEffect } from 'react';
import { motion, AnimatePresence } from 'framer-motion';
import { Menu, X } from 'lucide-react';

const navItems = [
  { id: 'hero', label: 'Visão Geral' },
  { id: 'problema', label: 'Problema' },
  // … um por secção
];

export default function Navigation() {
  const [active, setActive] = useState('hero');
  const [scrolled, setScrolled] = useState(false);
  const [mobileOpen, setMobileOpen] = useState(false);

  useEffect(() => {
    const handleScroll = () => {
      setScrolled(window.scrollY > 50);
      const sections = navItems.map(item => ({
        id: item.id,
        top: document.getElementById(item.id)?.offsetTop - 100 ?? 0,
      }));
      const current = sections.reduce((prev, curr) =>
        window.scrollY >= curr.top ? curr : prev
      );
      if (current) setActive(current.id);
    };
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);

  return (
    <>
      <motion.nav
        initial={{ y: -80 }}
        animate={{ y: 0 }}
        className={`fixed top-0 inset-x-0 z-50 transition-all duration-300 ${
          scrolled ? 'bg-[#0A0A12]/80 backdrop-blur-lg border-b border-white/5' : ''
        }`}
      >
        <div className="max-w-7xl mx-auto px-4 sm:px-6">
          {/* Desktop links + mobile hamburger */}
        </div>
      </motion.nav>

      <AnimatePresence>
        {mobileOpen && (
          <motion.div
            initial={{ x: '100%' }}
            animate={{ x: 0 }}
            exit={{ x: '100%' }}
            className="fixed inset-y-0 right-0 z-50 w-72 bg-[#0F0F1A] border-l border-white/10 p-6"
          >
            {/* Mobile links */}
          </motion.div>
        )}
      </AnimatePresence>
    </>
  );
}
```

### 4.3 Section Header (cabeçalho de secção)

Padrão repetido em todas as secções:

```tsx
<section id="SECTION_ID" className="relative py-24 sm:py-32">
  <div className="max-w-7xl mx-auto px-4 sm:px-6">
    {/* Eyebrow badge */}
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      whileInView={{ opacity: 1, y: 0 }}
      viewport={{ once: true }}
      className="inline-flex items-center gap-2 px-3 py-1 rounded-full
                 bg-emerald-500/10 border border-emerald-500/20
                 text-emerald-400 text-xs font-medium mb-4"
    >
      <EyebrowIcon size={12} />
      CATEGORIA
    </motion.div>

    {/* Gradient title */}
    <motion.h2
      initial={{ opacity: 0, y: 20 }}
      whileInView={{ opacity: 1, y: 0 }}
      viewport={{ once: true }}
      className="text-3xl sm:text-5xl font-bold mb-4"
    >
      <span className="gradient-text">Título da Secção</span>
    </motion.h2>

    {/* Subtitle */}
    <motion.p className="text-slate-400 text-base sm:text-lg max-w-2xl mb-12">
      Descrição do que esta secção cobre.
    </motion.p>

    {/* Conteúdo específico */}
  </div>
</section>
```

### 4.4 Hero Section

```tsx
<section id="hero" className="relative min-h-screen flex items-center justify-center overflow-hidden pt-16">
  {/* Background blobs — blur radial sutil */}
  <div className="absolute inset-0">
    <div className="absolute top-1/4 left-1/4 w-96 h-96 bg-emerald-500/5 rounded-full blur-3xl" />
    <div className="absolute bottom-1/4 right-1/4 w-96 h-96 bg-teal-500/5 rounded-full blur-3xl" />
    {/* Grid pattern opcional */}
  </div>

  <div className="relative max-w-5xl mx-auto px-4 sm:px-6 text-center">
    {/* Badge pill */}
    {/* H1 com gradient-text */}
    {/* P subtitle */}
    {/* Feature pills — grid de ícones */}
    {/* Tech stack pills */}
    {/* CTA button com animate-pulse-glow */}
  </div>
</section>
```

### 4.5 Interactive SVG Diagram (C4 pattern)

Para diagramas clicáveis com detail panel:

```tsx
'use client';
import { useState } from 'react';
import { motion, AnimatePresence } from 'framer-motion';

// 1. Definir metadata dos nós num Record tipado
interface NodeDetail {
  title: string;
  description: string;
  tech: string[];
  role: string;
}
const nodeDetails: Record<string, NodeDetail> = {
  platform: { title: '…', description: '…', tech: ['…'], role: '…' },
  // …
};

// 2. Componente C4Node reutilizável
function C4Node({ id, label, icon: Icon, x, y, width = 180, height = 70,
  color = '#10B981', bgColor = 'rgba(16,185,129,0.1)', active, onClick }: {
  id: string; label: string; icon: React.ElementType;
  x: number; y: number; width?: number; height?: number;
  color?: string; bgColor?: string;
  active: boolean; onClick: () => void;
}) {
  return (
    <g className="c4-node" style={{ cursor: 'pointer' }} onClick={onClick}>
      {/* rect com fill=bgColor, stroke=color */}
      {/* icon como <foreignObject> ou SVG <text> */}
      {/* label */}
    </g>
  );
}

// 3. Connection com animate-dash
function Connection({ x1, y1, x2, y2, label, color = '#334155' }) {
  return (
    <>
      <line x1={x1} y1={y1} x2={x2} y2={y2}
        stroke={color} strokeWidth={1.5}
        strokeDasharray="8 4"
        className="c4-connection" />
      {label && <text>…</text>}
    </>
  );
}

// 4. Section principal com SVG + detail panel
export default function Diagram() {
  const [activeNode, setActiveNode] = useState<string | null>(null);
  const detail = activeNode ? nodeDetails[activeNode] : null;

  return (
    <section id="diagram-id" className="relative py-24 sm:py-32">
      {/* Section header (padrão 4.3) */}
      <div className="grid lg:grid-cols-[1fr_360px] gap-8">
        {/* SVG canvas */}
        <svg viewBox="0 0 900 500" className="w-full">
          <C4Node id="…" active={activeNode === '…'} onClick={() => setActiveNode('…')} … />
          <Connection … />
        </svg>

        {/* Detail panel — AnimatePresence */}
        <AnimatePresence mode="wait">
          {detail && (
            <motion.div
              key={activeNode}
              initial={{ opacity: 0, x: 20 }}
              animate={{ opacity: 1, x: 0 }}
              exit={{ opacity: 0, x: 20 }}
              className="glass-card rounded-2xl p-6"
            >
              {/* title, description, tech[], role */}
            </motion.div>
          )}
        </AnimatePresence>
      </div>
    </section>
  );
}
```

CSS de interação SVG:

```css
.c4-node { transition: all 0.3s ease; cursor: pointer; }
.c4-node:hover { filter: brightness(1.3); transform: scale(1.05); }
.c4-node.active {
  filter: brightness(1.4) drop-shadow(0 0 12px rgba(16,185,129,0.6));
}
.c4-connection {
  stroke-dasharray: 8 4;
  animation: dash-flow 1s linear infinite;
}
```

### 4.6 User Flow — tabs + step cards

```tsx
export default function UserFlowSection() {
  const [activePhase, setActivePhase] = useState(0);
  const [activeStep, setActiveStep] = useState(0);

  return (
    <section id="userflow" className="relative py-24 sm:py-32">
      {/* Phase tabs — clique muda activePhase */}
      <div className="flex gap-2 mb-8">
        {useCases.map((uc, i) => (
          <button
            key={i}
            onClick={() => { setActivePhase(i); setActiveStep(0); }}
            className={`px-4 py-2 rounded-xl border transition-all ${
              activePhase === i
                ? 'bg-white/10 border-white/20'
                : 'border-white/5 opacity-50 hover:opacity-80'
            }`}
          >
            <uc.icon size={16} style={{ color: uc.color }} />
            {uc.phase}
          </button>
        ))}
      </div>

      {/* Steps do phase ativo — AnimatePresence com mode="wait" */}
      <AnimatePresence mode="wait">
        <motion.div
          key={activePhase}
          initial={{ opacity: 0, y: 10 }}
          animate={{ opacity: 1, y: 0 }}
          exit={{ opacity: 0, y: -10 }}
        >
          {/* Step cards com navegação prev/next */}
        </motion.div>
      </AnimatePresence>

      {/* Comparison table — antes vs depois */}
    </section>
  );
}
```

### 4.7 Specs Grid — cards de categoria

```tsx
const specs = [
  { category: 'Infraestrutura', icon: Server, color: '#10B981', items: [...] },
  // …
];

<div className="grid md:grid-cols-2 lg:grid-cols-3 gap-6">
  {specs.map((spec, i) => (
    <motion.div
      key={i}
      variants={itemVariants}
      className="glass-card rounded-2xl p-6"
    >
      <div className="flex items-center gap-3 mb-4">
        <div className="w-10 h-10 rounded-xl flex items-center justify-center"
             style={{ background: `${spec.color}20`, color: spec.color }}>
          <spec.icon size={20} />
        </div>
        <h3 className="text-white font-semibold">{spec.category}</h3>
      </div>
      <ul className="space-y-2">
        {spec.items.map((item, j) => (
          <li key={j} className="text-slate-400 text-sm flex items-center gap-2">
            <span className="w-1 h-1 rounded-full" style={{ background: spec.color }} />
            {item}
          </li>
        ))}
      </ul>
    </motion.div>
  ))}
</div>
```

### 4.8 Footer

```tsx
<footer className="relative py-16 border-t border-white/5">
  <div className="max-w-7xl mx-auto px-4 sm:px-6">
    <div className="grid md:grid-cols-3 gap-8 mb-12">
      {/* Col 1: Brand + descrição */}
      {/* Col 2: Links de navegação */}
      {/* Col 3: Tech stack pills */}
    </div>
    <div className="section-divider mb-8" />
    {/* Copyright */}
  </div>
</footer>
```

---

## 5. Estrutura de Ficheiros

```
src/
├── app/
│   ├── page.tsx          # Page shell — compõe secções + dividers
│   ├── layout.tsx        # metadata + globals.css import
│   └── globals.css       # keyframes + utilidades + tema base
├── components/
│   └── arch/
│       ├── Navigation.tsx       # scroll-spy nav + mobile drawer
│       ├── HeroSection.tsx      # full-viewport hero
│       ├── ProblemSolution.tsx  # problema → solução comparativo
│       ├── C4SystemContext.tsx  # SVG interativo nível 1
│       ├── C4Container.tsx      # SVG interativo nível 2
│       ├── C4Deployment.tsx     # SVG interativo nível 3
│       ├── UserFlowSection.tsx  # tabs + step cards + comparison
│       ├── TechSpecsSection.tsx # grid de specs categorizados
│       └── Footer.tsx           # footer 3-col
├── tailwind.config.js
├── postcss.config.js
├── tsconfig.json
└── next.config.js
```

---

## 6. Checklist de Replicação

Para criar uma nova apresentação interativa seguindo este padrão:

1. **Setup** — Next 14 + deps listadas acima + `globals.css` com keyframes e tema
2. **Navigation** — lista de `navItems` (um por secção), scroll-spy, mobile drawer
3. **Hero** — blobs de fundo, badge, H1 gradient, pills de features, CTA
4. **Secções de conteúdo** — cada uma com:
   - `<section id="…">` com `py-24 sm:py-32` + `max-w-7xl mx-auto px-4 sm:px-6`
   - Header (eyebrow → gradient title → subtitle)
   - Conteúdo com `containerVariants`/`itemVariants` stagger
   - `section-divider` entre secções
5. **Diagramas SVG** — `C4Node` + `Connection` reutilizáveis, detail panel com `AnimatePresence`
6. **User flow** — phase tabs + step cards animados + comparison table
7. **Footer** — 3-col grid + divider + copyright
8. **Verificação** — `bun run dev`, confirmar scroll-spy, animações, interações SVG, responsividade mobile

---

## 7. Classes Tailwind Recorrentes (quick reference)

| Uso | Classe |
|---|---|
| Fundo base | `bg-[#0A0A12]` |
| Glass card | `bg-white/[0.03] backdrop-blur-md border border-white/[0.08]` |
| Texto muted | `text-slate-400` / `text-slate-500` |
| Accent emerald | `text-emerald-400` / `bg-emerald-500/10` / `border-emerald-500/20` |
| Gradient text | `.gradient-text` (class CSS) |
| Pill badge | `inline-flex items-center gap-2 px-3 py-1 rounded-full bg-X/10 border border-X/20` |
| Icon wrapper | `w-10 h-10 rounded-xl flex items-center justify-center` com `style={{ background: color+'20', color }}` |
| Section padding | `py-24 sm:py-32` |
| Container | `max-w-7xl mx-auto px-4 sm:px-6` |
