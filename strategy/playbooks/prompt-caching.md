# Playbook — Anthropic prompt caching (operacyjny)

**Data:** 16 maja 2026
**Status:** Aktywny — wdrożenie w tygodniu 2 roadmapy.

---

## Cel

60–80% redukcja $/decyzja na typowej rozmowie Claude Sonnet/Opus, przy <30 minutach implementacji.

---

## Jak to działa (skrótowo)

Anthropic prompt caching pozwala oznaczyć fragmenty promptu jako „cacheable". Pierwsze wywołanie pisze do cache (25% premium nad standardową ceną input). Każde kolejne wywołanie z tym samym prefixem płaci 10% standardowej ceny input za skeszowaną część (TTL 1h od ostatniego użycia).

**Cacheable elements:**
- System prompt
- Tool definitions (MCP)
- Long context (np. dokumentacja, FAQ, lista tokenów top-200)
- Few-shot examples

**Non-cacheable:**
- User input (oczywiście)
- Dynamic context (aktualne ceny, freshness <1h)

---

## Architektura cache w Proquant

```
┌──────────────────────────────────────────────────────────────────┐
│  System prompt (cacheable)                                       │
│  • Persona agenta                                                │
│  • Polityka MiCA (disclaimer template)                           │
│  • Klasyfikator wagi decyzji                                     │
│  • Format `decision_log` JSON schema                             │
│  ~ 6 000 tokenów, cache TTL 1h                                   │
├──────────────────────────────────────────────────────────────────┤
│  Tool definitions (cacheable)                                    │
│  • get_market_snapshot, evaluate_thesis, log_decision...         │
│  ~ 2 000 tokenów, cache TTL 1h                                   │
├──────────────────────────────────────────────────────────────────┤
│  Per-user context (cacheable per user, TTL 1h)                   │
│  • Portfel użytkownika (snapshot 5 min stale OK)                 │
│  • Historia ostatnich 10 decyzji                                 │
│  ~ 1 500 tokenów                                                 │
├──────────────────────────────────────────────────────────────────┤
│  Dynamic context (NIE cacheable)                                 │
│  • Aktualne ceny z CoinGecko                                     │
│  • User message                                                  │
│  ~ 200–500 tokenów                                               │
└──────────────────────────────────────────────────────────────────┘
```

**Rezultat:** ~9 500 tokenów cache'owanych vs. ~500 dynamic. Cache hit rate >80% przy aktywnym użytkowniku oznacza, że płacimy 90% mniej za 95% tokenów.

---

## Implementacja w `proquant-core` (Python)

```python
# app/llm/anthropic_client.py
from anthropic import Anthropic

client = Anthropic()

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=2048,
    system=[
        {
            "type": "text",
            "text": SYSTEM_PROMPT,
            "cache_control": {"type": "ephemeral"}  # ← cache marker
        }
    ],
    tools=[
        {**tool, "cache_control": {"type": "ephemeral"}}
        for tool in TOOLS
    ],
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": user_portfolio_summary,
                 "cache_control": {"type": "ephemeral"}},
                {"type": "text", "text": user_message}  # not cached
            ]
        }
    ]
)
```

---

## Monitoring

Loguj na każdym wywołaniu:

```python
log.info("anthropic.usage",
    input_tokens=resp.usage.input_tokens,
    cache_creation_input_tokens=resp.usage.cache_creation_input_tokens,
    cache_read_input_tokens=resp.usage.cache_read_input_tokens,
    output_tokens=resp.usage.output_tokens,
    model=resp.model,
    user_id=user_id,
    lane=lane
)
```

Metryki w Grafana / Railway dashboard:

- `anthropic.cache.hit_rate` (cache_read / (cache_read + cache_creation))
- `anthropic.cost_per_decision_eur`
- `anthropic.cache.miss_spike` — alert jeśli hit rate spada >20pp w 1h (oznacza zmianę promptu lub TTL expiry).

---

## Operacyjne zasady

1. **Każda zmiana system prompta** → bump wersji w `app/prompts/system.py` + deploy w godzinach low-traffic (3:00 UTC).
2. **Cache key porządek matters** — Anthropic cache'uje *prefix*. Kolejność: system → tools → user_context → user_message. Zmiana kolejności = cache invalidation.
3. **TTL 1h** — jeśli użytkownik nie pisał >1h, pierwsza wiadomość kolejnej sesji płaci cache write. To OK, ale liczymy w modelu kosztu.
4. **Nie cacheuj PII** — żadnych emaili, IP, telefon numerów w cache (one są w user_context per user, RLS-protected, ale lepiej nie ryzykować).

---

## Plan rollout

| Tydzień | Działanie |
|---|---|
| 2 | Cache system prompt + tools. Feature flag `prompt_cache_enabled`. |
| 3 | Cache per-user context. A/B test: 50% z cache, 50% bez, mierz $/decyzja. |
| 4 | Dodaj Redis LangCache jako warstwę #2 (semantic, dla zapytań cenowych). |
| 5 | Batch API dla nightly decision review (50% off, async, nie wpływa na user-facing UX). |

---

## Powiązane

- [ADR-0003](../adr/0003-prompt-caching-first.md) — uzasadnienie strategiczne.
- [Anthropic Prompt Caching Docs](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching)
- [`metrics.md`](../metrics.md) — `anthropic.cache.hit_rate` jako North Star operacyjna.
