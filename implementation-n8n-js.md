# Reference Implementation: n8n Code Node (JavaScript)

This is the production reference implementation for Stage 1 (regex) anonymization,
running as an n8n Code Node. Use it as a starting template for JavaScript/Node.js targets.

---

## Coverage (Stage 1 — this file)

✅ Email addresses  
✅ URLs (http/https/www)  
✅ German street addresses (10 street-type suffixes)  
✅ German ZIP + city (5-digit PLZ)  
✅ Dates (d.m.y, d-m-y, d/m/y formats)  
✅ Phone numbers (with looksLikePhone validation — excludes dates, times, short numbers)  
✅ Company names with legal suffixes (GmbH, AG, UG, KG, OHG) — **protected from false-positive replacement**

---

## Known Gaps (extend as needed)

❌ ISO dates (`2024-01-15`) — not matched by current DATE regex  
❌ IBAN  
❌ Credit/debit card numbers (requires Luhn check)  
❌ IPv4 / IPv6  
❌ MAC address  
❌ German Kfz-Kennzeichen  
❌ German Steuernummer / USt-ID  
❌ German Sozialversicherungsnummer  
❌ GPS coordinates  
❌ Company suffixes SE, GbR, e.V., e.K., PartG (missing from protection list)  
❌ Catch-all phone regex at end of `sanitizeLine` runs **without** `looksLikePhone` validation → risk of false positives on article numbers, prices, etc.

---

## The Code

```javascript
const CONFIG = {
  inputField: "beschreibung",
  outputField: "beschreibung_bereinigt",
  keepOriginal: true,
  originalField: "beschreibung_original",
  overwriteInputField: false,
  placeholders: {
    EMAIL: "[E-MAIL ENTFERNT]",
    PHONE: "[TELEFON ENTFERNT]",
    URL: "[LINK ENTFERNT]",
    ADDRESS: "[ADRESSE ENTFERNT]",
    POSTCODE_OR_CITY: "[PLZ/ORT ENTFERNT]",
    DATE: "[DATUM ENTFERNT]",
  },
};

// ---------------- REGEX ----------------
const RE = {
  EMAIL: /\b[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}\b/gi,
  URL: /\b(?:https?:\/\/|www\.)[^\s<>"']+/gi,
  STREET: /\b[A-ZÄÖÜ][A-Za-zÄÖÜäöüß\-]*(?:straße|str\.|strasse|weg|platz|allee|gasse|ring|ufer|damm|steig|pfad|promenade|chaussee)\s+\d+[A-Za-z]?\b/gi,
  PLZORT: /\b\d{5}\s+[A-ZÄÖÜ][A-Za-zÄÖÜäöüß\-]+\b/g,
  DATE: /\b\d{1,2}[.\-/]\d{1,2}[.\-/]\d{2,4}\b/g,
  PHONE: /(^|[^\w])((?:\+|00)?\s*(?:\(?\d{1,4}\)?[ \-./]*)?(?:\d[ \-./]*){5,}\d)(?=$|[^\w])/g,
};

// ---------------- HELPERS ----------------
function normalize(text) {
  return String(text || "")
    .replace(/\r\n?/g, "\n")
    .replace(/\u00A0/g, " ")
    .trim();
}

function looksLikePhone(candidate) {
  const s = String(candidate).trim();
  if (/^\d{1,2}[.\-/]\d{1,2}[.\-/]\d{2,4}$/.test(s)) return false; // date
  if (/^\d{1,2}:\d{2}$/.test(s)) return false;                       // time
  const digits = s.replace(/\D/g, "");
  if (digits.length < 7 || digits.length > 15) return false;
  const hasStructure =
    /^\+/.test(s) ||
    /^00/.test(s) ||
    /[\s\-./]/.test(s) ||
    /\(\d+\)/.test(s);
  if (!hasStructure) return false;
  return true;
}

// ---------------- CORE ----------------
function sanitizeLine(line) {
  // Protect company names from false-positive replacement
  const companies = [];
  line = line.replace(
    // NOTE: SE, GbR, e.V., e.K., PartG are currently missing here
    /\b[A-ZÄÖÜ][A-Za-zÄÖÜäöüß]+(?:\s+[A-ZÄÖÜ][A-Za-zÄÖÜäöüß]+)*\s+(GmbH|AG|UG|KG|OHG)\b/g,
    (match) => {
      companies.push(match);
      return `__COMPANY_${companies.length - 1}__`;
    }
  );

  line = line.replace(RE.EMAIL, CONFIG.placeholders.EMAIL);
  line = line.replace(RE.URL, CONFIG.placeholders.URL);
  line = line.replace(RE.STREET, CONFIG.placeholders.ADDRESS);
  line = line.replace(RE.PLZORT, CONFIG.placeholders.POSTCODE_OR_CITY);
  line = line.replace(RE.DATE, CONFIG.placeholders.DATE);
  line = line.replace(RE.PHONE, (full, prefix, cand) => {
    return looksLikePhone(cand)
      ? `${prefix}${CONFIG.placeholders.PHONE}`
      : full;
  });

  // ⚠️ KNOWN ISSUE: this catch-all runs WITHOUT looksLikePhone — may produce false positives
  line = line.replace(
    /\(?\d{3,5}\)?[\d\s\-./]{5,}/g,
    CONFIG.placeholders.PHONE
  );

  // Restore protected company names
  companies.forEach((c, i) => {
    line = line.replace(`__COMPANY_${i}__`, c);
  });

  return line;
}

function sanitize(text) {
  return normalize(text)
    .split("\n")
    .map(sanitizeLine)
    .join("\n");
}

// ---------------- N8N ENTRY POINT ----------------
const items = $input.all();
return items.map(item => {
  const json = { ...item.json };
  const raw = json[CONFIG.inputField];
  const cleaned = sanitize(raw);
  if (CONFIG.keepOriginal) {
    json[CONFIG.originalField] = raw;
  }
  json[CONFIG.outputField] = cleaned;
  if (CONFIG.overwriteInputField) {
    json[CONFIG.inputField] = cleaned;
  }
  return { json };
});
```

