# ✦ Strategia — Pole Decyzyjne

> *Strategia nie jest dokumentem. Jest stanem obserwowalnym pola — zbiorem decyzji, które kolapsują nieskończoność opcji w jeden akt budowy.*

Ten folder jest **inżynierskim śladem** strategii AI dla pierwszego produktu na bazie manifestu QuantuM® — *decision copilot dla solo-inwestora krypto*. Nie ma tu marketingu. Są **decyzje**, **playbooki**, **ślady kosztu i ryzyka**.

---

## Struktura

| Plik / folder | Zawartość |
|---|---|
| [`thesis.md`](./thesis.md) | Teza produktowa w trzech zdaniach. To, co odróżnia nas od konkurencji. |
| [`architecture.md`](./architecture.md) | Architektura referencyjna v2. Diagram, decyzje warstwowe, koszt. |
| [`adr/`](./adr/) | Architecture Decision Records — każda istotna decyzja techniczna z datą i powodem. |
| [`playbooks/`](./playbooks/) | Operacyjne instrukcje: MiCA, prompt caching, ewaluacja, on-call. |
| [`roadmap.md`](./roadmap.md) | Tydzień po tygodniu, do końca lipca 2026. |
| [`metrics.md`](./metrics.md) | Co mierzymy. Czego nie mierzymy. Definicja sukcesu i porażki. |

---

## Stan pola — 16 maja 2026

- **T-46 dni do MiCA** (1 lipca 2026). Każdy komunikat brzmiący jak personalizowana rekomendacja inwestycyjna jest po tej dacie ryzykowny w UE. → [`playbooks/mica.md`](./playbooks/mica.md).
- **Anthropic prompt caching** dojrzał — 90% off na cache hit. To główna dźwignia kosztu, nie router modelowy. → [`adr/0003-prompt-caching-first.md`](./adr/0003-prompt-caching-first.md).
- **Claude Opus 4.7 ujednolicony do $5/$25** za MTok. Trzy lane'y (Haiku 4.5 / Sonnet 4.6 / Opus 4.7) mają sens kosztowy. → [`adr/0002-three-lane-routing.md`](./adr/0002-three-lane-routing.md).
- **EIP-7702 + x402 + session keys** — agent działa on-chain bez klucza użytkownika. Sovereign tier z funkcji marketingowej staje się funkcją techniczną. → [`adr/0005-sovereign-tier-eip7702.md`](./adr/0005-sovereign-tier-eip7702.md).
- **MCP ekosystem** — 10 000+ publicznych serwerów, 97M+ pobrań SDK miesięcznie. Zero-CAC kanał dystrybucji. → [`adr/0004-public-mcp-server.md`](./adr/0004-public-mcp-server.md).

---

## Zasada redagowania

Każdy plik w tym folderze trzyma się czterech reguł manifestu QuantuM®:

1. **Superpozycja zanim koherencja.** ADR dokumentuje *alternatywy*, nie tylko wybór.
2. **Splątanie zanim izolacja.** Każda decyzja linkuje do innych decyzji i do kodu.
3. **Obserwacja zmienia system.** Każdy plik ma datę i autora — strategia jest *żywa*, nie *święta*.
4. **Koherencja zanim hałas.** Bez marketingu. Bez powtórzeń. Bez „synergii".

> *Strategia jest aktem obserwacji. Wybór jest aktem kolapsu. Kod jest dowodem.*
