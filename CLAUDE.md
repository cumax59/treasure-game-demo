# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm install      # install dependencies
npm run dev      # start Vite dev server on http://localhost:3000 (auto-opens browser)
npm run build    # production build, output to ./build (not ./dist)
```

There is no test runner, linter, or type-check script configured, and no `tsconfig.json`. TypeScript is transpiled (not type-checked) by `@vitejs/plugin-react-swc`.

## Architecture

A single-page React 18 + TypeScript app built with Vite. It is a "Treasure Hunt" game with no backend, no router, and no state library — all game logic lives in `src/App.tsx`.

- **Game logic (`src/App.tsx`)**: three chests, one randomly holds treasure. Opening the treasure (+$100) or opening all chests ends the game; skeletons are -$50. State is three `useState` hooks (`boxes`, `score`, `gameEnded`); `motion/react` drives the flip/scale animations. This is the only file with real application code.
- **Entry**: `index.html` → `src/main.tsx` (`createRoot`) → `App`.
- **Assets**: imported directly as ES modules (`import x from './assets/x.png'` / `'./audios/x.mp3'`) so Vite fingerprints and bundles them. Add new images/audio under `src/assets` and `src/audios` and import them the same way.
- **UI library (`src/components/ui/`)**: a large set of shadcn/Radix components. The game currently uses only `button.tsx`. Treat these as a vendored toolkit, not hand-written code.

## Project-specific conventions (important)

These two patterns are non-obvious and will break things if missed:

1. **Versioned import specifiers.** Files in `src/components/ui/` import packages with the version appended, e.g. `import { Slot } from "@radix-ui/react-slot@1.1.2"`. These are not valid npm specifiers — they resolve only because every one is aliased in `vite.config.ts`. If you add a UI component or a new dependency that uses this style, add a matching alias in `vite.config.ts`. The `@/` alias maps to `./src`.

2. **Tailwind CSS is precompiled and committed — there is no build step.** `tailwindcss`/`postcss` are not dependencies. `src/main.tsx` imports `src/index.css`, which is a ~4600-line static, precompiled Tailwind stylesheet. Tailwind classes are **not** generated at build time. A utility class only works if its rule already exists in `src/index.css`; using a class that isn't there silently produces no styling. (`src/styles/globals.css` holds the design-token source/`:root` variables but is **not imported** anywhere.)

## Notes

- `README.md` is a Claude Code demo walkthrough/script, not project documentation — don't treat its instructions (SQLite auth, Vercel/GitHub Pages deploy, etc.) as existing features.
- Comment convention used in this repo (from prior work): add a one-line comment atop every new function summarizing its purpose and documenting input/output parameters.
- `Attributions.md` lists required credit for the bundled sound/icon assets.
