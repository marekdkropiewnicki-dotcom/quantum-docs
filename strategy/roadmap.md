# Roadmap — maj→lipiec 2026

**Data:** 16 maja 2026
**Horyzont:** 7 tygodni do deadline MiCA (1 lipca 2026)
**Reguła:** każdy tydzień ma jeden deliverable, który da się zademonstrować w 5 minut.

---

## Tydzień 1 — 16–22 maja: Fundament + ślad strategii ✅

- [x] **Pole strategiczne w `quantum-docs`** — manifest, teza, architektura, ADR. *Commit [`525c217`](https://github.com/marekdkropiewnicki-dotcom/quantum-docs/commit/525c217).*
- [x] **Repo `proquant-core`** — struktura `app/`, `tests/`, `evals/`, `mcp/`, `infra/`. *[proquant-core](https://github.com/marekdkropiewnicki-dotcom/proquant-core).*
- [x] **FastAPI hello-world lokalnie** — endpoint `/health` + structured logging (structlog). *5/5 testów green, 83% coverage.*
- [x] **Supabase schema** — `users`, `consents`, `decisions` (APPEND-ONLY), `decision_outcomes`. RLS włączone od dnia 1. *Plik [`infra/supabase_schema.sql`](https://github.com/marekdkropiewnicki-dotcom/proquant-core/blob/main/infra/supabase_schema.sql).*
- [x] **Promptfoo skeleton w CI** — 2 smoke testy (MiCA disclaimer + format). Tydzień 3 rozszerza do 50.
- [x] **Realny deploy na Railway** — `/health` zwraca 200 na `https://api-production-11fb.up.railway.app/health`. Klucze sekretne (Anthropic, Supabase service_role) do uzupełnienia gdy operator wróci na desktop.

**Definicja done:** GitHub Actions zielone ✅, `/health` w produkcji → 200 ✅. Tydzień 1 closed.

---

## Tydzień 2 — 23–29 maja: Pierwsza decyzja end-to-end ✅

- [x] **M1 Decision Loop** — `POST /decide` z routerem (Haiku/Sonnet/Opus), prompt caching, persist do `decisions` + `usage_log`, cost guard z dual-budget (per-user €5, global €50). *PR [#13](https://github.com/marekdkropiewnicki-dotcom/proquant-core/pull/13), commit [`8854101`](https://github.com/marekdkropiewnicki-dotcom/proquant-core/commit/8854101).*
- [x] **Sonnet 4.6 lane** z prompt caching — system prompt + tools w cache (TTL 5 min, hit-rate measured).
- [x] **`decisions` zapis** — każda decyzja idzie do Supabase (APPEND-ONLY z RLS). Embedding via pgvector — w iteracji M2.
- [x] **Reverse-solicitation disclaimer** + MiCA pre-flight (`app/agent/compliance.py`).
- [x] **Telegram webhook** — `app/routers/telegram.py` z HMAC signature verify. Aiogram bootstrap → M2.
- [x] **Cost guard** — daily budget enforcement, atomic insert do `usage_log`, race-condition issue ([#14](https://github.com/marekdkropiewnicki-dotcom/proquant-core/issues/14)) tracked dla M2.

**Definicja done:** 42/42 testów green, 68% coverage, `POST /decide` zwraca strukturyzowaną odpowiedź zgodną ze schema. Tydzień 2 closed.

---

## Tydzień 3 — 30 maja–5 czerwca: Trzy lane'y + budget guard

- [ ] **Router (Haiku / Sonnet / Opus)** w hooks Claude Agent SDK.
- [ ] **Budget guard** — hard limit $5/dzień/user, hard limit $50/dzień globalny.
- [ ] **Counterfactual replay** — endpoint `/replay/{decision_id}` pokazujący „co by się stało, gdybyś nie zrobił".
- [ ] **Sleep-well score** — pytanie po każdej decyzji, zapis 1–10.

**Definicja done:** dashboard `/admin/cost` pokazuje koszt per lane per dzień, replay działa dla ostatnich 50 decyzji.

---

## Tydzień 4 — 6–12 czerwca: MCP server publiczny

- [ ] **`proquant-tools` MCP server** — OSS na GitHubie, MIT license.
- [ ] **3 narzędzia w MCP**: `get_market_snapshot`, `evaluate_thesis`, `log_decision`.
- [ ] **Dokumentacja MCP** w `quantum-docs/integrations/mcp.md`.
- [ ] **Submit do MCP Registry** (Anthropic publiczny katalog).

**Definicja done:** ktoś z Claude Desktop dodaje `proquant-tools` i wykonuje pierwsze zapytanie.

---

## Tydzień 5 — 13–19 czerwca: Płatności + ewaluacja

- [ ] **Stripe Checkout** — €19/m Pro tier, €99/m Sovereign tier (placeholder).
- [ ] **Telegram Stars** — tylko dla zakupu „extra reasoning calls" ≤€5.
- [ ] **Braintrust** — pierwsze production evals (regresja na 100 prawdziwych decyzjach).
- [ ] **PWA `proquant.app`** — landing + login + 1 strona „decision history".

**Definicja done:** test purchase działa, w Braintrust widać scoring decyzji z ostatnich 7 dni.

---

## Tydzień 6 — 20–26 czerwca: MiCA hardening + Sovereign POC

- [ ] **Reverse-solicitation playbook** zaimplementowany w UI — onboarding z explicit consent.
- [ ] **Disclaimer matrix** — różne teksty dla EU/UK/US/RoW.
- [ ] **EIP-7702 session key POC** — testnet, jeden swap.
- [ ] **`/legal/{mica,terms,privacy}`** endpoints + audit log dostępu.

**Definicja done:** prawnik (kontakt zewnętrzny) zatwierdza warstwę compliance jako „defensible" dla solo-foundera.

---

## Tydzień 7 — 27 czerwca–3 lipca: Beta launch

- [ ] **Soft launch** — 20 osób z prywatnej listy.
- [ ] **Monitoring** — Sentry + Railway metrics + Braintrust live.
- [ ] **MiCA D-day** (1 lipca) — wszystko musi być compliant.
- [ ] **Retrospektywa publiczna** — post na `quantum-docs/posts/` o pierwszych 7 dniach.

**Definicja done:** 20 użytkowników aktywnych, 0 incydentów P0/P1, raport pierwszych metryk.

---

## Reguły gry

1. **Każdy tydzień kończy się demo.** Jeśli nie ma demo — tydzień się nie skończył.
2. **Każdy slip dodaje 1 tydzień do końca, nie ścieśnia środka.** Nie ścinamy jakości.
3. **Każdy deliverable ma ADR.** Bez ADR nie ma merge.
4. **Każdy commit linkuje do roadmapy.** `[wk1]`, `[wk2]` w message.
