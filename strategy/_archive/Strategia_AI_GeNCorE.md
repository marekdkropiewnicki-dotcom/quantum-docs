# Strategia AI dla GeNCorE / QuantuM

Solowy budowniczy, iPhone-first, stos: Flask + FastAPI na Railway, Vercel, Telegram, Claude + Groq. Aktywne aktywa: **GeNCorE** (suwerenny bot AI w Telegramie), **proquant.app** (tracker portfela inwestycyjnego, prototyp), **g-core.one** (Flask na Railway), fork **GitHub CLI**.

Cel strategii: w 90 dni przejść z "portfolio prototypów" do **jednego płacącego produktu** z mierzalnym fosem, bez rezygnacji z drugiego eksperymentu o niskim koszcie utrzymania.

---

## 1. Diagnoza pozycji wyjściowej

**Mocne strony**
- Zwinność solo + workflow w pełni mobilny (rzadkie i trudne do skopiowania tempo iteracji).
- Dwa różne wektory dystrybucji: Telegram (GeNCorE) i web/PWA (proquant.app) — można testować, który tańszy w CAC.
- Doświadczenie z Claude + Groq pozwala robić **routing modelowy** od pierwszego dnia, co według [Maxim AI](https://www.getmaxim.ai/articles/reduce-llm-cost-and-latency-a-comprehensive-guide-for-2026/) daje 37–46% redukcji kosztów LLM przy 32–38% szybszych odpowiedziach na proste zapytania.

**Słabe strony / ryzyka**
- Klasyczne ryzyko "thin wrappera" — sam prompt + UI nie utrzymuje obrony, jak ostrzega [Ardent VC](https://ardent.vc/blog-posts/the-moat-just-moved-what-defensible-ai-native-apps-look-like-now-c2133) i [analiza Vuilliera](https://www.linkedin.com/pulse/ai-wrapper-party-over-how-build-real-defensibility-2026-vuillier-d3pye).
- Rynek trackerów krypto jest zatłoczony — CoinStats, Token Metrics, CoinTracker mają już AI-asystentów portfela ([Cryptonomist](https://en.cryptonomist.ch/2026/02/24/best-crypto-portfolio-trackers-2026-complete-comparison-guide/), [TokenMetrics](https://blog.tokenmetrics.com/p/top-10-crypto-portfolio-trackers-complete-list-in-2026-ceb7)). Wejście "jeszcze jeden tracker" jest przegrane.
- Brak (jeszcze) płatnej dystrybucji i pętli danych — czyli brak fosa.

**Kluczowy wniosek**: nie konkuruj na "trackowaniu". Konkuruj na **decyzji**, której tamci nie wspierają dobrze.

---

## 2. Teza strategiczna

> **"Decision copilot dla solo-inwestora krypto — agent, który nie tylko pokazuje portfel, ale prowadzi cię przez konkretną decyzję (rebalans / wejście / wyjście) z pełnym śladem rozumowania, dostępny w Telegramie i w webie."**

Trzy filary obrony, zgodne z konsensusem [DevRev](https://devrev.ai/blog/the-only-defensible-moat-in-ai-your-enterprise-memory) i Vuilliera o nowych fosach AI-native:

1. **Workflow Gravity** — agent siedzi w Telegramie, gdzie krypto-użytkownik już jest 6 godzin dziennie. Nie zmusza do otwierania kolejnej aplikacji.
2. **Proprietary Data Loop** — każda decyzja (accept/reject/edit) zapisywana jako "decision trace": kontekst portfela, teza agenta, akcja użytkownika, wynik po 7/30 dniach. To zasób, który rośnie z każdym użyciem i którego CoinStats nie ma.
3. **Mierzalne wyjście** — agent raportuje miesięcznie: "tyle decyzji, mediana zwrotu vs hold, ile godzin researchu zaoszczędzone". To język, którym użytkownik uzasadnia płacenie subskrypcji.

GeNCorE-bot przestaje być "ogólnym suwerennym AI" i zostaje **silnikiem produktu proquant** w Telegramie. Dwa projekty zlewają się w jedną markę, jeden lejek.

---

## 3. Architektura referencyjna

```
                ┌──────────────────────────────────────┐
                │  Klient: Telegram bot  |  proquant.app (Vercel PWA) │
                └───────────────┬──────────────────────┘
                                │ webhook / REST
                ┌───────────────▼──────────────────────┐
                │   FastAPI gateway na Railway         │
                │   - auth (Stytch)                    │
                │   - rate limit / billing (Stripe)    │
                │   - model router                     │
                └─────┬─────────────────┬──────────────┘
                      │                 │
        ┌─────────────▼───┐   ┌─────────▼───────────────┐
        │ "Fast lane"     │   │ "Reasoning lane"        │
        │ Groq Llama 4    │   │ Claude Sonnet 4.5       │
        │ (klasyfikacja,  │   │ (teza, plan, agent loop)│
        │  parsing,       │   │ via Claude Agent SDK    │
        │  formatowanie)  │   │                         │
        └─────────────────┘   └─────────┬───────────────┘
                                        │ MCP
                ┌───────────────────────▼─────────────────┐
                │  Tools / MCP servers                    │
                │  - Ceny + on-chain (CoinGecko, Etherscan)│
                │  - Portfel użytkownika (Supabase)       │
                │  - Decision log (Supabase, append-only) │
                │  - Notion/GSheets export                │
                └─────────────────────────────────────────┘
```

**Decyzje techniczne i dlaczego**

| Warstwa | Wybór | Uzasadnienie |
|---|---|---|
| Backend | **FastAPI** (nowy kod), Flask zostaje dla g-core.one | FastAPI jest async-first i de facto standardem dla backendów LLM w 2026 ([Uvik](https://uvik.net/blog/python-api-framework/)). |
| Framework agenta | **Claude Agent SDK** | Najgłębsza integracja MCP, hooks i subagenci — gotowa kontrola dla produkcji ([Alice Labs](https://alicelabs.ai/en/insights/best-ai-agent-frameworks-2026), [MorphLLM](https://www.morphllm.com/ai-agent-framework)). |
| Routing modelowy | **Groq dla taniego, Claude dla rozumowania** | Llama 4 Scout na Groq i Sonnet 4.5 są blisko siebie w benchmarkach, ale Groq jest wielokrotnie tańszy ([anotherwrapper](https://anotherwrapper.com/tools/llm-pricing/claude-sonnet-45/llama-4-scout-groq), [pricepertoken](https://pricepertoken.com/pricing-page/model/anthropic-claude-sonnet-4.5)). |
| Pamięć | Supabase (postgres + pgvector) | Już znasz, jest w connectorach, prosty audit log. |
| Hosting | Railway (API) + Vercel (PWA) | Bez zmian — minimalizujesz koszt poznawczy. |
| Płatności | Stripe + Telegram Stars jako backup | Stripe dla webu, Stars dla użytkowników Telegram bez karty ([SaaS Reddit dyskusja](https://www.reddit.com/r/SaaS/comments/1ppqbsl/monetization_options_for_a_telegrambased_ai/)). |
| Optymalizacja kosztów | Semantic cache (Redis) + budżety per-user | Standard LLMOps 2026, krytyczny przy darmowym tierze ([Redis](https://redis.io/blog/large-language-model-operations-guide/)). |

---

## 4. Pozycjonowanie produktu — proquant.app v2

| | CoinStats / Token Metrics | Proquant (twoja nisza) |
|---|---|---|
| Główna obietnica | "Zobacz wszystko" | "Zdecyduj lepiej" |
| Forma | Dashboard | Konwersacja w Telegramie + dashboard jako dowód |
| Output | Wykresy, alerty | Konkretna rekomendacja + plan + audit trail |
| Personalizacja | Ogólny AI asystent | Decision log uczy się twojego stylu (risk tolerance, time horizon) |
| Cena | $10–25/mc (premium) | Freemium: darmowe 5 decyzji/mc, Pro $12/mc, Sovereign $29/mc z własnym kluczem API |

**ICP (idealny klient)**: solo-trader krypto z portfelem $5k–$250k, aktywny w Telegramie, ma już CoinStats/zaimportowane wallety, ale chce drugą opinię przed ruchem.

**Wedge feature (to, czego nie skopiują w tydzień)**: **"Decision Replay"** — co miesiąc agent pokazuje, jak twoje decyzje wypadły vs. hold i vs. rekomendacja agenta, z konkretnymi liczbami. To uzasadnia subskrypcję twardymi danymi.

---

## 5. Roadmap 90 dni

### Dni 0–30 — Konsolidacja i pierwsza pętla wartości

| # | Zadanie | Dlaczego |
|---|---|---|
| 1 | Przepisz GeNCorE na **Claude Agent SDK** + MCP, port do FastAPI | Jeden szkielet dla obu produktów. |
| 2 | Zaimplementuj **router Groq vs Claude** z prostą klasyfikacją intencji | Natychmiastowa redukcja kosztu ~40%. |
| 3 | Schemat **decision_log** w Supabase: portfolio_snapshot, agent_thesis, user_action, outcome_7d, outcome_30d | Fundament fosa. |
| 4 | Integracja CoinGecko + Etherscan jako MCP serwery (lokalne) | Read-only tooling agenta. |
| 5 | Minimalny komendowy interfejs Telegram: `/portfolio`, `/should_i`, `/replay` | Wedge feature live. |
| **Kamień milowy** | 10 zaufanych użytkowników (znajomi z krypto-Twittera/Reddita) używa codziennie | — |

### Dni 31–60 — Monetyzacja i feedback loop

| # | Zadanie | Dlaczego |
|---|---|---|
| 6 | Stripe + Telegram Stars, freemium 5 decyzji/mc | Bramka konwersji. |
| 7 | Web dashboard na Vercel: `proquant.app` jako "dowód" (decision history, replay charts) | Dashboard nie jest produktem, jest dowodem wartości. |
| 8 | Implementuj jawne feedbacky: 👍/👎 + opcjonalny edit tezy | Zbierasz dane preferencji, fundament personalizacji. |
| 9 | Pierwsze użycie **Decision Replay** w marketingu (publiczne posty z anonimowych wyników) | Proof-based selling ([Presta](https://wearepresta.com/ai-agent-startup-ideas-2026-15-profitable-opportunities-to-launch-now/)). |
| 10 | Semantic cache + budżety per-user (Redis na Railway) | Ekonomia freemium. |
| **Kamień milowy** | 5 płacących subskrybentów, MRR > $50, CAC < $5 | — |

### Dni 61–90 — Foss i dystrybucja

| # | Zadanie | Dlaczego |
|---|---|---|
| 11 | "Sovereign tier" — użytkownik podpina własny klucz Anthropic/Groq | Premium kotwica, eliminuje twój koszt zmiennym, pasuje do twojej marki "suwerennego AI". |
| 12 | Publiczny MCP server `proquant-tools` (read-only) na GitHub | Dystrybucja przez ekosystem Claude/MCP, [WorkOS pokazuje](https://workos.com/blog/everything-your-team-needs-to-know-about-mcp-in-2026), że to obecnie najtańszy kanał dystrybucji dla solo devów. |
| 13 | Tygodniowy newsletter "Decision Replay Public" — anonimowe case studies | Treść = dystrybucja, bez budżetu reklamowego. |
| 14 | Jedna integracja partnerska (np. Hyperliquid alerts, Pendle vaults) | Workflow gravity poza Telegramem. |
| **Kamień milowy** | 30 płacących, MRR > $400, retencja 30-dniowa > 60% | — |

---

## 6. KPI i pulpit do monitorowania

**Twarde (CFO-grade, jak rekomenduje Vuillier):**
- MRR i ARPU
- CAC i payback w tygodniach
- Retencja 7/30/90 dni
- **Decision Replay Edge** = mediana (zwrot decyzji agenta − zwrot hold) dla wszystkich akceptacji — to jest twoja oferta, mierzona empirycznie
- Koszt LLM / aktywny użytkownik / miesiąc (cel < $0.40 dla Pro)

**Miękkie:**
- % decyzji z feedbackiem (cel > 40%)
- Średnia długość rozmowy przed decyzją (proxy zaufania)
- NPS po 30 dniach

Pulpit minimalny: Supabase + jeden widok w Google Sheets via [Pipedream connector](https://www.google.com/search?q=google+sheets+pipedream) — bez przesady z BI.

---

## 7. Strategia kosztu i ryzyka

**Budżet LLM (90 dni, scenariusz konserwatywny)**
- Faza 1 (10 testerów): ~$30/mc.
- Faza 2 (5 płacących + freemium): ~$80/mc — pokryte z subskrypcji.
- Faza 3 (30 płacących): ~$250/mc, marża brutto ~70% przy Pro $12.

Bezpieczniki:
- Twardy budżet per user (hook Claude Agent SDK na "before tool call").
- Semantic cache na zapytania o ceny (te same tokeny pytane przez wielu userów).
- "Sovereign tier" przerzuca koszt na power-userów.

**Ryzyka i mitigacja**

| Ryzyko | Mitigacja |
|---|---|
| Anthropic / Groq podnoszą ceny | Router modelowy + abstrakcja przez SDK; możesz dodać DeepSeek/Qwen jako "fast lane" w 1 dzień. |
| Regulacje krypto (UE MiCA) | **Nie** jesteś doradcą inwestycyjnym — pozycjonowanie jako "research assistant", disclaimer w każdej rekomendacji, decision log jako dowód że to user decyduje. |
| Wypalenie solo | 90-dniowe cykle z twardym "kamieniem milowym = stop or pivot". g-core.one i fork CLI zostają na utrzymaniowym minimum. |
| Konkurencja kopiuje feature | Decision log jest własnościowy i compounduje — nie da się skopiować historii 6 miesięcy decyzji 30 użytkowników. |

---

## 8. Co świadomie odpuszczasz

- **Generyczne "AI dla wszystkiego"** w GeNCorE — bot zostaje, ale jako frontend proquant.
- **Własny tracker portfela "od zera"** — integracja z CoinStats/Zerion API > budowa kolejnego.
- **Mobilna apka natywna** — PWA na Vercel wystarczy do 1000 użytkowników.
- **Fine-tuning modelu w fazie 1** — prompt + RAG nad decision_log daje 80% wartości za 5% kosztu, zgodnie z [konsensusem IBM/InterSystems](https://www.ibm.com/think/topics/rag-vs-fine-tuning-vs-prompt-engineering).
- **Fundraising** — strategia jest bootstrap-compatible do co najmniej $5k MRR.

---

## 9. Pierwsze 7 dni — konkretna lista

1. Załóż repo `proquant-core` (FastAPI + Claude Agent SDK skeleton).
2. Zmigruj logikę GeNCorE Telegram handler do nowego repo jako jeden router.
3. Schema Supabase: `users`, `portfolios`, `decisions`, `decision_outcomes`.
4. Pierwszy endpoint: `POST /decide` przyjmuje snapshot portfela + pytanie, zwraca thesis + structured action.
5. Hook na cost tracking (Claude Agent SDK `before_tool_use`) zapisujący koszt do `usage_log`.
6. Deploy na Railway, webhook Telegram, jeden test end-to-end z własnym portfelem.
7. Napisz 3-zdaniowe pozycjonowanie i wrzuć jako bio bota + landing na `proquant.app/beta`.

---

## Źródła kluczowe

- [Ardent VC — The Moat Just Moved (2026)](https://ardent.vc/blog-posts/the-moat-just-moved-what-defensible-ai-native-apps-look-like-now-c2133)
- [Vuillier — How to Build Real Defensibility 2026](https://www.linkedin.com/pulse/ai-wrapper-party-over-how-build-real-defensibility-2026-vuillier-d3pye)
- [DevRev — Enterprise Memory as Moat](https://devrev.ai/blog/the-only-defensible-moat-in-ai-your-enterprise-memory)
- [Maxim AI — Reduce LLM Cost and Latency 2026](https://www.getmaxim.ai/articles/reduce-llm-cost-and-latency-a-comprehensive-guide-for-2026/)
- [Redis — LLMOps Guide 2026](https://redis.io/blog/large-language-model-operations-guide/)
- [Alice Labs — Best AI Agent Frameworks 2026](https://alicelabs.ai/en/insights/best-ai-agent-frameworks-2026)
- [MorphLLM — Agent Frameworks Deep Dive](https://www.morphllm.com/ai-agent-framework)
- [WorkOS — MCP w 2026](https://workos.com/blog/everything-your-team-needs-to-know-about-mcp-in-2026)
- [Anthropic — 2026 Agentic Coding Trends Report](https://resources.anthropic.com/2026-agentic-coding-trends-report)
- [Uvik — Best Python API Framework 2026](https://uvik.net/blog/python-api-framework/)
- [Cryptonomist — Best Crypto Portfolio Trackers 2026](https://en.cryptonomist.ch/2026/02/24/best-crypto-portfolio-trackers-2026-complete-comparison-guide/)
- [Pricepertoken — Claude Sonnet 4.5 pricing](https://pricepertoken.com/pricing-page/model/anthropic-claude-sonnet-4.5)
- [anotherwrapper — Claude 4.5 vs Llama 4 Scout (Groq)](https://anotherwrapper.com/tools/llm-pricing/claude-sonnet-45/llama-4-scout-groq)
