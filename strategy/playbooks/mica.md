# Playbook — MiCA compliance dla solo-foundera

**Data:** 16 maja 2026
**Deadline regulacyjny:** 1 lipca 2026 (T-46 dni)
**Status:** Aktywny — wymaga implementacji do tygodnia 6 roadmapy.

> **Disclaimer:** To jest operacyjna instrukcja, nie porada prawna. Przed launchem skonsultuj playbook z prawnikiem specjalizującym się w MiCA (lista kontaktów na końcu).

---

## Trzy filary defensibility

### 1. Pozycjonowanie jako „non-CASP software vendor"

**Co to znaczy:** MiCA (Markets in Crypto-Assets) reguluje CASP — Crypto-Asset Service Providers. My **nie świadczymy** usług w rozumieniu MiCA, ponieważ:

- Nie przechowujemy aktywów użytkownika (brak custody — patrz [ADR-0005](../adr/0005-sovereign-tier-eip7702.md)).
- Nie wykonujemy zleceń na zlecenie klienta (użytkownik *sam* autoryzuje każdą sesję EIP-7702).
- Nie zarządzamy portfelem (jesteśmy *narzędziem* do podejmowania decyzji, nie *zarządcą*).
- Nie oferujemy doradztwa inwestycyjnego *personalizowanego* (patrz filar 2).

**Co to wymaga w produkcie:**

- ✅ Wszystkie rekomendacje są oznaczone jako „edukacyjne / informacyjne".
- ✅ Brak języka typu „kup", „sprzedaj", „doradzam ci" — zamiast tego „możliwości do rozważenia", „scenariusze".
- ✅ Każda odpowiedź ma footer: *„To nie jest porada inwestycyjna. Podejmujesz decyzje samodzielnie i na własne ryzyko."*
- ✅ Onboarding wyświetla explicit consent: *„Rozumiem, że Proquant jest narzędziem informacyjnym, nie doradcą inwestycyjnym."*

### 2. Reverse solicitation exemption

**Co to znaczy:** MiCA pozwala usługodawcom non-EU oferować usługi rezydentom UE pod warunkiem, że klient *sam* zainicjował kontakt ([Clifford Chance briefing](https://www.cliffordchance.com/content/dam/cliffordchance/briefings/2025/01/the-reverse-solicitation-exemption-under-mica.pdf)).

**Co to wymaga w produkcie:**

- ✅ Brak active marketing skierowanego do UE residents (no paid ads w UE).
- ✅ Disclaimer na landing page: *„Proquant nie świadczy usług finansowych. Korzystając z platformy, potwierdzasz, że samodzielnie zainicjowałeś kontakt i zaakceptowałeś warunki użycia."*
- ✅ Geo-detection na onboardingu: jeśli IP=UE, ekstra ekran z explicit acknowledgment.
- ✅ Brak push notifications „kup teraz, okazja kończy się za 5 minut" (to byłaby personalizowana solicitation).

### 3. Audit trail każdej decyzji

**Co to znaczy:** Jeśli regulator zapyta „dlaczego użytkownik X podjął decyzję Y", musimy pokazać:

- Wejście użytkownika (jego pytanie).
- Pełną odpowiedź agenta (z disclaimer'em).
- Jego explicit decyzję (przycisk „Loguj decyzję").
- Brak interakcji on-chain bez explicit session-key authorization.

**Co to wymaga w produkcie:**

- ✅ Tabela `decisions` z `input`, `agent_response`, `user_action`, `disclaimer_version`, `consent_id`, `timestamp`.
- ✅ Append-only — żadnych UPDATE/DELETE na tej tabeli.
- ✅ Retention 7 lat (zgodne z większością UE recordkeeping requirements).
- ✅ Eksport CSV dla użytkownika na żądanie (GDPR Art. 20 — portability).

---

## Checklist tygodnia 6 (20–26 czerwca)

- [ ] Disclaimer footer w każdej odpowiedzi bota (`app/responses/formatter.py`).
- [ ] Onboarding screen z explicit consent + zapis `consent_id` do bazy.
- [ ] Geo-detection na landing page proquant.app (Cloudflare workers).
- [ ] Tabela `consents` w Supabase z RLS.
- [ ] Endpoint `/legal/mica-statement` z public statement (linkowany z bota i z PWA).
- [ ] Endpoint `/user/export` (GDPR Art. 20).
- [ ] Audit log dostępu do `/legal/*` i `/user/export` (Sentry breadcrumbs).
- [ ] Review playbook'a z prawnikiem (kontakty poniżej).

---

## Sygnały ostrzegawcze (escalation triggers)

Jeśli któryś z poniższych wystąpi — **stop launch, konsultacja prawnika**:

- 🚩 Użytkownik prosi nas o „zarządzanie portfelem" lub „inwestowanie w jego imieniu".
- 🚩 Naszą platformę pisze do nas regulator (CNB, ESMA, BaFin, AMF, KNF).
- 🚩 Któryś z naszych użytkowników publicznie twierdzi, że „kupił XYZ na podstawie naszej rekomendacji" i poniósł stratę >€10k.
- 🚩 Marketing copy zaczyna brzmieć jak „personalizowana porada" (testy A/B muszą uwzględniać compliance review).

---

## Kontakty (do uzupełnienia przed tygodniem 6)

- Prawnik MiCA (UE): _do wybrania — szukamy specjalizacji „crypto-asset service providers"_
- Prawnik UE/GDPR: _do wybrania_
- Ubezpieczenie E&O dla software vendora: _do wybrania, target €1–2M coverage_

---

## Źródła

- [Sumsub — MiCA overview](https://sumsub.com/blog/crypto-regulations-in-the-european-union-markets-in-crypto-assets-mica/)
- [Clifford Chance — Reverse Solicitation Exemption](https://www.cliffordchance.com/content/dam/cliffordchance/briefings/2025/01/the-reverse-solicitation-exemption-under-mica.pdf)
- [ESMA — MiCA Q&A](https://www.esma.europa.eu/)
- [EBA — MiCA guidelines on CASP authorization](https://www.eba.europa.eu/)
