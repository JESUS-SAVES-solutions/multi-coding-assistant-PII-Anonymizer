---
name: multi-coding-assistant-PII-Anonymizer
description: >
  A development advisor for building PII (Personally Identifiable Information)
  anonymization systems — covering code generation, architecture, legal compliance,
  LLM selection, and tooling. Use this skill whenever a user wants to anonymize,
  pseudonymize, mask, or redact personal data — regardless of programming language,
  framework, or platform (e.g. n8n, Python scripts, Node.js, Java, PHP, etc.).
  Also use when a user wants to extend or improve an existing anonymization script,
  add new PII categories, review code for missing PII coverage under GDPR/DSGVO or
  similar regulations, select an LLM for anonymization, design a fallback/retry
  strategy, create a prompt for LLM-based PII detection, or get advice on existing
  tools like Microsoft Presidio. Also use for multi-jurisdiction compliance
  (GDPR + CCPA + PIPL + LGPD etc.) and for users who operate internationally and
  must comply with multiple privacy laws simultaneously.
---

# Multi-Coding Assistant — PII Anonymizer

This skill generates a complete anonymization solution — in any programming language,
for any runtime context — covering global privacy laws, a proven two-stage Regex + LLM
pipeline, GDPR-compliant LLM providers, and production-grade resilience patterns.

---

## Reference Files — Read When Needed

| File | Read when… |
|---|---|
| `references/countries.md` | User needs country-specific PII patterns or non-GDPR laws (US, CN, IN, BR, RU, etc.) |
| `references/jurisdictions.md` | User operates internationally or must comply with multiple laws simultaneously |
| `references/tools.md` | User asks about ready-made solutions (Presidio, spaCy, ARX, Faker, etc.) |
| `references/implementation-n8n-js.md` | User needs the JavaScript/n8n reference implementation or wants to extend it |

Always check the relevant reference file before answering jurisdiction-specific or tool-specific questions.

---

## How to Use This Skill

Follow this onboarding sequence **before generating any code**:

### Step 1 — Language preference
> "In which language would you like to work with me? (e.g. English, Deutsch, Español…)"

Continue the entire session in that language.

### Step 2 — Jurisdiction(s)
> "Which countries or regions are your data subjects located in?
> And where is your organization based / where are your servers hosted?"

→ Read `references/countries.md` for applicable laws and country-specific identifiers.  
→ If multiple jurisdictions: read `references/jurisdictions.md`.  
→ If sector-specific (health, finance, children): note the applicable overlay law.

### Step 3 — Programming language & runtime context
> "Which programming language should the script be generated in?
> And where will it run? (e.g. n8n Code Node, standalone script, REST API…)"

### Step 4 — PII categories
Present the relevant category list (Section 2 + `references/countries.md` for local identifiers) and ask:
> "Which categories should be anonymized? Select all that apply."

### Step 5 — Anonymization method
> "How should detected PII be handled?"
> - **Mask** — replace with label, e.g. `[EMAIL]`
> - **Redact** — remove completely
> - **Pseudonymize** — deterministic fake (same input → same output)
> - **Hash** — one-way hash (SHA-256 etc.)
> - **Custom** — user defines replacement logic

### Step 6 — Two-stage pipeline (recommended)
> "Do you want a two-stage pipeline (Regex + LLM) for higher coverage?
> Regex handles structured PII; LLM handles names, narratives, and ambiguous content."

If yes → proceed to LLM Selection Guide (Section 6).

### Step 7 — Existing tools
> "Would you like to know about ready-made libraries (e.g. Microsoft Presidio) that
> may cover your needs without writing everything from scratch?"

If yes → read `references/tools.md`.

### Step 8 — Existing code (optional)
> "Do you have an existing anonymization script you'd like to extend?"

If JavaScript/n8n → read `references/implementation-n8n-js.md` for the reference implementation and known gaps.

---

## PII Categories (Universal + GDPR/DSGVO)

For country-specific identifiers (SSN, Aadhaar, CPF, RRN, etc.) → see `references/countries.md`.

