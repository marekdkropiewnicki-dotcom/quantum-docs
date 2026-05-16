# ADR-0005: Sovereign tier — EIP-7702 + x402 + session keys

**Data:** 16 maja 2026
**Status:** Zaakceptowane (proof-of-concept w tygodniu 6, produkcyjnie po MiCA D-day)
**Autor:** GeNCorE

---

## Kontekst

W v1 strategii „Sovereign tier" był głównie konceptem marketingowym — własny klucz API, prywatność. To słabe pozycjonowanie, bo każdy konkurent może to skopiować w tydzień.

W maju 2026 dojrzały trzy standardy on-chain, które zmieniają definicję „sovereign":

1. **EIP-7702** — EOA może *tymczasowo* zachowywać się jak smart account z restrykcjami. Użytkownik nie traci kontroli klucza prywatnego.
2. **x402** — protokół mikropłatności HTTP 402. Agent może płacić za narzędzia (gas, RPC, oracle) bez user prompt na każdą transakcję.
3. **Session keys** (ERC-7715) — agent dostaje uprawnienia ograniczone czasowo i kwotowo („max €100 swap w przeciągu 24h").

Razem: **agent wykonuje plan on-chain, nie dotykając klucza prywatnego użytkownika, z hard limitami egzekwowanymi on-chain.**

---

## Rozważane opcje

| Opcja | Trust model | Compliance pod MiCA |
|---|---|---|
| **EIP-7702 + session keys + x402** | Użytkownik trzyma klucz, agent ma session-keyed uprawnienia z limitami | Defensible — agent jest *narzędziem*, użytkownik *autoryzuje* każdą sesję |
| Custodial wallet (my trzymamy klucz) | Wymaga licencji VASP/CASP | Toksyczne dla solo-foundera |
| MPC wallet (Fireblocks-style) | Współdzielony klucz, my wystawiamy infra | Komplikacje regulacyjne, vendor lock |
| Brak execution (tylko rekomendacja) | Brak | Najbezpieczniejsze, ale słabsza propozycja wartości |

---

## Decyzja

**Sovereign tier = EIP-7702 + ERC-7715 session keys + x402 mikropłatności.**

Model:

1. Użytkownik łączy wallet (Rainbow / Metamask / Rabby).
2. Wybiera „autoryzuj agenta na 24h, max €100, tylko Uniswap V4 + Aave V3".
3. Podpisuje EIP-7702 transakcję — wallet staje się tymczasowo smart account z restrykcjami.
4. Agent wykonuje plan w ramach limitów. Każda transakcja zostaje on-chain z metadata `proquant-execution`.
5. Po 24h restrykcje wygasają, wallet wraca do EOA.

Klucz prywatny użytkownika **nigdy nie opuszcza jego urządzenia**.

---

## Konsekwencje

**Pozytywne:**
- Najsilniejsza propozycja wartości w segmencie — żaden konkurent (CoinStats, Token Metrics) tego nie ma.
- **Compliance defensible**: my jesteśmy software vendorem, użytkownik podejmuje i autoryzuje każdą sesję ([Clifford Chance — reverse solicitation](https://www.cliffordchance.com/content/dam/cliffordchance/briefings/2025/01/the-reverse-solicitation-exemption-under-mica.pdf)).
- Pricing power — Sovereign tier €99/m vs. Pro €19/m jest uzasadniony funkcją, nie marketingiem.

**Negatywne:**
- EIP-7702 jest na mainnecie od marca 2026, jeszcze wczesne. Bugs możliwe. Mitygacja: testnet POC w tygodniu 6, mainnet dopiero po 8 tygodniach „bake time".
- Wymaga supportu wallet'ów — nie wszystkie wallety wspierają EIP-7702 w czerwcu 2026. Mitygacja: lista supported wallets, gracefull degradation do „recommendation only" mode.
- Audyt smart contractu session-key adapteru — koszt €5–15k. Mitygacja: użyj audytowanego open-source (np. Pimlico, Biconomy) zamiast pisać od zera.

**Następne kroki:**
- Tydzień 6: POC na testnet (Base Sepolia), jeden swap testowy.
- Tydzień 6: research audytowanych session-key library (Pimlico, ZeroDev, Biconomy).
- Po MiCA D-day (lipiec 2026): produkcyjne wdrożenie + audit zewnętrzny.

---

## Powiązane

- [playbooks/mica.md](../playbooks/mica.md) — dlaczego ten model jest defensible.
- [ADR-0004](./0004-public-mcp-server.md) — Sovereign tools wystawione przez MCP z auth.
