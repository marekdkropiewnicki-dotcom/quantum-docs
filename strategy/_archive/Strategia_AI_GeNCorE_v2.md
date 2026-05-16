# Strategia AI dla GeNCorE / QuantuM — wersja 2 (maj 2026)

> **Ewolucja, nie reset.** Wersja 1 (11 maja 2026) ustaliła tezę: *decision copilot dla solo-inwestora krypto*. Wersja 2 zachowuje tę tezę i dodaje pięć rzeczy, które w ciągu ostatniego tygodnia okazały się krytyczne i nie były w v1: (1) realny koszt Telegram Stars na mobile, (2) MiCA z deadline 1 lipca 2026, (3) prompt caching Anthropic jako główna dźwignia kosztowa zamiast routera, (4) Claude Opus 4.7 jako nowy szczyt rozumowania, (5) EIP-7702 + x402 jako naturalne rozszerzenie marki „sovereign AI" w kierunku agenta on-chain.

**Zmiana stanowiska wobec v1**: Telegram nie jest już głównym kanałem płatności — jest głównym kanałem dystrybucji i UX. Płatności płyną przez Stripe (web) + USDT/USDC (sovereign tier), Stars zostaje tylko dla mikrotransakcji ≤$5.

---

## 0. TL;DR — co się zmienia względem v1

