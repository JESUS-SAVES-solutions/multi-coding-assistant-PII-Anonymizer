# Multi-Jurisdiction Compliance Guide

Read this file when the user operates internationally, processes data of residents
from multiple countries, or needs to comply with more than one privacy law simultaneously.

---

## Step 1: Determine Which Laws Apply

Multiple laws can apply simultaneously. Ask the user:

1. **Where is your organization legally registered?**
2. **Where are your servers / data processors located?**
3. **Where are your data subjects (users/customers) located?**
4. **What sector are you in?** (health, finance, education — triggers sector-specific laws)
5. **What age group do you serve?** (children → COPPA, PIPA special rules, etc.)

### Trigger rules (simplified)

| Law | Triggered when |
|---|---|
| GDPR | Data subjects are in the EU, OR organization is established in EU |
| UK GDPR | Data subjects are in the UK, OR org established in UK |
| CCPA/CPRA | Data subjects are California residents + org meets revenue/volume threshold |
| HIPAA | US-based covered entity or business associate handles PHI |
| LGPD | Data subjects are in Brazil, OR data is processed in Brazil |
| PIPL | Data subjects are in China, OR data is processed in China |
| DPDP | Data subjects are in India (enforcement ongoing) |
| POPIA | Data subjects are in South Africa, OR responsible party is in SA |
| PIPA (KR) | Data subjects are in South Korea, OR org is based in KR |
| APPI | Data subjects are in Japan, OR org is based in Japan |

**Rule of thumb:** If you have users in a country, that country's law likely applies to you — regardless of where you're based.

---

## Step 2: The "Most Restrictive" Principle

When multiple laws apply, the safest approach is to build to the **most restrictive standard** across all applicable laws — this generally satisfies the weaker laws automatically.

**GDPR is the strongest baseline** for the following reasons:
- Broad territorial scope (applies globally if EU subjects are involved)
- Strict requirements: lawful basis, data minimization, purpose limitation, DPO requirement, breach notification (72h), DPIAs
- Heaviest fines: up to €20M or 4% of global annual turnover

However, some laws have requirements that go beyond GDPR in specific areas:

| Area | Stricter than GDPR | Law |
|---|---|---|
| Data localization | Yes | China PIPL/CSL, Russia 152-FZ |
| Children's data | Similar / stricter | COPPA (US), PIPA (KR) |
| Health data de-identification | Yes (specific 18-identifier standard) | HIPAA Safe Harbor |
| Cross-border transfer approval | Yes (active approval required) | PIPL (China) |
| Consent withdrawal mechanism | Similar | LGPD, PDPA (SG) |

---

## Step 3: Cross-Border Data Transfer Mechanisms

If your anonymization pipeline sends data to an LLM or processing service in another country, the transfer itself must be legally covered — especially if the text is not yet fully anonymized.

### Transfer mechanisms by law

**GDPR (EU → Third Country):**
- Adequacy decision (UK, Canada, Japan, Israel, NZ, Korea — check current list)
- Standard Contractual Clauses (SCCs) — most common commercial solution
- Binding Corporate Rules (for intra-group transfers)
- Certification (e.g. EU-US Data Privacy Framework for US transfers)

**PIPL (China → Abroad):**
- Standard contract with CAC approval
- Security assessment (mandatory for critical data / large volumes)
- Personal information protection certification
- Note: More complex and strict than GDPR SCCs

**DPDP (India → Abroad):**
- Transfer to "notified countries" by Indian government
- List not yet fully published as of 2026

**LGPD (Brazil → Abroad):**
- Adequacy decision (none published yet)
- Standard clauses (similar to GDPR SCCs)
- Consent of the data subject
- Transfer within a corporate group (BCR equivalent)

**PIPA (South Korea → Abroad):**
- Adequacy decision (EU adequacy granted)
- Consent of data subject
- Standard clauses

### Practical implication for the anonymization pipeline

```
BEFORE Stage 1 regex runs:
  Text contains raw PII → must NOT leave the legal jurisdiction
  unless transfer mechanism is in place

AFTER Stage 1 regex (partial anonymization):
  Text still contains soft PII (names, narratives) → same restriction applies
  
AFTER Stage 2 LLM (full anonymization):
  Text is anonymized → GDPR / most laws no longer apply to the output
  (anonymized data is outside the scope of data protection law)
```

**Bottom line:** Run both stages within the legal jurisdiction, or ensure transfer mechanisms are in place for the intermediate state. The safest architecture: self-hosted or EU-only pipeline for both stages before any data leaves the controlled environment.

---

## Step 4: Documentation Requirements

Many laws require documenting your anonymization process. Ask the user:

> "Do you need help with documentation for regulatory compliance?"

| Document | Required by |
|---|---|
| Records of Processing Activities (RoPA) | GDPR Art. 30 |
| Data Protection Impact Assessment (DPIA) | GDPR Art. 35 (high-risk processing) |
| Data Processing Agreement (DPA/DPA) | GDPR Art. 28 (for any processor) |
| Privacy Notice / Policy update | GDPR, CCPA, LGPD, PIPL, etc. |
| Technical and Organizational Measures (TOMs) | GDPR Art. 32 |
| Anonymization methodology documentation | HIPAA Expert Determination, GDPR recitals |

---

## Step 5: Sector-Specific Overlays

Beyond general privacy law, sector-specific rules may impose stricter anonymization requirements:

| Sector | Law | Key requirement |
|---|---|---|
| Healthcare (US) | HIPAA | 18 specific identifiers must be removed for Safe Harbor de-identification |
| Finance (US) | GLBA | Customer financial info; minimize and protect NPI |
| Children's services | COPPA (US) | Extra restrictions for under-13 data |
| Clinical research | GCP / ICH E6 | De-identification before publication; IRB approval |
| Payment data | PCI DSS | Card data must be masked/tokenized; not a privacy law but mandatory |
| Public sector (EU) | NIS2 | Cybersecurity requirements affecting data processing |

---

## Multi-Country Deployment Checklist

Use this when building an anonymization system for international deployment:

- [ ] Identified all applicable jurisdictions (see Step 1)
- [ ] Built to the most restrictive standard as baseline
- [ ] Country-specific PII patterns implemented for each active jurisdiction
- [ ] Transfer mechanisms documented for any cross-border LLM API calls
- [ ] DPA signed with every data processor (LLM provider, cloud host, etc.)
- [ ] DPIA completed if processing is high-risk
- [ ] RoPA entry created for the anonymization processing activity
- [ ] Audit log implemented (without logging PII content)
- [ ] Data residency requirements verified per country (especially CN, RU)
- [ ] Output validation step in place (re-run regex on LLM output)
- [ ] Retention policy for anonymized output defined
- [ ] Incident response plan covers anonymization pipeline failures

---

## Common Pitfall: "Pseudonymization ≠ Anonymization"

This distinction matters legally:

| Concept | GDPR status | Practical meaning |
|---|---|---|
| **Anonymization** | Outside scope of GDPR | Cannot be re-identified at all; data protection law no longer applies |
| **Pseudonymization** | Still within GDPR scope | Re-identification is possible with the key; additional safeguards required |

If your system replaces names with tokens and stores the mapping table:
→ That is **pseudonymization**, not anonymization.
→ The mapping table is itself personal data and must be protected.
→ GDPR still applies to the whole system.

True anonymization requires that re-identification is **not reasonably possible** — accounting for all means likely to be used, including the mosaic effect (see SKILL.md).
