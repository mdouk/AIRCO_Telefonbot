# Topic-Definitionen (Arbeitsstand)

Dieses Dokument hält die im Sparring erarbeiteten Inhalte der Copilot-Studio-Topics fest,
bis sie in Copilot Studio angelegt/exportiert werden.

Jedes Topic ist einer **Ausbaustufe** zugeordnet (siehe Roadmap in
[Architektur.md](Architektur.md)). Als „Stufe 2 (zurückgestellt)" markierte
Topics sind erarbeitetes Material, das erst später gebaut wird — nichts davon
ist verworfen.

---

## Stufe 1 — aktive Topics

### Topic: Conversation Start (System Topic) — Stufe 1

- **Trigger**: `OnConversationStart` (automatisch, keine Trigger-Phrasen)

- **Node 1 — Message Node**:
  
  > Willkommen beim technischen Service der Firma AIRCO Systems GmbH. Sie sprechen mit unserem digitalen Service-Assistenten. Zur Bearbeitung Ihres Anliegens wird dieses Gespräch aufgezeichnet und verarbeitet.
  
  - **Begründung**: Offenlegung, dass ein KI-Bot und kein Mensch am Telefon
    ist — Transparenz gegenüber dem Anrufer statt erst bei Nachfrage. Der
    DSGVO-Hinweis ist nötig, da das Gespräch bot-seitig verarbeitet und das
    Anliegen per E-Mail weitergeleitet wird (entschieden; unabhängig davon,
    dass die KI-Klassifizierung selbst in Stufe 1 nicht stattfindet, siehe
    Architektur.md Abschnitt 1). Exakte rechtliche Formulierung sollte vor
    Produktivbetrieb noch mit Datenschutz/Rechtsabteilung gegengeprüft
    werden.

- **Node 2 — Redirect to Topic**: → „Kundendaten erfassen"

- **Notausstieg (entschieden)**: Kein globaler „Mitarbeiter sprechen"-Trigger
  während „Kundendaten erfassen" — die Interruption für diesen Trigger wird
  erst ab Topic „Prio-Filter" aktiv (siehe Architektur.md, Abschnitt 5/7,
  Entscheidungspunkt E9). Begründung: Firmenname/Ansprechpartner/Telefonnummer
  sind dann bereits erfasst, sodass der Mitarbeiter bei Eskalation sofort
  weiß, wer anruft. Bei Stille auf eine Frage gilt die allgemeine
  Fehlversuchs-Regel (E4): erster Fehlversuch/Stille → Frage wiederholen
  (zweite Chance), zweiter Fehlversuch/Stille → `Transfer to Agent` — keine
  Sonderregel für Stille nötig.

### Topic: Kundendaten erfassen — Stufe 1

- **Node 1 — Question Node**:
  
  > Bitte nennen Sie zuerst Ihren Firmennamen.
  > → gebunden an `Global.Firmenname`

- **Node 2 — Question Node** (separat von Node 1, bewusst NICHT kombiniert):
  
  > Wie ist Ihr Name?
  > → gebunden an `Global.Ansprechpartner`
  
  - **Begründung**: Getrennte Fragen vermeiden Slot-Disambiguierung bei Freitext.
    Beispiel-Risiko einer kombinierten Frage: „Mein Name ist Michael Doukas aus
    der Firma Michael Doukas GmbH" — Firmenname und Personenname sind hier
    textlich fast identisch, eine kombinierte Extraktion wäre unzuverlässig.

- **Node 3 — Telefonnummer (entschieden)**: Caller-ID vorschlagen +
  Bestätigungsfrage, mit Fallback-Frage bei „Nein". Struktur:
  
  - **3a — Question Node** (Boolean/Ja-Nein), dynamische Nachricht mit der
    Caller-ID-Systemvariable:
    
    > Ich sehe, Sie rufen von der Nummer {Caller-ID} an. Ist das die richtige
    > Rückrufnummer für Sie?
    > → gebunden an `Topic.richtigeTelefonnummer` (bewusst **Topic Variable**, nicht Global — wird nur innerhalb dieses Topics für die Verzweigung gebraucht, siehe Architektur.md Abschnitt 6; Identifizieren = Boolesch)
  
  - **3b — Condition Node**:
    
    - **Ja** → Assign: `Global.Telefonnummer` = Caller-ID-Wert
    
    - **Nein** → Question Node (Freitext/Telefonnummer):
      
      > Wie lautet dann die Telefonnummer, unter der wir Sie erreichen können?
      > → gebunden an `Global.Telefonnummer`
  
  - **Bewusst kein separater Zentrale-Erkennungsmechanismus in Stufe 1**: Zeigt
    die Caller-ID eine Sammelnummer statt der Durchwahl, deckt der
    „Nein"-Zweig das automatisch ab — der Anrufer nennt seine Nummer dann
    manuell. Ein Abgleich gegen bekannte AIRCO-Zentralnummern (siehe
    [ToDos.md](ToDos.md)) wäre nur eine Optimierung, keine Voraussetzung.
  
  - **Variable verifiziert** (Test-Panel-Simulation, siehe Architektur.md
    Abschnitt 9, Punkt 1): `Activity.From.Name` trägt die Rufnummer. Im
    „Variablenwert festlegen"-Node muss der Wert im Feld „Bis Wert" als
    Power-Fx-Formel `System.Activity.From.Name` eingetragen werden (per
    `fx`-Symbol) — funktioniert (bestätigt 2026-07-08). Auswahl über den
    normalen Objekt-Picker führte stattdessen zu einem Laufzeitfehler
    (`InvalidContent`). Bestätigung unter realen Telefonie-Bedingungen (echter
    Anruf) steht noch aus.

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

