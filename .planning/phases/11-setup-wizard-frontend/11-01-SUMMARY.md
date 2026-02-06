---
phase: 11-setup-wizard-frontend
plan: 01
subsystem: ui
tags: [react, next-intl, setup-wizard, provider-selection, i18n]

# Dependency graph
requires:
  - phase: 10-connection-testing-endpoints
    provides: Backend test endpoints for OpenAI-compatible and OpenRouter
  - phase: 09-settings-schema-extension
    provides: Settings keys for new provider fields
provides:
  - Setup wizard UI for 4 LLM providers (Ollama, Groq, OpenAI-compatible, OpenRouter)
  - Connection testing integration with Phase 10 endpoints
  - Model selection after successful connection test
  - i18n support for EN, FR, IT
affects: [12-settings-panel-frontend, setup-wizard-e2e-tests]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - Provider grid pattern (2x2 layout for 4 choices)
    - Test function pattern (testOpenAICompatible, testOpenRouter following existing testOllama pattern)

key-files:
  modified:
    - frontend/components/setup/setup-wizard.tsx
    - frontend/components/setup/provider-step.tsx
    - frontend/messages/en.json
    - frontend/messages/fr.json
    - frontend/messages/it.json

key-decisions:
  - "OpenAI-compatible requires base_url + model but NOT api_key (optional for local servers)"
  - "OpenRouter requires api_key + model (same as Groq pattern)"
  - "Provider grid uses existing 2-column layout with 4 buttons (2x2)"

patterns-established:
  - "Provider step form sections: input + test button + success/error + model select"
  - "STT notes at bottom of each provider section explaining linked STT service"

# Metrics
duration: 4min
completed: 2026-02-06
---

# Phase 11 Plan 01: Setup Wizard Provider Extension Summary

**Extended setup wizard with OpenAI-compatible and OpenRouter provider choices including connection testing, model selection, and EN/FR/IT translations**

## Performance

- **Duration:** 4 min
- **Started:** 2026-02-06T00:00:00Z
- **Completed:** 2026-02-06T00:04:00Z
- **Tasks:** 3
- **Files modified:** 5

## Accomplishments
- Extended SetupData interface with 5 new fields for OpenAI-compatible and OpenRouter
- Expanded provider grid from 2 to 4 options in 2x2 layout
- Added test functions that call Phase 10 backend endpoints
- Added form sections with base URL, API key inputs, and model selection dropdowns
- Added all new i18n keys for EN, FR, IT

## Task Commits

Each task was committed atomically:

1. **Task 1: Extend SetupData interface and canProceed validation** - `278a32c` (feat)
2. **Task 2: Add provider grid and form sections with test functions** - `ffdd3c6` (feat)
3. **Task 3: Add i18n translations for all three languages** - `c89c19d` (feat)

## Files Created/Modified
- `frontend/components/setup/setup-wizard.tsx` - Extended SetupData interface, INITIAL_DATA, canProceed logic
- `frontend/components/setup/provider-step.tsx` - 4-provider grid, form sections, testOpenAICompatible, testOpenRouter
- `frontend/messages/en.json` - English translations for new providers
- `frontend/messages/fr.json` - French translations for new providers
- `frontend/messages/it.json` - Italian translations for new providers

## Decisions Made
- OpenAI-compatible API key is optional (for local servers without auth)
- OpenRouter follows Groq pattern (API key required, link to get key)
- Used existing 2-column grid for 4 buttons (2x2 layout)
- STT notes added explaining provider coupling (OpenAI-compatible -> Speaches, OpenRouter -> Groq Whisper)

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness
- Setup wizard UI complete for all 4 LLM providers
- Ready for Phase 12 settings panel extension (same providers in settings after initial setup)
- Manual verification should occur in Phase 12 visual verification checkpoint

---
*Phase: 11-setup-wizard-frontend*
*Completed: 2026-02-06*