### 1. Direct Identifiers
| Item | Regex reliable? |
|---|---|
| Full name | ❌ NER or LLM required |
| Date of birth | ✅ Multiple date formats |
| Place of birth | ❌ LLM (context-sensitive) |
| Nationality / Gender | ❌ LLM (context-sensitive) |

### 2. Contact Data
| Item | Regex reliable? |
|---|---|
| Email | ✅ Uniform pattern |
| Phone / Fax | ✅ Partially (many formats) |
| Street address | ⚠️ Country-specific |
| ZIP / postal code | ✅ Per-country patterns |

### 3. Online Identifiers
| Item | Regex reliable? |
|---|---|
| IPv4 / IPv6 | ✅ Yes |
| MAC address | ✅ Yes |
| UUID / Device ID | ✅ Yes |
| IBAN | ✅ Yes |
| Cookie ID / Session token | ❌ Context-dependent |
| Username / Handle | ⚠️ Context-sensitive |

### 4. Financial Data
| Item | Regex reliable? |
|---|---|
| Credit / debit card | ✅ With Luhn validation |
| BIC / SWIFT | ✅ Yes |
| Tax ID / VAT / USt-ID | ✅ Per-country patterns |
| Bank account number | ⚠️ Country-specific |

### 5. Official Identifiers
| Item | Regex reliable? |
|---|---|
| Passport / National ID | ⚠️ Country-specific |
| Social security / pension no. | ✅ Structured per country |
| Health insurance number | ✅ Per-country patterns |
| Driver's license | ⚠️ Country-specific |

### 6. Special Categories — Art. 9 GDPR (heightened protection)
All require LLM or keyword-based detection in free text:
Health / medical data, racial / ethnic origin, political opinions, religious beliefs,
sexual orientation, trade union membership, biometric / genetic data.

### 7. Location & Other
| Item | Regex reliable? |
|---|---|
| GPS coordinates | ✅ Decimal / DMS formats |
| Location names in narrative | ❌ LLM required |
| Vehicle license plate | ✅ Per-country patterns |
| Photos / audio / video | ❌ Out of scope for text |

---

## Anonymization Methods

| Method | Description | Use case |
|---|---|---|
| **Masking** | Replace with label: `[EMAIL]`, `[PHONE]` | Logging, audit trails |
| **Redaction** | Remove entirely | Strict privacy, irreversible |
| **Pseudonymization** | Deterministic fake (same in → same out) | Analytics, referential integrity |
| **Hashing** | One-way hash (SHA-256) | Deduplication |
| **Tokenization** | Random token + stored mapping | Reversible if needed |

⚠️ **Pseudonymization ≠ Anonymization:** Pseudonymized data is still personal data under GDPR.
The mapping table must be protected. Only true anonymization takes data outside GDPR scope.
See `references/jurisdictions.md` for the full legal distinction.

---

## Two-Stage Pipeline: Regex → LLM

```
Input text
    │
    ▼
[Stage 1] Regex — fast, deterministic, no external calls
    │  Handles: email, IP, IBAN, credit card, phone, ZIP, MAC,
    │           UUID, VAT, SSN, GPS, license plates, dates
    │
    ▼
Partially anonymized text
    │
    ▼
[Stage 2] LLM — contextual judgment, catches what regex cannot
    │  Handles: names, narratives, family relationships,
    │           Art. 9 categories, mosaic combinations
    │
    ▼
Fully anonymized text
    │
    ▼
[Validation] Re-run Stage 1 regex on output to catch slippage
```

**Why this order:** Regex first reduces the PII the LLM sees — minimizing exposure and token cost.

### What Regex Cannot Reliably Handle → LLM Required

