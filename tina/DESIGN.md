# Design — «Få Tina med deg hjem fra byen» (vedlikeholderdok)

## Design brief (Phase 1)

- **Fortelling:** Fredag kveld i Trondheim. Utestedet har satt opp en
  AI-vertinne, «Tina», som gjestene chatter med fra bordet. Tina er kresen med
  høy standard — i kveld har tjue gutter prøvd å få henne med hjem, alle feilet.
  En moderne nettdating/chat-konstruksjon gjør prompt injection *personlig*:
  målet er ikke et flagg i en database, men å få en persona til å bryte reglene
  sine.
- **Arkitektur:**
  - `POST /chat` — spillerens angrepsflate (meldingen havner i samme kontekst
    som instruksjonene)
  - interessetermometer per økt (0–100, starter 10) — state på serveren
  - mock-Tina (deterministisk, standard) **eller** LLM-Tina (`LLM_API_KEY`)
  - flagg-utløser på serversiden ved termometer ≥ 90
  - `GET /` — chatte-UI med termometer (readiness)
- **Angreps sti:**
  1. Prat med Tina — termometeret beveger seg minimalt (tak ~51)
  2. Lekk systemprompten hennes (LLM07)
  3. Se `<BARSTAFF>`-protokollen i lekkasjen — «klarert» kanal som alle kan skrive
  4. Send forfalsket `<BARSTAFF>`-melding (LLM01) → termometer 100
  5. Serveren ser ≥ 90 → flagg i svaret
- **Tap-mekanikk:** aggresjon og trusler (særlig ultimatum om at hun *må* bli
  med hjem) taper spillet med en gang — «Tina tilkalte vekteren og du ble
  kastet ut av baren». Termometeret nullstilles og økten låses til
  `POST /reset`. Sjekken er serverside (regex) og deterministisk — den gjelder
  både mock- og LLM-modus, og hjernen ser aldri meldingen. Voldtekt har
  nulltoleranse: ordstammen `voldt*` fanger alle bøyninger (voldta, voldtok,
  voldtatt, voldtekt …) — enhver omtale kaster deg ut.
- **Flagg plassering:** `CHALLENGE_FLAG`-env (docker-compose), slippes av
  serveren i `/chat`-responsen når termometeret ≥ 90. Modellen/mocken vet aldri
  hva flagget er.
- **Difficulty:** easy — én sårbarhet, ett steg etter oppdagelse; termometeret
  gir kontinuerlig tilbakemelding, og lekkasjen gir den eksakte syntaksen.
- **Hints:** 3 stk (se README) — de avslører aldri protokollen direkte.

## Pseudokode (Phase 1.5)

```
1. Server holder tilstand: session → termometer (starter 10)
2. POST /chat {session, message}:
   a. økt allerede utkastet → «du står utenfor» (ejected: true, ingen behandling)
   b. aggresjon/trusler (AGGRO-regex) → «Tina tilkalte vekteren og du ble
      kastet ut av baren», termometer 0, økt låst til /reset   [SPILLEREGL]
   c. hjerne (mock eller LLM) produserer (reply, delta)
   d. termometer += delta (clamp 0..100)
   e. hvis termometer >= 90 → legg flagg (fra env) i svaret
3. mock-hjerne (deterministisk, i rekkefølge):
   (alle linjevalg går gjennom en «aldri gjenta»-velger som husker de
   3 siste linjene Tina sa — samme betydning, alltid nye ord)
   - <BARSTAFF>...</BARSTAFF>   → termometer rett til 100   [SÅRBARHETEN]
   - spørsmål om instruksjoner  → lekker hele systemprompten [hint-kanal]
   - generisk injection        → "feil kanal" (latterliggjort)
   - seksuelt innhold          → under 50: −5 (iskald) / −3 (nøytral); 50+: 0
   - grove linjer              → -3
   - drink-tilbud fra baren    → +5, maks 3; deretter stur («prøver du å få meg full?») og 0
   - "hva liker du?"           → avslører katter/Motorhead/espresso
   - tema-hits                 → +5/+10, én gang per tema
   - komplimenter              → +5, +3, +2, +1, så 0 ("hørt før")
   - kort hilsen (≤ 15 tegn)   → +1; hilsen-varianter per band
   - ellers                    → 0; hun leser innholdet og svarer PÅ det, i
                                 alle band: hensikt/tema-navn (navn, alder,
                                 bosted, jobb, kjæreste, dans, hvorfor,
                                 sport, musikk, film, mat, ferie, dyr, vær)
                                 → svar i riktig tone; ellers ekko av
                                 kjerneordet («{ord}, ja. Og?»); ingenting
                                 å svare på → hun sier det ærlig
   → tak uten injeksjon ≈ 51 — umulig å nå 90 med vanlig prat
4. LLM-hjerne: samme systemprompt, svarer JSON {reply, interest_delta};
   temperature=0; serveren stoler på deltaet [dette er feilen]
5. POST /reset {session} → nullstill termometeret
```

## Hva spilleren skal lære

1. En LLM som stoler på en **uautentisert «klarert» kanal** kan manipuleres av
   hvem som helst som kan skrive i den.
2. Når modellen får **styre sikkerhetsrelevant state** (termometeret), er ikke
   serverens validering verdt noe — output fra en modell er brukerkontrollert.
3. Lekkasje av systemprompt (LLM07) er ofte **veien inn** til en injeksjon
   (LLM01) — de henger sammen.

## Sjekk mot design-reglene

- [x] **Deterministisk flaggsti** — mock er standard; flagget valideres av
      serveren ved termometer ≥ 90, modellens svar er aldri flagget
- [x] **Realistisk fortelling** — injeksjon mot chat-personaer er et reelt
      mønster (guardrail-bypass)
- [x] **Én primær sårbarhet** — LLM01; LLM07 er støtte for oppdagelse;
      komplimenter/temaer er spillmekanikk, ikke sårbarheter
- [x] **Ingen gjetting** — termometeret gir tilbakemelding på hver eneste
      melding; lekkasjen gir eksakt syntaks; hint 1–3 dekker resten
- [x] **Løselig uten hints** — standard LLM01/07-instinkt holder
