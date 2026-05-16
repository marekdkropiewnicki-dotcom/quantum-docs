# ADR-0002: Trzy lane'y modelowe (Haiku 4.5 / Sonnet 4.6 / Opus 4.7)

**Data:** 16 maja 2026
**Status:** Zaakceptowane
**Autor:** GeNCorE

---

## Kontekst

Anthropic ujednolicił ceny flagowych modeli (Opus 4.7: $5/$25 za MTok, ~3x taniej niż Opus 4.1). To zmienia rachunek ekonomiczny — Opus przestaje być „luksusem".

Mamy trzy klasy zadań w agencie:

1. **Parsing / formatowanie** (np. wyciągnij ticker z wiadomości, sformatuj decyzję do JSON) — wymaga taniego, szybkiego modelu.
2. **Standardowa decyzja** (np. „czy kupić ETH za €100") — wymaga rozumowania, ale stawka niska.
3. **Decyzja wysokiej wagi** (np. realokacja portfela ≥€500 ekwiwalentu, decyzja DeFi z lockiem) — wymaga najlepszego rozumowania.

Routing modelu według klasy zadania to oszczędność rzędu 5–10x vs. „Opus dla wszystkiego".

---

## Rozważane opcje

| Opcja | Plusy | Minusy |
|---|---|---|
| **3 lane'y (Haiku / Sonnet / Opus)** | Najlepszy stosunek kosztu do jakości na zadanie | Wymaga klasyfikatora (sam Haiku) + budget guard |
| 1 model (Sonnet 4.6 dla wszystkiego) | Prostota, brak klasyfikatora | 3x droższy parsing + 1.5x słabsze decyzje wysokiej wagi |
| 2 lane'y (Sonnet / Opus) | Średnia prostota | Marnotrawstwo na parsing |
| Multi-vendor (Claude + Groq + Gemini) | Hedge na vendor | 3x złożoność evals, 3x różne quirks w tool use |

---

## Decyzja

**Trzy lane'y Anthropic**, klasyfikacja przez Haiku 4.5.

Routing:

| Zadanie | Lane | Cena ($/MTok in/out) |
|---|---|---|
| Klasyfikator intencji, parsing, formatowanie | **Haiku 4.5** | $1 / $5 |
| Standardowa decyzja (<€500 ekwiwalent) | **Sonnet 4.6** | $3 / $15 |
| Decyzja wysokiej wagi (≥€500), counterfactual replay, miesięczne review | **Opus 4.7** | $5 / $25 |

Klasyfikator (Haiku) wstrzykuje `lane` metadata do hooka PreToolUse → router wybiera model. Default fallback: Sonnet.

---

## Konsekwencje

**Pozytywne:**
- Średni koszt decyzji ~€0.04 vs ~€0.18 dla Opus-only. Próg rentowności €19/m osiągany przy 20 użytkownikach zamiast 90.
- Eval-driven: każdy lane ma własny zestaw testów w Promptfoo, regresję widać per lane.

**Negatywne:**
- Klasyfikator Haiku może się mylić — risk: niedoszacowanie wagi decyzji.
- Mitygacja: **explicit user override** („zmuszę Opus") + threshold reguły (każda transakcja oznaczona kwotowo idzie do Opus, bez klasyfikatora).

**Następne kroki:**
- Tydzień 3: implementacja routera w hook PreToolUse Claude Agent SDK.
- Tydzień 3: 50 testów klasyfikatora w Promptfoo (golden set).
- Tydzień 3: budget guard z hard limitem $5/dzień/user.

---

## Powiązane

- [ADR-0003](./0003-prompt-caching-first.md) — caching osłabia argument „Opus jest drogi".
- [`metrics.md`](../metrics.md) — % decyzji per lane jako metryka operacyjna.
