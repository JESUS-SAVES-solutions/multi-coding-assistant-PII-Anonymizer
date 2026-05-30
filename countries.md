# Country-Specific PII Patterns & Privacy Laws

Read this file when the user needs anonymization for a specific country/region,
or when operating under a non-GDPR legal framework.

---

## How to Use This File

1. Identify the applicable jurisdiction(s) from the user's context
2. Check the relevant section for the applicable law and specific PII identifiers
3. Add the country-specific regex patterns to the generated script in addition to universal patterns
4. If multiple jurisdictions apply, see also: `jurisdictions.md`

---

## Universal PII Patterns (all jurisdictions)

These apply everywhere regardless of law:

| Category | Regex hint | Notes |
|---|---|---|
| Email | `[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}` | Uniform worldwide |
| IPv4 | `(?:\d{1,3}\.){3}\d{1,3}` | Universal |
| IPv6 | `[0-9a-fA-F]{1,4}(?::[0-9a-fA-F]{0,4}){2,7}` | Universal |
| MAC address | `[0-9A-Fa-f]{2}(?:[:\-][0-9A-Fa-f]{2}){5}` | Universal |
| UUID / Device ID | `[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}` | Universal |
| IBAN | `[A-Z]{2}\d{2}[\s]?(?:[A-Z0-9]{4}[\s]?){2,7}[A-Z0-9]{1,4}` | 80+ countries |
| Credit card | `(?:\d[ \-]?){13,19}` + Luhn check | Universal |
| GPS decimal | `[-]?\d{1,2}\.\d{4,},\s*[-]?\d{1,3}\.\d{4,}` | Universal |
| URL | `(?:https?:\/\/\|www\.)[^\s<>"']+` | Universal |

---

## Europe

### EU / EEA — GDPR (Regulation 2016/679)
**Applies to:** All 27 EU member states + Iceland, Liechtenstein, Norway.  
**Scope:** Any organization processing data of EU residents, regardless of where the org is based.  
**Key requirements:** Lawful basis, data minimization, right to erasure, DPA required for processors.  
**Anonymization standard:** Data is only considered anonymous if re-identification is not reasonably possible.

*Country-specific identifiers within the EU:*

| Country | Identifier | Pattern hint |
|---|---|---|
| DE | Steuernummer | `\d{2,3}\/\d{3}\/\d{4,5}` |
| DE | USt-ID | `DE\d{9}` |
| DE | Sozialversicherungsnummer | `\d{2}\d{6}[A-Z]\d{3}` |
| DE | Krankenversicherungsnummer | `[A-Z]\d{9}` |
| DE | Personalausweis | `[A-Z0-9]{9}` (9 chars alphanumeric) |
| DE | Kfz-Kennzeichen | `[A-ZÄÖÜ]{1,3}[\s\-][A-Z]{1,2}[\s]?\d{1,4}[HE]?` |
| FR | INSEE / NIR (Sécu) | `[12]\d{2}\d{2}\d{5}\d{3}\d{2}` |
| FR | SIRET | `\d{14}` |
| FR | SIREN | `\d{9}` |
| NL | BSN (Burger Service Nummer) | `\d{8,9}` (8-9 digits, validated by 11-proof) |
| IT | Codice Fiscale | `[A-Z]{6}\d{2}[A-Z]\d{2}[A-Z]\d{3}[A-Z]` |
| IT | Partita IVA | `IT\d{11}` or `\d{11}` |
| ES | DNI | `\d{8}[A-Z]` |
| ES | NIE | `[XYZ]\d{7}[A-Z]` |
| ES | CIF | `[A-Z]\d{7}[A-Z0-9]` |
| PL | PESEL | `\d{11}` (with checksum) |
| SE | Personnummer | `\d{6}[-+]\d{4}` |
| BE | INSZ/NISS | `\d{2}\.\d{2}\.\d{2}-\d{3}\.\d{2}` |
| AT | SVNr | `\d{4}\d{6}` |

