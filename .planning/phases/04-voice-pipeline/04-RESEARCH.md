# Phase 4: Voice Pipeline - Research

**Researched:** 2026-01-26
**Domain:** Voice pipeline i18n (STT, TTS, prompts, greetings)
**Confidence:** HIGH

## Summary

The CAAL voice pipeline has five i18n touchpoints: STT language parameter, TTS voice mapping, system prompts, wake greetings, and date/time formatting. The `language` setting already exists in `settings.json` (from Phase 1) but is unused by the voice pipeline. Each component needs a targeted change to read the language setting and adapt behavior.

**Primary recommendation:** Read language from settings at agent startup, pass to STT, select TTS voice per language, load per-language prompts, map greetings per language, and localize date/time formatting. The LLM will naturally reformulate tool responses in the configured language when the system prompt instructs it to.

## Current State Analysis

### 1. STT Language Parameter

**Current code** (`voice_agent.py:466-477`):
```python
# Groq STT - language hardcoded to "en"
base_stt = groq_plugin.STT(
    model="whisper-large-v3-turbo",
    language="en",  # HARDCODED
)

# Speaches STT - no language parameter at all
base_stt = openai.STT(
    base_url=f"{SPEACHES_URL}/v1",
    api_key="not-needed",
    model=WHISPER_MODEL,
    # language NOT passed
)
```

**Fix:** Both `livekit-plugins-openai` and `livekit-plugins-groq` STT accept a `language` parameter. Pass `settings["language"]` to both providers. Whisper supports ISO 639-1 codes ("en", "fr") natively.

**Speaches STT API:** OpenAI-compatible `/v1/audio/transcriptions` endpoint. The `language` field is passed as a form parameter. The `openai.STT()` constructor accepts `language` as a keyword argument.

### 2. TTS Voice Mapping

**Current code** (`voice_agent.py:552-568`):
```python
# Piper - voice baked into model ID
tts_instance = openai.TTS(
    model=runtime["tts_voice_piper"],  # "speaches-ai/piper-en_US-ryan-high"
    voice="default",
)

# Kokoro - separate model and voice params
tts_instance = openai.TTS(
    model=TTS_MODEL,
    voice=runtime["tts_voice_kokoro"],  # "am_puck"
)
```

**Piper voice model IDs** (format: `speaches-ai/piper-{lang}_{region}-{voice}-{quality}`):
- English: `speaches-ai/piper-en_US-ryan-high` (current default)
- French: `speaches-ai/piper-fr_FR-siwis-medium` (recommended for French)

**Fix:** Add voice mapping per language in settings. When language changes, auto-select the appropriate Piper voice. Keep Kokoro as English-only (limited French support — per prior decision).

**Settings approach:**
```python
# New settings for voice mapping
"tts_voice_piper_en": "speaches-ai/piper-en_US-ryan-high",
"tts_voice_piper_fr": "speaches-ai/piper-fr_FR-siwis-medium",
```

At runtime, select: `runtime[f"tts_voice_piper_{language}"]`

**Alternative:** A simpler approach is a hardcoded mapping dict in voice_agent.py:
```python
PIPER_VOICE_MAP = {
    "en": "speaches-ai/piper-en_US-ryan-high",
    "fr": "speaches-ai/piper-fr_FR-siwis-medium",
}
```
This avoids settings bloat and keeps the mapping close to where it's used. Users can still override via `tts_voice_piper` setting for custom voices.

### 3. System Prompts

**Current code** (`settings.py:268-349`):
- Prompts stored in `prompt/` directory: `default.md`, `custom.md`
- Loaded via `load_prompt_content()` → `load_prompt_with_context()`
- Template variables: `{{CURRENT_DATE_CONTEXT}}`, `{{TIMEZONE}}`
- `PROMPT_DIR` from env var `CAAL_PROMPT_DIR` or `src/caal/prompt/`

**Current prompt** (`prompt/default.md`):
- Entirely in English
- Instructions for tool usage, voice output formatting, home control
- 63 lines of behavioral instructions

**Fix:** Create per-language prompt directory structure:
```
prompt/
├── en/
│   └── default.md    # Current prompt (moved)
└── fr/
    └── default.md    # French translation
```

Update `load_prompt_content()` to accept language parameter and look in `prompt/{lang}/` first, falling back to `prompt/` for backward compatibility.

**Custom prompts:** Keep `prompt/custom.md` as-is (language-neutral). If a user writes a custom prompt, it's their responsibility to write it in their language. The custom prompt should take priority over per-language defaults.

### 4. Wake Greetings

**Current code** (`settings.py:38-46, voice_agent.py:493,688`):
```python
# Default greetings - English only
"wake_greetings": [
    "Hey, what's up?",
    "Hi there!",
    "Yeah?",
    "What can I do for you?",
    "Hey!",
    "Yo!",
    "What's up?",
],
```

**Fix:** Add per-language default greetings:
```python
DEFAULT_WAKE_GREETINGS = {
    "en": ["Hey, what's up?", "Hi there!", "Yeah?", "What can I do for you?", "Hey!", "Yo!", "What's up?"],
    "fr": ["Salut, quoi de neuf ?", "Bonjour !", "Oui ?", "Qu'est-ce que je peux faire pour toi ?", "Salut !", "Yo !", "Quoi de neuf ?"],
}
```

At runtime, select greetings based on language setting. If user has custom greetings in settings.json, use those instead.

### 5. Date/Time Formatting

**Current code** (`formatting.py:40-118`):
- `number_to_ordinal_word()` - English ordinals only ("first", "second", etc.)
- `format_date_speech_friendly()` - English format: "Monday, January twenty-first, 2026"
- `format_time_speech_friendly()` - English format: "3:30 PM", "noon", "midnight"

