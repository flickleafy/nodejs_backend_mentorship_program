# Internationalization (i18n) & Localization (l10n)

Below are **20 “Internationalization & Localization” build‑style exercises** for Node.js (JavaScript) that progress from easy to near mid‑level. Prefer **Express**. Suggested libs: negotiation (**accept-language-parser**), messages (**i18next**, **@formatjs/intl**, **messageformat/ICU**), dates/times (**Luxon** or **Temporal polyfill**), numbers & currency (**Intl.NumberFormat**), collation/search (**Intl.Collator**), phone/address (**libphonenumber‑js**, **CLDR** helpers), docs (**swagger‑ui‑express**).  
Use a consistent response envelope (e.g., `{ data, meta, error }`), stable error codes, and never echo secrets.

* * *

## 1) Locale negotiation with `Accept-Language` + allow‑list

**Goal:** Detect user locale and set a safe default.  
**Build steps:**

1. Add middleware that parses `Accept-Language` and matches against an allow‑list (e.g., `['en', 'pt-BR', 'es']`).

2. Implement a fallback chain (e.g., `pt-BR → pt → en`).

3. Expose `GET /whoami` returning `{ locale, fallbackChain }`.  
    **Acceptance criteria:**

- `Accept-Language: pt-BR,es;q=0.8` → `locale='pt-BR'`, `fallbackChain=['pt-BR','pt','en']`.

- Unknown locales pick default (e.g., `en`) and include a `meta.warning`.

* * *

## 2) Locale routing via path prefix + canonical redirects

**Goal:** Support routes like `/en/tasks` and `/pt-BR/tasks`.  
**Build steps:**

1. Add a router that extracts `:locale` prefix and validates it against the allow‑list.

2. Redirect bare `/tasks` → `/<best-locale>/tasks` using negotiation.

3. Add `Link: <...>; rel="alternate"; hreflang="..."` headers for SEO.  
    **Acceptance criteria:**

- `/tasks` 302‑redirects to `/<locale>/tasks`.

- Invalid locale prefix (e.g., `/xx/tasks`) → `404 { error.code:'UNSUPPORTED_LOCALE' }`.

* * *

## 3) Message catalogs & fallback (i18next or ICU)

**Goal:** Centralize translations with fallback.  
**Build steps:**

1. Organize catalogs: `/locales/<lang>/common.json`.

2. Initialize i18next with `fallbackLng:'en'` and `keySeparator:false`.

3. Add `t(key, vars)` helper and use it in controllers.  
    **Acceptance criteria:**

- Missing key logs once at DEBUG and returns a safe placeholder like `__key__`.

- Changing locale toggles message language without server restart.

* * *

## 4) ICU pluralization & ordinals

**Goal:** Correct messages for counts in different locales.  
**Build steps:**

1. Define ICU messages, e.g., `"{count, plural, one {# item} other {# items}}"`.

2. Add ordinal example: `"{n, selectordinal, one {#st} two {#nd} few {#rd} other {#th}}"`.

3. Expose `/items/count?count=` that returns localized strings.  
    **Acceptance criteria:**

- English: `count=1 → "1 item"`, `count=2 → "2 items"`.

- Portuguese handles plural correctly; ordinal examples render per locale rules.

* * *

## 5) Gender/select messages

**Goal:** Respect grammatical gender in messages.  
**Build steps:**

1. Add ICU `select` pattern: `"invite.{gender, select, male {Ele} female {Ela} other {Elu}} convidou você"`.

2. Endpoint: `POST /invites/preview { gender }`.  
    **Acceptance criteria:**

- `gender='female'` returns string with the correct pronoun in each supported language.

- Unknown gender falls back to `other`.

* * *

## 6) Time zone–aware date/time formatting

**Goal:** Show local times correctly, including DST.  
**Build steps:**

1. Store timestamps in UTC; accept an optional `tz` param (IANA name).

2. Format with Luxon/Intl to localized date/time styles.

3. Add `/events/when?id=&tz=` that returns localized `start`, `end`.  
    **Acceptance criteria:**

- Supplying `tz='America/Sao_Paulo'` reflects DST transitions correctly.

- Missing/invalid `tz` → default to server policy and include `meta.note`.

