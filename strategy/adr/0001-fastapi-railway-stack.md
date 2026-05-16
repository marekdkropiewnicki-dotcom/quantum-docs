# ADR-0001: FastAPI + Railway jako bazowy stack

**Data:** 16 maja 2026
**Status:** Zaakceptowane
**Autor:** GeNCorE

---

## Kontekst

Potrzebujemy zaplecza dla agenta AI z webhookami Telegrama, MCP serverem i orkiestracją modeli. Solo-developer, iOS-first workflow (Code App), budżet $50/m do MVP. Musi obsługiwać:

- webhooks Telegrama (niska latencja, długie połączenia)
- streaming odpowiedzi Anthropic (SSE)
- background jobs (nightly decision review)
- skalowanie do ~1000 WAD bez przepisywania

---

## Rozważane opcje

| Opcja | Plusy | Minusy |
|---|---|---|
| **FastAPI + Railway** | Native async, Pydantic v2, dojrzały ekosystem AI (Anthropic/OpenAI SDK), Railway $5/m baseline + autoscale, push-to-deploy | Vendor lock na Railway (mitigated: Docker → portable) |
| Flask + Vercel Functions | Znajome, serwerless | Brak długich połączeń (10s limit), brak background jobs natywnie |
| Node.js (Fastify) + Fly.io | TypeScript = jeden język z frontendem | Słabszy ekosystem AI niż Python, mniej dojrzały Anthropic SDK |
| Cloudflare Workers | Globalna edge latencja | Brak Python, ograniczenia czasu CPU, problem z długimi LLM stream'ami |

---

## Decyzja

**FastAPI 0.111+ na Railway**, Python 3.12, Pydantic v2.

Dlaczego:

1. Anthropic + OpenAI SDK są first-class w Pythonie. Claude Agent SDK jest Python-first.
2. Railway ma najprostszy DX dla solo-dev: `git push` deployuje, $5/m baseline, autoscale bez konfiguracji.
3. FastAPI obsługuje SSE i WebSockets natywnie — kluczowe dla streamingu odpowiedzi LLM.
4. Docker → jeśli Railway zawiedzie, migracja na Fly.io / własny VPS to godzina pracy.

---

## Konsekwencje

**Pozytywne:**
- Jeden język (Python) dla backend + ML + skryptów.
- Pydantic v2 = automatyczne schematy dla MCP tools.
- Railway zarządza Redis, Postgres, cron — minimum DevOps.

**Negatywne:**
- Python jest wolniejszy niż Go/Rust. Mitygacja: większość czasu spędzimy czekając na LLM API, nie w CPU.
- Lock na Railway przy databases. Mitygacja: Supabase jako primary DB (już zewnętrzne), Railway tylko hosting aplikacji + Redis.

**Następne kroki:**
- Tydzień 1: `proquant-core` repo + `app/main.py` z `/health` endpointem.
- Tydzień 1: `infra/railway.toml` z konfiguracją usług.
