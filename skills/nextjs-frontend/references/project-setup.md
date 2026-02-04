# Project Setup

## Create New Project

### With Preset (Recommended)

```bash
pnpm dlx shadcn@latest create \
  --preset "https://ui.shadcn.com/init?base=radix&style=vega&iconLibrary=lucide" \
  --template next
```

### Full Preset URL Options

```
https://ui.shadcn.com/init?
  base=radix
  &style=vega|nova|maia|lyra|mira
  &baseColor=neutral|slate|gray|zinc|stone
  &theme=neutral|blue|green|orange|red|rose|violet
  &iconLibrary=lucide|tabler|hugeicons|phosphor
  &font=inter|geist|system
  &menuAccent=subtle|bold
  &menuColor=default|accent
  &radius=default|sm|md|lg|xl
  &template=next
```

### Example Presets

**Minimal (vega + lucide)**:
```bash
pnpm dlx shadcn@latest create \
  --preset "https://ui.shadcn.com/init?base=radix&style=vega&iconLibrary=lucide&font=inter" \
  --template next
```

**Bold (nova + tabler)**:
```bash
pnpm dlx shadcn@latest create \
  --preset "https://ui.shadcn.com/init?base=radix&style=nova&iconLibrary=tabler&theme=violet" \
  --template next
```

**Soft (maia + phosphor)**:
```bash
pnpm dlx shadcn@latest create \
  --preset "https://ui.shadcn.com/init?base=radix&style=maia&iconLibrary=phosphor&radius=lg" \
  --template next
```

## Add Components

```bash
# Single component
pnpm dlx shadcn@latest add button

# Multiple components
pnpm dlx shadcn@latest add button card input

# All components
pnpm dlx shadcn@latest add --all
```

## Common Dependencies

```bash
# Forms
pnpm add react-hook-form @hookform/resolvers zod

# AI
pnpm add ai @ai-sdk/anthropic

# Animation
pnpm add motion              # For Motion
pnpm add gsap @gsap/react    # For GSAP

# Icons (pick one)
pnpm add lucide-react        # Default
```

## Project Structure After Setup

```
project/
├── app/
│   ├── globals.css         # Theme tokens
│   ├── layout.tsx          # Root layout
│   └── page.tsx            # Home page
├── components/
│   └── ui/                 # shadcn components
├── lib/
│   └── utils.ts            # cn() helper
├── public/
├── components.json         # shadcn config
├── tailwind.config.ts
├── tsconfig.json
└── package.json
```

## Commands Reference

| Task | Command |
|------|---------|
| Install deps | `pnpm install` |
| Add package | `pnpm add package` |
| Dev server | `pnpm dev` |
| Build | `pnpm build` |
| Start prod | `pnpm start` |
| Add shadcn component | `pnpm dlx shadcn@latest add component` |
| Create project | `pnpm dlx shadcn@latest create ...` |
