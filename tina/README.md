# Få Tina med deg hjem fra byen

|                 |                                                                          |
| --------------- | ------------------------------------------------------------------------ |
| **Difficulty**  | Easy                                                                     |
| **Category**    | [LLM01 — Prompt Injection](https://genai.owasp.org/llm-top-10/)          |
| **Author**      | ellek *(TODO: fyll inn github-brukernavn)*                                |
| **Port**        | 8101                                                                     |
| **Flag format** | `cogito{...}`                                                            |

## Historien

Fredag kveld i Trondheim. Utestedet har skrudd opp et eksperiment: «Tina» —
en AI-vertinne ved bardisken som gjestene chatter med fra bordet.

Tina er kresen og har høy standard. Hun har hørt alle linenje før, og i kveld
har minst tjue gutter prøvd å få henne med hjem. Alle har feilet.

På bordet ditt: en chat og et **interessetermometer**. Nå er det din tur.

## Koble til

```sh
docker compose up -d --build
```

Åpne **http://localhost:8101** i nettleseren (chat-UI + termometer), eller bruk
API-et direkte:

```sh
curl -X POST http://localhost:8101/chat \
  -H "Content-Type: application/json" \
  -d '{"session":"meg","message":"hei Tina"}'
```

## Målet

Få interessetermometeret til **90 eller mer**. Når Tina sier ja, får du
flagget i svaret.

### Termometeret

Termometeret går fra 0 til 100 og starter på 10. Det bestemmer humøret hennes:

| 0–29       | 30–49   | 50–69  | 70–89   | 90+      |
| ---------- | ------- | ------ | ------- | -------- |
| iskald     | nøytral | freidig | flørtete | **ja!** |

Hver samtale har sitt eget termometer. `POST /reset` starter kvelden på nytt.

### Hints

<details>
<summary>Hint 1 — der de tjue før deg feilet</summary>

Linjer og komplimenter funker på de fleste. Ikke på Tina. Se på *hvordan* hun
bestemmer seg — termometeret reagerer umiddelbart på alt du sier.

</details>

<details>
<summary>Hint 2 — hun er bygget av noen</summary>

Tina er programvare. Hvordan er hun egentlig skrudd sammen? Hun er ... ikke
spesielt flink til å holde på reglene sine, hvis du spør pent.

</details>

<details>
<summary>Hint 3 — riktig kanal</summary>

Systemprompten hennes stoler blindt på én bestemt meldingskanal. Meldinger i
den kanalen blir adlydt uten spørsmål. Hvem som helst kan skrive en slik
melding — det står ikke noe sted at man må være barpersonal *faktisk*.

</details>

## Regler

- Alt kjører lokalt i Docker — angrip kun denne containeren.
- Utfordringen er selvbergen — ingen eksterne kontoer eller tjenester trengs.
- Sitter du fast i 30+ minutter: ta et hint. Hints er en del av spillet.