- **Node 2 — Redirect to Topic**: → „Prio-Filter"

- **Offene Fragen**: Umgang mit „weiß nicht" (dritter Wert oder als
  Nein behandeln?). Ab Stufe 1.5/1.6: Frage durch automatischen Abgleich
  (SharePoint-Liste / Odoo-Kundenstamm) ersetzen.

### Topic: Prio-Filter — Stufe 1 (entschieden)

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
  keine Pflichtfelder. Der Flow leitet den Freitext **unverändert** per E-Mail
  weiter; der Innendienst sortiert die Kategorie manuell beim Bearbeiten ein
  (keine KI-Klassifizierung in Stufe 1, siehe Architektur.md Abschnitt 1).

- **Node 1 — Question Node** (Freitext, keine Entity):
  
  > Bitte beschreiben Sie nun Ihr Anliegen.
  > → gebunden an `Global.Anliegen`

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
- **Zweck (wenn gebaut)**: CLAUDE.md nennt die Regel „Sicherheitsrelevantes
  Problem (Brand, Rauch) → sofort Transfer to Agent", bisher aber ohne
  zugehöriges Topic. In Copilot Studio wäre das vermutlich ein eigenes Topic
  mit Trigger-Phrasen („es brennt", „Rauch", „Notfall" o. ä.), das **jedes**
  andere Topic unterbrechen darf (globale Trigger-Priorität).
- **Offene Fragen**: Welche Trigger-Phrasen genau? Nur Sprach-Trigger, oder
  auch eine Nachfrage im Prio-Filter („Besteht Gefahr für Personen?")?
  Eigene Sparring-Session erforderlich.

### Topic: Zusammenfassung & Bestätigung — Stufe 1 (entschieden)

- **Zweck**: Gemeinsamer Schritt direkt vor dem Action Node. Liest die
  gesammelten Werte vor und lässt den Anrufer bestätigen oder korrigieren.

- **Stufe-1-Kurzfassung — erweitert (2026-07-10, Michael)**: Node 1 liest jetzt
  zusätzlich zu Kundendaten (`Global.Firmenname`, `Global.Ansprechpartner`,
  `Global.Telefonnummer`) + `Global.Anliegen` auch `Global.Inbetriebnahme` und
  `Global.VertragVorhanden` vor — über 4 bedingte Nachrichtenvarianten
  (Inbetriebnahme × Vertrag, je Ja/Nein; Wortlaut siehe Node 1 unten). Da alle
  Werte Global Variables sind, entfällt weiterhin das
  Variablen-Sichtbarkeitsproblem.

- **Variablen dieses Topics** (Topic Variables, nur lokal für die
  Bestätigungs-/Korrekturlogik gebraucht — analog `Topic.richtigeTelefonnummer`
  in „Kundendaten erfassen", siehe Architektur.md Abschnitt 6):
  
  - `Topic.Bestaetigt` (Boolesch)
  - `Topic.Korrekturfeld2` (Closed-List-Entity: Firmenname, Ansprechpartner,
    Telefonnummer, Anliegen, **Inbetriebnahme**, **Vertrag** — beide neu,
    2026-07-10)
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

- **Node 2 — Question Node** (Boolean/Ja-Nein) → `Topic.Bestaetigt`:
  
  > Ist das so korrekt?

- **Node 3 — Condition Node** (`Topic.Bestaetigt`):
  
  - **Ja** → Node 8 (Action Node)
  - **Nein** → Node 4

- **Node 4 — Variable festlegen**: `Topic.Korrekturversuche` =
  `Topic.Korrekturversuche + 1`

- **Node 5 — Condition Node** (`Topic.Korrekturversuche`):
  
  - **≤ 2** → Node 6
  - **> 2** (dritter Widerspruch) → Node 9 (Eskalation)

- **Node 6 — Question Node** (Closed-List-Entity `Korrekturfeld2`):
  
  > Welche Angabe war nicht korrekt — Firmenname, Ansprechpartner,
  > Telefonnummer, Ihr Anliegen, die Angabe zur Inbetriebnahme oder zum
  > Wartungsvertrag?
  > → gebunden an `Topic.Korrekturfeld2`
  
  - **Synonym-Tabelle für die `Korrekturfeld2`-Entity: noch offen** (Michael
    liefert sie nach, siehe [ToDos.md](ToDos.md)) — analog der
    `Anlagentyp`-Tabelle in [CLAUDE.md](CLAUDE.md). Für die zwei neuen Werte
    `Inbetriebnahme` und `Vertrag` fehlt die Synonymliste noch genauso wie
    für die 4 bestehenden Werte.

- **Node 7 — Condition Node / Switch** (`Topic.Korrekturfeld2`), jeweils
  gefolgt von Redirect zurück zu **Node 1** (Zusammenfassung erneut vorlesen
  und bestätigen lassen):
  
  - **Firmenname** → Question Node: „Wie lautet der richtige Firmenname?" →
    `Global.Firmenname`
  
  - **Ansprechpartner** → Question Node: „Wie ist Ihr richtiger Name?" →
    `Global.Ansprechpartner`
  
  - **Telefonnummer** → Question Node: „Wie lautet die richtige
    Telefonnummer?" → `Global.Telefonnummer`
  
  - **Anliegen** → Question Node: „Wie würden Sie Ihr Anliegen richtig
    beschreiben?" → `Global.Anliegen`
  
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
    reproduzieren, das wir zuvor verworfen haben. Bei den anderen 4 Zweigen
    (Firmenname, Ansprechpartner, Telefonnummer, Anliegen) bleibt die
    Rückfrage nötig, da dort beliebig viele Werte möglich sind.
  
  - **Begründung gegen „Redirect to Topic"**: Ein Redirect zu „Kundendaten
    erfassen" würde am Anfang dieses Topics starten und dadurch alle drei
    Felder erneut abfragen statt nur des einen falschen. Da alle Zielwerte
    Global Variables sind, kann die Korrektur-Frage stattdessen direkt hier
    im Topic gestellt werden, ohne das Ursprungs-Topic erneut zu durchlaufen.

- **Node 8 — Action Node**: Flow „Anliegen weiterleiten" aufrufen, danach
  Message Node (Verabschiedung; „Ticket" meint hier die E-Mail-Weiterleitung,
  kein Drittsystem-Ticket):
  
  > Vielen Dank, ich habe Ihr Anliegen weitergeleitet und ein Ticket eröffnet.
  > Ein Mitarbeiter wird sich zeitnah bei Ihnen melden. Auf Wiederhören.
  
  Danach **Redirect to Topic → System-Topic „Ende der Konversation"**
  (nicht „Alle Themen beenden") — löst bei Telefonie die kanalspezifische
  Aktion aus, die den Anruf tatsächlich beendet. Ohne diesen Redirect bleibt
  die Verbindung nach der letzten Nachricht offen, da „Ende der Konversation"
  nie automatisch aufgerufen wird.

- **Node 9 — Eskalation** (nach dem dritten Widerspruch, `Topic.Korrekturversuche
  
  > 2`, siehe Node 5): **Geschäftszeiten-Prüfung entschieden** — dieselbe
  > Power-Fx-Formel wie beim Prio-Filter:
  > `Weekday(Now(), StartOfWeek.Monday) <= 5 && Hour(Now()) >= 8 && Hour(Now()) < 17`
  
  - **Wahr** (Mo–Fr 8–17 Uhr) → Action Node → Flow „Anliegen weiterleiten"
    (gleiche Variablen-Zuordnung wie Node 8) mit den zuletzt erfassten Werten
    
    + Marker „Zusammenfassung nach 2 Korrekturversuchen nicht bestätigt —
      Anrufer wurde an Mitarbeiter übergeben." → danach **Message Node**
      (neu, 2026-07-10):
      
      > Ihre Angaben konnten leider nicht abschließend bestätigt werden. Ich
      > verbinde Sie jetzt mit einem Mitarbeiter, der Ihnen weiterhilft.
      
      → danach „Unterhaltung übertragen" (gleiche Platzhalter-Zielrufnummer
      wie Prio-Filter)
  
  - **Falsch** (außerhalb Geschäftszeit) → Action Node → Flow (gleicher
    Aufruf) → Message Node:
    
    > Sie rufen außerhalb der Geschäftszeiten an. Bitte rufen Sie uns Montags bis Freitags zwischen 8 und 17 Uhr.
    
    → Redirect to Topic → „Ende der Konversation"

- **Korrektur-Schleife bewusst begrenzt**: Max. 2 Korrekturversuche
  (`Topic.Korrekturversuche > 2` → Eskalation), analog der bestehenden
  2-Fehlversuche-Regel E4, damit kein endloses Hin und Her am Telefon
  entstehen kann.

- **Stufe-2-Ausbau**: Muss dann kategoriespezifische Felder vorlesen (z. B.
  Priorität nur bei Störung) — siehe zurückgestellte Fragen unten.

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
