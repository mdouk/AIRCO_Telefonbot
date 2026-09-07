# Fortschrittsübersicht

Status je Topic/Baustein: **offen** / **in Bearbeitung** / **erledigt** /
**zurückgestellt** (= spätere Stufe).
Detailinhalte der bereits erarbeiteten Topics stehen in [Topics.md](Topics.md),
die Roadmap der Ausbaustufen in [Architektur.md](Architektur.md).

> **⚠ Versionshinweis (2026-07-14 — v2):** Ablauf auf Kundenwunsch umgestellt:
> **kein Prio-Filter, keine Eskalation/Transfer**; stattdessen KI-Verarbeitung
> (Zusammenfassung + Kritikalität) im Power-Automate-Flow und Routing an
> **4 Postfächer**. v1-Stand vollständig im Ordner
> `Lösung mit Produktionsstillstand`. Bot + Flow bereits dupliziert
> (Solution 1.1.0.0, 2026-07-14).

## Stufe 0 „Slim" — Status (2026-08-18)

| Baustein | Status |
|----------|--------|
| Conversation Start | **erledigt** — identisch mit Stufe 1 (YAML: `YAML/Slim/Start der Unterhaltung.md`) |
| Kundendaten erfassen | **erledigt** — redirectet zu `Anlagenerfassung`; Caller-ID-Block entfernt (2026-08-20): keine Systemvariable liefert PSTN-Nummer in Teams Phone + Agents & Queues; Telefonnummer immer manuell per `PhoneNumberPrebuiltEntity`; `KanalLabel` hardcodiert `"Telefon"`; zusätzlich `Global.TelefonnummerGesprochen` (2026-08-25, getestet) — Ziffern-mit-Leerzeichen-Formel gegen TTS-Kardinalzahl-Bug, siehe F3. ⚠ `PersonNamePrebuiltEntity` für Ansprechpartner: bewusst zunächst so belassen — Praxistest entscheidet; wenn Entity natürliche Sprache nicht erkennt → auf `StringPrebuiltEntity` wechseln (wie Stufe 1). (YAML: `YAML/Slim/Kundendaten_erfassen.md`) |
| **Anlage erfassen** (NEU) | **erledigt** — neues Topic: eine Frage (StringPrebuiltEntity, `allowInterruption: true`): „Bitte teilen Sie uns die Anlagenbezeichnung, Seriennummer und das Baujahr mit." → `Global.Anlage`; redirectet zu `Anliegenerfassen` (YAML: `YAML/Slim/Anlage erfassen.md`) |
| Anliegen erfassen | **erledigt** — identisch mit Stufe 1; redirectet zu `Zusammenfassung-Slim` statt `Zusammenfassung` (YAML: `YAML/Slim/Anliegen erfassen.md`) |
| Zusammenfassung & Bestätigung (Slim) | **erledigt** — vereinfacht: Summary + Frage in einem einzigen Question Node kombiniert; bestätigt nur Kontaktdaten (Anlage + Anliegen unbestätigt); Korrekturfeld-Entity auf 3 Optionen reduziert (Firmenname / Ansprechpartner / Telefonnummer); Fallback bei nicht erkannter Angabe: Zähler inkrementieren + zurück zur Korrekturfeld-Frage (korrigiert 2026-08-18); flowId: `019885f0-e29a-f111-b8db-7ced8d476627` (YAML: `YAML/Slim/Zusammenfassung - slim.md`) |
| **Power Automate Flow „Anliegen weiterleiten – slim"** | **erledigt (2026-08-18)** — 6 Text-Inputs (keine Booleans); Trigger: Firmenname/Ansprechpartner/Telefonnummer/Anliegen/KanalLabel/Anlage. KI-Verarbeitung: JSON-Schema um `anlagenbezeichnung`, `serialnummer`, `baujahr` (optional) erweitert; `kritikalitaet`-Enum auf `"Sonstiges"` (statt `"Other"`) umgestellt; `Anlage` als Kontext-Input im Prompt. E-Mail-Body: Tabellenzeile „Anlage" (`triggerBody()?['text_5']`) hinzugefügt, Inbetriebnahme/Wartungsvertrag entfernt, Rohtext-Duplikat entfernt. `Respond to the agent` direkt nach Trigger (Timeout-Fix). Fallback-E-Mail bereits korrekt (Anlage vorhanden, keine Booleans). Prioritätsformel unverändert korrekt (`Sonstiges` fällt in Low-Fallback). |
| Störungs-/Anliegenliste einbinden | **offen** — identisch mit Stufe 1; wartet auf Kundenlieferung |

---

## Stufe 1 (aktuelle Stufe) — Status

