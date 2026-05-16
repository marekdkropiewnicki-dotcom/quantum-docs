# Architektura referencyjna v2

**Data:** 16 maja 2026
**Status:** Aktywna
**Powiązane ADR:** [0001](./adr/0001-fastapi-railway-stack.md), [0002](./adr/0002-three-lane-routing.md), [0003](./adr/0003-prompt-caching-first.md), [0004](./adr/0004-public-mcp-server.md), [0005](./adr/0005-sovereign-tier-eip7702.md).

---

## Diagram

```
                ┌───────────────────────────────────────────────┐
                │  Klienci                                       │
                │  • Telegram bot (główny UX)                    │
                │  • proquant.app PWA (Vercel) — „dowód"         │
                │  • Claude Desktop / Cursor (przez MCP)         │
                └────────────────────┬──────────────────────────┘
                                     │ webhook / REST / MCP
                ┌────────────────────▼──────────────────────────┐
                │ FastAPI gateway (Railway)                     │
                │ • Stytch auth + Telegram Login                │
                │ • Stripe billing (web) / Stars (mikro ≤$5)    │
                │ • Model router + budget guard                 │
                │ • Eval shadow-run (10% ruchu → Promptfoo)     │
                └─────┬────────────────────┬────────────────────┘
                      │                    │
        ┌─────────────▼───┐   ┌────────────▼────────────────────┐
        │  Trzy lane'y    │   │  Claude Agent SDK orkiestrator  │
        │                 │   │  • główna pętla                  │
        │ Haiku 4.5       │   │  • subagenci: research/risk/exec │
        │ ($1/$5)         │   │  • hooks: PreToolUse =          │
        │ → parsing       │   │      budget+kompliance gate      │
        │                 │   │  • PostToolUse = decision_log    │
        │ Sonnet 4.6      │   │                                  │
        │ ($3/$15)        │   └────────┬────────────────────────┘
        │ → standard      │            │ MCP
        │                 │   ┌────────▼────────────────────────┐
        │ Opus 4.7        │   │ MCP servers (własne + publiczne)│
        │ ($5/$25)        │   │ • proquant-tools (public, OSS)  │
        │ → decyzje       │   │ • CoinGecko, Etherscan, DefiLlama│
        │   ≥$500 EUR     │   │ • Portfel (Supabase RLS)        │
        │                 │   │ • Decision log (append-only)    │
        └─────────────────┘   │ • EIP-7702 wallet adapter       │
                              │   (Sovereign tier)              │
                              └─────────────────────────────────┘

         Optymalizacja kosztu (3 warstwy):
         1. Anthropic prompt caching: system prompt 1h → 90% off na hit
         2. Redis LangCache: semantic cache nad zapytaniami o ceny
         3. Batch API dla nightly „decision review" (50% off, async OK)
```

---

## Wybory warstwowe

| Warstwa | Wybór | ADR | Komentarz |
|---|---|---|---|
| Backend | FastAPI (Python 3.12) | [0001](./adr/0001-fastapi-railway-stack.md) | Native async, Pydantic v2, dojrzały ekosystem AI. |
| Hosting | Railway | [0001](./adr/0001-fastapi-railway-stack.md) | $5/m baseline, autoscale, najprostsze dla solo-deva. |
| Frontend | Vercel + Next.js 15 PWA | — | Manifest i marketing. Główny UX w Telegramie. |
| Auth | Stytch + Telegram Login | — | OAuth + Telegram bez logiki samodzielnej. |
| Baza | Supabase Postgres + pgvector | — | RLS, pgvector dla decision embeddings, S3-compatible storage. |
| LLM orkiestracja | Claude Agent SDK | [0002](./adr/0002-three-lane-routing.md) | Hooks (PreToolUse/PostToolUse) = naturalne miejsce dla compliance gate. |
| Reasoning | Haiku 4.5 / Sonnet 4.6 / Opus 4.7 | [0002](./adr/0002-three-lane-routing.md) | Trzy lane'y. Opus tylko dla decyzji ≥$500 EUR ekwiwalentu. |
| Cache | Anthropic prompt caching + Redis LangCache | [0003](./adr/0003-prompt-caching-first.md) | 60–80% redukcja kosztu na typowej rozmowie. |
| Dystrybucja | Telegram bot + publiczny MCP | [0004](./adr/0004-public-mcp-server.md) | Sześć klientów AI za cenę jednego serwera. |
| On-chain (Sovereign) | EIP-7702 + x402 + session keys | [0005](./adr/0005-sovereign-tier-eip7702.md) | Agent działa bez klucza prywatnego użytkownika. |
| Płatności | Stripe (web, dominujący) + Stars (mikro ≤$5) + USDT/USDC (Sovereign) | — | Stars na mobile = ~32% prowizji. Tylko mikro. |
| Evals | Promptfoo (CI) + Braintrust (produkcja) | — | Eval-driven dev od dnia 1. |

---

## Model kosztu — przybliżenie

Założenia: 1 000 aktywnych użytkowników, średnio 5 decyzji/tydzień, 50% dotyka Sonnet, 10% Opus, 40% tylko Haiku.

| Pozycja | Koszt miesięczny | Notatka |
|---|---|---|
| Anthropic API (po cache) | ~$320 | 60% off na cache hit + 20% off na Batch API. |
| Railway (FastAPI + Redis) | ~$25 | Hobby + 1 service add-on. |
| Vercel (PWA) | $0 | Hobby tier. |
| Supabase (Pro) | $25 | Konieczny dla pgvector + RLS w produkcji. |
| Stripe prowizja | ~2.9% × przychód | Zależy od ARR. |
| Stytch | $0 | Free do 25 active users / month → potem $99. |
| **Razem (fixed)** | **~$370 + skala** | |

Próg rentowności przy €19/m Pro tier: **20 płacących użytkowników**.

---

## Powiązane

- Pełna strategia → [`_archive/Strategia_AI_GeNCorE_v2.md`](./_archive/Strategia_AI_GeNCorE_v2.md).
- Roadmap → [`roadmap.md`](./roadmap.md).
- Metryki sukcesu → [`metrics.md`](./metrics.md).