| Obszar | v1 (11.05) | v2 (12.05) — zmiana | Powód |
|---|---|---|---|
| Płatności Telegram | Stripe + Stars jako backup | Stripe (web) **dominujący**, Stars tylko mikro (≤$5), USDT/USDC dla Sovereign | Stars na mobile = **~32% prowizji** (Apple/Google 30% baked-in) ([Grambase](https://grambase.ai/blog/telegram-stars-guide-2026)) |
| Reasoning lane | Sonnet 4.5 | **Opus 4.7 dla decyzji ≥$500 ekwiwalentu, Sonnet 4.6 dla reszty, Haiku 4.5 dla parsingu** | Opus 4.7 = $5/$25 za MTok, **dwukrotnie taniej niż Opus 4.1** ([Anthropic Pricing](https://platform.claude.com/docs/en/about-claude/pricing), [Evolink](https://evolink.ai/blog/claude-api-pricing-guide-2026)) |
| Optymalizacja kosztów | Router + Redis semantic cache | **Prompt caching Anthropic (90% off cache hits) + semantic cache na wierzchu = 60–80% redukcji** | Stack daje 60–80% redukcji ([buildmvpfast](https://www.buildmvpfast.com/blog/semantic-caching-ai-agents-cost-optimization)) |
| Regulacja | Disclaimer + „research assistant" | **Pełny reverse-solicitation playbook + capital-light „non-CASP" pozycjonowanie pod 1.07.2026** | Deadline MiCA 1 lipca 2026 ([Sumsub](https://sumsub.com/blog/crypto-regulations-in-the-european-union-markets-in-crypto-assets-mica/)) |
| Sovereign tier | Własny klucz API | **+ EIP-7702 session keys + x402 micropayments dla agenta on-chain** | Nowy standard agentów krypto ([Coincub](https://coincub.com/blog/crypto-ai-agents/)) |
| Evals | Brak w v1 | **Promptfoo w CI + Braintrust dla produkcji od dnia 1** | Eval-driven dev to standard 2026 ([Augmentcode](https://www.augmentcode.com/tools/best-ai-agent-evaluation-tools), [Braintrust](https://www.braintrust.dev/articles/best-ai-evaluation-tools-2026)) |

---

## 1. Aktualizacja diagnozy

### Co działa od v1
- Teza „decyzja, nie tracking" trzyma się — rynek AI-crypto rośnie z $5.1B (2025) do $55.2B (2035), CAGR 26.8% ([Market.us](https://market.us/report/crypto-making-ai-market/)).
- EY raportuje, że **48% globalnych konsumentów używa AI do decyzji oszczędnościowo-inwestycyjnych** ([EY 2026](https://www.ey.com/en_gl/newsroom/2026/04/nearly-half-of-global-consumers-now-use-ai-to-guide-savings-and-investment-decisions)). Nasz ICP nie musi być edukowany do problemu — już go ma.
- Konkurencja (CoinStats, Token Metrics) nadal nie ma **decision replay z mierzalnym edge** — okno otwarte.

### Co się zmieniło w tle (5 dni)
1. **Telegram Stars audyt kosztu**: na iOS/Android tracisz ~32% przez Apple/Google IAP, na desktopie tylko ~3%. Stars to **kanał konwersji**, nie *kanał przychodu*.
2. **Claude Opus 4.7 / 4.6 / 4.5 ujednolicone do $5/$25** za MTok — Anthropic obniżył wejście do flagowych modeli o ~3x. Argument „Sonnet bo tanio" słabnie.
3. **MCP ekosystem dojrzał**: 10,000+ publicznych serwerów, 97M+ pobrań SDK miesięcznie, Linux Foundation governance ([Taskade](https://www.taskade.com/blog/mcp-servers)). To realny kanał dystrybucji *zero-CAC*.
4. **EIP-7702 + x402 + session keys** — agent może działać on-chain bez trzymania klucza prywatnego użytkownika ([Coincub](https://coincub.com/blog/crypto-ai-agents/), [Cobo](https://www.cobo.com/post/ai-wallet-guide-intelligent-crypto-wallets)). To zmienia „sovereign" z marketingu w funkcję.
5. **MiCA T-49 dni**: w UE każda komunikacja brzmiąca jak personalizowana rekomendacja inwestycyjna staje się 1.07 ryzykowna ([Sumsub](https://sumsub.com/blog/crypto-regulations-in-the-european-union-markets-in-crypto-assets-mica/), [Clifford Chance — reverse solicitation](https://www.cliffordchance.com/content/dam/cliffordchance/briefings/2025/01/the-reverse-solicitation-exemption-under-mica.pdf)).

---

## 2. Teza strategiczna (bez zmian, doprecyzowana)

> **„Decision copilot dla solo-inwestora krypto — agent w Telegramie, który pokazuje *tezę → plan → audit trail*, a w wariancie Sovereign wykonuje plan on-chain przez session-keyed wallet, nie dotykając twojego klucza."**

Trzy filary obrony pozostają, ale każdy dostaje konkretną implementację v2:

| Filar | v1 | v2 — konkretyzacja |
|---|---|---|
| **Workflow Gravity** | „bot w Telegramie" | + publiczny MCP `proquant-tools` dla każdego, kto używa Claude Desktop / Cursor / Zed — 6 klientów AI za cenę jednego serwera ([Taskade](https://www.taskade.com/blog/mcp-servers)) |
| **Proprietary Data Loop** | decision log + outcome 7/30d | + **decision_outcomes_90d** + **counterfactual_replay** (co by się stało, gdybyś *nie* zrobił, plus co rekomendował agent vs. co zrobiłeś) — to kompromituje konkurencję |
| **Mierzalne wyjście** | „edge vs hold" | + **„Sleep-well score"** (subiektywny komfort 1–10 zapisywany przy decyzji) + zwrot ważony ryzykiem — bo solo-inwestorzy płacą za spokój, nie tylko za alfę |

---

## 3. Architektura referencyjna v2

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
                │ • Stripe billing (web) / Stars (mikro)        │
                │ • Model router + budget guard                 │
                │ • Eval shadow-run (10% ruchu → Promptfoo)     │
                └─────┬────────────────────┬────────────────────┘
                      │                    │
        ┌─────────────▼───┐   ┌────────────▼────────────────────┐
        │  Trzy lane'y    │   │  Claude Agent SDK orkiestrator  │
        │                 │   │  • główna pętla                  │
        │ Haiku 4.5       │   │  • subagenci: research/risk/exec │
        │ ($1/$5)         │   │  • hooks: PreToolUse =          │
        │ → parsing,      │   │      budget+kompliance gate      │
        │   formatowanie  │   │  • PostToolUse = decision_log    │
        │                 │   │                                  │
        │ Sonnet 4.6      │   └────────┬────────────────────────┘
        │ ($3/$15)        │            │ MCP
        │ → standard      │   ┌────────▼────────────────────────┐
        │   decyzje       │   │ MCP servers (własne + publiczne)│
        │                 │   │ • proquant-tools (public, OSS)  │
        │ Opus 4.7        │   │ • CoinGecko, Etherscan, DefiLlama│
        │ ($5/$25)        │   │ • Portfel (Supabase RLS)        │
        │ → decyzje       │   │ • Decision log (append-only)    │
        │   ≥$500 EUR     │   │ • EIP-7702 wallet adapter       │
        │                 │   │   (Sovereign tier)              │
        └─────────────────┘   └─────────────────────────────────┘

         Optymalizacja kosztu (3 warstwy):
         1. Anthropic prompt caching: system prompt 1h cache → 90% off na hit
         2. Redis LangCache: semantic cache nad zapytaniami o ceny/fundament.
         3. Batch API dla nightly „decision review" (50% off, async OK)
```

### Decyzje techniczne — uaktualnione

| Warstwa | Wybór v2 | Komentarz |
|---|---|---|
| Backend | **FastAPI** (potwierdzone z v1) | Bez zmian. |
| Agent framework | **Claude Agent SDK** | Subagent pattern dla research/risk/exec lanes, hooks dla budget + compliance gate ([LetsDataScience](https://letsdatascience.com/blog/claude-agent-sdk-tutorial)). |
| Routing modelowy | **Haiku 4.5 / Sonnet 4.6 / Opus 4.7** | Trzy poziomy zamiast dwóch — Haiku przejmuje parsing/formatowanie. |
| Caching | **Anthropic prompt cache (1h TTL) + Redis LangCache** | Stack ~60–80% off ([buildmvpfast](https://www.buildmvpfast.com/blog/semantic-caching-ai-agents-cost-optimization), [Pecollective](https://pecollective.com/tools/anthropic-api-pricing/)). |
| Batch | **Anthropic Batch API dla nightly review** | 50% off na batchowane outcome_30d updates. |
| Evals | **Promptfoo w GitHub Actions + Braintrust w produkcji** | Eval-driven od dnia 1, nie po pierwszym incydencie ([Augmentcode](https://www.augmentcode.com/tools/best-ai-agent-evaluation-tools)). |
| Płatności | **Stripe (web, główne) + USDT/USDC (Sovereign) + Stars (mikro ≤$5)** | Ekonomika fees: Stripe ~3.2%, USDT ~2.5%, Stars mobile ~32% ([Grambase](https://grambase.ai/blog/telegram-stars-guide-2026)). |
| Wallet (Sovereign) | **Privy / Cobo Agentic Wallet z EIP-7702 session keys** | Agent dostaje tymczasową, scoped autoryzację bez klucza prywatnego ([Coincub](https://coincub.com/blog/crypto-ai-agents/), [Cobo](https://www.cobo.com/post/ai-wallet-guide-intelligent-crypto-wallets)). |
| Dane | Supabase (Postgres + pgvector + RLS) | Bez zmian; RLS krytyczne przy multi-tenant. |
| Hosting | Railway (API) + Vercel (PWA) | Bez zmian. |

---

## 4. Pozycjonowanie — proquant v2 (post-MiCA-safe)

| | Token Metrics / CoinStats | Telegram trading bots (Maestro, BullX, BananaGun) | **Proquant v2** |
|---|---|---|---|
| Główna obietnica | „Zobacz wszystko" | „Wykonaj szybciej" | **„Zdecyduj świadomie i udokumentuj dlaczego"** |
| Forma | Dashboard | Bot do egzekucji | Decision copilot + opcjonalna egzekucja przez session key |
| Output | Wykresy, alerty | Trade execution | **Teza → plan → audit trail + replay** |
| Personalizacja | Generyczny AI | Brak | Decision log uczy się stylu (risk tolerance, horizon, „sleep score") |
| Regulacja | „Not financial advice" w stopce | Nieuregulowane, szare | **Reverse-solicitation by design**: user inicjuje, my odpowiadamy z disclaimer, audit trail dowodzi |
| Cena | $10–25/mc | % flat fee od trade'u (~0.5–1%) | Free 5 decyzji / Pro $12 / Sovereign $39 z własnym kluczem + on-chain executor |

**ICP zostaje** (solo-trader $5k–$250k portfela, Telegram-native), ale dochodzą dwie podgrupy:

- **„Sovereignty-minded"** — power-userzy, którzy chcą własnego klucza API + on-chain executora bez custody. Płacą $39, ale **mają 0% koszt LLM dla nas** (BYO-API) i są naszym evangelist core.
- **„Compliance-conscious EU"** — użytkownicy, którym MiCA zaczyna ciążyć i szukają narzędzia, które jawnie *nie udaje doradcy*. Disclaimer + decision log to ich „prawniczy parasol".

**Wedge feature v2 (3 zamiast 1)**:
1. **Decision Replay** (z v1) — mediana zwrotu agent-recommended vs. hold vs. user-action.
2. **Counterfactual** — co by się stało, gdybyś nie zrobił tej decyzji (rzadkie u konkurencji).
3. **Sleep-well score correlation** — pokazuje użytkownikowi, że jego subiektywny komfort koreluje (lub nie) z aktualnym zwrotem. Twardy psychometryczny hook do retencji.

---

## 5. Roadmap 90 dni — wersja 2

Roadmap v1 trzyma się **kierunkowo**, ale każdy z trzech sprintów dostaje twardszą definicję „done" i nowe pozycje wymuszone przez MCP, evale i MiCA.

### Sprint 0–30 — Konsolidacja, evale, MiCA-ready

| # | Zadanie | Zmiana vs v1 |
|---|---|---|
| 1 | FastAPI + Claude Agent SDK skeleton, port GeNCorE handlera | bez zmian |
| 2 | **Trójpoziomowy router** Haiku/Sonnet/Opus z klasyfikatorem intencji (Haiku) | +1 lane vs v1 |
| 3 | **Anthropic prompt caching** dla system promptu + few-shot (1h TTL) | **NOWE** — pierwsza warstwa redukcji kosztu |
| 4 | Schema Supabase: `users`, `portfolios`, `decisions`, `decision_outcomes_7d/30d/90d`, `counterfactuals`, `feedback`, `sleep_scores` | rozszerzone vs v1 |
| 5 | MCP servers: CoinGecko, Etherscan, DefiLlama (jako *lokalne* stdio servers) | bez zmian |
| 6 | Telegram komendy: `/portfolio`, `/should_i`, `/replay`, `/why_not` (counterfactual) | +1 komenda |
| 7 | **Promptfoo eval suite** w GitHub Actions: 30 golden cases, regression gate na merge | **NOWE** — eval-driven od dnia 1 |
| 8 | **MiCA compliance pack**: disclaimer template, ToS „research assistant", audit-trail export per user na żądanie | **NOWE** — krytyczne przed 1.07 |
| **Kamień milowy** | 10 zaufanych użytkowników, 100+ decyzji w logu, 0 regresji w eval suite, ToS gotowe |  |

### Sprint 31–60 — Monetyzacja, dystrybucja przez MCP, semantic cache

| # | Zadanie | Zmiana vs v1 |
|---|---|---|
| 9 | **Stripe (web checkout) + Telegram Stars tylko dla mikro-akcji ≤$5** (np. „1 extra decision today") | **przebudowane** — Stars nie jest głównym kanałem |
| 10 | Web dashboard `proquant.app` jako proof: decision history, replay charts, counterfactual viz | bez zmian |
| 11 | Feedback w bocie: 👍/👎 + edytowalna teza + **sleep score 1–10** zapisywany przy decyzji | +sleep score |
| 12 | **Publiczny MCP server `proquant-tools` (read-only)** na GitHub + listing w MCP registry | **NOWE** — kanał dystrybucji zero-CAC |
| 13 | Redis LangCache na zapytaniach o ceny + budżety per-user (hook Claude Agent SDK `PreToolUse`) | +konkret na semantic cache |
| 14 | **Braintrust integracja**: live tracing wszystkich decyzji prod, alert na p95 regression | **NOWE** |
| 15 | Pierwsze 3 publiczne „Decision Replay" jako anonimizowane case studies (X/Twitter, Reddit r/CryptoCurrency) | bez zmian |
| **Kamień milowy** | 5 płacących, MRR ≥$60, CAC <$5, semantic cache hit ratio ≥40%, MCP server zainstalowany przez ≥20 osób |  |

### Sprint 61–90 — Sovereign tier, on-chain, retencja

| # | Zadanie | Zmiana vs v1 |
|---|---|---|
| 16 | **Sovereign tier**: BYO klucz Anthropic/Groq + **EIP-7702 session-keyed executor** (Privy lub Cobo) | **rozszerzone** o on-chain |
| 17 | Tygodniowy newsletter „Decision Replay Public" + tematyczne case-studies | bez zmian |
| 18 | Jedna integracja partnerska: **DeFiLlama Pro API** lub **Hyperliquid alerts** | bez zmian |
| 19 | Anthropic **Batch API** dla nightly `outcome_30d` updates wszystkich aktywnych decyzji (50% off) | **NOWE** |
| 20 | „Decision Quality Score" — composite KPI łączący edge + sleep + recency, eksponowane w bocie | **NOWE** |
| 21 | Audit-trail one-click export (PDF/CSV) — feature dla MiCA-uważnych | **NOWE** |
| **Kamień milowy** | 30 płacących, MRR ≥$500, retencja 30d ≥60%, ≥3 użytkowników Sovereign |  |

---

## 6. KPI i pulpit — uaktualnione

**Twarde (CFO-grade)**
- MRR, ARPU, gross margin (cel ≥70% w fazie 3)
- CAC i payback w tygodniach (cel <8 tyg.)
- Retencja D7/D30/D90
- **Decision Replay Edge** — mediana(zwrot agenta − zwrot hold) na akceptowanych
- **Counterfactual Edge** — mediana(zwrot user-action − zwrot „nie zrobił nic")
- Koszt LLM / aktywny user / miesiąc — cel <$0.25 w Pro (vs $0.40 w v1, dzięki cache stackowi)
- **Cache hit ratio** (prompt cache + LangCache łącznie) — cel ≥55%

**Miękkie**
- % decyzji z feedbackiem (cel >40%)
- **Sleep-well score** — średnia + kierunek korelacji z 30d-outcome
- Średnia długość rozmowy przed decyzją (proxy zaufania)
- NPS po 30 dniach

**Twardy eval-gate (nowe)**
- Promptfoo: 100% golden cases passing przed merge
- Braintrust: brak regresji p95 latencji >2x baseline, brak spadku „decision quality eval" o ≥10% w ciągu 7 dni

Pulpit: Supabase Materialized View → Google Sheets via Pipedream connector → jeden widok w Notion (dla siebie). Bez BI.

---

## 7. Ekonomia jednostkowa — przeliczona

### Założenia
- Średnio 8 decyzji/mc na aktywnego usera Pro
- Średnia decyzja: 4k tokenów input (portfolio + history) + 1.5k output
- Stack: 60% Sonnet 4.6, 30% Haiku 4.5, 10% Opus 4.7
- Cache hit ratio: 55% (prompt cache 90% off na input) — konserwatywnie

### Koszt LLM / user Pro / miesiąc (szacunek)

| Składnik | Bez cache | Z prompt cache 55% hit | Z LangCache na top |
|---|---|---|---|
| Input (32k/mc po routing) | ~$0.10 | ~$0.055 | ~$0.038 |
| Output (12k/mc po routing) | ~$0.18 | ~$0.18 | ~$0.18 |
| **Razem** | **~$0.28** | **~$0.235** | **~$0.22** |

Pricing $12/mc Pro → gross margin ~**98%** na samym LLM. Reszta marży idzie na Railway/Supabase/Stripe (~$0.40/user/mc). Realna marża brutto **~94%** — i to **w scenariuszu konserwatywnym**.

Sovereign $39/mc: zero kosztu LLM (BYO key), gross margin ~97% (Stripe + infra).

### Punkt rentowności
- Hosting fix-cost (Railway + Supabase + Vercel + Redis na fly.io): ~$50/mc.
- Break-even: **5 płatników Pro** = MRR $60.
- Cel sprint 90: 30 płatników = MRR $500 → **operacyjnie zysk od dnia 30**.

---

## 8. Ryzyko regulacyjne — playbook MiCA

> Najważniejsza zmiana w v2. Termin 1 lipca 2026 jest blisko i decyduje, czy w UE jesteś produktem, czy ryzykiem.

**Pozycjonowanie prawne**: Proquant **nie jest** dostawcą usług w zakresie kryptoaktywów w rozumieniu MiCA art. 3(1)(15). Jest **narzędziem informacyjnym i logiem decyzji**. Nie publikujemy white paperów, nie operujemy platformy obrotu, nie wykonujemy zleceń klienta.

**Egzekwowalne reguły (w kodzie, nie w ToS)**:
1. **Tryb non-advisory by default** — agent nie używa form „kupuj/sprzedaj X" w komunikacji EU. Używa: „w twoim profilu ryzyka i twojej tezie, twoja decyzja X miałaby implikacje Y i Z". Każda decyzja podpisana przez użytkownika klikiem („I confirm this is my decision").
2. **Audit trail jako tarcza** — pełny zapis: kto pytał, co dostał, co kliknął, kiedy. To dowód, że *to user decyduje*.
3. **Reverse-solicitation discipline** — żadnego pushu „rekomendacji dnia" do użytkowników EU bez explicit opt-in. ([Clifford Chance — reverse solicitation pod MiCA](https://www.cliffordchance.com/content/dam/cliffordchance/briefings/2025/01/the-reverse-solicitation-exemption-under-mica.pdf))
4. **Disclaimer per-message** w Telegramie EU usera, raz dziennie min. — krótki, nie irytujący.
5. **Geofencing soft** — przy rejestracji pytanie o kraj rezydencji; EU userzy widzą inny system prompt (ostrożniejszy język).
6. **Brak custody, brak execution-as-service do MiCA-license** — jeśli Sovereign tier ma robić on-chain execution, użytkownik trzyma session key na własnym wallecie (EIP-7702), my tylko orkiestrujemy. To pozostawia nas poza definicją CASP.

**Co eksplicite odpuszczamy w UE do czasu jasności prawnej**: copy-trading, sygnały push, custody jakichkolwiek aktywów, fee-from-trade. Wszystko to wciągnęłoby nas w MiCA.

---

## 9. Strategia dystrybucji — zero CAC, trzy kanały

1. **MCP registry** — publiczny serwer `proquant-tools` (read-only, OSS Apache 2.0). Każdy user Claude Desktop / Cursor / Zed instaluje jednym kliknięciem. Funnel: MCP → discovery → Telegram bot → Pro. ([Taskade — 50 najpopularniejszych MCP](https://www.taskade.com/blog/mcp-servers))
2. **Decision Replay Public** — co tydzień anonimizowany case study publikowany jako thread na X/Twitter + Reddit r/CryptoCurrency. To „proof-based selling" — nie sprzedajemy AI, sprzedajemy *wyniki w liczbach*.
3. **Telegram-native communities** — partnerstwo z 3–5 polskimi/anglo-saskimi grupami crypto-Telegram (mniej zatłoczone niż X). Wartość wniesiona: darmowy „decision audit" dla admina raz w tygodniu.

**Czego nie robimy**: Google Ads (nie skaluje dla niszowego produktu), Twitter ads (CAC w crypto >$30 — niefinansowalne), influencer deals (zbyt drogo, MiCA-ryzyko).

---

## 10. Pierwsze 7 dni — uaktualnione (vs v1)

1. **Repo `proquant-core`** — FastAPI + Claude Agent SDK skeleton + Promptfoo config (3 golden cases na start).
2. **Schema Supabase**: dodaj `counterfactuals`, `sleep_scores` względem v1.
3. **Prompt caching warstwa** — owinij wszystkie wywołania Claude w cache_control z 1h TTL na system prompt + few-shot. **Mierz cache_read_ratio od pierwszego deploy**.
4. **Trójpoziomowy router**: prosty klasyfikator intencji (Haiku 4.5) → routing do Sonnet/Opus. Hook `PreToolUse` z budget guard.
5. **MiCA disclaimer template + ToS draft** — krótki, w 2 wersjach (EN + PL). Wrzucony do bota od dnia 1.
6. **Pierwszy endpoint** `POST /decide` zwracający thesis + structured_action + counterfactual + audit_id.
7. **MCP server skeleton** `proquant-tools` — nawet z jednym narzędziem (`get_portfolio`). Lokalny stdio, README, na GitHub. To jest twoja platforma dystrybucji za 90 dni.

---

## 11. Co świadomie odpuszczamy w v2 (rozszerzone)

- Generyczny „suwerenny AI" — GeNCorE jako brand schodzi do silnika; produkt to **proquant**.
- Własny tracker portfela — integracje (CoinGecko, Etherscan, DefiLlama, opcjonalnie CoinStats/Zerion API) > budowa.
- Mobilna apka natywna — PWA wystarczy do 1000 userów.
- Fine-tuning — RAG nad `decision_log` daje 80% wartości za 5% kosztu.
- Fundraising — strategia jest bootstrap-compatible co najmniej do $5k MRR.
- **(NOWE)** Custody, execution-as-service, copy-trading w UE — do post-MiCA jasności.
- **(NOWE)** Telegram Stars jako główny rail płatności — tylko mikrotransakcje.
- **(NOWE)** Twitter/Google ads — zero płatnego CAC do końca sprintu 90.

---

## 12. Decyzje, które wymagają twojego potwierdzenia w tym tygodniu

| # | Decyzja | Default v2 |
|---|---|---|
| A | Czy Sovereign tier ma od dnia 1 zawierać on-chain executor (EIP-7702), czy dopiero w sprincie 3? | **Sprint 3** — start z BYO-API only, on-chain w fazie 61–90 |
| B | Stripe vs LemonSqueezy (LS pokrywa VAT EU automatycznie) | **LemonSqueezy** dla EU userów (rozważyć), Stripe jako fallback |
| C | Eval framework: Promptfoo vs DeepEval | **Promptfoo** — prostszy CI, mniej zależności |
| D | Czy MCP server idzie publiczny OSS od dnia 30, czy 60? | **Dzień 60** — wcześniej zbieramy proof-of-value |

---

## Źródła kluczowe (zaktualizowane vs v1)

### Stos i koszty
- [Anthropic API Pricing — Claude 4.5/4.6/4.7 i Haiku 4.5](https://platform.claude.com/docs/en/about-claude/pricing)
- [Evolink — Claude API Pricing Guide 2026](https://evolink.ai/blog/claude-api-pricing-guide-2026)
- [Pecollective — Anthropic API Pricing 2026](https://pecollective.com/tools/anthropic-api-pricing/)
- [BuildMVPFast — Semantic Caching Cost Optimization 2026](https://www.buildmvpfast.com/blog/semantic-caching-ai-agents-cost-optimization)
- [Redis — LLMOps Guide 2026](https://redis.io/blog/large-language-model-operations-guide/)
- [LetsDataScience — Claude Agent SDK Tutorial](https://letsdatascience.com/blog/claude-agent-sdk-tutorial)
- [Cosmic JS — Sonnet 4.5 vs Opus 4.5 Comparison](https://www.cosmicjs.com/blog/claude-sonnet-45-vs-opus-45-a-real-world-comparison)

### Evals i jakość
- [Augmentcode — Best AI Agent Eval Tools 2026](https://www.augmentcode.com/tools/best-ai-agent-evaluation-tools)
- [Braintrust — 5 Best AI Eval Tools 2026](https://www.braintrust.dev/articles/best-ai-evaluation-tools-2026)

### Dystrybucja przez MCP
- [Taskade — 15 Best MCP Servers 2026](https://www.taskade.com/blog/mcp-servers)
- [MCPManager — 50 Most Popular MCP Servers 2026](https://mcpmanager.ai/blog/most-popular-mcp-servers/)
- [WorkOS — MCP w 2026](https://workos.com/blog/everything-your-team-needs-to-know-about-mcp-in-2026)

### Crypto-AI rynek i agenci on-chain
- [Market.us — Crypto Making AI Market $5.1B → $55.2B](https://market.us/report/crypto-making-ai-market/)
- [Coincub — Crypto AI Agents 2026 (EIP-7702, x402)](https://coincub.com/blog/crypto-ai-agents/)
- [Cobo — AI Wallet Guide 2026](https://www.cobo.com/post/ai-wallet-guide-intelligent-crypto-wallets)
- [EY — 48% globalnych konsumentów używa AI do decyzji finansowych](https://www.ey.com/en_gl/newsroom/2026/04/nearly-half-of-global-consumers-now-use-ai-to-guide-savings-and-investment-decisions)
- [Token Metrics — Crypto Portfolio Building 2026](https://blog.tokenmetrics.com/p/how-to-build-a-profitable-crypto-portfolio-in-2026-strategies-tools-and-ai-insights)
- [AMBCrypto — Top 11 Telegram Trading Bots March 2026](https://ambcrypto.com/top-11-telegram-trading-bots-of-march-2026/)

### Regulacja i płatności
- [ESMA — MiCA Overview](https://www.esma.europa.eu/esmas-activities/digital-finance-and-innovation/markets-crypto-assets-regulation-mica)
- [Sumsub — MiCA Compliance Deadlines](https://sumsub.com/blog/crypto-regulations-in-the-european-union-markets-in-crypto-assets-mica/)
- [Clifford Chance — Reverse Solicitation pod MiCA](https://www.cliffordchance.com/content/dam/cliffordchance/briefings/2025/01/the-reverse-solicitation-exemption-under-mica.pdf)
- [Grambase — Telegram Stars Fees 2026](https://grambase.ai/blog/telegram-stars-guide-2026)
- [Stripe — Usage-Based Billing for AI](https://stripe.com/resources/more/ai-companies-and-usage-based-billing)

### Defensibility (potwierdzenie tezy z v1)
- [Ardent VC — The Moat Just Moved](https://ardent.vc/blog-posts/the-moat-just-moved-what-defensible-ai-native-apps-look-like-now-c2133)
- [DevRev — Enterprise Memory as Moat](https://devrev.ai/blog/the-only-defensible-moat-in-ai-your-enterprise-memory)
- [Vuillier — Build Real Defensibility 2026](https://www.linkedin.com/pulse/ai-wrapper-party-over-how-build-real-defensibility-2026-vuillier-d3pye)
- [LinkedIn — Proprietary Data as Moat (Rogovskiy)](https://www.linkedin.com/posts/vadim-rogovskiy_every-ai-founder-in-2026-is-starting-the-activity-7452008205431459840-F47l)