**Fix:** Add language parameter to formatting functions:
```python
def format_date_speech_friendly(dt: datetime, language: str = "en") -> str:
    if language == "fr":
        # French: "lundi premier janvier 2026" or "lundi 21 janvier 2026"
        ...
    else:
        # Current English logic
        ...
```

French date format:
- Day names: lundi, mardi, mercredi, jeudi, vendredi, samedi, dimanche
- Month names: janvier, février, mars, avril, mai, juin, juillet, août, septembre, octobre, novembre, décembre
- Ordinals: Only "premier" for 1st, rest use cardinal numbers (2, 3, 4...)
- Format: "lundi premier janvier 2026" or "lundi 21 janvier 2026"
- No comma between day name and date

French time format:
- 24-hour clock (standard in French): "15 heures 30" or "15h30"
- Special: "midi" (noon), "minuit" (midnight)

### 6. Tool Response Reformulation (VOICE-04)

**No code changes needed.** The LLM naturally reformulates tool responses in whatever language the system prompt instructs. When the French system prompt says "Réponds en français", the LLM will reformulate JSON tool responses into French speech.

This is verified by how `llm_node.py` works:
1. Tool results are appended to messages as `role: "tool"` (raw data)
2. LLM generates follow-up response based on system prompt + tool results
3. System prompt language determines output language

**Only requirement:** The French system prompt must explicitly instruct the agent to respond in French and format numbers/dates in French style.

## Architecture Patterns

### Pattern 1: Language-Aware Agent Startup

```python
# In voice_agent.py get_runtime_settings() or entrypoint()
language = settings.get("language", "en")

# Pass to STT
base_stt = openai.STT(..., language=language)

# Select TTS voice
piper_voice = PIPER_VOICE_MAP.get(language, PIPER_VOICE_MAP["en"])

# Load prompt
prompt = load_prompt_with_context(..., language=language)

# Select greetings
greetings = DEFAULT_WAKE_GREETINGS.get(language, DEFAULT_WAKE_GREETINGS["en"])
```

### Pattern 2: Prompt Directory Structure

```
prompt/
├── en/
│   └── default.md    # English system prompt
├── fr/
│   └── default.md    # French system prompt
└── custom.md         # User custom prompt (language-neutral, takes priority)
```

### Pattern 3: Backward Compatibility

- If `prompt/en/default.md` doesn't exist, fall back to `prompt/default.md`
- If language not in voice map, fall back to English
- If user has custom `wake_greetings` in settings.json, use those regardless of language
- Existing installations with no language setting default to "en" (already handled by Phase 1)

## Common Pitfalls

### Pitfall 1: Piper Model Download on First Use
**What:** Speaches downloads Piper models on first use. French voice model will download when first selected.
**Impact:** First French TTS response may be slow (~5-10s download).
**Mitigation:** Document this behavior. Optionally pre-download in Docker entrypoint.

### Pitfall 2: Whisper Language Parameter Affects Accuracy
**What:** Setting `language="fr"` forces Whisper to transcribe as French, even if user speaks English.
**Impact:** Mixed-language households may get garbled transcription.
**Mitigation:** Language is set globally, not per-utterance. Document this as expected behavior (matches "Out of Scope: Code-switching support").

### Pitfall 3: Kokoro French Support
**What:** Kokoro has limited French voice support. The `am_puck` voice is English.
**Impact:** If user sets language to French with Kokoro TTS, English voice speaks French text.
**Mitigation:** When language is "fr" and TTS provider is "kokoro", either auto-switch to Piper or warn user. Per prior decision: "Piper TTS for French."

### Pitfall 4: Custom Prompts Override Language
**What:** If user has `"prompt": "custom"` with English content, French language setting won't change prompt language.
**Impact:** Agent personality stays English even with French setting.
**Mitigation:** Custom prompt takes priority by design. Document that custom prompts should match the configured language.

## Key Files to Modify

| Component | File | Lines | Change |
|-----------|------|-------|--------|
| STT language | voice_agent.py | 466-477 | Pass `language` to STT constructors |
| TTS voice mapping | voice_agent.py | 552-568 | Select voice based on language |
| Runtime settings | voice_agent.py | 104-139 | Add language to runtime dict |
| Prompt loading | settings.py | 268-349 | Language-aware prompt path resolution |
| Prompt content | prompt/en/default.md | New | Move current prompt |
| Prompt content | prompt/fr/default.md | New | French prompt translation |
| Wake greetings | voice_agent.py | 493 | Language-aware greeting selection |
| Date formatting | formatting.py | 40-118 | Add French date/time formatting |
| Prompt context | settings.py | 317-349 | Pass language to formatting functions |

## Open Questions

1. **Piper French voice quality**: `siwis-medium` vs `siwis-high` — need to test which sounds better. Recommend `siwis-medium` as a safe default (lighter model, faster download).

2. **Auto-switch TTS provider for French**: Should the agent auto-switch from Kokoro to Piper when language is set to French? Or just warn? Recommend: auto-switch with log warning.

## Sources

- Codebase analysis of voice_agent.py, settings.py, formatting.py, llm_node.py
- Speaches API documentation (OpenAI-compatible endpoints)
- Piper TTS voice list: speaches-ai/piper-* model naming convention
- Prior decisions from STATE.md (Piper TTS for French, ISO 639-1 codes)

## Metadata

**Confidence breakdown:**
- STT language param: HIGH - Standard Whisper/OpenAI API parameter
- TTS voice mapping: HIGH - Piper model ID format is documented
- Prompt localization: HIGH - Simple directory restructure
- Wake greetings: HIGH - Data-only change
- Date formatting: HIGH - Standard localization pattern

**Research date:** 2026-01-26
**Valid until:** 2026-03-26
