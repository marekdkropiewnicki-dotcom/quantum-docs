# ADR-0003: Prompt caching jako główna dźwignia kosztu, nie router

**Data:** 16 maja 2026
**Status:** Zaakceptowane
**Autor:** GeNCorE

---

## Kontekst

W v1 strategii zakładaliśmy, że główną dźwignią kosztu będzie router modelowy + semantic cache w Redis. W ciągu ostatniego tygodnia dwa fakty zmieniły kalkulację:

1. **Anthropic prompt caching** dojrzał do produkcyjnego SLA: 1h TTL, 90% off na cache hit, 25% premium na cache write. Dla typowego agenta z systemowym promptem 5–10k tokenów to dramatyczna oszczędność.
2. **System prompt agenta krypto** jest *stabilny* między rozmowami — narzędzia MCP, polityka MiCA, format `decision_log`, klasyfikator wagi. To idealny target dla cache.

---

## Rozważane opcje

| Opcja | Redukcja kosztu | Złożoność |
|---|---|---|
| **Prompt caching jako warstwa #1** + semantic cache jako #2 | 60–80% | Niska — flag w Anthropic SDK |
| Tylko Redis semantic cache | 30–40% | Średnia — wymaga embeddings + similarity threshold |
| Tylko router (taniej dla tanich zadań) | 50–60% | Średnia — klasyfikator + ewaluacje per lane |
| Wszystko razem (prompt cache + semantic + router) | **70–85%** | Wyższa, ale każda warstwa działa niezależnie |

---

## Decyzja

**Trzy warstwy optymalizacji w kolejności wdrożenia:**

1. **Anthropic prompt caching** (tydzień 2) — system prompt + tools w cache, TTL 1h.
2. **Redis LangCache (semantic)** (tydzień 4) — dla powtarzalnych zapytań o ceny i fundamenty.
3. **Batch API** (tydzień 5) — dla nightly „decision review" jobs, 50% off, async OK.

Router modelowy ([ADR-0002](./0002-three-lane-routing.md)) **uzupełnia** te warstwy, nie zastępuje ich.

---

## Konsekwencje

**Pozytywne:**
- Główna dźwignia ($/decyzja) zaadresowana w tygodniu 2, nie 4–5.
- Cache hit rate jest mierzalną metryką operacyjną — łatwo zauważyć regresję.
- Stabilny system prompt = mniej tokenów w każdej rozmowie = niższa latencja p50.

**Negatywne:**
- 25% premium na cache write — pierwsza rozmowa dnia jest droższa.
- Cache invalidation przy każdej zmianie prompta = krótki spike kosztu po deployu.
- Mitygacja: deploye nowych promptów w godzinach low-traffic, monitoring cache hit rate w Grafana.

**Następne kroki:**
- Tydzień 2: feature flag `prompt_cache_enabled=true` w `app/config.py`.
- Tydzień 2: structured logging cache hit/miss → metryka `anthropic.cache.hit_rate`.
- Tydzień 4: Redis LangCache dla zapytań cenowych (TTL 60s dla cen, 24h dla fundamentów).

---

## Powiązane

- [playbooks/prompt-caching.md](../playbooks/prompt-caching.md) — operacyjna instrukcja wdrożenia.
- [ADR-0002](./0002-three-lane-routing.md) — router uzupełnia caching.