### UK — UK GDPR + Data Protection Act 2018
**Post-Brexit:** UK GDPR mirrors EU GDPR with minor deviations. Adequacy decision from EU in place (check current status).  
**Key difference:** ICO (Information Commissioner's Office) is the supervisory authority, not an EU DPA.

| Identifier | Pattern hint |
|---|---|
| NI number | `[A-Z]{2}\s?\d{2}\s?\d{2}\s?\d{2}\s?[A-Z]` |
| NHS number | `\d{3}\s?\d{3}\s?\d{4}` |
| UK postcode | `[A-Z]{1,2}\d[A-Z\d]?\s?\d[A-Z]{2}` |
| UK phone | `(?:\+44\|0)\d{4}[\s\-]?\d{6}` or `(?:\+44\|0)\d{3}[\s\-]?\d{3}[\s\-]?\d{4}` |
| UK driving license | `[A-Z]{5}\d{6}[A-Z]{2}\d[A-Z]{2}` |
| UK passport | `[0-9]{9}` |

### Switzerland — DSG (revised Federal Act on Data Protection, nDSG, since 2023)
Similar to GDPR in scope and requirements. Not an EU member but maintains equivalence.

| Identifier | Pattern hint |
|---|---|
| AHV-Nummer | `756\.\d{4}\.\d{4}\.\d{2}` |
| CH phone | `\+41\s?\d{2}\s?\d{3}\s?\d{2}\s?\d{2}` |

---

## North America

### United States — Sector-specific (no single federal law)

The US has no single federal privacy law equivalent to GDPR. Multiple sector-specific laws apply:

| Law | Scope | Key PII |
|---|---|---|
| **HIPAA** | Health data (covered entities + BAs) | PHI — 18 identifiers (names, dates, phone, fax, email, SSN, MRN, health plan numbers, account numbers, cert/license numbers, VINs, device identifiers, URLs, IPs, biometrics, full-face photos, geographic < 3 digits) |
| **CCPA / CPRA** | California residents; businesses above threshold | Broad: any info that identifies or is linkable to a CA resident |
| **FERPA** | Student education records (US) | Student names, IDs, grades, schedules, addresses |
| **COPPA** | Children under 13 (US) | Name, address, phone, persistent ID, geolocation |
| **GLBA** | Financial institutions | Account numbers, SSN, financial transaction data |
| **SHIELD Act** | New York state | SSN, driver's license, account + security code |

**State laws:** California (CCPA/CPRA), Virginia (CDPA), Colorado (CPA), Connecticut (CTDPA), Texas (TDPSA), Florida (FDBR) — and growing. When targeting US users, check the most restrictive applicable state law.

| Identifier | Pattern hint |
|---|---|
| SSN | `\d{3}-\d{2}-\d{4}` |
| EIN (tax ID) | `\d{2}-\d{7}` |
| US ZIP code | `\d{5}(?:-\d{4})?` |
| US phone (NANP) | `(?:\+?1[\s.-]?)?\(?\d{3}\)?[\s.-]?\d{3}[\s.-]?\d{4}` |
| US passport | `[A-Z]\d{8}` |
| Medicare (US) | `[1-9][A-Z]{2}\d{2}-?[A-Z]{2}\d-?\d{2}` (MBI format) |
| VIN (vehicle) | `[A-HJ-NPR-Z\d]{17}` |
| MRN (medical record) | Facility-specific, no universal pattern — use keyword context |

**HIPAA safe harbor** (18 identifiers to remove for de-identification):  
Names, geographic < state level, all date elements (except year) for persons > 89, phone, fax, email, SSN, MRN, health plan number, account numbers, certificate/license numbers, VINs, device identifiers, URLs, IPs, biometrics, full-face photos, unique identifiers.

### Canada — PIPEDA + Quebec Law 25 (Bill 64)
**PIPEDA:** Federal, applies to private sector organizations in interprovincial/international commerce.  
**Quebec Law 25:** Stricter than PIPEDA; requires privacy impact assessments, explicit consent, breach notifications.  
**Alberta / BC:** Have their own substantially-similar provincial laws.

| Identifier | Pattern hint |
|---|---|
| SIN (Social Insurance Number) | `\d{3}-\d{3}-\d{3}` |
| CA phone (NANP, same as US) | `(?:\+?1[\s.-]?)?\(?\d{3}\)?[\s.-]?\d{3}[\s.-]?\d{4}` |
| Health card (provincial) | Province-specific (e.g. ON: `\d{10}[A-Z]{2}`) |
| CA postcode | `[A-Z]\d[A-Z]\s?\d[A-Z]\d` |

---

## Latin America

### Brazil — LGPD (Lei Geral de Proteção de Dados, Law 13.709/2018)
Similar structure to GDPR. ANPD is the supervisory authority. Applies to any org processing data of Brazilian residents.

| Identifier | Pattern hint |
|---|---|
| CPF (Cadastro de Pessoas Físicas) | `\d{3}\.\d{3}\.\d{3}-\d{2}` |
| CNPJ (company) | `\d{2}\.\d{3}\.\d{3}/\d{4}-\d{2}` |
| RG (state ID, format varies) | `\d{2}\.\d{3}\.\d{3}-[\dX]` (SP format) |
| BR phone | `(?:\+55\s?)?\(?\d{2}\)?\s?\d{4,5}-\d{4}` |
| CEP (postal code) | `\d{5}-\d{3}` |
| CNH (driver's license) | `\d{11}` |
| PIS/PASEP | `\d{3}\.\d{5}\.\d{2}-\d` |

### Mexico — LFPDPPP (Ley Federal de Protección de Datos Personales)
| Identifier | Pattern hint |
|---|---|
| CURP | `[A-Z]{4}\d{6}[HM][A-Z]{5}[A-Z\d]\d` |
| RFC (tax) | `[A-Z]{3,4}\d{6}[A-Z\d]{3}` |
| MX phone | `(?:\+52\s?)?\d{2,3}[\s.-]?\d{3,4}[\s.-]?\d{4}` |
| CP (postal code) | `\d{5}` |

### Argentina — Law 25.326 (Personal Data Protection)
| Identifier | Pattern hint |
|---|---|
| DNI | `\d{7,8}` |
| CUIL/CUIT | `\d{2}-\d{8}-\d` |
| AR phone | `(?:\+54\s?)?\(?\d{2,4}\)?\s?\d{6,8}` |

### Chile — Law 19.628 + Law 21.719 (new, 2024)
| Identifier | Pattern hint |
|---|---|
| RUT (Rol Único Tributario) | `\d{1,2}\.\d{3}\.\d{3}-[\dKk]` |
| CL phone | `(?:\+56\s?)?\d{1}[\s.-]?\d{4}[\s.-]?\d{4}` |

---

## Asia Pacific

### China — PIPL (Personal Information Protection Law, 2021) + CSL + DSL
**Strictest cross-border transfer rules globally.** Data localization for "critical information infrastructure operators." Cross-border transfers require: standard contract, security assessment, or certification. Consent-based regime.

| Identifier | Pattern hint |
|---|---|
| Resident ID (身份证) | `\d{17}[\dXx]` (18-char: 6 region + 8 DOB + 3 seq + 1 check) |
| CN phone | `(?:\+86\s?)?1[3-9]\d{9}` |
| UnionPay card | `62\d{14,17}` |
| CN passport | `[EeGg]\d{8}` or `[Pp]\d{7}` |
| Bank account | `\d{16,19}` (overlap with credit card — use context) |

### India — DPDP Act 2023 (Digital Personal Data Protection Act)
Enforcement ongoing as of 2026. Consent required; data fiduciaries must notify breaches. Cross-border transfer to approved countries only.

| Identifier | Pattern hint |
|---|---|
| Aadhaar | `\d{4}\s?\d{4}\s?\d{4}` (12 digits) |
| PAN | `[A-Z]{5}\d{4}[A-Z]` |
| Passport | `[A-Z]\d{7}` |
| Voter ID | `[A-Z]{3}\d{7}` |
| IN phone | `(?:\+91\s?)?[6-9]\d{9}` |
| UPI ID / VPA | `[a-zA-Z0-9._-]+@[a-zA-Z]+` (overlap with email — check domain) |
| PIN code (postal) | `\d{6}` |

### Japan — APPI (Act on the Protection of Personal Information, revised 2022)
Third-party transfer restrictions; mandatory breach notification; PPC (Personal Information Protection Commission) oversight.

| Identifier | Pattern hint |
|---|---|
| My Number (個人番号) | `\d{4}\s?\d{4}\s?\d{4}` (12 digits) |
| JP phone | `(?:\+81\s?)?0\d{1,4}[\s.-]?\d{1,4}[\s.-]?\d{4}` |
| JP postal code | `\d{3}-\d{4}` |

### South Korea — PIPA (Personal Information Protection Act, revised 2023)
One of the strictest regimes in Asia. Explicit consent per purpose. Cross-border transfers require consent or adequacy.

| Identifier | Pattern hint |
|---|---|
| Resident Registration Number (RRN) | `\d{6}-\d{7}` |
| KR phone | `(?:\+82\s?)?0\d{1,2}[\s.-]?\d{3,4}[\s.-]?\d{4}` |
| KR passport | `[A-Z]{2}\d{7}` |
| Driver's license | `\d{2}-\d{2}-\d{6}-\d{2}` |

### Australia — Privacy Act 1988 + Australian Privacy Principles (APPs)
OAIC (Office of the Australian Information Commissioner). Notifiable Data Breaches scheme.

| Identifier | Pattern hint |
|---|---|
| TFN (Tax File Number) | `\d{3}\s?\d{3}\s?\d{3}` |
| Medicare number | `\d{10}` or `\d{4}\s?\d{5}\s?\d{1}` |
| AU phone | `(?:\+61\s?)?(?:0\d{1})[\s.-]?\d{4}[\s.-]?\d{4}` |
| AU postcode | `\d{4}` |
| ABN (company) | `\d{2}\s?\d{3}\s?\d{3}\s?\d{3}` |
| Driver's license | State-specific (no universal pattern) |

### Singapore — PDPA (Personal Data Protection Act 2012, amended 2020)
PDPC oversight. Mandatory breach notification (3 days for significant breaches). Cross-border: adequacy or contractual protection.

| Identifier | Pattern hint |
|---|---|
| NRIC / FIN | `[STFGM]\d{7}[A-Z]` |
| SG phone | `(?:\+65\s?)?\d{4}\s?\d{4}` |
| SG postal code | `\d{6}` |

---

## Russia & CIS

### Russia — Federal Law No. 152-FZ (On Personal Data)
Data localization requirement: personal data of Russian citizens must be stored on servers within Russia. Roskomnadzor oversight.

| Identifier | Pattern hint |
|---|---|
| SNILS (pension) | `\d{3}-\d{3}-\d{3}\s?\d{2}` |
| INN (tax, individual) | `\d{12}` |
| INN (company) | `\d{10}` |
| Passport series + number | `\d{4}\s\d{6}` |
| OGRN (company reg) | `\d{13}` |
| RU phone | `(?:\+7\|8)[\s.-]?\(?\d{3}\)?[\s.-]?\d{3}[\s.-]?\d{2}[\s.-]?\d{2}` |

---

## Middle East & Africa

### South Africa — POPIA (Protection of Personal Information Act 4/2013)
POPIA is broadly equivalent to GDPR. POPIA started enforcement in July 2021. Information Regulator oversight.

| Identifier | Pattern hint |
|---|---|
| SA ID number | `\d{13}` (6 DOB + 4 gender/seq + 1 citizen + 1 race legacy + 1 check) |
| SA phone | `(?:\+27\s?)?0?\d{2}[\s.-]?\d{3}[\s.-]?\d{4}` |
| SA postal code | `\d{4}` |
| SARS tax number | `\d{10}` |

### UAE — PDPL (Personal Data Protection Law, Federal Decree-Law No. 45/2021)
TDRA oversight. Applies to UAE residents' data. Sector-specific laws in Dubai DIFC and Abu Dhabi ADGM apply their own regimes.

| Identifier | Pattern hint |
|---|---|
| Emirates ID | `784-\d{4}-\d{7}-\d` |
| UAE phone | `(?:\+971\s?)?0?\d{2}[\s.-]?\d{7}` |
| UAE postcode | `\d{5,6}` (not universal) |

### Saudi Arabia — PDPL (Personal Data Protection Law 2021, amendments 2023)
NDMO and SAMA oversight. Explicit consent, data localization for sensitive data. Cross-border requires NDMO approval.

### Kenya — Data Protection Act 2019
ODPC oversight. First comprehensive data protection law in East Africa.

### Nigeria — NDPR (Nigeria Data Protection Regulation 2019) → NDPA 2023
NDPC oversight.

---

## Custom / Enterprise-Defined PII

Beyond statutory categories, organizations often define additional sensitive fields:

| Type | Examples | Approach |
|---|---|---|
| Internal employee IDs | `EMP-12345`, `DE-2024-001` | Custom regex matching the org's ID format |
| Customer account numbers | Org-specific format | Custom regex |
| Contract / case numbers | Org-specific format | Custom regex |
| Project codes | Org-specific format | Custom regex |
| Internal location codes | Floor/room/building codes | Keyword list |
| Salary / compensation data | Keywords: "salary", "Gehalt", "compensation" | Keyword + LLM |
| Trade secrets / IP identifiers | Product codes that are confidential | Keyword list |

**Approach:** Ask the user explicitly:
> "Does your organization have custom identifiers or internal data categories that need anonymization?
> Please describe the format — I'll generate a regex or keyword matcher for it."

---

## Quick Law Reference Table

| Region | Primary Law | Supervisory Authority | Data Residency Rule |
|---|---|---|---|
| EU/EEA | GDPR | National DPAs / EDPB | Transfers require adequacy/SCCs |
| UK | UK GDPR + DPA 2018 | ICO | Adequacy or safeguards |
| Switzerland | nDSG (revised DSG) | FDPIC | Adequacy / safeguards |
| USA (federal) | Sector-specific (HIPAA, GLBA, FERPA…) | FTC + sector regulators | No general residency rule |
| USA (California) | CCPA / CPRA | CPPA | No residency rule |
| Canada | PIPEDA + provincial | OPC | No strict residency (Quebec stricter) |
| Brazil | LGPD | ANPD | No strict residency |
| China | PIPL + CSL + DSL | CAC / SAMR | Strict — critical data must stay in CN |
| India | DPDP Act 2023 | DPB | Approved countries only |
| Japan | APPI | PPC | Approved countries / contractual |
| South Korea | PIPA | PIPC | Consent or adequacy |
| Australia | Privacy Act / APPs | OAIC | APPs require equivalent protection abroad |
| Singapore | PDPA | PDPC | Adequate protection required |
| Russia | 152-FZ | Roskomnadzor | Citizens' data must be stored in Russia |
| South Africa | POPIA | Information Regulator | Comparable protection required |
| UAE | PDPL | TDRA | Approval required |
| Mexico | LFPDPPP | INAI | No strict residency |
| Argentina | Law 25.326 | AAIP | Adequacy required |
