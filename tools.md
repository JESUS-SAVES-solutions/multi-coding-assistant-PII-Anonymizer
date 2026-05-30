# Existing Anonymization Tools & Libraries

Read this file when the user asks about ready-made solutions, wants to evaluate
alternatives to custom code, or needs a specific NLP/NER library.

---

## When to Use a Ready-Made Tool vs. Custom Code

| Situation | Recommendation |
|---|---|
| Broad coverage needed immediately, standard PII types | Use **Presidio** — fastest path to production |
| Single language, simple patterns | Custom regex script sufficient |
| Free text with names/orgs in any language | Presidio + multilingual NER model |
| Structured dataset anonymization (k-anonymity, generalization) | **ARX** |
| Need pseudonymization with realistic fake data | **Faker / Mimesis** |
| Java/JVM environment | **OpenNLP** or Presidio via REST |
| Constrained runtime (n8n Code Node) | Custom regex + external LLM API (Presidio as separate service) |
| Maximum control over deployment and data flow | Custom code + self-hosted LLM |

---

## Microsoft Presidio ⭐ (Top Recommendation)

**Type:** Open source (MIT License)  
**Language:** Python  
**GitHub:** https://github.com/microsoft/presidio  
**Docs:** https://microsoft.github.io/presidio/

The most comprehensive open-source PII detection and anonymization framework available.
Battle-tested, actively maintained, used in production by large organizations.

### What it does
- Detects 50+ PII entity types out of the box
- Supports regex recognizers, NER-based recognizers, rule-based recognizers, and custom recognizers
- Anonymizes by masking, redacting, replacing, hashing, or encrypting
- Supports custom operators (your own anonymization logic per entity type)
- Two main components: **Analyzer** (detect) + **Anonymizer** (transform)
- Optional: **Image Redactor** for photos/documents
- REST API server available — usable from any language via HTTP

### GDPR-relevant entities supported (selection)
PERSON, EMAIL_ADDRESS, PHONE_NUMBER, CREDIT_CARD, IBAN_CODE, IP_ADDRESS,
DATE_TIME, LOCATION, NRP (nationality/religion/political group),
US_SSN, UK_NHS, UK_NINO, IN_AADHAAR, IN_PAN, SG_NRIC_FIN, AU_ABN, AU_TFN,
AU_MEDICARE, ES_NIF, IT_FISCAL_CODE, FI_PERSONAL_IDENTITY_CODE, PL_PESEL,
and many more — see full list at https://microsoft.github.io/presidio/supported_entities/

### NER backends supported
- **spaCy** (default) — fast, multilingual, good for names/places/orgs
- **Hugging Face Transformers** — higher accuracy, slower, better multilingual support
- **Azure Text Analytics** (if Azure is acceptable in context)
- Custom models

### Self-hosting
Presidio runs as a Python service (FastAPI). Can be containerized with Docker.
No data leaves your infrastructure — fully compatible with strict GDPR/PIPL requirements.

```bash
# Quick start
pip install presidio-analyzer presidio-anonymizer
python -m spacy download en_core_web_lg
```

### Using Presidio via REST API from n8n
Run Presidio as a Docker container, call it from n8n HTTP Request node:
```
POST http://localhost:5001/analyze
{
  "text": "John Smith lives at 123 Main St, his email is john@example.com",
  "language": "en"
}
```
Returns detected entities → feed into anonymizer endpoint.

### Multilingual support
Presidio itself is language-agnostic. NER accuracy depends on the spaCy/HF model used.
- English: `en_core_web_lg` (spaCy) — strong
- German: `de_core_news_lg` (spaCy) — good
- Other languages: check https://spacy.io/models

---

## spaCy

**Type:** Open source (MIT)  
**Language:** Python  
**Website:** https://spacy.io

Industrial-strength NLP library. Core NER capability used by Presidio internally.
Use directly if you want more control than Presidio provides, or are building a custom pipeline.

### Models relevant for PII
- `en_core_web_trf` — transformer-based, best accuracy, high RAM
- `de_core_news_lg` — German, good for names/orgs
- `xx_ent_wiki_sm` — multilingual, lighter, covers many languages

### When to use standalone spaCy
- You want NER only (no regex framework)
- You're building a fully custom pipeline
- You need tighter integration with your ML stack

---

## Hugging Face NER Models

**Type:** Open source (model-dependent, many MIT/Apache 2.0)  
**Website:** https://huggingface.co

Transformer-based NER models often outperform spaCy for names and multilingual text.

