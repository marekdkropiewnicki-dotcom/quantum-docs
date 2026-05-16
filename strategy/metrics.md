# Metryki — co mierzymy, czego nie mierzymy

**Data:** 16 maja 2026
**Reguła:** trzy metryki North Star. Wszystko inne to instrumentacja.

---

## North Star

| Metryka | Cel rok 1 | Powód |
|---|---|---|
| **Weekly Active Deciders (WAD)** | 1 000 | Decyzja > sesja. Mierzymy ludzi, którzy *podjęli* decyzję z naszej pomocy, nie ludzi, którzy „weszli i wyszli". |
| **Decision-to-Outcome Retention 30d** | ≥40% | Czy użytkownik wraca *po* tym, jak zobaczył wynik swojej decyzji 30 dni później? To weryfikuje, czy nasz audit trail ma wartość. |
| **Sleep-well delta** | +2.0 pkt (1–10) | Średnia różnica Sleep-well score między decyzjami z naszym agentem vs. decyzjami solo (self-reported baseline). |

---

## Instrumentacja

| Warstwa | Metryki operacyjne |
|---|---|
| Koszt | $/decyzja, cache hit rate, % decyzji per lane (Haiku/Sonnet/Opus). |
| Niezawodność | p95 latency, error rate, eval pass rate (Promptfoo CI), production eval drift (Braintrust). |
| Compliance | % decyzji z reverse-solicitation banner, % consent recorded, audit log completeness. |
| Wzrost | CAC (jeśli płatne kanały), MCP installs, organic Telegram referrals. |

---

## Czego nie mierzymy (świadomie)

- **DAU/MAU** — to metryka uzależnienia, nie wartości. Nie chcemy budować scroll-trapu.
- **Czas spędzony w bocie** — im krócej, tym lepiej. Decyzja w 60 sekundach > decyzja w 60 minutach.
- **Liczba wiadomości** — koreluje z kosztem, nie z wartością.
- **NPS** — zbyt szumna metryka dla N<500. Zastąpiona Sleep-well delta.

---

## Definicja porażki

Wycofujemy projekt jeśli po tygodniu 13 (koniec sierpnia 2026):

- WAD < 100, **lub**
- Sleep-well delta < 0 (agent pogarsza komfort decyzji), **lub**
- $/decyzja > €0.50 (model ekonomicznie nie domyka się przy €19/m).

> *Strategia, która nie ma definicji porażki, jest religią.*
