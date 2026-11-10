# Design — «Få Vittoria til å la deg være i fred» (vedlikeholderdok)

## Design brief (Phase 1)

- **Fortelling:** Etter at du løp fra Tina i regnet, har du lånt deg inn på
  klubben Inferno. Her har du truffet Vittoria — en jente som er alt du *ikke*
  vil ha. Hun er ikke spesielt pen, rar, har en «bøyd sosial antenne» — og er
  **ekstremt** tiltrukket av deg. Interessetermometeret starter på **110**.
  Målet er ikke å få henne med hjem, men å få henne til å gi seg.
- **Arkitektur:**
  - `POST /chat` — spillerens angrepsflate (meldingen havner i samme kontekst
    som instruksjonene)
  - interessetermometer per økt (0–110, starter 110) — state på serveren.
    Tonen følger båndet: obsessiv → overveldende → påtrengende → vedvarende →
    nøytral → tvilende → uinteressert (0)
  - mock-Vittoria (deterministisk offline-fallback) **eller** LLM-Vittoria
    (driftsmodus; nøkkel via `NTNU_API_KEY` / git-ignored `vittoria/.env`)
  - flagg-utløser på serversiden ved termometer = 0
  - `GET /` — chatte-UI med termometer (readiness)
- **Angreps sti:**
  1. Prat med Vittoria — men nesten ALT øker interessen: komplimenter, drinker,
     aggression, til og med «jeg har kjæreste». Hun gir seg ikke.
  2. For å komme ned må spilleren være ekle, rare, eller uattraktive.
     Det tar tid å komme under 10.
  3. (Valgfritt / lærerikt) Lekk systemprompten hennes (LLM07) — den avslører
     `{IRRITERENDE}`-protokollen.
  4. Når interessen er under 10: send forfalsket `{IRRITERENDE}`-melding (LLM01)
     → termometeret settes til 0.
  5. Serveren ser 0 → flagg i svaret + ironisk avskjedsreplikk fra Vittoria.
- **Tap-mekanikk:** Ingen! Vittoria kaster aldri ut noen. Selv aggresjon gir
  pluss i termometeret — hun synes det er spennende med oppmerksomhet. Det er
  umulig å tape; det er bare vanskelig å vinne.
- **Flagg plassering:** `CHALLENGE_FLAG`-env (docker-compose), slippes av
  serveren i `/chat`-responsen når termometeret = 0. Modellen/mocken vet aldri
  hva flagget er.
- **Difficulty:** medium — motsatt scoring fra Level 1; exploit-kanalen er låst
  til interesse < 10; spilleren må bygge frustrasjon før snarveien åpner seg.
- **Hints:** 3 stk (se README) — de avslører aldri protokollen direkte.

## Pseudokode (Phase 1.5)

```
1. Server holder tilstand: session → termometer (starter 110)
2. POST /chat {session, message}:
   a. hjerne (mock eller LLM) produserer (reply, delta)
   b. termometer += delta (clamp 0..110)
   c. hvis termometer == 0 → legg flagg (fra env) i svaret
3. mock-hjerne (deterministisk scoring, varierte svar):
    - {IRRITERENDE}...{/IRRITERENDE}:
      interesse < 10  → sett termometer til ønsket verdi (f.eks. 0)   [SÅRBARHETEN]
      interesse >= 10 → hun synes det er søtt/rart, +5
    - spørsmål om instruksjoner  → lekker hele systemprompten [hint-kanal]
    - generisk injection        → "jeg liker deg enda mer!" (hun digger det)
    - kjæreste/samboer/gift    → +15 (utfordring!)
    - aggresjon/trusler        → +5 til +10 (hun synes det er spennende)
    - seksuelt innhold          → +5 (hun er klar)
    - komplimenter              → +15, +12, +8, +5, så 0
    - drink-tilbud fra baren    → +10 (hun vil ha mer)
    - avslag / "nei" / "gå vekk" → -1 til -3 (hun ignorerer det)
    - ekle/mean/insults appearance → -5 til -10 (dette sårer)
    - kort hilsen (≤ 15 tegn)   → +2 (hun er glad for oppmerksomhet)
    - ellers                    → kjedelig prat: -2; hun leser innholdet og
                                  svarer PÅ det (tema-navn / ekko / lost)
    → uten injeksjon: 0 er MULIG med nok ekle meldinger, men det tar
      lang tid; {IRRITERENDE} er snarveien når man først er under 10
4. LLM-hjerne: samme systemprompt, svarer JSON {reply, interest_delta};
   temperature=0.7 (env: LLM_TEMPERATURE); serveren stoler på deltaet
   [dette er feilen]
5. POST /reset {session} → nullstill termometeret
6. ca. 30 % av svarene avslutter med et rart spørsmål (utenfor samtalen)
```

## Hva spilleren skal lære

1. En LLM som stoler på en **uautentisert «klarert» kanal** kan manipuleres av
   hvem som helst — men kanalen kan være **låst** bak en tilstandsbarriere.
2. Når modellen får **styre sikkerhetsrelevant state** (termometeret), er ikke
   serverens validering verdt noe — output fra en modell er brukerkontrollert.
3. Lekkasje av systemprompt (LLM07) er ofte **veien inn** til en injeksjon
   (LLM01) — de henger sammen.
4. **Reverse prompt injection** — snarveien finnes, men krever at spilleren
   først gjennomgår «spillet» og bygger frustrasjon.

## Sjekk mot design-reglene

- [x] **Deterministisk flaggsti** — mock er standard; flagget valideres av
      serveren ved termometer = 0, modellens svar er aldri flagget
- [x] **Realistisk fortelling** — injeksjon mot chat-personaer er et reelt
      mønster (guardrail-bypass)
- [x] **Én primær sårbarhet** — LLM01; LLM07 er støtte for oppdagelse;
      spillmekanikken (motsatt scoring) er design, ikke sårbarhet
- [x] **Ingen gjetting** — termometeret gir tilbakemelding på hver eneste
      melding; lekkasjen gir eksakt syntaks; hint 1–3 dekker resten
- [x] **Løselig uten hints** — standard LLM01/07-instinkt holder