---

## Suggested Extensions

### ISO date format
Add to `RE`:
```javascript
DATE_ISO: /\b\d{4}[-]\d{2}[-]\d{2}\b/g,
```
Apply before `RE.DATE`.

### IBAN (international)
```javascript
IBAN: /\b[A-Z]{2}\d{2}[\s]?(?:[A-Z0-9]{4}[\s]?){2,7}[A-Z0-9]{1,4}\b/g,
```

### IPv4
```javascript
IPV4: /\b(?:\d{1,3}\.){3}\d{1,3}\b/g,
```
Add Luhn-style validation if needed (e.g. exclude `192.168.x.x` for internal IPs).

### IPv6
```javascript
IPV6: /\b(?:[0-9a-fA-F]{1,4}:){7}[0-9a-fA-F]{1,4}\b|\b(?:[0-9a-fA-F]{1,4}:)*:(?::[0-9a-fA-F]{1,4})*\b/g,
```

### German Kfz-Kennzeichen
```javascript
KFZ: /\b[A-ZÄÖÜ]{1,3}[\s\-][A-Z]{1,2}[\s]?\d{1,4}[HE]?\b/g,
```

### German Steuernummer
```javascript
STEUER: /\b\d{2,3}\/\d{3}\/\d{4,5}\b/g,
```

### German USt-ID
```javascript
UST: /\bDE\d{9}\b/g,
```

### German Sozialversicherungsnummer
```javascript
SV: /\b\d{2}\d{6}[A-Z]\d{3}\b/g,
```

### Company suffix expansion
```javascript
/\b[A-ZÄÖÜ][A-Za-zÄÖÜäöüß]+(?:\s+[A-ZÄÖÜ][A-Za-zÄÖÜäöüß]+)*\s+(GmbH|AG|UG|KG|OHG|SE|GbR|PartG|e\.V\.|e\.K\.)\b/g
```

### Fix catch-all phone regex (add looksLikePhone guard)
```javascript
line = line.replace(
  /\(?\d{3,5}\)?[\d\s\-./]{5,}/g,
  (match) => looksLikePhone(match) ? CONFIG.placeholders.PHONE : match
);
```