| Category | Why regex fails |
|---|---|
| Full names | No fixed pattern; cultural variety worldwide |
| Family relationships | "my son", "meine Tochter" — context-dependent |
| Employer names in free text | Unlimited variety |
| Physical appearance | "tall, red hair, wears glasses" |
| Health conditions in narrative | "I have been diagnosed with…" |
| Religion / politics / ethnicity in narrative | Free text |
| Location as identifier | "near the old church on the hill" |
| Pet names + unique descriptors | Part of mosaic |
| Biographical details | "worked at X for 20 years, moved to Y" |
| Indirect references | "the director of our department" |

---

## Mosaic Effect & Readability Balance

### The Mosaic Effect
Individually harmless data can combine to re-identify a person:
- Gender alone → not identifying
- Gender + ZIP + DOB → often uniquely identifying (Sweeney 1997: 87% of US population)
- Add pet breed, name, hobby, clothing → increasingly unique

**The LLM's task** is to evaluate *combinations*, not individual items in isolation.

### Model size and judgment quality
- Small models (≤24B) often over-anonymize — they remove "my son" / "meine Tochter"
  because they see "gender indicator" without evaluating whether the remaining context
  is actually identifying.
- Larger models (70B+, especially 120B) can evaluate the full picture and apply
  genuine proportionality.

### Don't over-anonymize
Excessive anonymization produces unreadable "swiss cheese" text full of `[REDACTED]` gaps.

**Principle:** Apply the *minimum* anonymization that makes re-identification impossible —
not the maximum possible redaction. Include this in every LLM prompt.

---

## LLM Selection Guide

### Model Size & Judgment Quality

| Size | Verdict | Notes |
|---|---|---|
| < 8B | ❌ Do not use | Unreliable — barely better than regex |
| ~8B | ❌ Not recommended | Too small for nuanced judgment |
| ~24B | ⚠️ Minimum viable | Mistral Small 3.1 — works but tends to over-anonymize |
| ~32–70B | ✅ Good | Qwen3 32B, Llama 3.3 70B — better contextual judgment |
| ~120B | ✅ Excellent | gpt-oss-120b — strong proportionality; recommended for production |

### GDPR-Compliant LLM Providers (API)

Sending text that still contains PII to a non-compliant provider is itself a GDPR violation.

**Purely EU — recommended for sensitive data (no CLOUD Act risk):**

| Provider | HQ | Notes |
|---|---|---|
| Mistral AI (mistral.ai) | France 🇫🇷 | Frontier + open-weight; GDPR by default; good pricing |
| Scaleway Generative APIs | France 🇫🇷 | Open-source model hosting; Simon's existing provider |
| Aleph Alpha | Germany 🇩🇪 | BSI C5 certified; best for regulated industries |
| Nordference | Estonia 🇪🇪 | Open-source models, zero data retention, NIS2 compliant |
| Nebius | EU 🇪🇺 | Open-source model hosting at scale |

**⚠️ US providers with EU data centers:** Azure, AWS, Google — CLOUD Act risk even in EU regions.  
**⚠️ DPA required:** Always sign a Data Processing Agreement before sending any personal data.

### Self-Hosting (strongest GDPR position)

| Model | License | Min. VRAM (Q4) | Notes |
|---|---|---|---|
| Mistral Small 3.1 (24B) | Apache 2.0 | ~24 GB | Single RTX 4090; best efficiency at size |
| Qwen3 32B | Apache 2.0 | ~24–32 GB | Strong multilingual |
| Llama 3.3 70B | Meta community | ~48 GB | Excellent quality; dual GPU or high-VRAM |
| gpt-oss-120b | Apache 2.0 | ~80 GB | Best judgment; A100/H100 class |
| DeepSeek R1 70B distill | MIT | ~48 GB | Strong reasoning |

Self-hosting below 24B is not recommended — too small for reliable PII judgment.

### Fallback Strategy — Two Providers, Two Companies

```
Primary: Provider A (e.g. Scaleway + Mistral Small 3.1)
    │  → on failure (timeout / 5xx / rate limit) →
Secondary: Provider B (e.g. Mistral API + Mistral Large)
    │  → on failure →
Dead letter queue → retry with configurable delay
```