### Recommended models for PII
| Model | Language | Notes |
|---|---|---|
| `dslim/bert-base-NER` | English | Fast, good baseline |
| `Jean-Baptiste/roberta-large-ner-english` | English | Higher accuracy |
| `dbmdz/bert-large-cased-finetuned-conll03-english` | English | Strong for names/orgs |
| `flair/ner-german` | German | Good for German names |
| `Babelscape/wikineural-multilingual-ner` | Multilingual | 9 languages |

### Usage
Can be integrated into Presidio as a custom recognizer, or used standalone via
Hugging Face `transformers` pipeline.

---

## Scrubadub

**Type:** Open source (MIT)  
**Language:** Python  
**GitHub:** https://github.com/LeapBeyond/scrubadub

Simpler alternative to Presidio. Good for English text, quick setup.
Fewer entity types than Presidio but easier to extend with custom detectors.

### Best for
- English-language text
- Quick prototyping
- Environments where Presidio's complexity is overkill

---

## ARX — Data Anonymization Tool

**Type:** Open source (Apache 2.0)  
**Language:** Java (GUI + API)  
**Website:** https://arx.deidentifier.org

Specialized for **structured datasets** (tables/CSV/databases), not free text.
Implements k-anonymity, l-diversity, t-closeness, and differential privacy.

### Best for
- Database exports, CSV files, structured datasets
- Statistical disclosure limitation (research, health data, census data)
- Compliance with HIPAA safe harbor / expert determination methods

### Not suitable for
- Free-text anonymization
- Real-time/stream processing

---

## Faker / Mimesis — Pseudonymization Libraries

**Faker (Python/JS/PHP/Ruby/...):**  
**GitHub:** https://github.com/faker-ruby/faker (Ruby), https://fakerjs.dev/ (JS), https://faker.readthedocs.io/ (Python)

**Mimesis (Python):**  
**GitHub:** https://github.com/lk-geimfari/mimesis

These generate realistic fake data for pseudonymization:
names, addresses, phone numbers, emails, IDs — in many locales.

### Use case
Replace real PII with plausible-looking fake data (same type, same format).
Useful for creating anonymized test datasets, demo environments, or when
masking with `[PLACEHOLDER]` would make text unreadable.

### Deterministic pseudonymization
Use a seeded random generator (same seed → same fake output for same input),
so that references to the same person within a document remain consistent.

---

## OpenNLP

**Type:** Open source (Apache 2.0)  
**Language:** Java  
**Website:** https://opennlp.apache.org

Apache project for NLP tasks including NER. Use when the runtime is Java/JVM
and adding a Python service is not feasible.

Less accurate than modern transformer-based models, but works in constrained environments.

---

## Managed / Cloud Services (Use with GDPR Caution)

| Service | Provider | Notes |
|---|---|---|
| Amazon Comprehend | AWS (US) | PII detection API; US-hosted by default; EU region available |
| Azure Text Analytics / Language | Microsoft (US) | PII detection; EU data centers available; CLOUD Act risk if US parent |
| Google DLP (Data Loss Prevention) | Google (US) | PII detection + redaction API; EU region available; CLOUD Act risk |
| AWS Macie | AWS (US) | Focused on S3 bucket PII scanning |

**⚠️ GDPR note:** US providers with EU data centers are subject to the CLOUD Act —
US courts can compel access to data even when stored in EU. For sensitive personal data,
prefer purely EU providers or self-hosted solutions. At minimum, sign a DPA and use
the EU-region endpoint. See `jurisdictions.md` for transfer mechanism options.

---

## PrivacyProxy

**Website:** https://privacyproxy.dev  
**Type:** Commercial SaaS  

A proxy service that sits in front of any LLM API and automatically masks PII
before sending the request, then unmasks in the response. Useful as a quick
compliance layer when you cannot change your LLM provider.

**Note:** You're trusting PrivacyProxy with your PII — evaluate their own GDPR compliance.

---

## Summary: Decision Tree

```
Do you need free-text anonymization?
├── Yes, with names/orgs/complex PII
│   ├── Python available? → Use Presidio (+ spaCy or HF model)
│   ├── Java available?   → OpenNLP or Presidio via REST
│   └── Any language?     → Presidio as REST service + HTTP calls
│
├── Yes, structured patterns only (email, phone, ID numbers)
│   └── Custom regex script (see implementation files)
│
└── No, structured dataset (table/CSV/DB)?
    └── ARX (k-anonymity, l-diversity)

Do you need pseudonymization (realistic fake data)?
└── Faker / Mimesis (locale-aware)

Do you need LLM-based second stage?
├── EU-compliant API needed → Mistral API, Scaleway
├── Self-hosted             → Mistral Small 3.1, Llama 3.3 70B, gpt-oss-120b
└── See SKILL.md LLM Selection Guide
```
