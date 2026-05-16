# Teza produktowa

**Data:** 16 maja 2026
**Status:** Aktywna (v2)
**Poprzednie wersje:** v1 (11.05.2026) — w archiwum sesji.

---

## Jedno zdanie

> **„Decision copilot dla solo-inwestora krypto — agent w Telegramie, który pokazuje *tezę → plan → audit trail*, a w wariancie Sovereign wykonuje plan on-chain przez session-keyed wallet, nie dotykając twojego klucza prywatnego."**

---

## Trzy filary obrony

| Filar | Implementacja | Dlaczego konkurencja nie skopiuje przez 6 miesięcy |
|---|---|---|
| **Workflow Gravity** | Telegram bot + publiczny MCP `proquant-tools` (Claude Desktop / Cursor / Zed) | Konkurenci (CoinStats, Token Metrics) mają własne aplikacje — przeniesienie się na Telegram + MCP wymaga przebudowy całego stacku. |
| **Proprietary Data Loop** | `decision_log` (append-only) + `decision_outcomes_90d` + `counterfactual_replay` | Dane uczące, których nikt inny nie zbiera. Każda decyzja użytkownika + co rekomendował agent + co się wydarzyło 7/30/90 dni później. |
| **Mierzalne wyjście** | Zwrot ważony ryzykiem + **Sleep-well score** (subiektywny komfort 1–10) | Solo-inwestor płaci za *spokój*, nie za alfę. Konkurencja optymalizuje wyłącznie zwrot. |

---

## Co odpada

- **Nie budujemy** ogólnego asystenta krypto („chat o krypto" jest tani i przegrany).
- **Nie budujemy** narzędzia dla funduszy ani trading desków (CAC za wysoki, regulacja toksyczna).
- **Nie budujemy** automatycznego tradera bez human-in-the-loop (regulacyjnie martwe pod MiCA).

---

## ICP (Ideal Customer Profile)

- Solo-inwestor, $5k–$250k w krypto.
- Wiek 25–45, technicznie biegły, używa Telegrama codziennie.
- Już używa AI do decyzji finansowych ([48% globalnych konsumentów wg EY 2026](https://www.ey.com/en_gl/newsroom/2026/04/nearly-half-of-global-consumers-now-use-ai-to-guide-savings-and-investment-decisions)).
- Pain point: **brak pamięci między decyzjami** — co tydzień analizuje to samo od zera.

---

## Rynek

- **AI-crypto:** $5.1B (2025) → $55.2B (2035), CAGR 26.8% ([Market.us](https://market.us/report/crypto-making-ai-market/)).
- **TAM solo-inwestor krypto z AI:** ~30M osób globalnie (ekstrapolacja z EY × Chainalysis adoption).
- **Realistyczny SAM (rok 1):** 5 000 płacących użytkowników w UE + globalna anglojęzyczna diaspora krypto.

---

## Powiązane

- Strategia pełna v2 → [Strategia_AI_GeNCorE_v2 (archiwum sesji)](../../strategy/_archive/Strategia_AI_GeNCorE_v2.md) *(zostanie skopiowane przy commit)*.
- Architektura → [`architecture.md`](./architecture.md).
- Konkurencja → [`_archive/GeNCorE_Competitive_Teardown.md`](./_archive/GeNCorE_Competitive_Teardown.md).
