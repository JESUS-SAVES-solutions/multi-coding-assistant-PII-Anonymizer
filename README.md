# Multi-Coding Assistant — PII Anonymizer - Skill

A Claude skill that acts as a full development advisor for building PII (Personally
Identifiable Information) anonymization systems — from architecture and code generation
to legal compliance, LLM selection, and production resilience.

---

## What This Skill Does

When installed, this skill guides Claude to help you:

- **Generate anonymization scripts** in any programming language (JavaScript, Python, PHP, Java, Rust, and more)
- **Design a two-stage pipeline** — regex for structured PII, LLM for names and free text
- **Comply with global privacy laws** — GDPR, CCPA, HIPAA, LGPD, PIPL, DPDP, POPIA, and more
- **Select a GDPR-compliant LLM** for the second stage, including EU-only providers
- **Choose the right model size** — based on the trade-off between judgment quality and hardware cost
- **Build for resilience** — retry logic, fallback providers, circuit breakers, dead letter queues
- **Evaluate existing tools** — Microsoft Presidio, spaCy, ARX, Faker, and others
- **Handle multi-jurisdiction compliance** — when your users span multiple countries

Claude will ask you which language you want to communicate in before starting,
and generates all code in your chosen programming language.

---

## Included Files

```
multi-coding-assistant-PII-Anonymizer/
├── SKILL.md                              ← Main skill file (loaded by Claude)
└── references/
    ├── countries.md                      ← Country-specific PII patterns + privacy laws
    │                                       (EU, US, BR, CN, IN, RU, ZA, JP, KR, AU, SG, …)
    ├── jurisdictions.md                  ← Multi-jurisdiction compliance guide
    │                                       (which laws apply, transfer mechanisms,
    │                                        pseudonymization vs. anonymization)
    ├── tools.md                          ← Existing tools and libraries
    │                                       (Microsoft Presidio, spaCy, ARX, Faker, …)
    └── implementation-n8n-js.md          ← Reference implementation: JavaScript / n8n Code Node
                                            (production code + known gaps + suggested extensions)
```

---

## Installation

### Claude.ai (via Skills / Projects)

1. Download or clone this repository
2. In Claude.ai, open **Settings → Skills**
3. Create a new skill and upload `SKILL.md`
4. Upload the `references/` folder files as additional resources
5. The skill will activate automatically when relevant topics are detected

### Claude Code / Cowork

```bash
git clone https://github.com/YOUR_USERNAME/multi-coding-assistant-PII-Anonymizer.git
```

Place the folder in your skills directory and reference it in your Claude configuration.

---

## Quick Start

Once the skill is active, start with something like:

> *"I need to anonymize customer support tickets before sending them to an LLM. We have users in Germany and the US."*

or

> *"I have an existing n8n Code Node that anonymizes text — help me extend it to cover more PII categories."*

Claude will walk you through the onboarding sequence:
1. Preferred language
2. Applicable jurisdiction(s)
3. Target programming language and runtime
4. PII categories to cover
5. Anonymization method
6. Whether to add an LLM second stage
7. Whether to review existing tools first

---

## Coverage

### Privacy Laws
GDPR · UK GDPR · nDSG (CH) · CCPA/CPRA · HIPAA · FERPA · COPPA · GLBA ·
LGPD (BR) · PIPL + CSL (CN) · DPDP (IN) · POPIA (ZA) · APPI (JP) · PIPA (KR) ·
Privacy Act (AU) · PDPA (SG) · 152-FZ (RU) · PDPL (UAE) · PDPL (SA) ·
NDPA (NG) · DPA (KE) · Law 25.326 (AR) · LFPDPPP (MX) · Law 21.719 (CL) + more

### Country-Specific PII Identifiers
Germany (Steuernummer, USt-ID, SVNr, Kfz-Kennzeichen) ·
France (INSEE, SIRET) · Netherlands (BSN) · Italy (Codice Fiscale) ·
Spain (DNI, NIE) · Poland (PESEL) · Sweden (Personnummer) ·
UK (NI number, NHS number) · Switzerland (AHV) ·
USA (SSN, EIN, HIPAA 18 identifiers) · Canada (SIN) ·
Brazil (CPF, CNPJ, CEP) · Mexico (CURP, RFC) · Argentina (DNI, CUIL) ·
China (Resident ID 18-digit) · India (Aadhaar, PAN) ·
Japan (My Number) · South Korea (RRN) · Australia (TFN, Medicare) ·
Singapore (NRIC/FIN) · Russia (SNILS, INN) · South Africa (SA ID 13-digit) ·
UAE (Emirates ID) + more

### Recommended LLM Providers (GDPR-compliant, EU-hosted)
Mistral AI · Scaleway · Aleph Alpha · Nordference · Nebius

---

## Contributing

Contributions welcome — especially:
- Additional country-specific PII patterns
- Reference implementations in other languages (Python, PHP, Java, etc.)
- Updated legal information as privacy laws evolve
- Corrections to regex patterns

Please open an issue or pull request.

---

## License

MIT License — free to use, modify, and distribute.

---

## Disclaimer

This skill provides technical and informational guidance. It does not constitute legal advice.
For binding legal assessments of your anonymization implementation, consult a qualified
data protection lawyer or certified Data Protection Officer (DPO).