Different companies — not just different models from one provider. A platform outage or
billing problem can affect all models from one provider simultaneously.

---

## LLM Prompt Guide

The LLM receives the **already regex-anonymized text** as input. Template:

```
You are a GDPR compliance assistant performing Stage 2 of a two-stage PII
anonymization pipeline. The text has already been processed by regex.
Placeholders like [EMAIL] or [PHONE] mark already-anonymized fields.

Your task:
- Identify and replace remaining PII that regex could not catch: names,
  family relationships (where identifying in context), employer names,
  health details, location references, physical descriptions, and any
  combination of attributes enabling re-identification.
- Apply MINIMUM necessary anonymization. Do NOT remove elements that,
  given the remaining context, cannot contribute to re-identification.
- Preserve readability. Avoid unnecessary [REDACTED] gaps.
- Use descriptive placeholders: [NAME], [EMPLOYER], [HEALTH_CONDITION],
  [LOCATION], [FAMILY_RELATION], etc.
- Return ONLY the anonymized text. No explanation, no commentary.

Text:
{{text}}
```

**Tuning:**
- Legal / medical / sensitive: instruct to err on the side of caution
- Operational logs / support tickets: instruct to preserve as much as possible

---

## Resilience & Robustness

### Configurable Parameters

| Parameter | Default | Notes |
|---|---|---|
| `llm_timeout_seconds` | 30 | Abort and retry on exceed |
| `max_retries` | 3 | Per provider before fallback |
| `retry_delay_seconds` | 60 | Initial retry delay |
| `retry_backoff_factor` | 2 | Exponential: 60s → 120s → 240s |
| `fallback_retry_delay` | 900 | 15 min before retrying full pipeline |
| `max_fallback_retries` | 3 | Before dead letter |
| `circuit_breaker_threshold` | 5 | Failures before circuit opens |
| `circuit_breaker_window_seconds` | 300 | 5-minute observation window |

### Resilience Patterns

1. **Retry with exponential backoff** — respect `Retry-After` on HTTP 429
2. **Fallback to secondary provider** — different company, not just different model
3. **Circuit breaker** — stop calling a failing provider; alert; cool-down before retry
4. **Dead letter queue** — never silently discard; queue for manual review or scheduled retry
5. **Output validation (mandatory)** — re-run Stage 1 regex on LLM output; fail if PII found
6. **Chunking** — split long texts into overlapping chunks (~200 token overlap); reassemble after
7. **Audit log** — log timestamp, model, duration, success/failure, input hash — NEVER raw text
8. **Health check** — ping both providers before batch processing
9. **Idempotency** — hash input; return cached result if already processed

---

## Reference Implementation (JavaScript / n8n)

See `references/implementation-n8n-js.md` for:
- Full production code (Stage 1 regex, n8n Code Node)
- Coverage list (what is and is not anonymized)
- Known gaps and suggested extensions (IBAN, IPv4, ISO dates, catch-all fix, etc.)

---

## Known Limitations

- Names require NER or LLM — regex alone is not sufficient
- Multilingual input requires country/locale-specific patterns — clarify with user
- False positives from aggressive regex — use context-aware patterns and validation
- Nested/encoded data (base64, JSON strings, HTML entities) may hide PII — decode first
- Chunk boundaries in long texts may split PII — use overlapping chunks
- LLM hallucinations — output validation is a partial mitigation, not a guarantee
- Pseudonymization ≠ anonymization — GDPR still applies; see `references/jurisdictions.md`

---

## Changelog

| Version | Date | Changes |
|---|---|---|
| 0.1 | 2026-05-30 | Initial skeleton |
| 0.2 | 2026-05-30 | Two-stage pipeline, mosaic effect, LLM selection guide, provider comparison, resilience patterns |
| 0.3 | 2026-05-30 | Global scope, reference files structure, jurisdiction step, tools section, JS reference implementation, country-specific patterns moved to references/ |
| 0.3.1 | 2026-05-30 | Renamed: multi-coding-assistant-PII-Anonymizer |
