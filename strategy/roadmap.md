# Roadmap — maj→lipiec 2026

**Data:** 16 maja 2026
**Horyzont:** 7 tygodni do deadline MiCA (1 lipca 2026)
**Reguła:** każdy tydzień ma jeden deliverable, który da się zademonstrować w 5 minut.

---

## Tydzień 1 — 16–22 maja: Fundament + ślad strategii

- [x] **Pole strategiczne w `quantum-docs`** — manifest, teza, architektura, ADR (ten commit).
- [ ] **Repo `proquant-core`** — pusta struktura `app/`, `tests/`, `evals/`, `mcp/`, `infra/`.
- [ ] **FastAPI hello-world na Railway** — endpoint `/health` + structured logging (structlog).
- [ ] **Supabase schema** — tabele `decisions`, `decision_outcomes`, `users`, RLS włączone od dnia 1.
- [ ] **Promptfoo skeleton w CI** — 10 testów ewaluacyjnych dla wstępnego prompta tezy.

**Definicja done:** `curl proquant-core.up.railway.app/health` → 200 OK, GitHub Actions zielone.

---

## Tydzień 2 — 23–29 maja: Pierwsza decyzja end-to-end

- [ ] **Telegram bot bootstrap** (`aiogram` 3.x) — `/start`, `/decision`, `/log`.
- [ ] **Sonnet 4.6 lane** z prompt caching — system prompt + tools w cache.
- [ ] **`decision_log` zapis** — każda decyzja idzie do Supabase z embeddingiem.
- [ ] **Reverse-solicitation disclaimer** na każdej odpowiedzi (MiCA pre-flight).

**Definicja done:** użytkownik pisze do bota „czy kupić ETH za €500", dostaje strukturyzowaną odpowiedź *teza → plan → audit trail*, decyzja jest w bazie.

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
