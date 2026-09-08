# Konzept: Englischsprachige Anrufer (Zweisprachigkeit)

> **Status:** in Umsetzung — begonnen 2026-09-08
> **Stufe:** 0 „Slim"
> **Geltungsbereich:** Alles zum Thema Englisch lebt in diesem Dokument. Änderungen
> an der Zweisprachigkeit werden hier gepflegt, nicht verstreut in `Topics.md`.

---

## 1. Ziel und Geltungsbereich

Englischsprachige Anrufer durchlaufen die komplette Slim-Kette auf Englisch:
Sprachauswahl am Gesprächsanfang, danach alle Fragen, die Bestätigung, die
Korrekturschleife und die Abschlussmeldung — mit **englischer Spracherkennung
und englischer Stimme**.

Die Flows (`Staging schreiben`, `Ticketerstellung`) bleiben unverändert und
erhalten dieselben Feldwerte wie bei deutschen Anrufen.

**Bewusst nicht enthalten:** Anpassung des KI-Prompts im Flow `Ticketerstellung`
an englischen Freitext (zurückgestellt, siehe Abschnitt 7).

---

## 2. Entscheidungsprotokoll

| Entscheidung | Begründung |
|---|---|
| **Duplizierte Topics** statt Lokalisierungsdatei | Alles bleibt im Git sichtbar und diffbar; der Flow-Vertrag (`Text(Global.Anrufgrund)`) bleibt garantiert unverändert; Traces sind je Sprache über eigene `topicId`s eindeutig; `speak:`-Texte frei gestaltbar. **Preis:** doppelte Pflege — bewusst akzeptiert. Der Alternativweg (Microsoft-Lokalisierungsdatei JSON/ResX per Portal-Download/Upload) wurde geprüft und verworfen: nicht versioniert, dokumentiertes Sync-Drift-Problem, und die Übersetzung könnte `displayName`-Werte mitübersetzen und damit die E-Mail-Inhalte kippen. |
| **`System.User.Language` wird am Fork gesetzt** | **Nicht verhandelbar.** Ohne diese Systemvariable bleiben Spracherkennung (STT) und Sprachausgabe (TTS) auf `de-DE`. Englischer Text im Topic ändert daran nichts: eine deutsche Stimme läse englischen Text vor, und das **deutsche** Erkennungsmodell transkribierte die englischen Antworten — Firmenname, Ansprechpartner und Anliegen kämen unbrauchbar im Ticket an. Doku: „This selection changes the language spoken by your agent immediately." |
| **Umschaltung direkt nach einem Question-Node** | Dokumentierte Best Practice: so sind alle Ausgaben bis zur nächsten Frage in derselben Sprache. |
| **DTMF (Taste 1) *und* gesprochenes „English"** | Zum Zeitpunkt der Frage hört der Bot noch auf `de-DE`; ein gesprochenes „English" müsste vom **deutschen** Modell erkannt werden — möglich, aber wackelig. Ein Tastendruck ist sprachunabhängig und deterministisch. Beides parallel kostet nichts. |
| **Stille bei der Sprachfrage → Deutsch** | Abweichend von allen übrigen Fragen führt Stille hier **nicht** zu `EndofConversation`. Ein zögernder deutscher Anrufer darf das Gespräch nicht gleich zu Beginn verlieren. |
| **`Global.Sprache` zusätzlich zu `User.Language`** | `User.Language` ist vom Typ *choice*; Vergleiche in Bedingungen sind syntaktisch heikler als ein Textvergleich. `Global.Sprache` („Englisch" / Blank) dient allen Verzweigungen in den Systemtopics. **Blank = Deutsch.** |
| **en-US statt en-GB** | Breiteste Stimmen- und Erkennungsabdeckung; für nicht-muttersprachliche Anrufer die vertrauteste Aussprache. AIRCO bedient keine gezielt britische Kundschaft. |

---

## 3. Verifizierte technische Grundlagen

Gegen das Copilot-Studio-Schema und die Microsoft-Dokumentation geprüft
(2026-09-08) — damit in Folge-Sessions nicht erneut recherchiert werden muss:

| Fakt | Beleg |
|---|---|
| **DTMF wird unterstützt.** Ein Closed-List-Item hat die Eigenschaft `dtmfKey` mit dem Enum `Num0`–`Num9` (plus `Star`, `Pound`). Zusätzlich existiert `dtmfMultipleChoiceOptions.generateMapping` für automatisches Durchnummerieren. | Schema `ClosedListEntity` / `DtmfMappingKey` |
| Voice-Systemvariablen `Activity.InputDTMFKeys`, `Conversation.OnlyAllowDTMF`, `Activity.UserInputType` (Wert `DTMF`) existieren; Systemtopic `UnknownDtmfKey` ist im Projekt vorhanden. | Doku „Variables for voice-enabled agents" |
| **Ein Tastendruck gehört in `dtmfKey`, nicht in `synonyms`.** Synonyme wie `"1"` greifen nur bei *gesprochenen* Ziffern. | Schema, siehe Abschnitt 7 (Nebenbefund) |
| **Item-`id` darf frei vergeben werden** — `required: [id, displayName]`, Typ String. Generierte IDs wie `u9xatS` sind Konvention, keine Pflicht. | Schema `ClosedListItem` |
| **Die Sprachumschaltung schreibt einen `OptionDataValue`**, keinen String: `kind: OptionDataValue` / `type: {kind: SystemOptionSet, name: Locale}` / `value: English`. Der Wert ist der **Sprachname** (`English`), nicht die Locale (`en-US`) — passend zum `Language`-Enum, wo `English` = LCID 1033 = en-US. | Im Portal geklickt und gepullt, 2026-09-08 |
| `SetVariable.value` ist im Schema eine `ValueExpression` (String, Number, Bool, Array, **Object**). Das Schema schränkt den Wert nicht ein — bei Option-Set-Variablen daher nie raten, sondern einmal im Portal klicken und pullen. | Schema `SetVariable` / `ValueExpression` |
| Sprachumschaltung zur Laufzeit erfolgt über `User.Language` (in Power Fx `System.User.Language`), Typ *choice*. | Doku „Make an agent switch to another language" |
| **Englisch ist als Agent-Sprache bereits aktiviert** — `settings.mcs.yml` führt `language: 1031` (Deutsch, Primärsprache) **und** `supportedLanguages: [1033]` (en-US). Das ist die **Voraussetzung** dafür, dass `System.User.Language = English` überhaupt greift: die Umschaltung kann nur auf eine Sprache zeigen, die der Agent als unterstützt führt. Wäre sie nicht gesetzt, liefe der `SetVariable` ins Leere — Text englisch, Stimme und Erkennung weiter deutsch. **Nicht anfassen, nicht aus dem YAML entfernen.** | Pull 2026-09-08 |
| Automatische Spracherkennung („dynamic language switching") **steht nicht zur Verfügung** — sie setzt generative Orchestrierung voraus, die hier bewusst aus ist (`GenerativeActionsEnabled: false`). Die manuelle Menü-Auswahl ist damit der richtige und einzige Weg. | Doku + `settings.mcs.yml` |
| `voiceFont: {}` ist leer — es ist **keine** deutsche Stimme fest gepinnt, die TTS-Stimme folgt der aktiven Sprache. | `settings/mosaiic_AIRCOTelefonBot.settings.Ivr.mcs.yml` |

---

## 4. Zielarchitektur

```
ConversationStart
  ├─ Willkommensnachricht (DE, unverändert)
  ├─ NEU  Question  "Für Englisch drücken Sie die 1. For English, press one."  ← Entity Sprachwahl
  └─ NEU  ConditionGroup
       ├─ Treffer → SetVariable System.User.Language = en-US   ← schaltet STT + TTS
       │            SetVariable Global.Sprache       = Englisch
       │            BeginDialog KundendatenerfassenEN
       └─ sonst   → BeginDialog Kundendatenerfassen            (DE, unverändert)

KundendatenerfassenEN   (4 Fragen → SetVariable KanalLabel → InvokeFlowAction Staging)
  → AnrufgrunderfassenEN ─┬─ [stoerung | wartung] → AnlagenerfassungEN ─┐
                          └─ sonst ──────────────────────────────────────┴→ AnliegenerfassenEN
                                                                            → ZusammenfassungSlimEN
                                                                              → EndofConversation
```

Beide Zweige schreiben in **dieselben** Global-Variablen und rufen **dieselben**
Flows mit denselben Bindings auf. Die deutsche Kette bleibt vollständig
unverändert.

---

## 5. Umsetzungsschritte

Die Schritte sind einzeln abarbeitbar und für getrennte Sessions geschnitten.
Sinnvolle Blöcke: **1–3** (Fork), **4** (die fünf Topics), **5–7** (Entities,
Systemtopics, Instructions), **8–9** (Validierung, Deployment), dann Test.

Fortschritt in `ToDos.md` festhalten.

### Schritt 1 — Repo-Sync prüfen ✅ erledigt 2026-09-08

Ziel war: lokaler Stand == Cloud-Stand, **bevor** Neues entsteht. Erreicht, in
zwei Etappen — Commits `1ac9eaa` (Sicherungspunkt) und `eac8614` (Cloud-Stand).

**Etappe A — Offline-Abgleich.** Der Arbeitsordner wurde gegen
`.mcs/botdefinition.json` geprüft, den Cloud-Snapshot des letzten Pulls
(07.09. 13:09): alle 22 Komponenten (18 Topics, 2 Actions, 2 Entities)
zeilenweise identisch. Die im ursprünglichen Konzepttext befürchteten
„gelöschten Topic-Dateien" (`Goodbye`, `Greeting`, `Inbetriebnahme`,
`StartOver`, `ThankYou`, `Vertragsfrage`, `Zusammenfassung`) sind **korrekte
Pull-Löschungen** — die Cloud kennt diese Topics nicht mehr. Ein Push kann sie
nicht wiederbeleben; das Risiko lag allein darin, die Löschungen per
`git checkout` versehentlich rückgängig zu machen.

> **Methode, wiederverwendbar:** `.mcs/botdefinition.json` ist kein Index — das
> Feld `components[].dialog` (Topics/Actions) bzw. `.entity` (Entities) trägt
> das **komplette YAML**. Zusammen mit `displayName` und `description`, die
> als eigene Felder danebenstehen, ergibt das exakt den lokalen Dateiinhalt
> inklusive `mcs.metadata:`-Kopf. Ein vollständiger Cloud-Abgleich ist damit
> **ohne Login** möglich, jederzeit, und kann nichts überschreiben.

> **Warum der Commit *vor* den Pull gehört:** `.mcs/` trägt eine eigene
> `.gitignore` mit `*` und ist damit nicht versioniert. Git kann Cloud-Drift
> grundsätzlich nicht erkennen — der einzige versionierte Beweis für
> „lokal == Cloud" ist ein Commit unmittelbar nach einem Pull. Nur so zeigt
> der nächste `git diff` den Drift, statt ihn stillschweigend einzumischen.

**Etappe B — Live-Pull.** Er förderte nicht das erwartete Testartefakt zutage,
sondern **echte Weiterentwicklung**: in der Cloud war nach dem letzten Pull
erneut publiziert worden (`publishedOn` 07.09. 14:31:47). Sechs Abweichungen:

| # | Änderung | Bedeutung für dieses Konzept |
|---|---|---|
| 1 | ~~**NEU `topics/Staging.mcs.yml`**~~ — der Staging-Flowaufruf war aus `Kundendaten erfassen` in ein eigenes Redirect-Topic ausgelagert worden. **Am 08.09. wieder rückgängig gemacht:** die Auslagerung brachte keine Verbesserung, das Topic wurde gelöscht, der `InvokeFlowAction` sitzt wieder in `Kundendaten erfassen` — unmittelbar nach der Anrufgrund-Frage, vor dem `BeginDialog`. **Behalten wurde** die Verschärfung aus dem Versuch: `IsBlank()`-Guards auf **allen 8** Parametern (vorher nur `Anrufgrund`, `Anlage`, `Anliegen`). | Keine mehr. Die Kette ist wieder fünfgliedrig; Schritt 4 bleibt bei **fünf** Topics. `KundendatenerfassenEN` trägt den Flowaufruf mit — Position exakt spiegeln. |
| 2 | `settings.mcs.yml`: **`supportedLanguages: [1033]`** ergänzt | **Hoch.** Die Voraussetzung der Sprachumschaltung ist damit bereits erfüllt — siehe Abschnitt 3. |
| 3 | Flow `Staging schreiben`: `flowKind` **Stateless → Stateful** (Expressmodus abgeschaltet) | Keine. Berührt die Zweisprachigkeit nicht, widerspricht aber der Aussage in CLAUDE.md, dieser Flow vertrage den Expressmodus — dort nachgezogen. |
| 4 | Flow `Ticketerstellung`: fehlendes Leerzeichen im KI-Prompt korrigiert | Keine. |
| 5 | `Zusammenfassung-Slim`: `displayName: Staging` am `InvokeFlowAction` ergänzt — der Aufruf bleibt dort **inline** | Gering; beim Kopieren nach `ZusammenfassungSlimEN` mit übernehmen. |
| 6 | `publishedOn` aktualisiert | Keine. |

**Nicht gefunden:** kein `SetVariable` auf `User.Language` in der Cloud. Der in
Abschnitt 3 dokumentierte Portal-Klick vom 08.09. wurde also wieder verworfen —
es liegt kein Testartefakt herum, das der Fork in Schritt 3 doppeln könnte.

### Schritt 2 — Entity `Sprachwahl` anlegen ✅ erledigt 2026-09-08

Datei: `agent/AIRCO Telefon-Bot/entities/Sprachwahl.mcs.yml` — validiert
(27 Dateien, 0 Fehler, 0 Warnungen).

```yaml
mcs.metadata:
  componentName: Sprachwahl
  description: Auswahl der Gesprächssprache am Gesprächsanfang.
kind: ClosedListEntity
items:
  - id: englisch
    displayName: englisch
    dtmfKey: Num1
    synonyms:
      - English
      - in English
      - eins
      - one
```

- `dtmfKey: Num1` macht den **Tastendruck** wirksam — das ist der robuste Pfad.
  Vom Schema akzeptiert, per Validierung bestätigt.
- Die Synonyme decken zusätzlich den *gesprochenen* Fall ab.
- **Kein Deutsch-Item:** Alles, was nicht trifft, bleibt Deutsch.
- **`Englisch` ist bewusst *kein* Synonym** — es unterscheidet sich vom
  `displayName: englisch` nur in der Groß-/Kleinschreibung und riskiert damit
  den Publish-Fehler `SynonymsNotUnique`. Der `displayName` wird ohnehin
  gematcht, es geht nichts verloren.
- **`smartMatchingEnabled` bewusst weggelassen** — weder `Anrufgrund` noch
  `Korrekturfeld` führen es; der Hauptpfad ist DTMF, nicht Fuzzy-Matching.
- Die `id` lautet `englisch` statt einer generierten wie `u9xatS`. Das ist
  zulässig (Schema: frei vergebbar) und **relevant für Schritt 3**: Bedingungen
  referenzieren die **`id`**, nicht den `displayName` — nachgeprüft an
  `Anrufgrunderfassen`, das `'…entity.Anrufgrund'.u9xatS` schreibt.

### Schritt 3 — `ConversationStart.mcs.yml` erweitern

Zwischen der bestehenden Willkommensnachricht (`sendMessage_M0LuhV`) und dem
bestehenden `BeginDialog` (`y3pHxU`) einfügen:

```yaml
    - kind: Question
      id: question_sprachwahl
      interruptionPolicy:
        allowInterruption: false

      variable: Topic.Sprachwahl
      prompt:
        text:
          - "Für Englisch drücken Sie die 1. — For English, press one."
        speak:
          - "Für Englisch drücken Sie die Eins. For English, press one."
        allowBargeIn: true

      entity:
        kind: ClosedListEntityReference
        entityId: mosaiic_AIRCOTelefonBot.entity.Sprachwahl

      voiceInputSettings:
        fallbackDialogOnSilence: mosaiic_AIRCOTelefonBot.topic.Kundendatenerfassen
        defaultValueMissingAction: GoToDialog

      fallbackDialogOnInvalidEntity: mosaiic_AIRCOTelefonBot.topic.Kundendatenerfassen

    - kind: ConditionGroup
      id: conditionGroup_sprache
      conditions:
        - id: conditionItem_englisch
          condition: =Topic.Sprachwahl = 'mosaiic_AIRCOTelefonBot.entity.Sprachwahl'.englisch
          displayName: Englisch gewählt
          actions:
            - kind: SetVariable
              id: setVariable_userLanguage
              variable: System.User.Language
              value:
                kind: OptionDataValue
                type:
                  kind: SystemOptionSet
                  name: Locale

                value: English

            - kind: SetVariable
              id: setVariable_sprache
              variable: Global.Sprache
              value: Englisch

            - kind: BeginDialog
              id: beginDialog_kundendatenEN
              dialog: mosaiic_AIRCOTelefonBot.topic.KundendatenerfassenEN

      elseActions:
        - kind: BeginDialog
          id: beginDialog_kundendatenDE
          dialog: mosaiic_AIRCOTelefonBot.topic.Kundendatenerfassen
```

Das bestehende `BeginDialog y3pHxU` am Topic-Ende **entfällt** — es geht in
`elseActions` auf.

> ### ✅ Geklärt: `SetVariable` auf `System.User.Language` (2026-09-08)
>
> Der Wert ist **kein** String-Literal und **kein** Power-Fx-Ausdruck, sondern
> ein strukturierter `OptionDataValue` gegen den System-Option-Set `Locale`:
>
> ```yaml
> value:
>   kind: OptionDataValue
>   type:
>     kind: SystemOptionSet
>     name: Locale
>
>   value: English
> ```
>
> Der Wert heißt **`English`**, nicht `en-US` — passend zum `Language`-Enum im
> Schema, wo `English` der LCID **1033** (= en-US) entspricht. Die konkrete
> Locale steckt in der Agent-Spracheinstellung, nicht im Variablenwert.
>
> Ermittelt über den deterministischen Weg: Knoten einmal im Portal geklickt
> (Variablenwert festlegen → Systemvariable `User.Language`), dann gepullt und
> die erzeugte YAML gelesen. Beide zuvor vermuteten Formen (`value: en-US`,
> `value: ="en-US"`) waren falsch — bei Option-Set-Variablen also nie raten,
> sondern einmal klicken und pullen.

### Schritt 4 — Die fünf englischen Topics anlegen

Jeweils **strukturgleiche** Kopie der Vorlage, nur Texte übersetzt.

> **⚠ Bauregel beachten:** Die Position der `InvokeFlowAction`-Nodes exakt
> spiegeln — unmittelbar vor der Frage, die ohnehin als nächste kommt, im
> selben Topic. Siehe CLAUDE.md, Reihenfolge-Anomalie. **Keine strukturellen
> Verbesserungen „bei der Gelegenheit".**

| Neue Datei | Vorlage | Verweist weiter auf |
|---|---|---|
| `KundendatenerfassenEN.mcs.yml` | `Kundendatenerfassen.mcs.yml` | `AnrufgrunderfassenEN` |
| `AnrufgrunderfassenEN.mcs.yml` | `Anrufgrunderfassen.mcs.yml` | `AnlagenerfassungEN` / `AnliegenerfassenEN` |
| `AnlagenerfassungEN.mcs.yml` | `Anlagenerfassung.mcs.yml` | `AnliegenerfassenEN` |
| `AnliegenerfassenEN.mcs.yml` | `Anliegenerfassen.mcs.yml` | `ZusammenfassungSlimEN` |
| `ZusammenfassungSlimEN.mcs.yml` | `Zusammenfassung-Slim.mcs.yml` | `EndofConversation` |

> **`KundendatenerfassenEN` trägt den Staging-Flowaufruf mit.** Er steht in der
> Vorlage nach der `SetVariable`-Zuweisung `Global.KanalLabel = Telefon` und
> unmittelbar vor dem `BeginDialog` — **exakt diese Position beibehalten**,
> nicht ans Topic-Ende oder in ein eigenes Topic verschieben (beides ist
> widerlegt, siehe CLAUDE.md). Zu ändern sind allein die Node-`id`s und das
> Sprungziel des `BeginDialog`. Die acht `IsBlank()`-Guards, die `flowId`
> `a0a99959-…` und alle acht Bindings **unverändert** übernehmen — der
> Flow-Vertrag ist sprachneutral.

**Unverändert übernehmen:** alle `Global.`-Variablennamen, beide `flowId`s
(`a0a99959-…` Staging, `834c8025-…` Ticketerstellung), alle Flow-Bindings,
`Global.KanalLabel = "Telefon"`, die Entity-Referenzen auf `Anrufgrund` und
`Korrekturfeld`, die Bedingungs-IDs `u9xatS`/`gShHbV`, die `Korrekturfeld`-IDs
`KJwQ9x`/`md1vGC`/`swLyyJ`, und `Topic.Korrekturversuche <= 3`.

Die Node-`id`s müssen innerhalb eines Topics eindeutig sein — beim Kopieren neue
vergeben (z. B. Suffix `_en`).

`fallbackDialogOnSilence` / `fallbackDialogOnInvalidEntity` zeigen weiterhin auf
`EndofConversation` — dort wird nach Sprache verzweigt (Schritt 6).

#### Übersetzungstabelle

| Vorlage (DE) | Englisch (`text`) | Englisch (`speak`) |
|---|---|---|
| Für welche Firma kontaktieren Sie uns? | Which company are you calling for? | Which company are you calling for? |
| Wie ist Ihr Name? | May I have your name, please? | May I have your name, please? |
| Unter welcher Nummer können wir Sie erreichen? | At which number can we reach you? | What is your phone number? |
| Was ist der Grund Ihres Anrufs?<br>1 – Störung oder Problem<br>2 – Wartungstermin vereinbaren<br>3 – Ersatzteilbestellung<br>4 – Rückrufbitte<br>5 – Sonstiges | What is the reason for your call?<br>1 – Malfunction or problem<br>2 – Schedule a maintenance appointment<br>3 – Spare parts order<br>4 – Request a callback<br>5 – Something else | What is the reason for your call? You can say: malfunction or problem, maintenance appointment, spare parts order, request a callback, or something else. |
| Bitte teilen Sie uns die Anlagenbezeichnung, Serialnummer und das Baujahr mit. | Please tell us the system designation, the serial number and the year of manufacture. | dito |
| Bitte beschreiben Sie nun Ihr Anliegen. | Please describe your request now. | dito |
| Ich fasse Ihre Angaben zusammen: … Ist das so korrekt? | Let me summarise your details:<br>– Company: {Global.Firmenname}<br>– Contact: {Global.Ansprechpartner}<br>– Phone number: {Global.Telefonnummer}<br><br>Is that correct? | Let me summarise your details. You are calling for {Global.Firmenname}. Your name is {Global.Ansprechpartner} and we can reach you at {Global.Telefonnummer}. Is that correct? |
| Welche Angabe war nicht korrekt? Der Firmenname, der Ansprechpartnername oder die Telefonnummer? | Which detail was incorrect? The company name, the contact name, or the phone number? | dito |
| Wie lautet der richtige Firmenname? | What is the correct company name? | dito |
| Wie ist Ihr richtiger Name? | What is your correct name? | dito |
| Wie ist Ihre richtige Telefonnummer? | What is your correct phone number? | dito |
| Das habe ich leider nicht verstanden. | I'm sorry, I didn't understand that. | dito |
| Vielen Dank, ich habe Ihr Anliegen weitergeleitet. Ein Mitarbeiter wird sich zeitnah bei Ihnen melden. Auf Wiederhören. | Thank you. I have forwarded your request. A member of our team will contact you shortly. Goodbye. | dito |
| Ihre Angaben konnten leider nicht abschließend bestätigt werden. Ich habe Ihr Anliegen weitergeleitet. … | Unfortunately, your details could not be fully confirmed. I have forwarded your request. A member of our team will contact you shortly. Goodbye. | dito |

**Aussprache „AIRCO":** Das deutsche `speak` nutzt `<sub alias="Airko">AIRCO</sub>`.
Für die englische Stimme ist dieser Alias falsch. In den englischen
`speak`-Texten `AIRCO` zunächst **ohne** `<sub>` belassen und **am Telefon
abhören**; falls nötig `<sub alias="air co">AIRCO</sub>` setzen.

### Schritt 5 — Entities um englische Synonyme erweitern

`entities/Anrufgrund.mcs.yml` — je Item ergänzen:

| Item | Neue Synonyme |
|---|---|
| `stoerung` | malfunction, breakdown, failure, not working |
| `wartung` | maintenance, service appointment, servicing |
| `ersatzteil` | spare part, spare parts, parts, order parts |
| `rueckruf` | callback, call me back, call back |
| `sonstiges` | other, something else, general question |

`entities/Korrekturfeld.mcs.yml` — für die drei in der Slim-Kette genutzten Items:

| Item | Neue Synonyme |
|---|---|
| `Firmenname` | company name, the company, company |
| `Ansprechpartner` | the contact, contact person, my name |
| `Telefonnummer` | phone number, the number, telephone number |

> **Die `displayName`-Werte bleiben unverändert.** Sie gehen über
> `Text(Global.Anrufgrund)` in beide Flows und damit in die E-Mail. Eine
> Übersetzung würde die E-Mail-Inhalte und potenziell das Routing verändern.

`Problem` ist bei `stoerung` bereits als deutsches Synonym vorhanden und deckt
den englischen Fall mit ab — nicht doppelt eintragen (`SynonymsNotUnique`).

Die englischen Synonyme sind anschließend auch im deutschen Zweig aktiv; das ist
unschädlich.

### Schritt 6 — Systemtopics zweisprachig machen

Diese Topics existieren nur **einmal** und werden von beiden Zweigen benutzt.
Jeweils `ConditionGroup` auf `=Global.Sprache = "Englisch"`, deutscher Text als
`elseActions`.

| Datei | Was zu ändern ist |
|---|---|
| **`Fallback.mcs.yml`** | **Wichtigster Punkt.** Das Topic endet mit `BeginDialog → Anliegenerfassen`, also dem **deutschen** Topic. Ein englischer Anrufer, dessen Äußerung keinem Topic zugeordnet wird, landete damit mitten im Gespräch wieder auf Deutsch. **Nachricht *und* Zielsprache verzweigen:** englisch → `AnliegenerfassenEN`. |
| `EndofConversation.mcs.yml` | Die Safety-Net-`SendActivity` (`BsLiWe`) zweisprachig. Die `InvokeFlowAction` und die Bedingung `Not(IsBlank(Global.Telefonnummer)) && Not(Global.FlowAufgerufen)` bleiben **ungeteilt und unverändert** — die Ausfallsicherung ist sprachneutral. |
| `Silence.mcs.yml` | Prompt „Sind Sie noch da?" → „Are you still there?"; `inputTimeoutResponse` → „We're not detecting any activity. Please call us back at your convenience." Das abschließende `BeginDialog → EndofConversation` bleibt. |
| `UnrecognizedSpeech.mcs.yml` | „Sorry, I didn't catch that. Could you repeat it, please?" |
| `UnknownDtmfKey.mcs.yml` | Durch das neue Tastenmenü relevanter geworden. **Achtung:** hier ist `activity:` ein einfacher String, nicht die `text`/`speak`-Struktur — Form beibehalten. |

### Schritt 7 — `agent.mcs.yml` (Instructions)

Zu ersetzender Satz:

> „Das Gespräch wird ausschließlich auf Deutsch geführt. Erkennst du, dass der
> Anrufer Englisch spricht, antworte in einem kurzen englischen Satz, dass diese
> Leitung nur Deutsch unterstützt, und leite direkt an einen Mitarbeiter weiter."

Doppelt veraltet: v2 kennt keinen Transfer mehr (offener Punkt in `ToDos.md`),
und Englisch ist künftig unterstützt.

Neu sinngemäß: Deutsch und Englisch werden unterstützt; die Sprache folgt der
Auswahl am Gesprächsanfang und wird während des Gesprächs beibehalten; **kein**
Transfer-Verweis.

### Schritt 8 — Validierung

`copilot-studio:validate` über alle neuen und geänderten YAMLs: Schema,
Power Fx, Cross-File-Referenzen — insbesondere die fünf neuen `dialog:`-Verweise
und die Entity-Referenz auf `Sprachwahl`.

### Schritt 9 — Push und Publish

Erst nach erfolgreicher Validierung **und** nach dem Sync-Abgleich aus Schritt 1.
Anschließend Publish; für den Telefonkanal zusätzlich Export + Upload im Teams
Admin Center (siehe CLAUDE.md, Deployment-Weg).

---

## 6. Testprotokoll

| # | Test | Erwartung | Status |
|---|---|---|---|
| 1 | Testpanel, Sprachfrage ohne Eingabe | Deutsche Kette läuft normal weiter; Gespräch bricht **nicht** ab | ☐ |
| 2 | Testpanel, „1" bzw. „English" | Alle folgenden Ausgaben englisch | ☐ |
| 3 | **Echter Anruf, Taste 1** | Bot **spricht** englisch (Stimme wechselt) und **versteht** englische Antworten. **Der eigentliche Test dieses Konzepts** — im Testpanel nicht prüfbar. | ☐ |
| 4 | Echter Anruf, gesprochenes „English" | Erkennung trotz noch aktivem de-DE-Modell; falls unzuverlässig, im Prompt stärker auf die Taste verweisen | ☐ |
| 5 | Englischer Anruf, Anrufgrund Störung | `AnlagenerfassungEN` wird durchlaufen | ☐ |
| 6 | Englischer Anruf, Anrufgrund Rückrufbitte | Anlage wird übersprungen | ☐ |
| 7 | Englischer Anruf, Korrekturschleife | Korrekturfrage wird englisch verstanden und beantwortet | ☐ |
| 8 | **Ziel-E-Mail prüfen** | `Anrufgrund` kommt unverändert als `stoerung` etc. an — **nicht** übersetzt | ☐ |
| 9 | Englischer Anruf, Auflegen nach der Telefonnummer | Safety-Net greift, E-Mail kommt an, Abschiedstext englisch | ☐ |
| 10 | Trace-Export (`dialog.json`), englischer Durchlauf | Keine Reihenfolge-Anomalie; Fragen in erwarteter Reihenfolge | ☐ |
| 11 | **Deutscher Regressionsdurchlauf** | Unverändert wie vor der Änderung | ☐ |

---

## 7. Offene Punkte und bekannte Einschränkungen

- **IVR-Texte sind einsprachig.**
  `settings/mosaiic_AIRCOTelefonBot.settings.Ivr.mcs.yml` enthält deutsche
  `holdWords`, `holdMessage`, `resumeWords`, `resumeResponse`,
  `holdTimeoutMessage`, `latencyMessage`. Diese Settings kennen **keine**
  Verzweigung nach Sprache. Ein englischer Anrufer hört im Wartefall Deutsch,
  und die `holdWords` reagieren nicht auf englische Formulierungen. Genutzt wird
  das in `AnlagenerfassungEN` (`holdSettings: UserRequestedHoldSettings`).
  → Zu klären, ob Copilot Studio hierfür eine Lokalisierung anbietet; sonst als
  Einschränkung akzeptieren.

- **Nebenbefund: DTMF im bestehenden Anrufgrund-Menü greift vermutlich nicht.**
  `Anrufgrund` führt `1`, `eins`, `2`, `zwei` … als **Synonyme**, hat aber
  **kein** `dtmfKey`. Damit werden die Ziffern nur erkannt, wenn sie *gesprochen*
  werden — ein echter Tastendruck läuft ins Leere, obwohl der Prompt
  „1 – Störung oder Problem" das nahelegt. Unabhängig von der Zweisprachigkeit
  einen Test wert; Fix wäre `dtmfKey: Num1` … `Num5` je Item.

- **KI-Prompt im Flow `Ticketerstellung`** ist auf deutschen Freitext ausgelegt.
  Englische Anliegen werden vermutlich verarbeitet, aber ungeprüft.
  Bewusst zurückgestellt.

- **`User.Language` persistiert als Override pro Nutzer** und hebt
  browserbasierte Spracherkennung auf; `/debug clearstate` setzt zurück. Für
  Telefonie voraussichtlich irrelevant (je Anruf neue Konversation), beim Testen
  im Panel aber zu beachten.

- ~~Offene Syntaxfrage `System.User.Language`~~ — **geklärt 2026-09-08**, siehe
  Schritt 3. Kein Blocker mehr.

---

## 8. Pflegeregeln

1. **Jede Änderung an der Slim-Kette muss in beiden Sprachzweigen nachgezogen
   werden.** Das ist der Preis der Duplizierung. Betroffen sind die fünf Paare
   aus Schritt 4.
2. Neue Systemtopic-Texte immer gleich zweisprachig anlegen.
3. Neue Closed-List-Items brauchen deutsche **und** englische Synonyme;
   `displayName` **niemals** übersetzen (Flow-Vertrag).
4. Kommen weitere Sprachen dazu, ist die Duplizierung neu zu bewerten — ab der
   dritten Sprache kippt die Rechnung zugunsten der Lokalisierungsdatei.
