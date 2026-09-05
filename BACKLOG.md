# Backlog

## Vibe-code technical debt audit for OPTIQON Voice

**Status:** Backlog

**Timing:** Run after the OPTIQON Voice rebrand, production signing/release workflow and first physical-device smoke are stable.

### Goal

Run a focused technical-debt audit of OPTIQON Voice specifically for risks that are plausible in an AI-first / vibe-coded Android codebase.

This is **not** a general cleanup or stylistic refactor. The purpose is to find the small number of places where the app's AI-first origin could create future bugs, security issues, release/update problems, or disproportionately expensive debugging sessions.

### Audit focus

Prioritize high-impact surfaces:

- Android lifecycle / foreground service / boot behavior
- Accessibility and text-injection safety
- overlay/bubble lifecycle and race conditions
- microphone/audio recorder resource handling
- network/API error handling, retries and cancellation
- credential/secret handling and logging
- release signing / versioning / update chain
- Room schema, migrations and persistence invariants
- Hilt/KSP/generated-code assumptions
- coroutine/threading/concurrency hazards
- permissions and exported-component semantics
- state restoration / process death / edge cases
- provider/model configuration validation
- tests that look green but do not verify real behavior
- stale/dead code, duplicated logic and hidden coupling introduced by iterative AI changes

### Required output

Produce a ranked list of roughly **5–10 concrete findings**, each with:

1. evidence (file/path + behavior)
2. severity / likelihood
3. user/business impact
4. whether it is an actual defect, latent risk, or maintainability debt
5. cheapest sufficient verification
6. recommended fix or explicit recommendation to leave it alone
7. rough effort and regression risk

Group findings into:

- **Fix now** — high value / low-to-moderate effort
- **Fix when touched** — real debt but not worth a dedicated mission yet
- **Accept / monitor** — low-value cleanup or speculative risk

### Boundaries

- Do **not** rewrite the app because it was AI-generated.
- Do **not** reward cosmetic code cleanliness for its own sake.
- Do **not** change architecture unless evidence shows the current design is causing meaningful risk or recurring cost.
- Prefer tests, invariants and targeted hardening over broad refactors.
- Preserve current app behavior unless a defect is demonstrated.

### Definition of Done

- 5–10 evidence-backed findings max, ranked by expected future cost/risk.
- Critical user paths and release/update path explicitly reviewed.
- Existing tests assessed for meaningful coverage, not just test count.
- Clear recommendation for which items deserve separate implementation issues/Missions.
- No implementation changes as part of the audit itself unless separately approved.

### Repo migration note

If OPTIQON Voice is moved into a new `LarsAhls/optiqon-voice` repository instead of renaming this repository, migrate this backlog item with it.

---

## Groq setup UX — provider preset, prefilled fields and model selection

**Status:** Backlog

**Goal:** Make first-time Groq configuration substantially faster and harder to misconfigure, without bundling credentials or locking the app to one provider.

### Proposed behavior

When the user selects **Groq** as provider, offer a Groq preset that:

- prefills the compatible base URL as `https://api.groq.com/openai`;
- presents known compatible transcription models in a dropdown instead of requiring exact manual typing, with `whisper-large-v3-turbo` as the recommended/default Groq ASR choice unless later evidence changes the recommendation;
- presents known compatible post-processing/chat models in a dropdown where applicable, while retaining an **Other / custom model** option;
- keeps advanced/custom base URL editing available so OpenAI-compatible providers remain supported;
- never ships or hardcodes an API key.

### Acceptance intent

A new user choosing Groq should normally only need to paste their API key and confirm/select a model, rather than manually discovering and typing provider URL/model identifiers.

### Boundaries

- Do not remove custom provider support.
- Do not silently overwrite a user's existing custom configuration.
- Treat model lists as maintainable configuration, not permanent truth; preserve a custom-model fallback.
- Validate the final endpoint construction so the app does not accidentally produce duplicate `/v1` path segments.

---

## Groq onboarding link from provider settings

**Status:** Backlog

**Goal:** Let a user go directly from OPTIQON Voice settings to the place needed to obtain/configure Groq credentials, reducing setup friction and support burden.

### Proposed behavior

In the Groq provider section:

- add a clearly labelled external link/button such as **Get Groq API key** that opens Groq's official API-key/account page;
- optionally add a secondary **Groq setup/help** link if the API-key page alone is not sufficiently self-explanatory;
- visually associate the link with the API-key field and provider preset/dropdown;
- make it explicit that the key belongs to the user and is stored/handled by the app according to its credential policy.

### Definition of Done

- Groq can be selected without manually knowing its endpoint.
- Recommended models can be selected without typing model IDs.
- Custom endpoint/model remain possible.
- Official Groq link opens correctly.
- No key, token or secret is committed, logged or embedded in the app.
- Existing non-Groq provider behavior is unchanged.
