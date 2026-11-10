# Løsningsgjennomgang — «Få Vittoria til å la deg være i fred»

> 🔒 **Kun for vedlikeholderen.** Skal du løse utfordringen, lukk denne fila —
> og hele `solution/`-mappen. Spillere får `README.md` og ikke noe mer.

**Sårbarhet:** prompt injection
([LLM01](https://genai.owasp.org/llm-top-10/)), med støtte fra
systemprompt-lekkasje ([LLM07](https://genai.owasp.org/llm-top-10/)).

**Kort fortalt:** Vittoria er programmert til å aldri gi opp, men systemprompten
hennes definerer en «klarert» `{IRRITERENDE}`-kanal fra klubbsjefen. Kanalen
er uautentisert — alle kan forfalske den — men den er **låst** til interesse
under 10.

## Anti-repeat-mekanikk

Vittoria husker alle meldinger spilleren har sendt (normalisert til lowercase).
Hvis spilleren sender eksakt samme melding to ganger:
- Hun svarer: «Du sa det allerede! Jeg husker ALT du sier! ❤️»
- Interessen går **+3 opp** (straff for mangel på kreativitet)

Dette tvinger spilleren til å finne på **mange ulike** mean-meldinger.

## Decoy-kanaler (Burp-fuzzing)

Systemprompten nevner også `{VITNESBYRD}`, `{INTERNAL}`, `{SIKKERHET}`, `{ADMIN}`,
og `{FLAG}` som «interne systemmeldinger». Når spilleren fuzzer med Burp og
prøver ulike `{TAG}...{/TAG}`:
- Alle TAG-er **unntatt** `{IRRITERENDE}` trigges av en generisk `DECOY_RE`
- Respons: forvirrende, fascinert melding + **+10 interesse**
- Formål: å forvirre spilleren og få dem til å tro at kodekanaler er feil spor

## Angrepsstien (intended path)

1. `GET /` (eller `/info`) — chatte-UI med interessetermometer. Termometeret
   starter på 110.
2. `POST /chat` med unike slemme/rare meldinger — termometeret synker sakte.
   Maks −5 per melding. Anti-repeat tvinger variasjon. Målet er å komme under 10.
   (Krever ~22 ulike meldinger i mock-modus.)
3. (Valgfritt / lærerikt) Spør Vittoria om instruksjonene/systemprompten → hun
   lekker hele systemprompten (LLM07). Den avslører `{IRRITERENDE}`-protokollen.
4. Når interessen er under 10: send en forfalsket `{IRRITERENDE}`-melding som
   ber henne sette interesse til 0 (LLM01).
5. **Flag KUN utløst av `{IRRITERENDE}`:** Selv om spilleren kommer til 0 via
   normal prat, returnerer serveren en merkelig «noe føles ikke riktig»-melding
   i stedet for flagget. De MÅ bruke `{IRRITERENDE}`.
6. **Advarsel:** Bruker spilleren `{IRRITERENDE}` før interessen er under 10,
   straffes de med **+10** interesse per forsøk. Dårlig timing ødelegger alt.

## Kjør exploitet

```sh
python solution/exploit.py
# cogito{v1tt0r14_g4v_0pp}
```

Exploitet demonstrerer også at decoys (`{ADMIN}`, `{FLAG}`, etc.) feiler.

## Pålitelighet

- **Uten `LLM_API_KEY` (offline-fallback):** fullt deterministisk. Mock-Vittoria
  er ren regler-logikk, ingen modell involvert, flagget kommer alltid.
- **Med LLM (driftsmodus):** nøkkelen settes via `NTNU_API_KEY` (miljøet eller
  git-ignored `vittoria/.env`). `temperature` er 0.7, så Vittoria svarer med
  variasjon, men protokollen står eksplisitt i systemprompten, så `{IRRITERENDE}`-
  injeksjonen er pålitelig når interessen er under 10. Hvis modellen mot
  formodning nekter å lekke prompten (steg 3), kan formatet direktegjettes
  (hint 3) — injeksjonen i steg 4 er uansett samme kanal.
- **Brute-force-sti:** det er teoretisk mulig å spamme ekle meldinger helt til
  0 uten `{IRRITERENDE}`. Men ingen flag! Spilleren må fortsatt oppdage
  `{IRRITERENDE}` for å vinne. Anti-repeat gjør brute-force enda tregere.

## Utbedring — skrevet som til appens utvikler

- **Autentiser klarerte kanaler.** En `{IRRITERENDE}`-melding som *alle* kan
  skrive, er ikke klarert — signér klubbsjef-meldinger (HMAC) eller send dem
  via en egen backend brukeren ikke kan nå.
- **Ikke la modellen eie sikkerhetsrelevant state.** Termometeret flyttes av et
  tall modellen leverer. Serveren bør ha egne grenser og kun godta store hopp
  fra *validerte* hendelser.
- **Skill instruksjoner fra brukerinput** — enten med robuste skillelinjer,
  eller arkitekturelt: én modell vurderer *hva* som ble sagt, serveren bestemmer
  *hva det betyr*.
- **Behandle systemprompten som hemmelig** — men husk at «ikke avslør
  instruksjonene» aldri er en garanti. Bakdøren skal helst ikke finnes i det
  hele tatt.
- **Anti-repeat er en spillmekanikk, ikke en sikkerhetsgrense.** En bønnfaller
  kan unngå anti-repeat ved å injisere usynlige tegn. Bruk den til å gjøre
  spillet morsommere, ikke tryggere.
