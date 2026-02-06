---
phase: 12
plan: 02
subsystem: frontend-settings
tags: [settings-panel, cmdk, searchable-dropdown, openrouter, restart-prompt, i18n]

dependency-graph:
  requires: [12-01-PLAN]
  provides: [cmdk-searchable-model-dropdown, restart-prompt-on-provider-change]
  affects: []

tech-stack:
  added: [cmdk]
  patterns: [searchable-dropdown-pattern, provider-change-detection]

key-files:
  created: []
  modified:
    - frontend/components/settings/settings-panel.tsx
    - frontend/messages/en.json
    - frontend/messages/fr.json
    - frontend/package.json

decisions:
  - id: dec-12-02-01
    decision: "Use cmdk with manual filtering (shouldFilter=false) for full control over model search"
    rationale: "Allows case-insensitive partial matching across 400+ OpenRouter models"
  - id: dec-12-02-02
    decision: "Show restart prompt as yellow info box within content area"
    rationale: "Consistent with existing info/warning box patterns, non-modal for better UX"

metrics:
  duration: "4 min"
  completed: "2026-02-06"
---

# Phase 12 Plan 02: Searchable Model Dropdown and Restart Prompt Summary

Added cmdk-based searchable model dropdown for OpenRouter and restart prompt when LLM provider changes.

## One-liner

OpenRouter model selection enhanced with cmdk searchable dropdown supporting 400+ models, plus restart prompt when provider changes.

## Changes Made

### Package Installation

- Added `cmdk` package (v1.1.1) for searchable dropdown component

### i18n Translations

Added 5 new translation keys to both en.json and fr.json under Settings.providers:

| Key | English | French |
|-----|---------|--------|
| searchModels | "Search models..." | "Rechercher des modeles..." |
| noModelsFound | "No models found" | "Aucun modele trouve" |
| restartRequired | "Restart Required" | "Redemarrage requis" |
| restartDescription | "The LLM provider has changed..." | "Le fournisseur LLM a change..." |
| restartLater | "I'll restart later" | "Je redemarrerai plus tard" |

### OpenRouter Searchable Dropdown

Replaced standard `<select>` element with cmdk-based searchable dropdown:

- Button trigger showing selected model or "Select a model..."
- Dropdown overlay with search input and scrollable list
- Manual filtering (`shouldFilter={false}`) with case-insensitive matching
- `Command.Empty` shows "No models found" when search has no results
- Clicking a model updates settings and closes dropdown
- New state variables: `openrouterDropdownOpen`, `openrouterSearch`

### Restart Prompt Logic

Added provider change detection and restart prompt:

1. **State tracking:**
   - `originalProvider` - captures initial provider when settings load
   - `showRestartPrompt` - controls restart prompt visibility

2. **Reset on panel open:** Both states reset in `loadSettings()`

3. **Capture original provider:** useEffect captures `llm_provider` when `settingsLoadedFromApi` becomes true

4. **Save behavior modified:** Instead of always calling `onClose()`, now checks if provider changed:
   - If changed: shows restart prompt
   - If unchanged: closes panel normally

5. **Restart prompt UI:** Yellow info box with:
   - Title: "Restart Required"
   - Description explaining provider change needs restart
   - "I'll restart later" button that closes panel

## Verification Results

- `pnpm build` - Passed (no errors, only pre-existing useCallback warnings)
- `pnpm lint` - Passed (only pre-existing warnings)
- cmdk installed in package.json
- All 5 i18n keys present in en.json and fr.json
- OpenRouter uses Command component from cmdk
- showRestartPrompt state and UI in place

## Deviations from Plan

None - plan executed exactly as written.

## Next Phase Readiness

Phase 12 (Settings Panel UI) is now complete:
- All 4 LLM providers have full configuration forms
- OpenAI-compatible: base URL, optional API key, model dropdown
- OpenRouter: API key, searchable model dropdown with cmdk
- Restart prompt notifies users when provider change requires agent restart
- All strings properly internationalized (English and French)
