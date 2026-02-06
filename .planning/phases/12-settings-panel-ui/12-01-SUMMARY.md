---
phase: 12
plan: 01
subsystem: frontend-settings
tags: [settings-panel, provider-configuration, openai-compatible, openrouter]

dependency-graph:
  requires: [08-backend-provider-foundation, 09-settings-schema-extension, 10-connection-testing-endpoints, 11-setup-wizard-frontend]
  provides: [settings-panel-4-providers, openai-compatible-test-function, openrouter-test-function]
  affects: [12-02-PLAN]

tech-stack:
  added: []
  patterns: [provider-form-pattern, flex-wrap-toggle-layout]

key-files:
  created: []
  modified:
    - frontend/components/settings/settings-panel.tsx

decisions:
  - id: dec-12-01-01
    decision: "Use flex-wrap for 4 provider buttons"
    rationale: "Allows natural wrapping on mobile while keeping layout simple"
  - id: dec-12-01-02
    decision: "OpenAI-compatible uses Speaches for STT (local), OpenRouter uses Groq Whisper (cloud)"
    rationale: "Matches the pattern of local vs cloud providers established in prior phases"

metrics:
  duration: "3 min"
  completed: "2026-02-06"
---

# Phase 12 Plan 01: Settings Panel Infrastructure Summary

Extended settings panel infrastructure to support OpenAI-compatible and OpenRouter providers with test functions and 4-button toggle.

## One-liner

Settings panel extended with 4 LLM providers (Ollama, Groq, OpenAI-compatible, OpenRouter), dedicated test functions, and flex-wrap button layout.

## Changes Made

### Settings Interface Extended

Added new fields to `Settings` interface:
- `llm_provider` union type extended: `'ollama' | 'groq' | 'openai_compatible' | 'openrouter'`
- `openai_base_url: string` - Base URL for OpenAI-compatible API servers
- `openai_api_key: string` - Optional API key for authenticated servers
- `openai_model: string` - Selected model from the server
- `openrouter_api_key: string` - Required API key for OpenRouter
- `openrouter_model: string` - Selected model from OpenRouter

All new fields default to empty strings in `DEFAULT_SETTINGS`, matching the backend schema.

### Test State and Functions

Added state management:
- `openaiModels` / `setOpenaiModels` - Available models from OpenAI-compatible server
- `openrouterModels` / `setOpenrouterModels` - Available models from OpenRouter
- `openaiTest` / `setOpenaiTest` - Test status for OpenAI-compatible
- `openrouterTest` / `setOpenrouterTest` - Test status for OpenRouter

Added test functions:
- `testOpenAICompatible()` - POSTs to `/api/setup/test-openai-compatible` with base_url and optional api_key
- `testOpenRouter()` - POSTs to `/api/setup/test-openrouter` with api_key

Added auto-fetch useEffect for OpenRouter models when API key is available.

### Provider Toggle UI

Changed from `inline-flex` with 2 buttons to `flex flex-wrap gap-2` with 4 buttons:
- Ollama
- Groq
- OpenAI Compatible
- OpenRouter

Each button updates `llm_provider` state. Removed stt_provider auto-setting from buttons (handled by STT info display).

### Provider Form Sections

Added conditional rendering for each provider with full form sections:
- OpenAI-compatible: Base URL with test button, optional API key, model dropdown
- OpenRouter: API key with test button, model dropdown, link to openrouter.ai/keys

STT info box updated to show Speaches for ollama/openai_compatible and Groq Whisper for groq/openrouter.

## Verification Results

- `pnpm build` - Passed (no TypeScript errors)
- `pnpm lint` - Passed (only pre-existing warnings about useCallback deps)
- Settings interface includes all 5 new fields
- DEFAULT_SETTINGS has matching empty string defaults
- Test functions exist and reference correct endpoints
- 4 provider buttons visible in toggle with flex-wrap layout

## Deviations from Plan

None - plan executed exactly as written.

## Next Phase Readiness

Plan 12-02 can now build the configuration form sections for each provider using the infrastructure established here:
- Settings interface extended with new fields
- Test functions wired to backend endpoints
- Provider toggle shows all 4 options
- State management in place for models and test status
