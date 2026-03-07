# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev          # Dev server (Vite)
npm run build        # Type-check + build
npm run type-check   # vue-tsc only
npm run lint         # oxlint --fix, then eslint --fix (runs in sequence)
```

No test runner is configured yet.

## Architecture

**Stack:** Vue 3 + TypeScript + Pinia + Vue Router. Tailwind and shadcn-vue are planned but not yet installed.

**Path alias:** `@/` maps to `src/`.

**Linting:** Two-layer setup — oxlint runs first (fast, configured via `.oxlintrc.json`), then ESLint handles Vue-specific rules. Both auto-fix on `npm run lint`. ESLint config is in `eslint.config.ts` (flat config format).

**TypeScript:** `noUncheckedIndexedAccess` is enabled — array/object lookups return `T | undefined`.

## Project Structure

Hybrid modular architecture: shared layers (api, stores, types, composables) are flat; components are grouped by feature.

```
src/
├── api/                    # Backend communication (pure functions, no Vue imports)
│   ├── client.ts           #   fetch wrapper (VITE_API_URL, error handling)
│   ├── recipes.ts          #   Recipe CRUD + generation endpoints
│   ├── ingredients.ts      #   Ingredient CRUD endpoints
│   ├── chat.ts             #   Chat message endpoints
│   └── voice.ts            #   Audio upload → transcription
├── composables/            # Reusable reactive logic
│   ├── useSSE.ts           #   SSE connection, chunk parsing, reconnect
│   ├── useVoiceInput.ts    #   MediaRecorder lifecycle + audio capture
│   ├── useRecipeStream.ts  #   Orchestrates SSE + recipe store for generation
│   └── useMediaQuery.ts    #   Reactive mobile/desktop detection
├── stores/                 # Pinia stores (global singletons, flat)
│   ├── ingredients.ts      #   Product list, categories, CRUD
│   ├── recipes.ts          #   Generated recipes, favorites, current recipe
│   ├── chat.ts             #   Chat history per recipe, streaming state
│   └── ui.ts               #   Theme (light/dark), sidebar, loading
├── types/                  # Shared TypeScript interfaces (no src/ imports)
│   ├── ingredient.ts       #   Ingredient, IngredientCategory
│   ├── recipe.ts           #   Recipe, RecipeIngredient, RecipeStep
│   ├── chat.ts             #   ChatMessage, ChatRole
│   └── api.ts              #   ApiResponse<T>, ApiError, SSEEvent
├── components/
│   ├── ui/                 #   Design system primitives (stateless, props/events only)
│   ├── layout/             #   AppShell, BottomNav, SideNav, PageHeader
│   ├── home/               #   RecipePrompt, CategoryChips, ProductsSummary, FavoritesList
│   ├── fridge/             #   CategoryTabs, IngredientList, IngredientItem, AddIngredient
│   ├── recipe/             #   RecipeHero, RecipeDescription, IngredientChecklist, StepList
│   └── chat/               #   ChatPanel, ChatMessage, ChatInput
├── views/                  # Route-level components (thin orchestrators)
│   ├── HomeView.vue
│   ├── FridgeView.vue
│   ├── RecipeView.vue
│   └── HistoryView.vue
├── router/
│   └── index.ts            # Routes with lazy-loading
├── assets/
├── App.vue
└── main.ts
```

### Layer rules
- **ui/** never imports stores, api, or composables — props/events only
- **api/** — pure functions, never import Vue or stores
- **types/** — never import anything from src/
- **composables** — orchestrate api + stores
- **views** — thin, only compose feature components

### Routes
```
/              → HomeView       (name: 'home')
/fridge        → FridgeView     (name: 'fridge')
/recipe/:id    → RecipeView     (name: 'recipe-detail')
/history       → HistoryView    (name: 'history')
```

## Key Implementation Notes

- LLM responses stream via **SSE** (not WebSocket). Use `useSSE.ts` composable for all streaming.
- LLM returns **structured JSON** (function calling / structured outputs) for recipe data — `{ title, servings, time_minutes, ingredients[], steps[] }`. Each ingredient includes `available: boolean` to split "have at home" vs "need to buy".
- Voice input flow: `MediaRecorder` → send audio blob to backend → Whisper transcription → second LLM call to parse free-form speech into a structured ingredient list.
- **All** LLM and Whisper API calls go through the backend. Never call external AI APIs directly from the frontend.
- Backend URL comes from `VITE_API_URL` env variable.