* * *

## 7) Numbers, currencies & rounding policies

**Goal:** Render amounts per locale rules.  
**Build steps:**

1. Use `Intl.NumberFormat` with `{ style:'currency', currency }`.

2. Implement banker’s rounding or defined business rule.

3. Endpoint `/prices/quote?amount=1234.567&currency=BRL`.  
    **Acceptance criteria:**

- `pt-BR` uses comma decimal separator; `en` uses dot.

- Currency symbol/code is correct; total is rounded per policy.

* * *

## 8) Locale‑aware sorting & diacritics‑insensitive search

**Goal:** Make search and sort feel native.  
**Build steps:**

1. Use `Intl.Collator(locale, { sensitivity:'base', ignorePunctuation:true })`.

2. Implement `/search?q=` that matches `"café"` when user types `"cafe"`.

3. Sort results with the same collator.  
    **Acceptance criteria:**

- `"cafe"` finds `"café"` in `pt-BR` and `es`.

- Sorting order differs appropriately between `en` and `sv` (documented example).

* * *

## 9) Pseudolocalization toggle

**Goal:** Surface truncation/concatenation issues.  
**Build steps:**

1. Add a pseudo‑locale (e.g., `en-XA`) that elongates and brackets strings.

2. Toggle via header `X-Pseudo-Locale: 1` or `?pseudo=1`.

3. Do not pseudolocalize dynamic numbers/dates.  
    **Acceptance criteria:**

- Responses show elongated text like `⟦Ēxámplē tēxt⟧`.

- Numeric/date fields remain unaffected.

* * *

## 10) Localized email templates (subject + body)

**Goal:** Send emails in the user’s language.  
**Build steps:**

1. Create per‑locale email templates using ICU; support HTML + plaintext.

2. Add `POST /emails/welcome` that picks templates by user locale.

3. Include a test transporter (no real sends) to capture outputs.  
    **Acceptance criteria:**

- Two locales render differing subjects and bodies with correct placeholders.

- Missing locale falls back to default with a flag in `meta.fallback=true`.

* * *

## 11) Localized validation errors (Zod/Ajv + i18next)

**Goal:** Translate validation feedback.  
**Build steps:**

1. Map library error codes to translation keys (e.g., `errors.required`, `errors.minLength`).

2. Use `req.locale` to pick messages; include `path` and limits.  
    **Acceptance criteria:**

- Submitting bad input returns errors in the chosen language.

- Error codes remain stable across locales.

* * *

## 12) RTL support & bidi safety

**Goal:** Support Arabic/Hebrew and prevent spoofing.  
**Build steps:**

1. For RTL locales, add `dir:'rtl'` in response `meta`.

2. Wrap interpolated user content with Unicode **bidi isolates** (`\u2068...\u2069`) in server‑rendered strings.

3. Sanitize and reject inputs containing forbidden bidi control characters unless escaped.  
    **Acceptance criteria:**

- RTL locales include `meta.dir='rtl'`.

- Injected LTR/RTL controls cannot visually spoof neighboring text in rendered HTML.

* * *

## 13) Locale‑aware file exports (CSV/Excel/PDF)

**Goal:** Export data with proper formats.  
**Build steps:**

1. Localize column headers, dates, and numbers.

2. Use semicolon separators for locales where comma is decimal (e.g., `pt-BR` CSV).

3. Set `Content-Disposition` filename encoded per RFC 5987 with localized name.  
    **Acceptance criteria:**

- `pt-BR` CSV uses `;` and localized decimals; `en` uses `,`.

- Filenames download with correct localized names across major browsers.

* * *

## 14) Caching & `Vary: Accept-Language`

**Goal:** Serve correct language from caches.  
**Build steps:**

1. Add `Vary: Accept-Language` to cacheable endpoints.

2. Key the server‑side cache by `locale` (and `tz` if applicable).

3. Distinct ETags per locale.  
    **Acceptance criteria:**

- Same resource differs by locale and returns different ETags.

- Intermediary cache does not cross‑serve languages.

* * *

## 15) Localized OpenAPI (multi‑language docs)

**Goal:** Show API docs in user’s language.  
**Build steps:**

