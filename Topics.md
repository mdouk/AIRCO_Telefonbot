# Topic-Definitionen (Arbeitsstand)

Dieses Dokument hält die im Sparring erarbeiteten Inhalte der Copilot-Studio-Topics fest,
bis sie in Copilot Studio angelegt/exportiert werden.

Jedes Topic ist einer **Ausbaustufe** zugeordnet (siehe Roadmap in
[Architektur.md](Architektur.md)). Als „Stufe 2 (zurückgestellt)" markierte
Topics sind erarbeitetes Material, das erst später gebaut wird — nichts davon
ist verworfen.

> **⚠ Versionshinweis (2026-07-14 — v2):** Der Ablauf wurde auf Kundenwunsch
> umgestellt. **Topic „Prio-Filter" entfällt** (kein Produktionsstillstand,
> kein Transfer), und **jeder Transfer an einen Menschen** ist gestrichen. Die
> Priorisierung erfolgt nachgelagert **per KI im Power-Automate-Flow**
> (Zusammenfassung + Kritikalität → E-Mail-Routing an 4 Postfächer, siehe
> Architektur.md Abschnitt 4). Die vollständigen v1-Topics liegen im Ordner
> `Lösung mit Produktionsstillstand`. Angepasste Stellen unten sind mit
> „**v2:**" markiert; obsolete v1-Detailblöcke bleiben als Referenz stehen.

---

## Stufe 0 — Slim-Variante (aktive Topics, 2026-08-18)

> **Kontext:** Die Slim-Variante entfernt die Boolean-Fragen zu Inbetriebnahme
> und Wartungsvertrag und ersetzt sie durch eine neue freie Frage zur
> Anlagenbeschreibung (Bezeichnung + Seriennummer + Baujahr → `Global.Anlage`).
> Die Zusammenfassung bestätigt nur die 3 Kontaktfelder; Anlage und Anliegen
> gehen unbestätigt in den Flow „Anliegen weiterleiten – slim".
> Die PPTX-Bezeichnung für diese Variante ist **„Stufe 0"** (Folie 8, 18.08.2026).
> YAML-Quelle: `YAML/Slim/`.

### ⚠ Ist-Stand der Kette (2026-09-04, per Pull aus dem Studio verifiziert)

Die Beschreibungen der einzelnen Stufe-0-Topics weiter unten sind teilweise
älter. **Maßgeblich ist diese Kette:**

```
Conversation Start
  → Kundendaten erfassen      (3 Fragen, SetVariable KanalLabel)
  → Anrufgrund erfassen       (ClosedList, NEU — in den Topic-Texten unten noch nicht dokumentiert)
      ├ Störung / Wartung → Anlage erfassen → Anliegen erfassen
      └ sonst             → Anliegen erfassen
  → Zusammenfassung - slim    (Staging-Flow, Bestätigung, Korrekturschleife, Ticketerstellung)
  → Ende der Unterhaltung     (Safety-Net)
```

