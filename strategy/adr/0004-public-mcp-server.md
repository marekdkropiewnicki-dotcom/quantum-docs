# ADR-0004: Publiczny MCP server `proquant-tools` jako zero-CAC kanał dystrybucji

**Data:** 16 maja 2026
**Status:** Zaakceptowane
**Autor:** GeNCorE

---

## Kontekst

MCP (Model Context Protocol) ekosystem dojrzał w Q1/Q2 2026:

- 10 000+ publicznych serwerów MCP ([Taskade — MCP Servers](https://www.taskade.com/blog/mcp-servers)).
- 97M+ pobrań SDK miesięcznie.
- Linux Foundation governance — protokół jest neutralny.
- Klienci: Claude Desktop, Cursor, Zed, Continue, Windsurf, Goose. Sześć platform, jeden serwer.

Solo-deweloperzy buildujący na MCP zyskują **organic distribution** — użytkownicy Claude Desktop sami szukają nowych narzędzi.

---

## Rozważane opcje

| Opcja | Zasięg | Koszt utrzymania |
|---|---|---|
| **Publiczny OSS MCP server + komercyjny tier z autoryzacją** | 6 klientów AI + brand awareness | Średni — wymaga semver + dokumentacji |
| Prywatny MCP tylko dla naszego bota | Tylko nasi użytkownicy | Niski |
| Brak MCP, tylko REST API | Wymaga, żeby ktoś napisał integrację | Najniższy |
| Tylko integracja z Claude Desktop (bez MCP) | 1 klient | Średni |

---

## Decyzja

**Publiczny MCP server `proquant-tools`** (MIT license, repo `proquant-tools` na GitHubie).

Strategia warstwowa:

| Warstwa | Tools | Auth |
|---|---|---|
| **Free public** | `get_market_snapshot`, `evaluate_thesis_lite` (Haiku-only) | Brak auth, rate-limit per IP |
| **Pro (API key)** | `evaluate_thesis_full` (Sonnet/Opus), `log_decision`, `replay_decision` | API key z proquant.app |
| **Sovereign (API key + wallet)** | `execute_plan_onchain` (EIP-7702) | API key + wallet signature |

Free tier = lejek do Pro. Każde zapytanie do Free dostaje subtelne „upgrade to log this decision" w odpowiedzi.

---

## Konsekwencje

**Pozytywne:**
- **Zero-CAC** dla pierwszych 100 użytkowników — wystarczy submit do Anthropic MCP Registry + 1 dobry post na X.
- MCP-first oznacza, że produkt jest **portable** — jeśli Telegram zaczyna nas blokować, użytkownicy mają już alternatywne klienty.
- **Branding** — `proquant-tools` jako referencyjna implementacja dla MCP w finansach.

**Negatywne:**
- Free tier kosztuje (Haiku $1/$5 + infra). Mitygacja: hard rate limit 10 zapytań/IP/dzień, fallback do statycznego snapshotu po przekroczeniu.
- Konkurencja może forkować. Mitygacja: wartość siedzi w `decision_log` + outcomes (private), nie w narzędziach MCP.
- Audit logs publicznych zapytań — musimy nie logować PII (IP hashowane, brak query content w produkcji).

**Następne kroki:**
- Tydzień 4: repo `proquant-tools` (OSS, MIT), submit do MCP Registry.
- Tydzień 4: dokumentacja w `quantum-docs/integrations/mcp.md`.
- Tydzień 4: pierwsza wersja `proquant-tools.app` (subdomena `proquant.app`) jako landing page MCP.

---

## Powiązane

- [`architecture.md`](../architecture.md) — MCP w diagramie warstwy klientów.
- [ADR-0005](./0005-sovereign-tier-eip7702.md) — Sovereign tier wymaga MCP + wallet.