1. Externalize OpenAPI `summary`/`description` strings; offer `?lang=` to switch.

2. Serve `swagger-ui-express` with a language toggle that swaps prebuilt spec variants or overlays translations at runtime.  
    **Acceptance criteria:**

- `/docs?lang=pt-BR` shows Portuguese titles/descriptions.

- Undocumented strings fall back to English with a visible badge.

* * *

## 16) Units & measurement localization

**Goal:** Display units audiences expect.  
**Build steps:**

1. Normalize to SI internally; convert for display when needed (e.g., miles for `en-US`, km otherwise—document policy).

2. Format using `Intl.NumberFormat(locale, { style:'unit', unit:'kilometer' })` where supported.

3. Endpoint `/distance?meters=` returns localized strings.  
    **Acceptance criteria:**

- `pt-BR` shows kilometers with the right unit string.

- Conversion rules are deterministic and documented; rounding is consistent.

* * *

## 17) Phone & address normalization + display

**Goal:** Store canonical forms, render localized.  
**Build steps:**

1. Normalize phone numbers to **E.164** with `libphonenumber-js`.

2. Store address components normalized; render address lines per locale conventions (country‑specific ordering).

3. Validate and return localized hints for invalid inputs.  
    **Acceptance criteria:**

- `+55` numbers round‑trip and format as `(+55 xx) xxxx‑xxxx` for display (example).

- Invalid numbers show localized guidance; storage remains canonical.

* * *

## 18) i18n testing: snapshots, coverage & missing keys CI gate

**Goal:** Prevent regressions and untranslated UI.  
**Build steps:**

1. Snapshot critical responses in at least two locales (stable fields only).

2. Add a test that fails if translation keys are missing or extra (compare to `en`).

3. CI job prints a diff and fails on missing keys.  
    **Acceptance criteria:**

- Removing a key from a non‑EN catalog fails CI with a clear message.

- Snapshots catch accidental English strings in other locales.

* * *

## 19) Translation workflow & linting (placeholders, ICU syntax)

**Goal:** Safe translation handoffs.  
**Build steps:**

1. Export/import catalogs (JSON/PO) and document the process (e.g., with locize/crowdin or a repo submodule).

2. Add a linter that validates ICU syntax and **placeholder parity** between source and target.

3. Add a script that merges new keys and marks obsolete ones.  
    **Acceptance criteria:**

- A mismatched placeholder name or count fails CI.

- Obsolete keys are detected and reported.

* * *

## 20) Locale switching UX + persistence

**Goal:** Respect user choice and make switching predictable.  
**Build steps:**

1. `POST /me/locale { locale }` validates against allow‑list and sets a secure cookie (httpOnly not required, but `SameSite=Lax`, `Secure` in prod).

2. Middleware prioritizes explicit cookie over `Accept-Language`.

3. Include `meta.localeSource: 'cookie'|'header'|'default'`.  
    **Acceptance criteria:**

- After setting the cookie, subsequent requests use the chosen locale even if headers differ.

- Clearing the cookie falls back to negotiation; `meta.localeSource` reflects the source.

* * *

### General expectations (apply to all 20)

- **Security:** Treat locale, currency, and timezone as **untrusted input**. Validate against allow‑lists. Sanitize strings (strip control chars), especially around bidi controls and filenames.

- **Performance:** Cache per‑locale where safe; avoid synchronous filesystem reads on hot paths; lazy‑load catalogs and keep them in memory.

- **Observability:** Log `requestId`, `locale`, `tz`, and negotiation source; avoid logging personal data. Expose counters for missing keys and fallback hits.

- **Docs:** Document supported locales/time zones, fallback rules, date/number policies, and how to add a new locale. Keep OpenAPI/docs localized and version‑controlled.

- **Validation:** All user‑facing errors must be localized; keep **stable machine codes** (e.g., `errors.REQUIRED`) independent of human‑readable messages.

- **Consistency:** Avoid string concatenation; use ICU/messageformat with named placeholders; maintain parity of placeholders across locales.

- Add a minimal **README** with how to run, env vars (allowed locales, default tz), and curl examples for each exercise.

- If you want, turn the acceptance criteria into **automated tests with Supertest/Jest**,