**Regel, die dabei gilt (hart erarbeitet, siehe ToDos.md „Reihenfolge-Anomalie"):**
Im Frageablauf steht **kein** `InvokeFlowAction`. Jeder Flow-Aufruf zwischen
den Fragen spaltet die Dialogausführung — die nachfolgende `ConditionGroup`
sieht die gerade beantwortete Global-Variable noch als leer und nimmt den
falschen Ast. Staging-Aufrufe existieren daher nur noch in
`Zusammenfassung - slim` und `Ende der Unterhaltung`.

Die aktuelle flowId für die Ticketerstellung ist
`834c8025-3da8-f111-b8dd-70a8a52f67fc` (Agent-Flow), **nicht** mehr
`019885f0-…` wie in den Abschnitten unten teils angegeben.

### Topic: Conversation Start — Stufe 0 (identisch mit Stufe 1)

Keine Änderungen gegenüber Stufe 1. Selber Begrüßungstext (beide Kanäle „transkribiert"),
selbes Redirect zu `Kundendatenerfassen`. Siehe Stufe-1-Dokumentation unten.

### Topic: Kundendaten erfassen — Stufe 0

Weitgehend identisch mit Stufe 1, mit folgenden **Abweichungen**:

1. Letzter Node redirectet zu `mosaiic_AIRCOTelefonBot.topic.Anrufgrunderfassen` (Stand 2026-09-04; früher `Anlagenerfassung`, davor `Inbetriebnahme`).
2. **Caller-ID-Block entfernt** (2026-08-20): Die automatische Rufnummern-Erkennung
   wurde nach Untersuchung aller verfügbaren Systemvariablen aufgegeben — keine
   Variable liefert die PSTN-Caller-ID im Teams Phone + Agents & Queues Kanal
   (Details siehe Stufe-1-Dokumentation Node 3 unten). Der Bot fragt die
   Telefonnummer **immer manuell** ab.
3. **`Global.KanalLabel` wird hardcodiert** auf `"Telefon"` gesetzt — kein
   Bedingungsblock. Wenn der Chat-Kanal in Zukunft hinzukommt, wird die
   Bedingung `Activity.ChannelId = "directline"` → `"Chat"` ergänzt.
4. **`Global.TelefonnummerGesprochen` neu (2026-08-25, getestet, ✅ erfolgreich)**:
   SetVariable-Node direkt nach der Telefonnummer-Frage berechnet eine
   Ziffern-mit-Leerzeichen-Variante der Nummer für die Sprachausgabe.
   **Root Cause**: Azure-Neural-TTS liest eine zusammenhängende Ziffernfolge ohne
   Formatierung standardmäßig als Kardinalzahl vor (z. B. „123456789" →
   „einhundertdreiundzwanzig Millionen..." statt einzelner Ziffern) — bestätigt
   durch Praxistest per echtem Anruf. Copilot Studios `speak`-Feld-Editor bietet
   nur die SSML-Tags Audio/Pause/Betonung/Prosodie, **kein** `say-as
   interpret-as="telephone"` → rohes SSML manuell einfügen ist keine Option.
   **Fix**: Power-Fx-Formel baut die gesprochene Variante mit Leerzeichen zwischen
   den Ziffern:
   ```
   Concat(Sequence(Len(Global.Telefonnummer)), Mid(Global.Telefonnummer, Value, 1), " ")
   ```
   `"123456789"` → `"1 2 3 4 5 6 7 8 9"`. Wird **nur** im `speak`-Feld der
   Zusammenfassung verwendet (siehe unten); `text`-Feld, Flow-Input und E-Mail
   bleiben bei der unformatierten `Global.Telefonnummer`.

**YAML-Stand (2026-09-02):**
```yaml
- Question → Global.Firmenname (StringPrebuiltEntity, allowBargeIn: false)
- Question → init:Global.Ansprechpartner (StringPrebuiltEntity, allowBargeIn: false)
- Question → Global.Telefonnummer (StringPrebuiltEntity, allowBargeIn: false)
- SetVariable: Global.KanalLabel = "Telefon"
- BeginDialog → Anrufgrunderfassen   # 2026-09-04; kein InvokeFlowAction mehr in diesem Topic
```
**Änderungen 2026-09-02:**
- `PersonNamePrebuiltEntity` → `StringPrebuiltEntity`: Entity übersetzte Namen ins Englische („Ich bin der Detlef" → „I am Detlef").
- `PhoneNumberPrebuiltEntity` → `StringPrebuiltEntity`: Entity lehnte ausländische/dialektale Formate ab → Telefonnummer blieb leer → Flow schlug fehl.
- `Global.TelefonnummerGesprochen` + SetVariable-Node **entfernt**: speak-Feld der Zusammenfassung nutzt jetzt `{Global.Telefonnummer}` direkt; TTS liest den Rohtext korrekt vor, da er bereits als gesprochen gespeichert ist.
- `allowBargeIn: false` in allen prompt-Feldern ergänzt.

### Topic: Anlage erfassen — Stufe 0 (NEU)

- **Trigger**: `OnRedirect` (von „Kundendaten erfassen")
- **Topic-ID im Dialog**: `mosaiic_AIRCOTelefonBot.topic.Anlagenerfassung`

- **Node 1 — Question Node** (`allowInterruption: true`, Entity: `StringPrebuiltEntity`, `sensitivityLevel: None`):
  
  > **text**: Bitte teilen Sie uns die Anlagenbezeichnung, Seriennummer und das Baujahr mit.
  > **speak**: Bitte teilen Sie uns die Anlagenbezeichnung, Seriennummer und das Baujahr mit.
  > → gebunden an `Global.Anlage` (kein `init:`-Präfix)
  
  - **Design-Entscheidung**: Alle drei Felder (Bezeichnung, Seriennummer, Baujahr) werden
    in einer einzigen Frage als Freitext erfasst. Die KI im Flow extrahiert die Einzelwerte.
    Kein separates Question Node je Feld — reduziert Gesprächsdauer.

- **Node 2 — BeginDialog**: → `mosaiic_AIRCOTelefonBot.topic.Anliegenerfassen`

### Topic: Anliegen erfassen — Stufe 0

Identisch mit Stufe 1, **einzige Änderung**: letzter Node redirectet zu
`mosaiic_AIRCOTelefonBot.topic.Zusammenfassung-Slim` (statt `Zusammenfassung`).

### Topic: Zusammenfassung & Bestätigung (Slim) — Stufe 0

- **Trigger**: `OnRedirect` (von „Anliegen erfassen")
- **Topic-ID im Dialog**: `mosaiic_AIRCOTelefonBot.topic.Zusammenfassung-Slim`
- **flowId**: `019885f0-e29a-f111-b8db-7ced8d476627` (Flow „Anliegen weiterleiten – slim")

**Kernunterschiede zur Stufe-1-Zusammenfassung:**

1. **Summary + Frage in einem Question Node kombiniert** (kein separater SendActivity für die Zusammenfassung — der Prompt-Text des Question Nodes enthält die Zusammenfassung):
   
   > **text**:
   > ```
   > Ich fasse Ihre Angaben zusammen:
   > - Firma: {Global.Firmenname}
   > - Ansprechpartner: {Global.Ansprechpartner}
   > - Telefonnummer: {Global.Telefonnummer}
   >
   > Ist das so korrekt?
   > ```
   > **speak**: Ich fasse Ihre Angaben zusammen. Sie rufen für die Firma {Global.Firmenname} an. Ihr Name lautet {Global.Ansprechpartner} und Sie sind unter {Global.Telefonnummer} erreichbar. Ist das so korrekt?
   > → gebunden an `init:Topic.korrekt` (BooleanPrebuiltEntity)
   
   - **Anlage und Anliegen werden nicht vorgelesen** — nur die 3 Kontaktfelder werden bestätigt.
   - **Kein Node 0b / keine 4-Varianten-Condition** — da Inbetriebnahme und Vertrag entfallen.
   - **`{Global.Telefonnummer}` direkt im `speak`-Feld** (2026-09-02 geändert): Da die Telefonnummer jetzt als `StringPrebuiltEntity` (Rohtext, so wie gesprochen) gespeichert wird, liest TTS sie korrekt vor — `{Global.TelefonnummerGesprochen}` ist damit hinfällig und wurde entfernt.

2. **Korrekturfeld-Entity auf 3 Optionen reduziert** (Inbetriebnahme + Vertrag entfernt):
   
   > Welche Angabe war nicht korrekt? Der Firmenname, der Ansprechpartnername oder die Telefonnummer?
   > → `init:Topic.Korrekturfeld` (ClosedListEntityReference: `mosaiic_AIRCOTelefonBot.entity.Korrekturfeld`)
   
   | Entity-Wert | Condition-Key |
   |-------------|---------------|
   | Firmenname | `KJwQ9x` |
   | Ansprechpartner | `md1vGC` |
   | Telefonnummer | `swLyyJ` |
   
   Gleiche Entity (`Korrekturfeld`) wie Stufe 1 — Werte YvfEJQ (Vertrag) und gjJ4ZJ (Inbetriebnahme)
   werden schlicht nicht mehr abgefragt. Entity selbst muss nicht geändert werden.

3. **Fallback bei unerkannter Angabe**: Vereinfacht gegenüber Stufe 1 —
   
   - `SetVariable: Topic.Korrekturversuche = Topic.Korrekturversuche + 1` (Zähler wird inkrementiert — korrigiert 2026-08-18)
   - SendActivity: „Das habe ich leider nicht verstanden." (text + speak identisch)
   - **GotoAction → `question_ODxM04`** (zurück zur Korrekturfeld-Frage, nicht zur Re-Zusammenfassung)
   
   Nach einer erfolgreichen Korrektur: **GotoAction → `question_Qt7GUF`** (der kombinierte
   Summary+Question-Node), der dann die aktualisierte Zusammenfassung vorliest.

4. **Kein separater Re-Zusammenfassung-Node**: Da die Zusammenfassung Teil des Question Nodes
   `question_Qt7GUF` ist, wird durch `GotoAction → question_Qt7GUF` automatisch die
   Zusammenfassung + Frage erneut abgespielt — ohne eigene SendActivity.

**Variablen dieses Topics:**

- `Topic.Korrekturversuche` (Zahl, initialisiert mit `0` am Anfang)
- `Topic.korrekt` (Boolesch, `init:Topic.korrekt`)
- `Topic.Korrekturfeld` (Closed-List, `init:Topic.Korrekturfeld`)

**Flow-Bindings (beide InvokeFlowAction-Nodes — Bestätigung und Fallback nach 3 Versuchen):**

```yaml
text:   =If(IsBlank(Global.Firmenname), "nicht angegeben", Global.Firmenname)
text_1: =If(IsBlank(Global.Ansprechpartner), "nicht angegeben", Global.Ansprechpartner)
text_2: =If(IsBlank(Global.Telefonnummer), "nicht angegeben", Global.Telefonnummer)
text_3: =Global.Anliegen
text_4: =Global.KanalLabel
text_5: =Global.Anlage
```
**Änderung 2026-09-02:** Null-Checks für die 3 Kontaktfelder — verhindert Flow-Fehler wenn eine Variable leer geblieben ist (z.B. weil Anrufer die Telefonnummer nicht nennen konnte).

**Abschluss-Verhalten nach 3 erschöpften Korrekturversuchen** (identisch mit Stufe 1):
SendActivity + InvokeFlowAction + BeginDialog → EndofConversation.

---

## Stufe 1 — aktive Topics

### Topic: Conversation Start (System Topic) — Stufe 1

- **Trigger**: `OnConversationStart` (automatisch, keine Trigger-Phrasen)

- **Node 1 — Message Node** (text und speak identisch — im YAML geprüft 2026-08-10):
  
  - **text + speak** (beide Kanäle): „Willkommen beim technischen Service der Firma AIRCO Systems GmbH. Sie sprechen mit unserem digitalen Service-Assistenten. Zur Bearbeitung Ihres Anliegens wird dieses Gespräch **transkribiert** und verarbeitet."
  
  - **Hinweis**: Die ursprünglich geplante Unterscheidung (text = „gespeichert",
    speak = „transkribiert") wurde in der Implementierung nicht umgesetzt —
    beide Felder verwenden „transkribiert". Das ist für den Telefon-Hauptkanal
    korrekt; ein reiner Chat-Textkanal ist in Stufe 1 kein Ziel.
  
  - **Begründung**: Offenlegung, dass ein KI-Bot und kein Mensch am Telefon
    ist — Transparenz gegenüber dem Anrufer statt erst bei Nachfrage. Der
    DSGVO-Hinweis ist nötig, da das Gespräch bot-seitig verarbeitet und das
    Anliegen per E-Mail weitergeleitet wird (entschieden; unabhängig davon,
    dass die KI-Klassifizierung selbst in Stufe 1 nicht stattfindet, siehe
    Architektur.md Abschnitt 1). Exakte rechtliche Formulierung sollte vor
    Produktivbetrieb noch mit Datenschutz/Rechtsabteilung gegengeprüft
    werden.

- **Node 2 — Redirect to Topic**: → „Kundendaten erfassen"

- **Notausstieg — v2 angepasst**: In v2 gibt es **keinen Transfer an einen
  Menschen** mehr, also auch keinen „Mitarbeiter sprechen"-Eskalationsausgang.
  Sagt der Anrufer dennoch, er wolle einen Mitarbeiter, weist der Bot freundlich
  darauf hin, dass er das Anliegen aufnimmt und sich jemand meldet (genaue
  Formulierung offen, siehe Architektur.md Abschnitt 9, Punkt 18). Bei Stille
  auf eine Frage gilt die allgemeine Fehlversuchs-Regel (E4): erster
  Fehlversuch/Stille → Frage wiederholen (zweite Chance), zweiter
  Fehlversuch/Stille → **kein Transfer** mehr, sondern Entschuldigung + (falls
  Daten vorliegen) weiter zum Flow bzw. Gesprächsende (Default, offen).

### Topic: Kundendaten erfassen — Stufe 1

- **Node 1 — Question Node** (`allowInterruption: true` — im YAML geprüft 2026-08-10):
  
  > **text**: Für welche Firma kontaktieren Sie uns?
  > **speak**: Für welche Firma rufen Sie an?
  > → gebunden an `Global.Firmenname` (kein `init:`-Präfix); Entity: `StringPrebuiltEntity` (`sensitivityLevel: None`)
  
  - **Praxistest erledigt (2026-07-16)**: `OrganizationPrebuiltEntity` erkennt
    Firmennamen ohne Suffix (GmbH, AG etc.) nicht — Bot wiederholt die Frage.
    **→ Entity auf `StringPrebuiltEntity` („Gesamte Antwort des Benutzers")
    geändert.** Nimmt jede Eingabe verbatim entgegen.
  - **Hinweis**: Im YAML ist `allowInterruption: true` implementiert (nicht `false`
    wie ursprünglich geplant). Da Unterbrechungen im Telefonkanal ohnehin selten
    sind und ein zu strenges `false` Randfälle erzeugen kann, wird der
    implementierte Wert beibehalten.

- **Node 2 — Question Node** (separat von Node 1, bewusst NICHT kombiniert,
  `allowInterruption: true`):
  
  > Wie ist Ihr Name?
  > → gebunden an `init:Global.Ansprechpartner`; Entity: `StringPrebuiltEntity` (direkte Entity-Referenz, kein `sensitivityLevel`)
  
  - **Begründung Trennung**: Getrennte Fragen vermeiden Slot-Disambiguierung bei
    Freitext — Risiko bei kombinierter Frage: „Mein Name ist Michael Doukas aus
    der Firma Michael Doukas GmbH" wäre textlich fast identisch.
  - **Praxistest erledigt (2026-07-21):** `PersonNamePrebuiltEntity` scheiterte
    an natürlicher Sprache („ich bin der Michael Doukas") → auf `StringPrebuiltEntity`
    umgestellt. Bereinigung (Rohtext → sauberer Name) übernimmt der KI-Node im Flow.

- **Node 3 — Telefonnummer (2026-08-20 vereinfacht)**: Bot fragt die
  Rufnummer **immer manuell** ab — kein Caller-ID-Erkennungsblock mehr.

  **Hintergrund — Untersuchung Caller-ID (2026-08-20, abgeschlossen):**
  Im Teams Phone + Agents & Queues Kanal liefert **keine** verfügbare
  Copilot-Studio-Systemvariable die PSTN-Rufnummer des Anrufers:
  
  | Variable | Befund |
  |---|---|
  | `Activity.From.Id` | Kryptische interne Teams-Objekt-ID (kein `tel:`-Format) |
  | `Activity.From.Name` | Leer bei PSTN-Anrufern |
  | `Activity.Recipient.Name` | Laut MS-Doku = Agent-Rufnummer im Telefonie-Kontext; in der Praxis ebenfalls leer |
  | `Activity.ChannelId/Channel/ChannelData` | Kanalidentifier, keine Rufnummer |
  | Telephonie-Variablen (`Activity.InputDTMFKeys` etc.) | Nur für Eingaben während des Gesprächs, nicht für Caller-ID |
  
  **Fazit**: Die PSTN-Caller-ID ist in Copilot Studio bei Teams Phone +
  Agents & Queues grundsätzlich nicht zugänglich. Kein Workaround bekannt.

  **Aktuelle Implementierung (Node 3):**
  
  > Unter welcher Nummer können wir Sie erreichen?
  > → gebunden an `Global.Telefonnummer`; Entity: `PhoneNumberPrebuiltEntity`
  
  - **Assign danach**: `Global.KanalLabel` = `"Telefon"` (hardcodiert,
    kein Bedingungsblock — Bot läuft ausschließlich über Telefon)

  **KanalLabel-Erkennung für zukünftigen Chat-Kanal:**
  Wenn ein Web-Chat-Widget hinzukommt, `Activity.ChannelId` prüfen:
  - `"directline"` → Chat → `KanalLabel = "Chat"` + Telefonnummer fragen
  - Alles andere → Telefon → `KanalLabel = "Telefon"`
  Teams-Chat würde ebenfalls `"msteams"` liefern wie der Telefonkanal —
  dann ist eine andere Unterscheidungsmethode nötig.

- **Node 4 — Redirect to Topic**: → „Inbetriebnahme"

### Topic: Inbetriebnahme — Stufe 1 (neu, Gerüst)

- **Zweck**: Erfassen, ob die Anlage in den letzten 12 Monaten in Betrieb
  genommen wurde — als **Selbstauskunft** per Ja/Nein-Frage. Die Info wird
  nur an den Flow weitergeleitet und **ändert nicht** den Gesprächsverlauf,
  analog zu `Global.VertragVorhanden` und `Global.Produktionsstillstand`.

- **Node 1 — Question Node** (Boolean/Ja-Nein):
  
  > Wurde die Anlage in den letzten 12 Monaten in Betrieb genommen?
  > → gebunden an `Global.Inbetriebnahme`

- **Node 2 — Redirect to Topic**: → „Vertragsfrage"

- **Offene Fragen**: Umgang mit „weiß nicht" (dritter Wert oder als Nein
  behandeln?) — analog zur gleichen offenen Frage bei „Vertragsfrage".

### Topic: Vertragsfrage — Stufe 1 (neu, Gerüst)

- **Zweck**: Erfassen, ob ein Wartungsvertrag besteht — als **Selbstauskunft**
  per Ja/Nein-Frage. Die Info steuert Routing/Priorisierung im Flow und wird
  in der E-Mail ausgewiesen; sie ändert **nicht** den Gesprächsverlauf.
  Verifikation übernimmt der Innendienst beim Bearbeiten.

- **Node 1 — Question Node** (Boolean/Ja-Nein, `allowInterruption: false`):
  
  > Haben Sie einen Wartungsvertrag mit Garantieverlängerung mit uns?
  > → gebunden an `Global.VertragVorhanden`

- **Node 2 — Redirect to Topic**: → „Anliegen erfassen" (**v2:** vorher
  „Prio-Filter", der entfällt)

- **Offene Fragen**: Umgang mit „weiß nicht" (dritter Wert oder als
  Nein behandeln?). Ab Stufe 1.5/1.6: Frage durch automatischen Abgleich
  (SharePoint-Liste / Odoo-Kundenstamm) ersetzen.

### Topic: Prio-Filter — ⚠ ENTFÄLLT IN v2 (2026-07-14)

> **v2:** Dieses Topic wird nicht mehr verwendet (kein Produktionsstillstand-
> Filter, kein Transfer). Die Priorisierung übernimmt die KI im Flow
> (Architektur.md Abschnitt 4); im v2-Ablauf springt „Vertragsfrage" direkt zu
> „Anliegen erfassen". Der folgende Abschnitt bleibt als **v1-Referenz** (auch
> im Backup-Ordner `Lösung mit Produktionsstillstand`) und ist für den v2-Bau
> irrelevant.

#### v1-Referenz (obsolet)

- **Zweck**: Produktionsstillstände herausfiltern und zum Mitarbeiter
  transferieren. **Reihenfolge bewusst geändert (2026-07-10)**: Prio-Filter
  läuft jetzt nach `Inbetriebnahme` und `Vertragsfrage`, nicht mehr direkt
  nach „Kundendaten erfassen". Ein Anrufer mit Produktionsstillstand
  durchläuft also zwei kurze Ja/Nein-Fragen, bevor transferiert wird — bewusst
  in Kauf genommen (Michael, 2026-07-10), im Gegensatz zur ursprünglichen
  Regel „sofort herausfiltern, bevor weitere Daten erfragt werden". Weiterhin
  gilt: kritische Anrufe warten **nicht** auf die Freitext-Anliegenerfassung.

- **Node 1 — Question Node** (Boolean/Ja-Nein):
  
  > Steht Ihre Produktion aktuell still?
  > → gebunden an `Global.Produktionsstillstand`

- **Node 2 — Condition Node** (Produktionsstillstand):
  
  - **Ja** → **Node 2b — Condition Node „Geschäftszeit?"** (Power Fx):
    `Weekday(Now(), StartOfWeek.Monday) <= 5 && Hour(Now()) >= 8 && Hour(Now()) < 17`
    
    - **Wahr** (Mo–Fr 8–17 Uhr) → „Unterhaltung übertragen" (Node-Typ in
      Copilot Studio, nicht „Transfer to Agent" — das war nur unser
      Arbeitsbegriff; Transfer type = „Externe Telefonnummer-Übertragung")
      **sofort + parallel** Action Node → Flow „Anliegen weiterleiten" mit
      [KRITISCH]-Kennzeichnung.
      **Zielrufnummer: noch offen** — keines der Quelldokumente (PDF/beide
      .docx) enthält echte Telefonnummern, nur Platzhalter „+49...". Michael
      muss bei AIRCO klären, wer Anrufe entgegennehmen soll und dessen
      Durchwahl (Kandidat laut Prozessdokumentation: Robert Wallner, dort als
      Eskalationskontakt für Service-Innendienst-Probleme genannt — aber
      nicht ausdrücklich für Telefon-Eskalationen bestätigt). Bis dahin
      Platzhalter-Nummer eintragen.
    
    - **Falsch** (außerhalb Geschäftszeit) → **kein** Transfer-Versuch;
      stattdessen Message Node:
      
      > Sie rufen außerhalb unserer Geschäftszeiten (Montag bis Freitag,
      > 8 bis 17 Uhr) an. Bitte schildern Sie trotzdem kurz Ihr Anliegen —
      > wir melden uns schnellstmöglich bei Ihnen.
      
      danach **Redirect to Topic → „Anliegen erfassen"** (normaler Flow
      weiter: → Zusammenfassung & Bestätigung → Flow „Anliegen
      weiterleiten"). Der Flow erkennt `Global.Produktionsstillstand = true`
      und routet automatisch als [KRITISCH] — kein separater Flow-Aufruf
      im Prio-Filter nötig. Das Anliegen steht damit vollständig in der
      [KRITISCH]-E-Mail. **(Geändert 2026-07-13: vorher Gesprächsende ohne
      Anliegenerfassung — Fehler, weil E-Mail ohne Anliegen kaum
      aussagekräftig.)**
  
  - **Nein** → Redirect to Topic: → „Anliegen erfassen"
  
  - **E-Mail-Inhalt der [KRITISCH]-Mail (aktualisiert, 2026-07-13)**:
    Kundendaten (`Global.Firmenname`, `Global.Ansprechpartner`,
    `Global.Telefonnummer`) + `Global.Inbetriebnahme` + `Global.VertragVorhanden`
    + `Global.Anliegen` (alle Werte vollständig — kein Marker mehr nötig,
    da auch außerhalb der Geschäftszeiten das Anliegen erfasst wird).
    Bei Transfer innerhalb Geschäftszeiten: `Global.Anliegen` ist leer /
    nicht erfasst — Flow setzt dort den Marker „Anrufer direkt
    transferiert, Anliegen nicht erfasst."
  
  - **Offene Frage für später**: Soll dieselbe Geschäftszeiten-Prüfung auch
    bei den anderen beiden Eskalationsauslösern gelten (Sicherheitsproblem,
    2× unerkannte Eingabe) — noch nicht entschieden, siehe
    [ToDos.md](ToDos.md).

### Topic: Anliegen erfassen — Stufe 1 (neu, Gerüst)

- **Zweck**: Das Anliegen als **Freitext** aufnehmen — keine Kategoriewahl,
  keine Pflichtfelder im Bot. **v2:** Der Freitext wird an den Flow übergeben,
  wo eine **KI** ihn gegen die Störungsliste prüft, intelligent zusammenfasst
  und eine Kritikalität ableitet (siehe Architektur.md Abschnitt 4). Der Bot
  selbst klassifiziert nichts; die Kategorie-Erkennung A–F bleibt
  Stufe-2-Material.

- **Node 1 — Question Node** (`allowInterruption: true`, Entity: `StringPrebuiltEntity`):
  
  > Bitte beschreiben Sie nun Ihr Anliegen.
  > → gebunden an `init:Global.Anliegen`

- **Node 2 — Redirect to Topic**: → „Zusammenfassung & Bestätigung"

- **Offene Fragen**: Exakter Wortlaut der offenen Frage (soll sie Beispiele
  nennen: „z. B. eine Störung, eine Ersatzteilanfrage oder ein Rückrufwunsch"?);
  Umgang mit sehr langen Monologen (Nachfassfrage? Bestätigungsschleife?).
  Eigene Sparring-Session erforderlich.

### Topic: Sicherheits-Eskalation — Stufe 1 (zurückgestellt, 2026-07-10)

- **Status**: **Zurückgestellt** (Michael, 2026-07-10) — bewusst nach hinten
  priorisiert zugunsten des Power Automate Flows. Bewusst in Kauf genommenes
  Risiko bis dahin: Ein Sicherheitsnotfall (Brand, Rauch) wird vom Bot
  vorerst wie jeder andere Anruf behandelt, keine Sofort-Eskalation.
- **Zweck (wenn gebaut) — v2**: In v2 gibt es **keinen Transfer**. Ein
  Sicherheitsnotfall (Brand, Rauch) kann daher nicht mehr an einen Menschen
  übergeben werden; vorläufiger Default ist ein dringlicher Hinweis an den
  Anrufer + höchste Kritikalität in der E-Mail (offen, siehe Architektur.md
  Abschnitt 9, Punkt 17). In Copilot Studio wäre das weiterhin ein eigenes
  Topic mit Trigger-Phrasen („es brennt", „Rauch", „Notfall" o. ä.), das
  **jedes** andere Topic unterbrechen darf (globale Trigger-Priorität).
- **Offene Fragen**: Welche Trigger-Phrasen genau? Und wie reagiert der Bot
  ohne Transfer sinnvoll auf einen echten Notfall (nur dringlicher Hinweis +
  höchste Kritikalität, oder zusätzliche Sofortmaßnahme)? Eigene
  Sparring-Session erforderlich.

### Topic: Zusammenfassung & Bestätigung — Stufe 1 (entschieden)

- **Zweck**: Gemeinsamer Schritt direkt vor dem Action Node. Liest die
  gesammelten Werte vor und lässt den Anrufer bestätigen oder korrigieren.

- **Stufe-1-Kurzfassung — erweitert (2026-07-10, Michael)**: Node 1 liest jetzt
  zusätzlich zu Kundendaten (`Global.Firmenname`, `Global.Ansprechpartner`,
  `Global.Telefonnummer`) + `Global.Anliegen` auch `Global.Inbetriebnahme` und
  `Global.VertragVorhanden` vor — über 4 bedingte Nachrichtenvarianten
  (Inbetriebnahme × Vertrag, je Ja/Nein; Wortlaut siehe Node 1 unten). Da alle
  Werte Global Variables sind, entfällt weiterhin das Variablen-Sichtbarkeitsproblem.

- **Speak-Feld (Sprachausgabe) vs. Text-Feld (2026-07-15)**: Die `speak`-Variante
  der 4 Nachrichtenvarianten enthält **kein `{Global.Anliegen}`** — bewusste
  Kürzung für die Telefonsprachausgabe, da das Anliegen beliebig lang sein kann
  und per Stimme schwer verständlich ist. Das `text`-Feld (für Textkanäle) liest
  das Anliegen weiterhin mit vor.

- **Variablen dieses Topics** (Topic Variables, nur lokal für die
  Bestätigungs-/Korrekturlogik gebraucht — analog `Topic.richtigeTelefonnummer`
  in „Kundendaten erfassen", siehe Architektur.md Abschnitt 6):
  
  - `Topic.korrekt` (Boolesch — **Achtung**: im YAML `Topic.korrekt`, nicht `Topic.Bestaetigt` wie ursprünglich geplant; im YAML geprüft 2026-08-10)
  - `Topic.Korrekturfeld2` (im YAML: `Topic.Korrekturfeld`; Closed-List-Entity: Firmenname, Ansprechpartner,
    Telefonnummer, **Inbetriebnahme**, **Vertrag** — beide neu 2026-07-10;
    **„Anliegen" ist kein Korrekturfeld** — im YAML nicht vorhanden)
  - `Topic.Korrekturversuche` (Zahl, Startwert 0)

- **Node 0 — Variable festlegen** (ganz am Anfang des Topics, vor Node 1):
  `Topic.Korrekturversuche` = `0` (Literalwert, keine Formel).
  
  - **Begründung**: Copilot Studio leitet den Variablentyp aus der
    Verwendung ab, nicht aus einer manuellen Auswahl. Die spätere
    Increment-Formel in Node 4 (`Topic.Korrekturversuche + 1`) referenziert
    die Variable auf sich selbst — ohne vorherige Zuweisung eines
    Literalwerts bleibt der Typ „unknown" und die Formel lässt sich nicht
    anlegen. Getestet 2026-07-09: Typ-Panel zeigte „unknown", solange nur
    der selbstreferenzierende Node existierte.

- **Node 0b — Condition Node** (direkt nach Node 0, vor Node 1): prüft alle 4
  Kombinationen aus `Global.Inbetriebnahme` × `Global.VertragVorhanden`
  (je Ja/Nein) und führt zur passenden Node-1-Nachrichtenvariante:

- **Node 1 — Message Node** (4 Varianten, je nach Node-0b-Zweig):
  
  - **Inbetriebnahme = Ja, VertragVorhanden = Ja**:
    
    > Ich fasse Ihre Angaben zusammen: Firma {Global.Firmenname}, Ansprechpartner {Global.Ansprechpartner}, erreichbar unter {Global.Telefonnummer}. Die Inbetriebnahme erfolgte in den letzten 12 Monaten und ein gültiger Wartungsvertrag liegt vor. Ihr Anliegen: {Global.Anliegen}.
  
  - **Inbetriebnahme = Ja, VertragVorhanden = Nein**:
    
    > Ich fasse Ihre Angaben zusammen: Firma {Global.Firmenname}, Ansprechpartner {Global.Ansprechpartner}, erreichbar unter {Global.Telefonnummer}. Die Inbetriebnahme erfolgte in den letzten 12 Monaten, ein Wartungsvertrag liegt jedoch nicht vor. Ihr Anliegen: {Global.Anliegen}.
  
  - **Inbetriebnahme = Nein, VertragVorhanden = Ja**:
    
    > Ich fasse Ihre Angaben zusammen: Firma {Global.Firmenname}, Ansprechpartner {Global.Ansprechpartner}, erreichbar unter {Global.Telefonnummer}. Die Inbetriebnahme liegt länger als 12 Monate zurück, ein gültiger Wartungsvertrag liegt vor. Ihr Anliegen: {Global.Anliegen}.
  
  - **Inbetriebnahme = Nein, VertragVorhanden = Nein**:
    
    > Ich fasse Ihre Angaben zusammen: Firma {Global.Firmenname}, Ansprechpartner {Global.Ansprechpartner}, erreichbar unter {Global.Telefonnummer}. Die Inbetriebnahme liegt länger als 12 Monate zurück und ein Wartungsvertrag liegt nicht vor. Ihr Anliegen: {Global.Anliegen}.
  
  Alle vier Varianten münden anschließend in **Node 2** (Bestätigungsfrage).

- **Node 2 — Question Node** (Boolean/Ja-Nein) → `init:Topic.korrekt` (im YAML geprüft; entity: `BooleanPrebuiltEntity`):
  
  > Ist das so korrekt?

- **Node 3 — Condition Node** (`Topic.korrekt`):
  
  - **Ja** → Node 8 (Action Node)
  - **Nein** → Node 4

- **Node 4 — Variable festlegen**: `Topic.Korrekturversuche` =
  `Topic.Korrekturversuche + 1`

- **Node 5 — Condition Node** (`Topic.Korrekturversuche`):
  
  - **≤ 3** → Node 6
  - **> 3** (vierter Widerspruch) → Node 9 (**v2:** Abschluss ohne Transfer)
  
  (Schwelle am 2026-07-20 von 2 auf 3 erhöht — Bedingung im YAML
  `Topic.Korrekturversuche <= 3`; erlaubt jetzt bis zu 3 Korrekturversuche,
  Abschluss beim 4. „Nein".)

- **Node 6 — Question Node** (Closed-List-Entity `Korrekturfeld2`):
  
  > Welche Angabe war nicht korrekt: Firmenname, Ansprechpartner,
  > Telefonnummer, die Angabe zur Inbetriebnahme oder zum Wartungsvertrag?
  > → gebunden an `Topic.Korrekturfeld2`
  
  - **Synonym-Tabelle für die `Korrekturfeld`-Entity (abgestimmt 2026-07-20)** —
    Closed-List, Match per **Teilstring**; Synonyme über die Einträge hinweg
    **überschneidungsfrei** halten (kein bloßes „Name" bei Ansprechpartner, da
    in „Firmen**name**" enthalten). Der Eintragsname selbst matcht ohnehin mit.
    
    | Eintrag (Entity-Wert) | Synonyme |
    |---|---|
    | **Firmenname** | Firma, die Firma, der Firmenname, Unternehmen, das Unternehmen, Betrieb, Firmenbezeichnung, wir heißen, die Firma heißt |
    | **Ansprechpartner** | ich heiße, wie ich heiße, mein Name, mein Name ist, der Name, der Ansprechpartner, Ansprechperson, Kontaktperson, der Kontakt |
    | **Telefonnummer** | *von Michael direkt in Copilot Studio gepflegt (2026-07-20)* — Vorsicht: „Nummer" steckt in „Seriennummer" |
    | **Inbetriebnahme** | die Inbetriebnahme, in Betrieb genommen, in Betrieb, Inbetriebnahmedatum, installiert, aufgestellt, die Installation, die zwölf Monate, die 12 Monate, das Datum |
    | **Wartungsvertrag** (Entity-Wert „Vertrag") | Wartungsvertrag, der Vertrag, Vertrag, Servicevertrag, die Wartung, Wartung, der Service, Wartungsvereinbarung |
    
    **Nicht verwenden** (zu generisch / Kollision): bloßes „Name" (→ Firmenname),
    bloßes „wann" oder „Datum" ohne Kontext bei mehreren Datumsfeldern. Nach dem
    Pflegen mit Sprech-Floskeln testen („nein, mein Name stimmt nicht", „die
    Firma ist falsch", „die Inbetriebnahme"), nicht nur mit Einzelwörtern.

- **Node 7 — Condition Node / Switch** (`Topic.Korrekturfeld2`), jeweils
  gefolgt von Redirect zurück zu **Node 1** (Zusammenfassung erneut vorlesen
  und bestätigen lassen):
  
  - **Firmenname** → Question Node (`StringPrebuiltEntity`): „Wie lautet der richtige Firmenname?" →
    `Global.Firmenname`
  
  - **Ansprechpartner** → Question Node (`StringPrebuiltEntity`): „Wie ist Ihr richtiger Name?" →
    `Global.Ansprechpartner`
  
  - **Telefonnummer** → Question Node (`StringPrebuiltEntity`): „Wie lautet die richtige
    Telefonnummer?" → `Global.Telefonnummer`
  
  - **⚠ Wichtig — Question wird sonst übersprungen (Fix 2026-07-20)**: Diese
    drei Korrektur-Fragen schreiben in **bereits gefüllte Global-Variablen**
    (Firmenname/Ansprechpartner/Telefonnummer aus „Kundendaten erfassen").
    Copilot Studio **überspringt** eine Question, deren Zielvariable schon einen
    Wert hat → die Frage wurde im Test nicht gestellt, der alte Wert blieb stehen
    (Symptom: nach „Firma" als Korrekturfeld sprang der Bot direkt zur
    Re-Zusammenfassung, Firmenname unverändert). **Fix**: **vor** jeder der drei
    Fragen ein „Variablenwert festlegen" einfügen, das die jeweilige Variable auf
    `Blank()` setzt (Code-View-Alternative: `variable: init:Global.…`). Betrifft
    **nur** die 3 Text-Fragen — die Inbetriebnahme-/Vertrag-Zweige sind
    Set-Variable-Nodes (`Not(...)`) und laufen ohnehin immer.
    **✅ Fix umgesetzt, getestet & bestätigt 2026-07-20** (alle drei Zweige
    stellen die Korrektur-Frage jetzt korrekt).
  
  - **Inbetriebnahme** (neu, 2026-07-10) → **Variable festlegen** (Assign,
    kein Question Node, keine Rückfrage): `Global.Inbetriebnahme` =
    `Not(Global.Inbetriebnahme)`
  
  - **Vertrag** (neu, 2026-07-10) → **Variable festlegen** (Assign, kein
    Question Node, keine Rückfrage): `Global.VertragVorhanden` =
    `Not(Global.VertragVorhanden)`
  
  - **Begründung gegen erneute Ja/Nein-Rückfrage**: Beide Felder sind echte
    Booleans mit nur 2 möglichen Zuständen. Sagt der Anrufer „das war
    falsch", ist der korrekte Wert zwangsläufig das jeweilige Gegenteil —
    eine erneute Sprachfrage wäre unnötig und würde nur das
    Boolean-Erkennungsrisiko der Kontrastformulierung („doch"/„oder nicht")
    reproduzieren, das wir zuvor verworfen haben. Bei den anderen 3 Zweigen
    (Firmenname, Ansprechpartner, Telefonnummer) bleibt die
    Rückfrage nötig, da dort beliebig viele Werte möglich sind.
    **„Anliegen" ist kein Korrekturfeld** in v2 — der Anrufer kann das
    Anliegen in der Bestätigungsphase nicht korrigieren (kein
    entsprechender YAML-Zweig vorhanden).
  
  - **Re-Zusammenfassung nach Korrektur**: Im Gegensatz zur initialen
    Zusammenfassung (4 Varianten per Condition Node) verwendet die
    Korrekturschleife eine **einzelne SendActivity** mit `If()`-Formeln:
    
    > Ich fasse Ihre Angaben zusammen:
    > - Firma: {Global.Firmenname}
    > - Ansprechpartner: {Global.Ansprechpartner}
    > - erreichbar unter: {Global.Telefonnummer}
    > - Inbetriebnahme in den letzten 12 Monaten? {If(Global.Inbetriebnahme, "ja", "nein")}
    > - Wartungsvertrag vorhanden? {If(Global.VertragVorhanden, "ja", "nein")}
    > - Ihr Anliegen: {Global.Anliegen}
    
    Danach **GotoAction → `question_Qt7GUF`** (= „Ist das so korrekt?", tatsächliche
    actionId im YAML — kein Redirect zu Node 1, da die Bestätigungsfrage direkt
    wiederverwendet wird.
    
    **Stand YAML (2026-08-10 geprüft):** Die Re-Zusammenfassung hat **nur ein
    `speak`-Feld**, kein `text`-Feld — die dokumentierte Ergänzung vom 2026-07-20
    ist im aktuellen YAML nicht vorhanden. Für den Chat-Test ist die
    Re-Zusammenfassung daher unsichtbar (nur Sprachausgabe). Offen: text-Feld
    nachträglich ergänzen, falls Chat-Testbarkeit benötigt wird.
  
  - **Begründung gegen „Redirect to Topic"**: Ein Redirect zu „Kundendaten
    erfassen" würde am Anfang dieses Topics starten und dadurch alle drei
    Felder erneut abfragen statt nur des einen falschen. Da alle Zielwerte
    Global Variables sind, kann die Korrektur-Frage stattdessen direkt hier
    im Topic gestellt werden, ohne das Ursprungs-Topic erneut zu durchlaufen.

- **Node 7b — Fallback bei nicht erkannter Angabe** (neu 2026-07-20):
  `elseActions` des Korrekturfeld-Switch (`conditionGroup_7MJ9bT`). Erkennt die
  Closed-List-Entity die genannte Angabe **nicht**, bleibt `Topic.Korrekturfeld`
  leer und bisher greift **kein** Zweig → stiller Durchfall (Bot las kommentarlos
  dieselbe Zusammenfassung erneut vor). Neu: eine SendActivity („Das habe ich
  leider nicht verstanden … Ich fasse Ihre Angaben noch einmal zusammen."),
  danach läuft der **bestehende** Weg weiter (Re-Zusammenfassung → „Ist das so
  korrekt?"). Dadurch ist die Schleife über `Topic.Korrekturversuche`
  **begrenzt** — kein neuer Zähler nötig; nach dem 4. „Nein" schließt Node 9 ab.

  - **Tatsächlicher Fallback-Text im YAML (2026-08-10 geprüft)**:
    - text: „Das habe ich leider nicht verstanden. Sie können Firmenname, Ansprechpartner, Telefonnummer, Inbetriebnahme oder Wartungsvertrag eintippen. Ich fasse Ihre Angaben noch einmal zusammen."
    - speak: „Das habe ich leider nicht verstanden. Sie können Firmenname, Ansprechpartner, Telefonnummer, die Angabe zur Inbetriebnahme oder zum Wartungsvertrag sagen. Ich fasse Ihre Angaben noch einmal zusammen."
  
  - **Trade-off (bewusst)**: Der Anrufer muss nach der Rückfrage erneut „Nein"
    sagen, um wieder zur Feldauswahl zu gelangen. Alternative (nicht umgesetzt):
    direkter `GotoAction` zurück zur Feldfrage mit eigenem Zähler
    `Topic.Feldversuche`, um das zusätzliche „Nein" zu sparen — dafür mehr
    Komplexität.

- **Node 8 — Action Node**: Message Node (Verabschiedung) → Flow „Anliegen weiterleiten (KI)" aufrufen
  → System-Topic „Ende der Konversation". Reihenfolge im YAML:
  1. SendActivity:
  
  > Vielen Dank, ich habe Ihr Anliegen weitergeleitet. Ein Mitarbeiter wird sich zeitnah bei Ihnen melden. Auf Wiederhören.
  
  2. `InvokeFlowAction` (flowId: `299a805a-9036-df8d-ed1d-ec8dbc0fcd99`) mit Bindings:
     `boolean`=VertragVorhanden, `boolean_1`=Inbetriebnahme, `text`=Firmenname,
     `text_1`=Ansprechpartner, `text_2`=Telefonnummer, `text_3`=Anliegen, `text_4`=KanalLabel
  3. `BeginDialog → EndofConversation` — löst bei Telefonie die kanalspezifische
     Aktion aus, die den Anruf tatsächlich beendet (nicht „Alle Themen beenden").
     Ohne diesen Schritt bleibt die Verbindung nach der letzten Nachricht offen.

- **Node 9 — Abschluss ohne Bestätigung** (nach dem dritten Widerspruch).
  **v2: kein Transfer mehr** ✅ **erledigt (2026-07-15)**. Ablauf im YAML
  (SendActivity **vor** Flow, nicht danach):
  1. SendActivity:
  
  > Ihre Angaben konnten leider nicht abschließend bestätigt werden. Ich habe Ihr Anliegen weitergeleitet. Ein Mitarbeiter wird sich zeitnah bei Ihnen melden. Auf Wiederhören!
  
  2. `InvokeFlowAction` (gleiche Bindings wie Node 8)
  3. `BeginDialog → EndofConversation`
  Der v1-Block (Geschäftszeiten-Prüfung + `TransferConversationV2`) wurde
  entfernt. **Kein Marker** „nicht bestätigt" an den Flow übergeben — die
  KI wertet den Freitext ohne diesen Kontext aus, was für Stufe 1 akzeptabel
  ist.

  *(v1-Referenzblock mit Geschäftszeiten-Prüfung + TransferConversationV2 entfernt — ist in `Lösung mit Produktionsstillstand` archiviert)*

- **Korrektur-Schleife bewusst begrenzt**: Max. 3 Korrekturversuche
  (`Topic.Korrekturversuche > 3` → Abschluss ohne Bestätigung; **v2:** kein
  Transfer; Schwelle am 2026-07-20 von 2 auf 3 erhöht, vormals analog
  Regel E4), damit kein endloses Hin und Her am Telefon entstehen kann. Der
  Fallback bei nicht erkannter Angabe (Node 7b) teilt sich diesen Zähler.

- **Stufe-2-Ausbau**: Muss dann kategoriespezifische Felder vorlesen (z. B.
  Priorität nur bei Störung) — siehe zurückgestellte Fragen unten.

---

## System-Topics — Sicherheitsnetz bei Gesprächsabbruch (2026-09-02)

Gilt für **Stufe 0 und Stufe 1**. Slim-Flow-ID: `019885f0-e29a-f111-b8db-7ced8d476627`.

### Globale Variable: `Global.FlowAufgerufen` (Boolean)

Wird direkt **vor** jedem `InvokeFlowAction`-Node in „Zusammenfassung & Bestätigung
(Slim)" auf `true` gesetzt. Kein Initialwert nötig — ungesetzt gilt als `false`.
Verhindert Doppel-Mail durch das Sicherheitsnetz.

### System-Topic: Ende der Unterhaltung

- **Trigger**: `OnSystemRedirect` — feuert bei Caller-Hang-Up (Channel-Signal),
  bei `BeginDialog → EndofConversation` aus anderen Topics, und nach
  Laufzeitfehlern (System-Topic „Bei Fehler" leitet hierher)
- **`startBehavior`**: `CancelOtherTopics`

**Node 1 — ConditionGroup** (Sicherheitsnetz):

Bedingung: `Not(IsBlank(Global.Telefonnummer)) && Not(Global.FlowAufgerufen)`

- **TRUE** → `InvokeFlowAction`:

  | Binding | Wert |
  |---|---|
  | `text` | `=If(IsBlank(Global.Firmenname), "nicht angegeben", Global.Firmenname)` |
  | `text_1` | `=If(IsBlank(Global.Ansprechpartner), "nicht angegeben", Global.Ansprechpartner)` |
  | `text_2` | `=If(IsBlank(Global.Telefonnummer), "nicht angegeben", Global.Telefonnummer)` |
  | `text_3` | `=If(IsBlank(Global.Anliegen), "Gespräch vorzeitig beendet – Anliegen nicht erfasst", Global.Anliegen)` |
  | `text_4` | `=If(IsBlank(Global.KanalLabel), "Telefon", Global.KanalLabel)` |
  | `text_5` | `=If(IsBlank(Global.Anlage), "nicht erfasst", Global.Anlage)` |

- **FALSE** → kein Flow-Aufruf

**Node 2 — `EndConversation`**: außerhalb der Condition, feuert immer.

Partial-Mails landen in Postfach D (`Other`/`Sonstiges`-Kritikalität).

### System-Topic: Stille-Erkennung

- **Node 1**: Frage „Sind Sie noch da?" (unverändert)
- **Node 2**: ~~`EndDialog`~~ → `BeginDialog → EndofConversation`

---

## Stufe 2 — zurückgestellte Topics (erarbeitetes Material)

### Topic: Kategorie auswählen — Stufe 2 (zurückgestellt)

> **Hinweis**: In Stufe 1 gibt es **keine Kategoriewahl** — weder im
> Gespräch noch nachgelagert per KI im Flow (Korrektur 2026-07-10, siehe
> Architektur.md Abschnitt 1). Der Innendienst sortiert die Kategorie
> manuell beim Bearbeiten der E-Mail ein. Dieses Topic wird erst in Stufe 2
> gebaut.

- **Node 1 — Question Node** mit Closed-List-Entity `Kategorie`:
  
  > Sagen Sie: Störung, Wartung, Ersatzteile, Angebot oder Rückruf.
  
  - 5 explizit genannte Optionen (A–E). „F. Allgemeine Anfrage" wird **nicht**
    in der Ansage genannt, sondern dient als **impliziter Fallback**: Erkennt
    der Bot eine Anfrage, die klar zu keiner der 5 Hauptkategorien passt, wird
    sie als „Allgemeine Anfrage" behandelt.
    - **Technisch noch ungeklärt**: Ein Question Node mit Closed-List-Entity
      re-promptet bei unerkannter Eingabe standardmäßig, statt automatisch in
      einen Fallback-Wert zu leiten. Zusätzlich existiert die querschnittliche
      Eskalationsregel „2× unerkannt → Transfer to Agent". Es ist noch nicht
      festgelegt, welche der drei Mechaniken (Entity-Reprompt, Fallback zu
      „Allgemeine Anfrage", 2-Versuche-Eskalation) in welcher Reihenfolge
      greift — siehe offene Architekturfrage 10 in
      [Architektur.md](Architektur.md).
  - Synonym-Tabelle für die `Kategorie`-Entity (analog `Anlagentyp`): **offen**,
    siehe [ToDos.md](ToDos.md). Die in Stufe 1 gesammelten Freitext-Anliegen
    liefern hierfür realistische Formulierungen.

- **Node 2 — Redirect to Topic**: je nach erkanntem Wert → „Störung", „Wartung",
  „Ersatzteil", „Angebot", „Rückruf" oder „Allgemeine Anfrage"

### Topic: Rückrufbitte — Stufe 2 (zurückgestellt)

Pflichtfelder laut Quelldokument („KI Lösung Servicetelefonate.docx", Abschnitt
„E. Rückrufbitte"): Name, Telefonnummer, Thema, Erreichbarkeit. Name und
Telefonnummer liegen bereits als `Global.Ansprechpartner` / `Global.Telefonnummer`
aus „Kundendaten erfassen" vor — werden **nicht** erneut abgefragt, sondern erst
im gemeinsamen Topic „Zusammenfassung & Bestätigung" vorgelesen. Das Topic
„Rückrufbitte" erfasst daher nur die zwei neuen Felder.

- **Node 1 — Question Node** (Freitext, keine Entity):
  
  > Worum geht es bei Ihrem Rückrufwunsch?
  > → gebunden an `Topic.Thema`

- **Node 2 — Question Node** (Freitext, keine Entity):
  
  > Wann sind Sie am besten erreichbar?
  > → gebunden an `Topic.Erreichbarkeit`
  
  - **Begründung Freitext statt Closed-List**: „Erreichbarkeit" hat viele
    plausible Formulierungen („vormittags", „nach 14 Uhr", „jederzeit außer
    Montag") — eine feste Werteliste würde Nuancen verlieren, die für die
    Rückrufplanung durchaus relevant sein können.

- **Node 3 — Redirect to Topic**: → „Zusammenfassung & Bestätigung"

### Topics: Störung / Wartungsanfrage / Ersatzteilanfrage / Angebotsanfrage / Allgemeine Anfrage — Stufe 2 (noch nicht erarbeitet)

- Kategoriespezifische Datenerfassung gemäß Pflichtfelder-Tabelle in
  [CLAUDE.md](CLAUDE.md). Wird erarbeitet, sobald Stufe 2 beginnt — mit den
  Erfahrungen aus den Stufe-1-Freitext-Anliegen.

### Zusammenfassung & Bestätigung — Stufe-2-Ausbau (zurückgestellt)

- **Offene Fragen** für den Ausbau, sobald die Kategorie-Topics feststehen:
  - Werden Global- und Topic-Variablen in einer Nachricht zusammen vorgelesen,
    oder getrennt (erst Kundendaten bestätigen, dann kategoriespezifische
    Angaben)?
  - Wie geht das Topic mit unterschiedlichen Feldern je Kategorie um (z. B.
    Priorität nur bei Störung, Expressversand nur bei Ersatzteil) — eine
    Power-Fx-Bedingung je Feld, oder kategoriespezifische Textbausteine?
  - Variablen-Sichtbarkeit bei Topic Variables (siehe Architektur.md,
    Abschnitt 8).
