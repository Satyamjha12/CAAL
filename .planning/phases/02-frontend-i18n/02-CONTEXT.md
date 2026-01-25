# Phase 2: Frontend i18n - Context

**Gathered:** 2026-01-25
**Status:** Ready for planning

<domain>
## Phase Boundary

Add internationalization to the Next.js frontend using next-intl. Users see all UI text in their configured language (EN or FR). A language selector in settings allows changing the language. The language setting comes from the backend (Phase 1).

</domain>

<decisions>
## Implementation Decisions

### Language Selector UX
- Placement: Claude decides most logical location in settings panel
- After changing language: Save triggers page reload to apply new locale
- Dropdown displays languages in their native names ("English", "Français")
- Visual feedback: Toast notification ("Language updated") before reload

### Translation Coverage
- Scope: All visible UI text (settings, main screen, modals, toasts)
- Error messages: Keep in English, add error codes for support
- Technical terms: Keep brand names in English (Groq, Ollama, Whisper, STT, TTS)
- Placeholders: Claude decides based on standard i18n practices

### Locale Detection & Defaults
- Source of truth: Backend /api/settings only (no localStorage, no URL routing)
- Missing translation fallback: Show English text (silent fallback)
- Loading strategy: Load active language only (smaller bundle)
- Initial state: Default to English while waiting for settings API

### Claude's Discretion
- Exact placement of language selector in settings panel
- Placeholder and hint text localization approach
- Message file namespacing and structure (standard next-intl patterns)
- Loading skeleton and error state design

</decisions>

<specifics>
## Specific Ideas

- Toast before reload provides clear feedback that the change was saved
- Native language names help French users recognize their language immediately
- English error codes help with support even when UI is in French

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 02-frontend-i18n*
*Context gathered: 2026-01-25*
