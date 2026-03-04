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

## Planned Structure (per Cooker_plan.md)

```
src/
├── api/             # fetch/axios wrapper, all backend calls
├── components/
│   ├── ui/          # shadcn-vue primitives
│   ├── ingredients/ # fridge/ingredient management
│   ├── recipe/      # recipe display with streaming markdown
│   └── chat/        # LLM chat window
├── composables/
│   ├── useVoiceInput.ts  # MediaRecorder → Whisper API
│   └── useSSE.ts         # Server-Sent Events for LLM streaming
├── stores/          # Pinia stores (ingredients, recipe, chat, auth)
├── views/           # Route-level components
└── types/           # Shared TypeScript interfaces
```

## Key Implementation Notes

- LLM responses stream via **SSE** (not WebSocket). Use `useSSE.ts` composable for all streaming.
- LLM returns **structured JSON** (function calling / structured outputs) for recipe data — `{ title, servings, time_minutes, ingredients[], steps[] }`. Each ingredient includes `available: boolean` to split "have at home" vs "need to buy".
- Voice input flow: `MediaRecorder` → send audio blob to backend → Whisper transcription → second LLM call to parse free-form speech into a structured ingredient list.
- **All** LLM and Whisper API calls go through the backend. Never call external AI APIs directly from the frontend.
- Backend URL comes from `VITE_API_URL` env variable.