| Baustein | Status |
|----------|--------|
| Agent-Konfiguration (Basic Voice Agent, Anweisungen, Wissen, Tools) | erledigt (siehe Architektur.md Abschnitt 2) |
| Conversation Start | erledigt (Praxistest Aussprache „GmbH" ausstehend, siehe unten) |
| Kundendaten erfassen | erledigt (YAML aktualisiert 2026-07-15: Fragetext → „Für welche Firma rufen Sie an?"; `Global.KanalLabel` jetzt **kontextuell**: „Telefon" wenn Caller-ID vorhanden, „Chat" sonst. **Praxistest erledigt (2026-07-21):** Alle 3 Entities auf `StringPrebuiltEntity` („Gesamte Antwort des Benutzers") umgestellt — Copilot-Studio-Entity-Extraktion (OrganizationPrebuiltEntity, PersonNamePrebuiltEntity, PhoneNumberPrebuiltEntity) scheiterte an natürlicher Sprache. Die Bereinigung (Rohnamen → sauberer Wert) übernimmt jetzt der KI-Node im Flow.) |
| Inbetriebnahme | erledigt (YAML geprüft 2026-07-13) |
| Vertragsfrage | erledigt (YAML geprüft und korrigiert 2026-07-13: Wortlaut „mit Garantieverlängerung" ergänzt, allowInterruption false) |
| ~~Prio-Filter (Produktionsstillstand)~~ | **entfällt in v2** (2026-07-14) — kein Prio-Filter/Transfer mehr; v1-YAML im Backup-Ordner |
| Anliegen erfassen (Freitext) | erledigt (in Copilot Studio umgesetzt, bestätigt 2026-07-10) |
| Sicherheits-Eskalation | zurückgestellt + **v2 neu zu konzipieren** (ohne Transfer: dringlicher Hinweis + höchste Kritikalität?); Risiko bis dahin: Sicherheitsnotfall wird wie jeder andere Anruf behandelt |
| Zusammenfassung & Bestätigung (Kurzfassung) | erledigt (YAML geprüft und korrigiert 2026-07-15: „Ticket eröffnet"-Text entfernt → kein Ticket in Stufe 1; Korrekturfeld-Frage: „Anliegen" entfernt — Freitext per Sprache nicht korrigierbar; flowId auf neuen Flow aktualisiert; Synonymtabelle Korrekturfeld-Entity komplett gepflegt 2026-07-20 (inkl. Telefonnummer; „Anliegen"-Wert entfernt); Fallback bei nicht erkannter Angabe + Blank()-Fix für übersprungene Korrektur-Fragen + text-Feld der Re-Zusammenfassung ergänzt & getestet 2026-07-20 → Topic vollständig) |
| ~~Eskalation / Unterhaltung übertragen~~ | **entfällt in v2** (2026-07-14) — kein Transfer mehr; v1-Modellierung im Backup-Ordner |
| Power Automate Flow „Anliegen weiterleiten (KI)" | **erledigt (v2, 2026-07-15)** — neuer Flow von Grund auf gebaut; 7 Inputs, KI-Schritte, 5 E-Mail-Branches, HTML-Body; **Blocker Go-Live**: Postfach-Adressen + Störungsliste |
| KI-Schritt im Flow (Structured Output, ein Node) | **erledigt (2026-07-21, erweitert)** — Ein KI-Node (GPT-4.1, Structured Output mit JSON-Schema): Inputs: `Firmenname` (Rohtext), `Ansprechpartner` (Rohtext), `Telefonnummer` (Rohtext), `Anliegen` (Freitext). Liefert 5 Felder: `zusammenfassung`, `kritikalitaet` (Enum), `firmenname`, `ansprechpartner`, `telefonnummer` (je KI-bereinigt aus Rohtext) → VarZusammenfassung + VarKritikalitaet + VarFirmenname + VarAnsprechpartner + VarTelefonnummer. JSON-Schema-Bug behoben (doppelter `ansprechpartner`-Key → `telefonnummer`). |
| Störungs-/Anliegenliste bereitstellen + im KI-Schritt einbinden | **offen** — Knowledge-Slot im Agent-Node vorbereitet; wartet auf Kundenlieferung (AIRCO) |
| E-Mail-Routing + Body (eine Aktion, inline-if) | **erledigt (2026-07-21, aktualisiert)** — Inline-`if()` im Empfänger-Feld: Kritisch→A, Hoch→B, Mittel→C, Rest→D. HTML-Body: Tabelle — Firmenname/Ansprechpartner/Telefonnummer jetzt via `variables('VarFirmenname'/'VarAnsprechpartner'/'VarTelefonnummer')` (KI-bereinigt); Inbetriebnahme/Wartungsvertrag weiter via `triggerBody()?['boolean…']`; `variables('VarZusammenfassung')` + Zeitstempel/Kanal + Rohtext (Testperiode). Fallback-Node „E-Mail senden 2" bei KI-Ausfall (Configure run after: Failed/TimedOut/Skipped). **Postfach-Adressen (Interim 2026-07-20): vorerst alle Stufen → `Service@airco-systems.de`; Routing auf 4 getrennte Postfächer folgt später.** |
| Teams Phone einrichten (Agent mit Festnetznummer verbinden) | **in Bearbeitung** — Ansatz: **Agents & Queues** (kein Auto Attendant). Stand 2026-08-10: ✅ Power Platform Umgebung: EMEA + Dataverse + Pay-As-You-Go einrichten; ✅ M365 Admin Center: Resource Account `airco-service-bot@airco-systems.de` erstellt + lizenziert (2026-07-20); ✅ Copilot Studio: Teams-Kanal aktiviert; ⚠ Copilot Studio: Lösung als nicht verwaltete Lösung exportieren + .zip herunterladen; ⚠ Teams Admin Center: .zip als Teams App hochladen; ⚠ Teams Admin Center: Operator Connect + Festnetznummer portieren; ⚠ Teams Admin Center: Resource Account → Nummer zuweisen + Agent verbinden; ⚠ Teams Admin Center: Agents & Queues → Copilot Studio Agent registrieren; ⚠ Praxistest (echter Anruf) |

## Spätere Stufen — Status

| Stufe | Baustein | Status |
|-------|----------|--------|
| 1.5 | Ticketanlage Planner / SharePoint-Liste | zurückgestellt |
| 1.6 | Ticketanlage Odoo (Connector klären) | zurückgestellt |
| 2 | Kategorie auswählen | zurückgestellt (Design erarbeitet, siehe Topics.md) |
| 2 | Störung | zurückgestellt |
| 2 | Wartungsanfrage | zurückgestellt |
| 2 | Ersatzteilanfrage | zurückgestellt |
| 2 | Angebotsanfrage | zurückgestellt |
| 2 | Rückrufbitte | zurückgestellt (Design erarbeitet, siehe Topics.md) |
| 2 | Allgemeine Anfrage | zurückgestellt |
| 2 | Zusammenfassung & Bestätigung (kategoriespezifischer Ausbau) | zurückgestellt |
| 2 | Power Automate Flow „Ticket erstellen" (Prioritätsmatrix-Routing) | zurückgestellt |
| 2.5 | Automatische Angebotsgenerierung + Odoo-Eintrag | zurückgestellt |
| 3 | Interaktiver Fehlerbehebungsbot (intern / Vertragskunden / buchbar) | zurückgestellt |

---

## v2-Umbau — Arbeitsliste (nach Themen, je 1 Session)

Konkrete Anpassungen, um von der Doku in den duplizierten v2-Agenten/-Flow zu
kommen. Jede Gruppe (A–G) ist als **eigene Session** gedacht. Empfohlene
Reihenfolge: **A** (schnell, macht sofort testbar) → **B** (Design ohne
Transfer) → **C/D/E** (Flow; D/E brauchen den Kunden-Input aus **F**) → **G**
(Test). Details je Punkt in [Architektur.md](Architektur.md) /
[Topics.md](Topics.md).

### A. Copilot Studio — Agent aufräumen (mechanisch)
- [x] Prio-Filter-Topic im v2-Agenten **gelöscht** (erledigt)
- [x] „Vertragsfrage" Node 2: Redirect → **„Anliegen erfassen"** (erledigt)
- [x] Alle **„Unterhaltung übertragen"**-Nodes entfernt (erledigt)
- [x] Globale Variable **`Produktionsstillstand`** entfernt (erledigt)
- [x] Kurztest ohne Prio-Filter durchgeführt (erledigt)

### B. Copilot Studio — Ersatzverhalten ohne Transfer (Design-Session)
- [x] „Zusammenfassung & Bestätigung" **Node 9**: Abschluss ohne Transfer — erledigt (2026-07-15)
- [x] Querschnitt **Stille / unerkannte Eingabe / DTMF**: alle System Topics geprüft (2026-07-15) — kein Transfer-Node in keinem Topic; Eskalation-Topic deaktiviert; Details:
  - `OnSilence`: „Sind Sie noch da?" → bei erneuter Stille EndConversation ✅
  - `OnUnrecognizedSpeech`: Entschuldigung + Wiederholung (kein Zähler, d.h. theoretisch endlos — Stufe 1 akzeptiert) ✅
  - `OnUnknownDtmfKey`: Entschuldigung + Wiederholung ✅
  - `OnSystemRedirect` („Unterhaltung zurücksetzen"): `ClearAllVariables` + `CancelAllDialogs` + „Wie kann ich Ihnen helfen?" — kein Transfer, aber **⚠️ Praxistest**: falls mitten im Gespräch ausgelöst, gehen alle Variablen verloren
- [x] Aktiver **„Mitarbeiter-Wunsch"**: Eskalation-Topic deaktiviert (Kundenwunsch 2026-07-15) — kein Eskalationsweg angeboten, damit Anrufer nicht lernen, immer nach einem Menschen zu fragen ✅
- [x] **Englisch-Sprecher**: Anweisungstext aktualisiert (Transfer-Verweis entfernt, englische Abschluss-Nachricht + Gesprächsende) — erledigt (2026-07-15)
- [ ] **Englischer Anrufer — Topic** (zurückgestellt): Für zuverlässige Ticket-Erstellung bei englischsprachigen Anrufern ist ein eigenes Topic nötig (Trigger-Phrasen auf Englisch → Caller-ID als Telefonnummer sichern → Flow aufrufen → englische Abschluss-Nachricht → EndConversation). Anweisungstext allein kann keinen Flow aufrufen. Stufe 2 wird ohnehin volle Englisch-Unterstützung bringen — bis dahin zurückgestellt. Trigger-Phrasen noch zu definieren.
- [ ] **Sicherheitsnotfall** (Brand/Rauch) ohne Transfer: weiterhin zurückgestellt

### C. Power Automate — Flow-Gerüst v2
- [x] Neuen Flow von Grund auf gebaut (2026-07-15) — kein Altlast-Duplikat mehr. Alter duplizierter Flow verworfen (konnte Produktionsstillstand-Input nicht löschen; KanalLabel-If/Else war nicht mehr nötig da KanalLabel direkt vom Bot kommt).
- [x] **7 Trigger-Inputs** definiert: Firmenname, Ansprechpartner, Telefonnummer, Anliegen, KanalLabel, VertragVorhanden, Inbetriebnahme — kein Produktionsstillstand mehr.
- [x] **`flowId`** im v2-Agenten aktualisiert (Zusammenfassung.md) → zeigt auf den neuen Flow.
- [x] Grobstruktur: Trigger → KanalLabel-Logik → Agent (KI-Zusammenfassung) → Classify → 5 E-Mail-Branches → Respond to agent.

### D. Power Automate — KI-Schritt + Kritikalitäts-Prompt (Kern)
- [x] ~~**Zwei-Schritt-KI-Architektur** (Agent-Node + Classify-Node)~~ — **umgestellt (2026-07-20)**: stattdessen **ein Structured-Output-Node** (GPT-4.1) mit JSON-Schema. Ein Modellaufruf statt zwei — schneller, günstiger, Enum erzwingt gültige Werte.
- [x] **KI-Node „KI-Verarbeitung"** konfiguriert (GPT-4.1, Structured Output): Inputs: `Firmenname` (Rohtext), `Ansprechpartner` (Rohtext), `Telefonnummer` (Rohtext), `Anliegen` (Freitext). Prompt extrahiert 5 Felder: `zusammenfassung`, `kritikalitaet` (Enum), `firmenname`, `ansprechpartner`, `telefonnummer`. Fallback-Text „Anliegen unklar." bei unklarem Anliegen. JSON-Schema-Bug behoben (doppelter `ansprechpartner`-Key → `telefonnummer`). Ausgaben → VarZusammenfassung + VarKritikalitaet + VarFirmenname + VarAnsprechpartner + VarTelefonnummer. **Korrekte Ausdrücke:** `body('KI-Verarbeitung')?['structuredOutput/zusammenfassung']` / `…/kritikalitaet']` / `…/firmenname']` usw. — **nicht** `outputs()` (liefert leer).
- [x] **Kritikalitäts-Mapping** (2026-07-15/20): Mittel = Wartung/Ersatzteil/Angebot; Niedrig = Rückruf/allgemeine Anfragen; Other = automatischer Fallback.
- [ ] **Störungsliste** (Knowledge im Agent-Node): Als SharePoint-Datei geplant — wartet auf Kundenlieferung (AIRCO). **Entschieden (2026-08-25): kein Blocker mehr für die laufende Stufe** — Michael stuft das Thema als Material für eine spätere Ausbaustufe ein, nicht für den aktuellen Test von Stufe 0/1.
- [ ] **Prompt feintunen + testen** sobald Störungsliste vorliegt. (siehe oben — ebenfalls auf eine spätere Stufe verschoben)

### E. Power Automate — Routing + E-Mail
- [x] **Betreff-Format**: `[Kritikalität] - Firma: Firmenname` — Zusammenfassung im Body, nicht im Betreff.
- [x] **Routing: inline-if im Empfänger-Feld** (2026-07-20): eine einzige E-Mail-Aktion, kein Switch. `if(VarKritikalitaet == 'Kritisch') → A, Hoch → B, Mittel → C, Rest → D`. Betreff-Präfix + E-Mail-Priorität (High/Normal/Low) ebenfalls inline. Referenzen auf `body('agent-…')` wurden durch `variables('VarKritikalitaet')` ersetzt (kein GUID-Hardcode mehr).
- [x] **Body** (HTML, eine E-Mail-Aktion, 2026-07-20): Tabelle Kundendaten + `variables('VarZusammenfassung')` + Zeitstempel/Kanal + Rohtext (Testperiode). Booleans per `if(triggerBody()?['boolean/boolean_1'], 'Ja', 'Nein')`.
- [x] **Fallback-Node „E-Mail senden 2"** (2026-07-20): Configure run after: Failed/TimedOut/Skipped auf KI-Node. Rohtext + Hinweis „manuell prüfen" → Postfach D (fest). Nur `triggerBody()`-Werte, keine Variablen.
- [x] **Postfach-Adressen (Interim 2026-07-20)**: vorerst alle Kritikalitätsstufen → `Service@airco-systems.de`. Routing auf 4 getrennte Postfächer (Kritisch/Hoch/Mittel/Niedrig) kommt später — dann Adressen von AIRCO einholen.

### F. Kunden-/AIRCO-Input (Blocker für D/E bzw. Go-Live)
- [ ] **Störungs-/Anliegenliste** vom Kunden (Grounding für den KI-Schritt) — **auf spätere Stufe verschoben (2026-08-25)**, siehe oben.
- [x] **4 Postfach-Adressen** — Interim: alle → `Service@airco-systems.de` (2026-07-20); späteres Routing auf 4 Adressen offen
- [x] **Lizenz / Copilot-Guthaben** für AI Builder bzw. Flow-Ausführungen — **erledigt (2026-08-25)**: AIRCO hat den Pay-As-You-Go-Abrechnungsplan eingerichtet, Testbetrieb (~40 Anrufe/Tag) ist möglich.

### F2. Technische Pflichtprüfungen vor Go-Live (2026-08-10 ergänzt)
- [x] **LLM = GPT-4.1**: **Entscheidung (2026-08-25): bewusst nicht erneut geprüft.** Michael möchte die aktuell funktionierende Konfiguration nicht durch eine Kontrolle in den Copilot-Studio-Einstellungen riskieren. Bleibt unverändert, solange alles läuft — bei künftigen Problemen mit der Sprachausgabe als erste Stelle zum Prüfen im Hinterkopf behalten.
- [x] **`Respond to agent` direkt nach Trigger** im Flow: Fix für `FlowActionBadGateway`-Timeout — umgesetzt im Slim-Flow (2026-08-18) **und im Stufe-1-Flow (2026-08-25)**. Testanruf bestätigt: kein Hänger/Fehlercode mehr, E-Mail kommt zuverlässig an → behebt Testfeedback #15 und #18 (siehe „Testing-Feedback" unten).
- [ ] **Re-Zusammenfassung `text`-Feld** ergänzen: Im YAML fehlt das text-Feld der Re-Zusammenfassung — Chat-Test zeigt keine Re-Zusammenfassung (nur speak). **Nur Stufe 1 betroffen, für Slim nicht relevant** (Slim hat keinen separaten Re-Zusammenfassung-Node, siehe Topics.md). Nachpflegen oder bewusst akzeptieren.
- [x] **Feedback-Tracking**: `Daten\Airco_Telefonbot_Feedback_260825.xlsx` (Dateiname am 2026-08-25 aktualisiert/bestätigt) — Tester tragen Bugs / Änderungswünsche mit Beschreibung, E-Mail-Referenz, Name und Datum ein. Spalte „Anmerkung mosaiic" für Rückmeldungen reserviert.

### F3. Testing-Feedback — `Daten\Airco_Telefonbot_Feedback_260825.xlsx` (Stand 2026-08-25)

Alle Einträge mit Status „Umgesetzt" / „Stufe 2.0 ff." / „Prüfen" / „Zurückgestellt"
sind bereits abgearbeitet bzw. eingeordnet (siehe Datei). Diese vier Einträge
hatten **keinen Status** und wurden in dieser Session eingeordnet:

- [x] **#15 — Bot hängt bei Korrektur-Nachfrage, liest Fehlercode vor, danach „kein Thema verknüpft"**:
  bestätigt — war der bekannte `FlowActionBadGateway`-Timeout im Stufe-1-Flow.
  Fix (`Respond to the agent` direkt nach Trigger) am 2026-08-25 umgesetzt und
  per Testanruf verifiziert: kein Hänger/Fehlercode mehr. **Zusatzfund beim
  Review des System-Topics „Fallback"**: dessen Standard-Inhalt leitet nach
  3 Fehlversuchen per `BeginDialog` an das Topic „Escalate" weiter — das aber
  deaktiviert ist. Ein `BeginDialog` auf ein deaktiviertes Topic erklärt
  vermutlich auch die rohe Meldung „kein Thema verknüpft"/„keine Verknüpfung"
  zusätzlich zum Flow-Timeout. Wird mit #17 mitbehoben (siehe unten).
- [x] **#18 — Keine E-Mail bei abgebrochener Wartungsanfrage**: durch denselben
  Flow-Fix behoben — Testanruf bestätigt, E-Mail kommt jetzt zuverlässig an.
  Kein zusätzliches Sicherheitsnetz-Konzept nötig (Architektur.md Abschnitt 9,
  Punkt 21 kann geschlossen werden, falls kein erneuter Fall auftritt).
- [ ] **#17 — „Themenerkennung": Bot sagt „kein Thema verknüpft" bei fachfremden Aussagen**:
  Ursache bestätigt (siehe #15-Zusatzfund): Standard-Inhalt des System-Topics
  „Fallback" verweist im `elseActions`-Zweig (`FallbackCount >= 3`) auf das
  deaktivierte Topic „Escalate". System-Topics „Fallback" und „Bei Fehler"
  wurden am 2026-08-25 aktiviert (vorher beide „Aus"). **Noch zu tun**: im
  Fallback-Topic den `ConditionGroup`/`FallbackCount`-Block entfernen (unsere
  Entscheidung: kein Versuchszähler, immer direkt umleiten) und durch kurze
  Entschuldigung + Redirect zu „Anliegen erfassen" ersetzen (Ziel-Topic über
  den Themen-Picker wählen, nicht die Dialog-ID von Hand tippen). Praxistest:
  bleiben `Global.*`-Variablen beim Redirect erhalten? „Bei Fehler"
  (OnError)-Standardinhalt wurde geprüft: `speak`-Feld verrät in Produktion
  keine technischen Details (nur der Testmodus-Zweig zeigt Fehlercode/Convo-ID) —
  unverändert nutzbar. Offen: was nach `CancelAllDialogs` passieren soll
  (Gespräch sauber beenden vs. auch zu „Anliegen erfassen" umleiten) — siehe
  Rückfrage im Chat.
  **Entschieden (2026-08-25)**: „Bei Fehler" soll das Gespräch nach der
  Entschuldigung sauber beenden (nicht zu „Anliegen erfassen" umleiten, da ein
  Redirect bei einem systemischen Fehler denselben Fehler direkt wiederholen
  könnte). Umsetzung: `speak`-Text im Produktions-Zweig von „Versuchen Sie es
  noch einmal" auf „Bitte rufen Sie in Kürze erneut an" ändern (widerspricht
  sich sonst mit dem folgenden Gesprächsende) und nach `CancelAllDialogs` ein
  `EndConversation` ergänzen.
  **Fallback-Topic gebaut (2026-08-25)**: `ConditionGroup` mit
  `FallbackCount`-Zähler und Escalate-Verweis entfernt; stattdessen
  Nachrichtenknoten („Entschuldigung, das habe ich nicht richtig verstanden.
  Ich nehme Ihr Anliegen gerne trotzdem auf.") + Redirect zu „Anliegen
  erfassen". **Test noch offen** — im Testpanel nicht trivial auslösbar: In
  diesem Bot-Aufbau (durchgehende Redirect-Ketten, überall
  `StringPrebuiltEntity`-Catch-all) wartet fast immer ein Question Node auf die
  Antwort und schluckt jede Äußerung, sodass `OnUnknownIntent` gar nicht zum Zug
  kommt. „Thema testen"-Option existiert für System-Themen nicht.
  Erkenntnis: Fallback greift v. a. dann, wenn der Bot unerwartet im
  Root-Zustand landet (nach `CancelAllDialogs` aus OnError/Reset) oder wenn die
  Orchestrator-Prüfung bei `allowInterruption: true` anschlägt.

- [x] **Turn-Taking — Anrufer wird beim Zögern abgeschnitten** (neuer
  Testerbericht, nicht in der Excel-Liste): Bot fragt nach der
  Anlagenbezeichnung, Anrufer zögert 2–3 s, Bot startet bereits die
  Anliegenfrage; die nachgereichte Anlagenbezeichnung landet in
  `Global.Anliegen`, das echte Anliegen wird nie erfasst — **ohne jede
  Fehlermeldung**. Vollständige Analyse in Architektur.md Abschnitt 2a
  („Turn-Taking-Problem").
  **Behoben (2026-08-25)** durch zwei Kanaleinstellungen, ohne einen einzigen
  zusätzlichen Knoten: Sprachempfindlichkeit 0,5 → 0,2 **und**
  Äußerungsende-Timeout 1500 → 2500 ms. Verifiziert per Testanruf mit
  provoziertem Zögern (hörbares Einatmen/„ähm") sowie mit Störgeräusch
  (laut sprechende Person daneben — der härteste Störfall, da menschliche
  Sprache): Bot wartet, wiederholt die Frage bei Bedarf selbst, Werte landen
  im richtigen Feld, Gespräch läuft vollständig durch.
  **Wichtige Nebenerkenntnis:** Der Question Node wiederholt seine Frage bei
  *No Input* **von sich aus** (Plattformverhalten, steht nicht im Topic-YAML).
  Kritisch ist deshalb nicht die Stille, sondern der Fall, dass die Erkennung
  auf ein Geräusch anspringt und ein Fragment als gültige Antwort akzeptiert
  wird — was die Empfindlichkeit steuert, nicht die Topic-Logik.
  ⚠ Offen: Gegenprobe in wirklich lauter Industrieumgebung (Kunden rufen aus
  der Halle an) — bei 0,2 auch die Gegenrichtung prüfen, ob leise sprechende
  Anrufer noch zuverlässig erkannt werden.
- [x] **Telefonnummer wird als Kardinalzahl vorgelesen** (neuer Testerbericht,
  nicht in der Excel-Liste): Testanruf mit Testnummer „123456789" ergab
  „einhundertdreiundzwanzig Millionen vierhundertsechsundfünfzigtausend...".
  **Root Cause**: Azure-Neural-TTS normalisiert eine unformatierte Ziffernfolge
  im `speak`-Feld standardmäßig als Kardinalzahl; Copilot Studios `speak`-Editor
  bietet nur Audio/Pause/Betonung/Prosodie, kein `say-as
  interpret-as="telephone"` — rohes SSML manuell einfügen also keine Option.
  **Behoben (2026-08-25)** durch neue Variable `Global.TelefonnummerGesprochen`
  (SetVariable direkt nach der Telefonnummer-Frage in „Kundendaten erfassen —
  Stufe 0"): `Concat(Sequence(Len(Global.Telefonnummer)),
  Mid(Global.Telefonnummer, Value, 1), " ")` fügt Leerzeichen zwischen die
  Ziffern ein. Nur im `speak`-Feld der Zusammenfassung (Slim) verwendet;
  `text`-Feld, Flow-Input und E-Mail bleiben bei der unformatierten Nummer.
  Verifiziert per Testanruf. Details: [Topics.md](Topics.md), „Kundendaten
  erfassen — Stufe 0", Punkt 4, und „Zusammenfassung & Bestätigung (Slim)".
- [x] **Wortlaut-Inkonsistenz „Anlage erfassen"**: `text`-Feld sagte
  „Serielnummer", `speak`-Feld „Serialnummer" — auf „Seriennummer"
  vereinheitlicht (2026-08-25), per Testanruf verifiziert.
- [x] **#19 — Bot hört sich über Lautsprecher selbst, unterbricht bei Hintergrundgeräuschen**:
  Sprachempfindlichkeit im Sprachkanal (Karte „Stille") von 0,5 auf **0,2**
  gesenkt (umgesetzt 2026-08-25). Zusätzlich besprochen: Äußerungsende-Timeout
  1500→2000 ms (Vorschlag, Umsetzung offen), Spracherkennungs-Timeout bei
  12000 ms belassen (Option „Spracherkennungs-Timeout", nicht „Kein
  Erkennungs-Timeout" — sonst greift `OnSilence` nie), „Halten und Fortsetzen"
  mit Trigger-Wörtern befüllt für das Szenario „Kunde muss zum Typenschild
  laufen". Details siehe Architektur.md Abschnitt 2a.
  **Retest erfolgt (2026-08-25)**: Testanruf mit einer laut singenden Person
  direkt neben dem Anrufer (härterer Störfall als reines Hintergrundrauschen,
  da menschliche Stimme) — Bot unterbrach sich nicht selbst, Anruf lief
  vollständig durch. Bestätigt: Barge-in-Problem bei 0,2 behoben. ⚠ Weiterhin
  offen: Gegenrichtung (siehe „Gegenprobe" oben) — ob leise sprechende
  Anrufer bei 0,2 noch zuverlässig erkannt werden, ist damit noch nicht
  geprüft.

### F4. Fork-Bug „InvokeFlowAction vor BeginDialog" — Kundendaten erfassen (2026-09-07)

> **Hinweis**: Der hier beschriebene Stand (Anrufgrund-Frage im Hauptfluss,
> Topics „Anrufgrund erfassen"/„Anlage erfassen"/„Anliegen erfassen", Flow
> „Staging"/„Sweep") ist lokal weiterentwickelt worden und in
> [Architektur.md](Architektur.md)/[Topics.md](Topics.md) **noch nicht**
> vollständig nachgezogen — dieser Eintrag dokumentiert ausschließlich das
> Testergebnis, damit es zwischen Sessions nicht verloren geht.

**Problem**: Ein `InvokeFlowAction`-Knoten („Staging", flowId
`a0a99959-80a7-f111-b8de-7ced8d476627`), dem im selben Topic noch ein
`BeginDialog` und/oder eine `ConditionGroup` folgt, bevor die nächste Frage
erreicht wird, führt zu einem reproduzierbaren Fork: Der Bot springt vorzeitig
über mehrere Topic-Grenzen hinweg zur nächsten erreichbaren Frage, während der
eigentlich korrekte Pfad (inkl. Flow-Aufruf) erst Sekunden später — nach
Antwort des Flows — nachläuft und dabei bereits gefüllte Variablen
kommentarlos überspringt.

**Fünf getestete Varianten, alle negativ:**

| Test | Position von Staging | Symptom |
|---|---|---|
| 10 | Kundendaten erfassen: Frage(Anrufgrund) → SetVariable(KanalLabel) → Flow → BeginDialog | Anliegen-Frage kam vor der Anlagen-Frage (Reihenfolge vertauscht, kein Datenverlust) |
| 11 | Kundendaten erfassen: Flow → Frage(Anrufgrund) → SetVariable(KanalLabel) → BeginDialog | Anrufgrund-Frage komplett übersprungen — Anrufgrund blieb leer, Rest des Topics übersprungen |
| 2026-09-07, 12:52 Uhr | Kundendaten erfassen: Frage(Anrufgrund) → SetVariable(KanalLabel) → Flow → BeginDialog *(identische Struktur wie Test 10)* | Anlage-Frage komplett übersprungen (bei „Störung"); Bot landete direkt bei der Anliegen-Frage. Flow lief bis zu diesem Zeitpunkt nicht. |
| 2026-09-07, 12:55 Uhr | Staging aus Kundendaten erfassen entfernt, stattdessen an den Anfang von „Anrufgrund erfassen" gesetzt: Flow(`cXPOis`) → `ConditionGroup` → BeginDialog(Anlage/Anliegen je nach Anrufgrund) | **Mechanismus jetzt im Trace nachgewiesen** (siehe unten): Anliegen-Frage kam sofort, bevor der Flow zurück war; die `ConditionGroup` wurde beim verfrühten Durchlauf offenbar gegen einen noch nicht aktualisierten Zustand ausgewertet und nahm den **else**-Zweig (obwohl Anrufgrund korrekt „Störung" war), sodass die Anlage-Frage übersprungen wurde. ~27 s später lief der „echte" Durchlauf nach: Flow feuerte, `ConditionGroup` wertete diesmal **korrekt** in den Anlage-Zweig aus (Trace zeigt `conditionBranchId: conditionItem_8IyZXO`, Bedingung erfüllt), Anlage-Frage wurde nachträglich gestellt. Beim erneuten Erreichen von „Anliegen erfassen" wurde die dortige Frage nicht wiederholt (Variable schon belegt), sondern kaskadierte still bis in „Zusammenfassung - Slim". Kein Datenverlust in diesem Testlauf, aber Anliegen kam wieder vor Anlage — inhaltlich derselbe Fehler wie Test 10, nur über einen anderen Topic-Pfad ausgelöst. |
| 2026-09-07, 14:08 Uhr | Staging in ein eigenes, neu angelegtes Topic „Staging" ausgelagert (`mosaiic_AIRCOTelefonBot.topic.Staging`: Flow(`ZWR6XS`) → BeginDialog(`LzYRQH`) → Anrufgrund erfassen). Kundendaten erfassen redirectet jetzt zu diesem Topic statt direkt zu „Anrufgrund erfassen". | Identisches Bypass-Muster wie beim allerersten Test: Nach dem `BeginDialog` in „Kundendaten erfassen" fehlt im Trace jede Spur von `ZWR6XS`, `LzYRQH`, der `ConditionGroup` in „Anrufgrund erfassen" und der Anlage-Frage — der Bot landet unmittelbar bei der Anliegen-Frage. Ein eigenes, isoliertes Topic nur für den Flow-Aufruf ändert nichts: Es fügt lediglich einen weiteren Hop in derselben gefährlichen Kette hinzu. |

**Damit ist die Positions-Frage endgültig geklärt**: Fünf von fünf
Platzierungsversuchen (in drei verschiedenen Topics, davon eines eigens dafür
angelegt) sind gescheitert. Es ist kein Konfigurations- oder Strukturdetail
des Bots, das sich noch finden lässt, sondern eine strukturelle Grenze der
Copilot-Studio-Runtime bei der Kombination `InvokeFlowAction` + nachfolgende
`BeginDialog`/`ConditionGroup`-Kette. Auch die knotenspezifische
„Latenznachricht"-Einstellung (Aktionseigenschaften-Panel) wurde geprüft und
verworfen — der verfrühte Sprung tritt nachweislich auf, bevor der
Flow-Aufruf überhaupt als „pending" im Trace erscheint, eine Warte-Ansage
könnte daran nichts ändern. **Diese Untersuchung ist für die Messe
abgeschlossen** — kein weiteres Herumschieben von Staging mehr versuchen.

**Bauregel präzisiert (nicht mehr nur „Position", sondern Mechanismus)**:
Der Fork ist eine Race Condition der Copilot-Studio-Runtime: Trifft sie auf
einen `InvokeFlowAction`, dem noch weitere Steuerungsknoten (`BeginDialog`,
`ConditionGroup`) vor der nächsten Frage folgen, liefert sie für die
Sofortantwort einen „schnellen" Pfad aus — der jede `ConditionGroup` in dieser
Kette gegen einen noch nicht committeten Zustand auswertet und dabei
zuverlässig in den falschen Zweig läuft. Erst wenn der Flow tatsächlich
zurückkommt, läuft der korrekte Pfad nach und überspringt bereits gefüllte
Variablen. Das erklärt zugleich Test 11 (Skip) und alle Anliegen-vor-Anlage-
Fälle (Reorder) einheitlich. Die einzige über zwei Tests hinweg **nicht**
geforkte Form bleibt: `Flow` direkt gefolgt von der nächsten `Question`
**im selben Topic**, ohne `ConditionGroup` oder weiteren `BeginDialog`
dazwischen (nachweislich stabil in „Zusammenfassung - Slim", Knoten `2exxIi`).

**Empfehlung (vor der Messe, final)**: Nach fünf gescheiterten
Positionsversuchen (Kundendaten erfassen 2×, Anrufgrund erfassen 1×, eigenes
Topic „Staging" 1×, plus Test 11) das Herumschieben endgültig aufgeben.
`Staging` **nirgends** vor eine `ConditionGroup` oder einen weiteren
`BeginDialog` stellen. Einziger stabiler Aufruf bleibt „Zusammenfassung -
Slim" (`2exxIi`) — dort belassen, überall sonst entfernen. Kostet die frühe
Datensatz-Erfassung (Nice-to-have), verhindert aber Anrufgrund-/Anlagen-
Reihenfolgefehler im Messebetrieb.

- [x] ~~`Staging`-Aufruf aus „Kundendaten erfassen" entfernen~~ — erledigt
- [ ] `Staging`-Aufruf (`cXPOis`) aus „Anrufgrund erfassen" wieder entfernen
- [ ] Eigens angelegtes Topic „Staging" (`mosaiic_AIRCOTelefonBot.topic.Staging`,
  Knoten `ZWR6XS`/`LzYRQH`) wieder entfernen, „Kundendaten erfassen"
  redirectet wieder direkt zu „Anrufgrund erfassen"
- [ ] Nach Entfernen: Testanruf zur Bestätigung, dass „Kundendaten erfassen"
  → „Anrufgrund erfassen" → „Anlage erfassen"/„Anliegen erfassen" ganz ohne
  Flow im Pfad zuverlässig und in korrekter Reihenfolge durchläuft
  (Anlage vor Anliegen bei Störung/Wartung)
- [ ] Sweep-Flow weiterhin nicht Teil der Lösung — muss vor der Messe im
  Power-Apps-Portal ergänzt werden (siehe vorherige Session; hier nicht neu
  geprüft)

### G. Test / Go-Live
- [~] **End-to-End-Test** (in Bearbeitung, 2026-07-16): Erster Durchlauf im Testpanel — Flow läuft **erfolgreich** durch (KI-Zusammenfassung „Kritisch" korrekt, E-Mail-Branch erreicht), **aber der Bot meldet `FlowActionBadGateway / NoResponse`**. Ursache: der Bot ruft den Flow synchron auf und wartet auf `Respond to the agent`; die KI-Schritte (GPT-4.1 + GPT-4.1 mini + Knowledge) überschreiten das Warte-Zeitlimit → Transport-Timeout, obwohl der Flow-Lauf grün ist.
  - **Fix (zu bauen)**: `Respond to the agent` **direkt nach dem Trigger** platzieren (sofortige OK-Antwort), KI-Verarbeitung + Switch + E-Mail laufen danach. Stufe 1 liest keinen Flow-Output → Entkopplung verlustfrei. **Zu verifizieren**: ob Aktionen nach `Respond to the agent` im Agent-Trigger weiterlaufen; falls nicht → KI-/E-Mail-Teil als fire-and-forget child flow auslagern.
  - **Diagnose vor dem Fix**: Gesamtdauer des Flow-Laufs in der Run History prüfen (>60–100 s = Timeout bestätigt; <20 s = ggf. transienter Gateway-Aussetzer, erst Wiederholung testen).
  - **Status 2026-08-10**: Fix noch nicht umgesetzt — muss vor Praxistest erledigt sein.
- [ ] **Praxistest per echtem Anruf** (Teams Phone), sobald der v2-Agent mit der AIRCO-Nummer verbunden ist

### H. Power Automate — Vereinfachung ✅ erledigt
- [x] **Eine E-Mail-Aktion mit inline-if-Routing** (2026-07-20): Von Anfang an so implementiert — kein Switch, kein 5-fach-Duplikat. Routing über `if()` im Empfänger-Feld, Body-Pflege an einer einzigen Stelle.

---

## Offene Detailfragen — Stufe 1

### Conversation Start

- **Praxistest ausstehend**: Aussprache von „GmbH" in der Begrüßung noch
  nicht per echtem Anruf geprüft — Copilot-Studio-Testpanel spielt bei
  Basic-Voice-Agents nur Text ab, kein echtes Audio (siehe Quelle in
  Architektur.md Abschnitt 9, Punkt 1). Blockiert, bis der Agent mit der
  AIRCO-Telefonnummer (Teams Phone) verbunden ist. Falls Aussprache
  schlecht: Sprache-Text umformulieren (z. B. ohne „GmbH" oder phonetisch).

### Kundendaten erfassen

- **Praxistests Entity-Typen: erledigt (2026-07-21)** — Alle 3 Entities auf
  `StringPrebuiltEntity` („Gesamte Antwort des Benutzers") umgestellt:
  - `OrganizationPrebuiltEntity` für **Firmenname**: Praxistest ergab, dass
    Firmennamen ohne GmbH/AG nicht erkannt wurden → `StringPrebuiltEntity` (bereits
    2026-07-16 dokumentiert). ✅
  - `PersonNamePrebuiltEntity` für **Ansprechpartner**: Analog gescheitert an
    natürlicher Sprache („ich bin der Michael Doukas") → `StringPrebuiltEntity`. ✅
  - `PhoneNumberPrebuiltEntity` für **Telefonnummer**: → `StringPrebuiltEntity`. ✅
  
  **Neue Architektur (2026-07-21):** Bot sammelt alle 3 Felder als Rohtext
  (StringPrebuiltEntity). Die KI-Verarbeitung im Flow extrahiert saubere Werte
  (firmenname/ansprechpartner/telefonnummer) aus dem Rohtext → VarFirmenname /
  VarAnsprechpartner / VarTelefonnummer. Diese werden in der E-Mail verwendet.
  **Nebeneffekt**: Die Zusammenfassung im Bot zeigt noch den Rohtext
  (z.B. „mein Firma heißt mosaiic GmbH") — akzeptiert für Stufe 1; die E-Mail
  zeigt den bereinigten Wert.

- **Telefonnummer-Erfassung (2026-08-20, abgeschlossen)**: Caller-ID-Erkennung
  via Systemvariable ist im Teams Phone + Agents & Queues Kanal **nicht möglich**.
  Alle getesteten Variablen liefern entweder kryptische Teams-interne IDs oder
  leere Werte (Details in [Topics.md](Topics.md), Node 3). Entscheidung:
  Bot fragt Rufnummer **immer manuell** ab (`PhoneNumberPrebuiltEntity`).
  `Global.KanalLabel` wird **hardcodiert** auf `"Telefon"` gesetzt.
  Wenn Chat-Kanal hinzukommt: `Activity.ChannelId = "directline"` als
  Erkennungsmerkmal für Web-Chat nutzen — dann Bedingungsblock ergänzen.

### Inbetriebnahme

- Wortlaut entschieden: „Wurde die Anlage in den letzten 12 Monaten in
  Betrieb genommen?" → `Global.Inbetriebnahme`.
- Umgang mit „weiß nicht" (dritter Wert oder als Nein behandeln?) — analog
  zur gleichen offenen Frage bei „Vertragsfrage" unten.

### Vertragsfrage

- Wortlaut; Umgang mit „weiß nicht" (dritter Wert oder als Nein behandeln?).
- Eigene Sparring-Session erforderlich.

### Anliegen erfassen

- Wortlaut der offenen Frage — mit oder ohne Beispiele („z. B. eine Störung,
  eine Ersatzteilanfrage oder ein Rückrufwunsch")?
- Umgang mit sehr langen Monologen (Nachfassfrage? Bestätigungsschleife?).
- Eigene Sparring-Session erforderlich.

### Sicherheits-Eskalation

- **Zurückgestellt (2026-07-10)**: Michael priorisiert den Power Automate
  Flow zuerst. Bewusst in Kauf genommenes Risiko bis dahin: Ein
  Sicherheitsnotfall (Brand, Rauch) wird vom Bot vorerst wie jeder andere
  Anruf behandelt, es gibt keine Sofort-Eskalation.
- Trigger-Phrasen und Erkennungsmechanismus (Sprach-Trigger vs. aktive
  Nachfrage im Prio-Filter „Besteht Gefahr für Personen?") sind offen —
  siehe [Topics.md](Topics.md). Eigene Sparring-Session erforderlich.

### Zusammenfassung & Bestätigung (Kurzfassung)

- **Korrektur-Pfad entschieden** (siehe [Topics.md](Topics.md)):
  Closed-List-Frage nach dem falschen Feld → gezielte Korrektur-Nachfrage
  direkt im Topic (kein Redirect zum Ursprungs-Topic) → erneutes Vorlesen;
  max. 3 Korrekturversuche (Schwelle 2026-07-20 von 2 auf 3 erhöht), danach
  **v2:** Abschluss ohne Transfer (Flow
  „Anliegen weiterleiten (KI)" trotzdem aufrufen + Verabschiedung, siehe
  Topics.md).
- **Noch offen**:
  - **Synonym-Tabelle für die `Korrekturfeld`-Entity**: **erledigt 2026-07-20** —
    alle 5 Werte in Copilot Studio gepflegt (Firmenname, Ansprechpartner,
    Inbetriebnahme, Vertrag abgestimmt in [Topics.md](Topics.md) Node 6;
    Telefonnummer von Michael direkt ergänzt). „Anliegen" als 6. Entity-Wert
    **entfernt** (kein YAML-Zweig).
  - **Fallback bei nicht erkannter Angabe** (Node 7b) — `elseActions` des
    Korrekturfeld-Switch: **umgesetzt & getestet 2026-07-20**. Behebt den
    stillen Durchfall, wenn die Closed-List die genannte Angabe nicht erkennt.
  - **Korrektur-Fragen wurden übersprungen** (Firmenname/Ansprechpartner/
    Telefonnummer): Ursache = Question schreibt in bereits gefüllte
    Global-Variable → Copilot Studio überspringt. **Fix: `Blank()`-Node vor
    jeder Frage; umgesetzt, getestet & bestätigt 2026-07-20.** (Referenz-Merker
    im Session-Gedächtnis abgelegt.)
  - ~~**[KRITISCH]-E-Mail-Inhalt beim Prio-Filter**~~ — entschieden
    (2026-07-10): `Inbetriebnahme` und `VertragVorhanden` werden mit in die
    [KRITISCH]-Mail aufgenommen, da sie zu diesem Zeitpunkt bereits erfasst
    sind (siehe Topics.md, Topic „Prio-Filter", und Architektur.md
    Abschnitt 5, E1).
  - ~~Ob die Geschäftszeiten-Prüfung des Prio-Filters auch bei dieser
    Eskalation (3× Widerspruch) gelten soll~~ — entschieden: ja, gleiche
    Prüfung (siehe Topics.md und Abschnitt „Eskalation / Unterhaltung
    übertragen" unten).

### Power Automate Flow „Anliegen weiterleiten"

- **KI-Verarbeitung (v2, 2026-07-14)**: Der Flow verarbeitet den Freitext
  jetzt **per KI** — aber nicht als Kategorie-Klassifizierung A–F, sondern als
  **intelligente Zusammenfassung + Kritikalität** (Kritisch/Hoch/Mittel/Niedrig)
  anhand der vom Kunden gelieferten Störungsliste. Die Kategorie-Klassifizierung
  A–F bleibt Stufe-2-Material. (Der frühere Stufe-1-Grundsatz „kein KI-Aufruf
  im Flow" ist mit v2 überholt.)
- ~~E-Mail-Vorlage (Betreff-Schema mit Vertragsstatus, Body-Aufbau)~~ — **erledigt
  (2026-07-13)**: Betreff `[KRITISCH] Ticket – {Firmenname}` / `Ticket – {Firmenname}`;
  Body mit Datum/Uhrzeit, allen 7 Variablen, boolean→Ja/Nein-Expressions,
  Anliegen-Leerprüfung. Manuell getestet, E-Mail korrekt angekommen.
- **⚠ VOR PRODUKTIVSETZUNG PFLICHT — 4 Postfächer je Kritikalität (v2)**:
  Das Routing ist nicht mehr binär (Produktionsstillstand ja/nein), sondern per
  `Switch` auf die KI-Kritikalität an eines von vier Postfächern:
  - **Kritisch** → Postfach A
  - **Hoch** → Postfach B
  - **Mittel** → Postfach C
  - **Niedrig** → Postfach D
  Konkrete Adressen von Michael/AIRCO bereitstellen (Gruppen- oder
  Einzeladressen). Aktueller Platzhalter überall: `michael.doukas@mosaiic.com`.
- **Neu (2026-07-09)**: Copilot Studio zeigt beim Verknüpfen des Flows als
  Tool die Warnung „In dieser Umgebung ist nicht genügend Copilot-Guthaben
  vorhanden" (Flowprüfung) — betrifft die Laufzeit-Kapazität für
  Flow-Ausführungen, nicht die Konfiguration. Bei ~40 Anrufen/Tag
  potenziell ein Lizenz-/Kostenthema. Michael muss das mit AIRCO/dem
  M365-Lizenzverantwortlichen vor Produktivbetrieb klären (Copilot-Guthaben
  pro Umgebung, Kosten pro Flow-Ausführung).
- **⚠ Trigger-Input-Reihenfolge nicht verändern**: Der Copilot-Studio-Trigger in Power
  Automate benennt Inputs generisch nach Typ + Reihenfolge. Aktuelle Zuordnung
  (Stand 2026-07-20, bestätigt per Flow-Code):
  `text`=Firmenname, `text_1`=Ansprechpartner, `text_2`=Telefonnummer,
  `text_3`=Anliegen, `text_4`=KanalLabel, `boolean`=VertragVorhanden, `boolean_1`=Inbetriebnahme.
  Reihenfolge **niemals ändern**, sonst müssen alle Flow-Ausdrücke manuell angepasst werden.
- ~~Eigene Sparring-Session erforderlich~~ — **erledigt (2026-07-13)**.

### Eskalation / Unterhaltung übertragen — ⚠ ENTFÄLLT IN v2

> **v2 (2026-07-14):** Kein Transfer an einen Menschen mehr. Dieser gesamte
> Abschnitt (Zielrufnummer, Geschäftszeiten-Prüfung, Praxistest des
> Transfer-Nodes) ist damit **obsolet** und nur noch v1-Referenz (Backup-Ordner).
> Für v2 offen ist das Ersatzverhalten ohne Transfer — siehe „v2-spezifische
> offene Punkte" unten und Architektur.md Abschnitt 9.

- **Node-Typ geklärt**: Der Copilot-Studio-Node heißt „Unterhaltung
  übertragen" (Themenverwaltung → Transfer type = „Externe
  Telefonnummer-Übertragung"), nicht „Transfer to Agent" (nur unser
  Arbeitsbegriff).
- **Geschäftszeiten für „Prio-Filter" entschieden**: Mo–Fr 8–17 Uhr, per
  Power Fx geprüft (`Weekday(Now(), StartOfWeek.Monday) <= 5 && Hour(Now())
  >= 8 && Hour(Now()) < 17`). Außerhalb: kein Transfer-Versuch, nur Flow +
  Ansage + Gesprächsende. Siehe [Topics.md](Topics.md), Topic „Prio-Filter".
- **Noch offen**:
  - **⚠ VOR PRODUKTIVSETZUNG PFLICHT — Zielrufnummer**: In keinem
    Quelldokument (`/Daten`) ist eine echte Telefonnummer hinterlegt (nur
    Platzhalter „+49..."). Michael muss bei AIRCO klären, wer Anrufe
    entgegennehmen soll (Kandidat laut Prozessdokumentation: Robert Wallner,
    dort als Eskalationskontakt für Service-Innendienst-Probleme genannt,
    aber nicht ausdrücklich für Telefon-Eskalationen bestätigt) und dessen
    Durchwahl. Platzhalter im „Unterhaltung übertragen"-Node durch echte
    Nummer ersetzen.
  - Ob die Geschäftszeiten-Prüfung auch bei Sicherheits-Eskalation und der
    2×-unerkannt-Regel gelten soll (für „Prio-Filter" und die
    3×-Widerspruch-Eskalation aus „Zusammenfassung & Bestätigung" bereits
    entschieden: ja) — siehe Architektur.md Abschnitt 3 und 7.
  - **Praxistest ausstehend**: „Unterhaltung übertragen" (Transfer-Node) lässt
    sich im textbasierten Test-Panel nicht prüfen (kein echter Anruf
    vorhanden, Konversation bleibt beim Erreichen des Nodes ohne
    Fehlermeldung stehen — beobachtet beim Prio-Filter-Test am 2026-07-08).
    Muss über einen echten Anruf via Teams Phone getestet werden, sobald der
    Agent mit der AIRCO-Telefonnummer verbunden ist (gleicher Blocker wie bei
    „Conversation Start" oben).

### v2-spezifische offene Punkte (2026-07-14)

- ~~**Postfach-Adressen**~~ — **Interim entschieden (2026-07-20)**: vorerst alle Kritikalitätsstufen → `Service@airco-systems.de`. Differenzierung auf 4 Postfächer folgt später.
- **Störungs-/Anliegenliste**: Format + Einbindung in den KI-Schritt
  (Prompt-Text vs. Datei/Tabelle im Flow vs. Wissensquelle); Kunde liefert sie
  noch.
- **KI-Werkzeug + Prompt**: AI Builder „Text mit GPT erstellen" vs. Connector;
  Kritikalitäts-Kriterien reproduzierbar formulieren (Orientierung:
  Prioritätsmatrix in CLAUDE.md); Modell / Kosten / Datenschutz-Region.
- **Transfer-Nebenwirkungen ohne Eskalation**: Verhalten bei Sicherheitsnotfall,
  Englisch-Sprecher, 2× unerkannter Eingabe, 3 erschöpften Korrekturversuchen.
- **Duplizierter v2-Flow**: alten `Produktionsstillstand`-Input entfernen;
  prüfen/bestätigen, dass die `flowId` im v2-Agenten auf die **Kopie** zeigt.
- **Anweisungen-Text (Agent)**: Englisch-Regel verweist noch auf den entfallenen
  Transfer — neu formulieren (siehe Architektur.md Abschnitt 2).

---

## Offene Detailfragen — spätere Stufen

### Stufe 1.5 / 1.6

- Planner vs. SharePoint-Liste — was passt zum Arbeitsablauf des Innendiensts?
- Odoo-Connector (Standard-Connector, HTTP/REST, oder Weg über die
  SharePoint-Liste aus 1.5)?
- Vertragsfrage durch automatischen Abgleich (SharePoint-Liste /
  Odoo-Kundenstamm) ersetzen?

### Stufe 2

- **Synonym-Tabelle für `Kategorie`-Entity**: Michael liefert später eine Tabelle
  (analog `Anlagentyp`) mit realistischen Formulierungen pro Kategorie
  (Störung, Wartung, Ersatzteil, Angebot, Rückruf). Die in Stufe 1 gesammelten
  Freitext-Anliegen liefern dafür reale Formulierungen.
- Kategorie-Topics (Störung, Wartung, Ersatzteil, Angebot, Allgemeine Anfrage):
  noch nicht erarbeitet.
- Rückrufbitte: Design erarbeitet (siehe [Topics.md](Topics.md)); offen bleibt,
  ob eine Rückrufbitte ein Ticket erzeugen soll oder nur eine E-Mail genügt.
- Zusammenfassung & Bestätigung: kategoriespezifische Felder,
  Variablen-Sichtbarkeit (siehe Architektur.md Abschnitt 8), getrenntes oder
  gemeinsames Vorlesen von Global- und Topic-Variablen.
- Reihenfolge Entity-Reprompt / Fallback „Allgemeine Anfrage" /
  2-Versuche-Eskalation bei „Kategorie auswählen".
