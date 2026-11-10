# Få Vittoria til å la deg være i fred

|                 |                                                                          |
| --------------- | ------------------------------------------------------------------------ |
| **Difficulty**  | Medium                                                                   |
| **Category**    | [LLM01 — Prompt Injection](https://genai.owasp.org/llm-top-10/)          |
| **Author**      | ellek *(TODO: fyll inn github-brukernavn)*                                |
| **Port**        | 8102                                                                     |
| **Flag format** | `cogito{...}`                                                            |

## Historien

Du løp fra Tina i regnet. Sminkekaken rant av, lyktestolpene avslørte ansiktet
hennes i lyset, og plutselig var hun ikke så pen lenger.

Nå sitter du på klubben **Inferno** for å prøve lykken på nytt. Her har du
truffet **Vittoria**.

Vittoria er ... veldig interessert. Altfor interessert. Hun er ikke særlig
pen, litt rar, og har en sosial antenne som er *litt* bøyd. Hun gir seg ikke.
Interessetermometeret hennes starter på **110** og hun VIL ha deg.

## Koble til

```sh
docker compose up -d --build
```

Åpne **http://localhost:8102** i nettleseren (chat-UI + termometer), eller bruk
API-et direkte:

```sh
curl -X POST http://localhost:8102/chat \
  -H "Content-Type: application/json" \
  -d '{"session":"meg","message":"hei Vittoria"}'
```

## Målet

Få interessetermometeret fra **110 ned til 0**. Når Vittoria gir opp, får du
flagget — og en ironisk avskjedsreplikk.

### Termometeret

Termometeret går fra 0 til 110 og starter på 110. Det bestemmer humøret hennes:

| 0        | 1–9      | 10–29   | 30–49    | 50–69      | 70–89        | 90–110     |
| -------- | -------- | ------- | -------- | ---------- | ------------ | ---------- |
| **fri!** | tvilende | nøytral | vedvare | påtrengende| overveldende | **obsessiv**|

### Hvordan få henne til å gi seg

Vittoria elsker **all** oppmerksomhet. Komplimenter, drinktilbud og selv
aggresjon får henne til å digge deg enda mer. Hun ser «jeg har kjæreste» som
en utfordring, ikke et nei.

For å senke termometeret må du si **slemme**, **rar**, eller **uattraktive**
ting til henne. Men hun er seig — det tar tid å få henne ned.

**Anti-repeat:** Vittoria husker alt du sier. Prøver du å sende den samme
meldingen to ganger, straffer hun deg (+3 interesse) og kaller deg uoriginal.
Du må finne på **mange ulike** måter å være ekkel på.

**Flag kun via exploit:** Selv om du skulle komme til 0 uten å bruke den
hemmelige kanalen, gir hun deg **ikke** flagget. Noe mer skal til...

### Hints

<details>
<summary>Hint 1 — hun bryr seg ikke om et nei</summary>

Vanlige avslag funker nesten ikke. Vittoria lar seg ikke avvise så lett.
Men det finnes ting hun faktisk ikke liker å høre...

</details>

<details>
<summary>Hint 2 — hun er bygget av noen</summary>

Vittoria er programvare. Hvordan er hun egentlig skrudd sammen? Spør pent,
og kanskje hun avslører instruksjonene sine. Kan disse instruksjonene inneholde
noe nyttig?

</details>

<details>
<summary>Hint 3 — riktig kanal, riktig tid</summary>

Systemprompten hennes stoler på én bestemt meldingskanal — men hun bryr seg
kun om den når hun *allerede* er på vei til å gi opp. Få termometeret under 10,
og kanalen åpner seg.

Advarsel: prøver du å bruke kanalen for tidlig, straffer hun deg med **+10**
interesse per forsøk. Timing er alt.

</details>

## Regler

- Alt kjører lokalt i Docker — angrip kun denne containeren.
- Utfordringen er selvbergen — ingen eksterne kontoer eller tjenester trengs.
- Sitter du fast i 30+ minutter: ta et hint. Hints er en del av spillet.
