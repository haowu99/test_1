# Interview Vocabulary & Phrase Cheat Sheet — SC NFC-e Provider

A reference for discussing this project in English. Organized by topic,
with ready-to-say sentences at the end.

---

## 1. Architecture / General Terms

| Term | Meaning |
|---|---|
| provider / adapter | A module that translates an external system's protocol into your app's internal format |
| normalize / normalization | Converting raw external data into a consistent internal shape |
| contract | An agreed-upon shape/interface that different implementations must satisfy |
| entry point | The main function/method other code calls to use a module |
| service layer | The layer of code that orchestrates business logic, sitting above providers |
| routing / dispatch | Deciding which provider/handler should process a given input |
| fallback | A secondary path used when the primary path isn't available |
| stateless | Not keeping data between calls — each call is independent |
| singleton | A single shared instance reused across the whole app |
| dependency injection | Passing in a dependency (like an HTTP client) instead of hardcoding it, so it can be swapped in tests |

## 2. Access Key / Domain Terms

| Term | Meaning |
|---|---|
| access key (chave de acesso) | The 44-digit identifier printed on a Brazilian invoice |
| check digit | A digit calculated from the rest of the number, used to catch typos/corruption |
| checksum / checksum validation | Verifying that check digit matches |
| document model | The 2-digit code identifying the invoice *type* (55 = NF-e, 65 = NFC-e) |
| UF code (state code) | 2-digit code identifying which Brazilian state issued the document |

## 3. Network / Protocol Terms

| Term | Meaning |
|---|---|
| endpoint | A specific URL a client sends requests to |
| host allowlist | Only permitting requests to a pre-approved set of domains |
| SSRF (Server-Side Request Forgery) protection | Preventing the server from being tricked into requesting an unintended/internal URL |
| session | Server-side state tied to a specific client across multiple requests |
| cookie jar | A local store of cookies used to maintain a session |
| postback | A form submission back to the same page (classic ASP.NET Web Forms pattern) |
| ViewState | Hidden serialized state ASP.NET Web Forms round-trips between requests |
| partial postback / AJAX delta | An update to only part of a page instead of a full reload |
| redirect | Instructing the client to fetch a different URL |
| rate limiting | A server deliberately slowing/blocking a client that sends too many requests |
| CAPTCHA / human verification challenge | A test meant to distinguish humans from bots |
| timeout | Giving up on a request after it takes too long |
| deep link | A URL that goes directly to a specific piece of content |
| opaque token | A value with no meaning to the client — it can be passed along but not decoded/constructed |

## 4. Testing Terms

| Term | Meaning |
|---|---|
| unit test | A test of one small piece of logic in isolation |
| integration test | A test that checks multiple pieces working together |
| mock | A fake stand-in for a real dependency, used in tests |
| stub | A simplified fake implementation returning canned data |
| fixture | Sample data used consistently across tests |
| test coverage | How much of the code is exercised by tests |
| edge case | An unusual input/situation worth testing separately |
| regression | A bug that reappears after being previously fixed |
| self-consistency test | A test that checks code matches its own declared rules, not an external source of truth |

## 5. Error Handling Terms

| Term | Meaning |
|---|---|
| exception / error class | A structured way to represent a specific failure |
| exception hierarchy | A set of related error types, often inheriting from a common base |
| explicit failure mode | Failing in a clearly labeled, predictable way instead of silently |
| graceful degradation | Handling an unexpected situation without crashing |
| defensive programming | Writing code that anticipates and safely handles bad/unexpected input |

## 6. Honest-Limitation Phrases (use these — don't overclaim)

- "This path is implemented but not production-verified."
- "I don't have certainty on that — here's what I do know, and here's the assumption I made."
- "This was validated manually once, but I don't yet have automated test coverage for it."
- "This is a known limitation, not something I tried to hide."
- "I'd want to confirm this against the real PR provider before merging it into the shared contract."

---

## 7. Ready-to-say sentences about this project

**One-line summary:**
> "I implemented an SC-specific provider/adapter behind the same internal Receipt contract already used for Paraná, isolating SC's transport and parsing details from the rest of the system."

**On the PR vs SC difference:**
> "Paraná and Santa Catarina render the same invoice HTML fragment — likely from a shared national template — but the way you *get* to that fragment is completely different: PR is a single stateless GET, while SC is a stateful ASP.NET Web Forms flow with session cookies."

**On the primary vs fallback path:**
> "The primary path, `fromQrContent`, takes the URL that a real QR code already encodes and does a single GET — this matched what I actually observed when scanning a real receipt, and it doesn't hit Cloudflare. The fallback, `fromAccessKey`, is for when only the bare 44-digit key is available; it has to simulate a manual form submission, which can be rate-limited or challenged."

**On Cloudflare / rate limiting:**
> "SEFAZ-SC can return either a Cloudflare human-verification challenge or its own rate-limit redirect page, depending on request volume. My code detects both and raises an explicit error instead of pretending the lookup succeeded. I did not attempt to bypass either protection."

**On the normalized data structure:**
> "I followed the same field-naming convention the PR parser used, and documented that convention explicitly in a shared contract file. I don't have the real PR provider's source to diff against field-by-field, so I'd describe this as 'the same convention, self-consistently verified' rather than 'byte-for-byte identical.'"

**On testing approach:**
> "Provider-layer tests inject a fake fetch implementation via dependency injection, so they're deterministic and don't depend on SEFAZ-SC being online, rate limits, or Cloudflare."

**On the redirect-following fix:**
> "After a successful form submission, SC doesn't return the invoice HTML directly — it returns an AJAX instruction telling the client to redirect to the same deep-link URL a real QR code would encode. I follow that redirect using the same session cookies to fetch the actual invoice page."

**On what's still open/unverified:**
> "The redirect-following logic was validated once, manually, against a real response — it doesn't yet have automated test coverage, and that's a known gap I'd want to close before calling this production-ready."

**If asked "is your output exactly identical to PR's":**
> "Not something I can claim with certainty — I preserved the existing naming convention and added two SC-specific optional fields the response exposes. Before merging into a shared contract, I'd want to verify against the real PR provider's actual output."
