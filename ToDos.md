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
| Kundendaten erfassen | **erledigt** — redirectet zu `Anlagenerfassung`; Caller-ID-Block entfernt (2026-08-20); Telefonnummer immer manuell; `KanalLabel` hardcodiert `"Telefon"`. **(2026-09-02, Robustheit-Fix):** Alle 3 Entities auf `StringPrebuiltEntity` umgestellt (Firmenname bereits, jetzt auch Ansprechpartner + Telefonnummer) — `PersonNamePrebuiltEntity` übersetzte Namen ins Englische, `PhoneNumberPrebuiltEntity` lehnte ausländische/dialektale Nummern ab. `Global.TelefonnummerGesprochen` + zugehöriger `SetVariable`-Node entfernt (nicht mehr benötigt: speak-Feld der Zusammenfassung nutzt jetzt `{Global.Telefonnummer}` direkt, TTS liest Rohtext korrekt vor). `allowBargeIn: false` in allen prompt-Feldern gesetzt. (YAML: `YAML/Kundendaten_erfassen.md`) |
| **Anrufgrund erfassen** (NEU, 2026-09-02) | **erledigt** — neues Topic zwischen `Kundendaten erfassen` und `Anlage erfassen`. ClosedList-Entity `mosaiic_AIRCOTelefonBot.entity.Anrufgrund` angelegt (5 Items + Synonyme inkl. Zahlen 1–5; „Sonstiges" als Synonym entfernt, da Duplikat des Item-Namens). Condition-IDs: `u9xatS` = stoerung, `gShHbV` = wartung. Nur bei Störung/Wartung folgt `Anlage erfassen`; sonst direkt `Anliegen erfassen`. `Kundendaten erfassen` redirectet jetzt auf dieses Topic. `Global.Anrufgrund` als `text_6` in beide InvokeFlowAction-Nodes der Zusammenfassung + Sicherheitsnetz eingetragen. (YAML: `YAML/Anrufgrund erfassen.md`, `YAML/Email Body.html`) |
| **Anlage erfassen** (NEU) | **erledigt** — neues Topic: eine Frage (StringPrebuiltEntity, `allowInterruption: false`): „Bitte teilen Sie uns die Anlagenbezeichnung, Seriennummer und das Baujahr mit." → `Global.Anlage`; redirectet zu `Anliegenerfassen`. **(2026-09-02):** `allowBargeIn: false` ergänzt — verhindert Doppel-Frage durch TTS-Echo im Voice-Kanal. (YAML: `YAML/Anlage erfassen.md`) |
| Anliegen erfassen | **erledigt** — identisch mit Stufe 1; redirectet zu `Zusammenfassung-Slim` statt `Zusammenfassung` (YAML: `YAML/Slim/Anliegen erfassen.md`) |
| Zusammenfassung & Bestätigung (Slim) | **erledigt** — vereinfacht: Summary + Frage in einem einzigen Question Node kombiniert; bestätigt nur Kontaktdaten (Anlage + Anliegen unbestätigt); Korrekturfeld-Entity auf 3 Optionen reduziert (Firmenname / Ansprechpartner / Telefonnummer); Fallback bei nicht erkannter Angabe: Zähler inkrementieren + zurück zur Korrekturfeld-Frage (korrigiert 2026-08-18); flowId: `019885f0-e29a-f111-b8db-7ced8d476627`. **(2026-09-02, Robustheit-Fix):** Beide `InvokeFlowAction`-Nodes: `If(IsBlank(...), "nicht angegeben", ...)` für Firmenname, Ansprechpartner, Telefonnummer — verhindert Flow-Fehler bei leeren Feldern. `speak`-Feld: `{Global.TelefonnummerGesprochen}` → `{Global.Telefonnummer}`. Korrektur-Zweige: `PersonNamePrebuiltEntity` → `StringPrebuiltEntity` (Ansprechpartner), `PhoneNumberPrebuiltEntity` → `StringPrebuiltEntity` (Telefonnummer). **(2026-09-02, Sicherheitsnetz):** `SetVariable Global.FlowAufgerufen = true` direkt vor beiden `InvokeFlowAction`-Nodes eingefügt — verhindert Doppel-Mail durch das Sicherheitsnetz in „Ende der Unterhaltung". (YAML: `YAML/Zusammenfassung - slim.md`) |
| **System-Topics: Sicherheitsnetz bei Gesprächsabbruch** (NEU) | **erledigt (2026-09-02)** — **„Ende der Unterhaltung"** (`OnSystemRedirect`, `CancelOtherTopics`): Vor `EndConversation` neuer ConditionGroup-Node: Bedingung `Not(IsBlank(Global.Telefonnummer)) && Not(Global.FlowAufgerufen)` → `InvokeFlowAction` (flowId `019885f0-e29a-f111-b8db-7ced8d476627`) mit `If(IsBlank(...))` Wrappern für alle 6 Felder (Anliegen-Fallback: „Gespräch vorzeitig beendet – Anliegen nicht erfasst"; Anlage-Fallback: „nicht erfasst"). Feuert bei jedem Gesprächsende — auch bei Caller-Hang-Up via Teams-Phone-Signal. **„Stille-Erkennung"**: `EndDialog` → `BeginDialog → EndofConversation` ersetzt — stellt sicher, dass auch lautlose Abbrüche das Sicherheitsnetz durchlaufen. **Globale Variable `Global.FlowAufgerufen`** (Boolean): verhindert Doppel-Mail wenn Flow bereits regulär aufgerufen wurde. (YAML: `YAML/Ende der Unterhaltung.txt`, `YAML/Stille-Erkennung.txt`) **⚠ Bekannte Lücke (MCP-Review 2026-09-03):** Tritt der Fehler auf, bevor `Global.Telefonnummer` erfasst wurde (z. B. Laufzeitfehler während „Kundendaten erfassen"), greift das Sicherheitsnetz nicht — kein Flow-Aufruf, keine E-Mail, das Anliegen geht komplett verloren. Noch nicht behoben/entschieden. |
| **Power Automate Flow „Anliegen weiterleiten – slim"** | **erledigt (2026-08-18)** — 6 Text-Inputs (keine Booleans); Trigger: Firmenname/Ansprechpartner/Telefonnummer/Anliegen/KanalLabel/Anlage. KI-Verarbeitung: JSON-Schema um `anlagenbezeichnung`, `serialnummer`, `baujahr` (optional) erweitert; `kritikalitaet`-Enum auf `"Sonstiges"` (statt `"Other"`) umgestellt; `Anlage` als Kontext-Input im Prompt. E-Mail-Body: Tabellenzeile „Anlage" (`triggerBody()?['text_5']`) hinzugefügt, Inbetriebnahme/Wartungsvertrag entfernt, Rohtext-Duplikat entfernt. `Respond to the agent` direkt nach Trigger (Timeout-Fix). Fallback-E-Mail bereits korrekt (Anlage vorhanden, keine Booleans). Prioritätsformel unverändert korrekt (`Sonstiges` fällt in Low-Fallback). **⚠ Korrektur (2026-09-04):** Das tatsächliche JSON-Schema (`YAML/Agent Definition - KI-Verarbeitung.md`) enthält **kein** `anlagenbezeichnung`/`serialnummer`/`baujahr` und das Enum lautet `"Other"`, nicht `"Sonstiges"` — die Felder wurden später zugunsten von `anrufgrund` wieder vereinfacht. **⚠ Abgelöst (2026-09-04):** Flow wegen Microsoft-Designer-Bug nicht mehr bearbeitbar → als Agent-Flow **`Ticketerstellung`** neu aufgebaut (Details + bewusste Abweichungen: `Konzept-Ausfallsichere-Weiterleitung.md`, Abschnitt „Neuaufbau 2026-09-04"). **flowId neu: `834c8025-3da8-f111-b8dd-70a8a52f67fc`**; alle 3 `InvokeFlowAction`-Nodes umgehängt und per Pull verifiziert (2026-09-04). Alter Flow + verwaistes Tool `Anliegenweiterleiten-slim` werden nach grünem End-to-End-Test gelöscht. |
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
| E-Mail-Routing + Body (eine Aktion, inline-if) | **erledigt (2026-07-21, aktualisiert)** — Inline-`if()` im Empfänger-Feld: Kritisch→A, Hoch→B, Mittel→C, Rest→D. HTML-Body: Tabelle — Firmenname/Ansprechpartner/Telefonnummer jetzt via `variables('VarFirmenname'/'VarAnsprechpartner'/'VarTelefonnummer')` (KI-bereinigt); Inbetriebnahme/Wartungsvertrag weiter via `triggerBody()?['boolean…']`; `variables('VarZusammenfassung')` + Zeitstempel/Kanal + Rohtext (Testperiode). Fallback-Node „E-Mail senden 2" bei KI-Ausfall (Configure run after: Failed/TimedOut/Skipped). **Postfach-Adressen: endgültig entschieden (2026-09-04) — dauerhaft eine Mailbox `service@airco-systems.de` für alle Stufen, kein Routing auf 4 Postfächer mehr geplant.** |
| Teams Phone einrichten (Agent mit Festnetznummer verbinden) | **erledigt (2026-09-04)** — Ansatz: **Agents & Queues** (kein Auto Attendant). Komplett im Teams Admin Portal eingerichtet. Vorläufige Testnummer: **+4961712779569** (gekauft vom Provider **easybell**), mit dem Agenten verbunden. Nach Abschluss der Testphase wird die eigentliche AIRCO-Hotline-Nummer auf diese Nummer weitergeleitet. |

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
- [x] **Telefonnummer-Präfix +49/+41 (2026-09-02):** KI-Node-Prompt um Regel ergänzt: Wenn
  Nummer mit Ländervorwahl ohne Plus beginnt (z.B. „49", „41", „43", „33"), wird
  `+` vorangestellt. Hintergrund: Speech-to-Text transkribiert „+49" nicht mit
  Plus-Zeichen — der KI-Node normalisiert den Wert nachgelagert.

### E. Power Automate — Routing + E-Mail
- [x] **Betreff-Format**: `[Kritikalität] - Firma: Firmenname` — Zusammenfassung im Body, nicht im Betreff.
- [x] **Routing: inline-if im Empfänger-Feld** (2026-07-20): eine einzige E-Mail-Aktion, kein Switch. `if(VarKritikalitaet == 'Kritisch') → A, Hoch → B, Mittel → C, Rest → D`. Betreff-Präfix + E-Mail-Priorität (High/Normal/Low) ebenfalls inline. Referenzen auf `body('agent-…')` wurden durch `variables('VarKritikalitaet')` ersetzt (kein GUID-Hardcode mehr).
- [x] **Body** (HTML, eine E-Mail-Aktion, 2026-07-20): Tabelle Kundendaten + `variables('VarZusammenfassung')` + Zeitstempel/Kanal + Rohtext (Testperiode). Booleans per `if(triggerBody()?['boolean/boolean_1'], 'Ja', 'Nein')`.
- [x] **Fallback-Node „E-Mail senden 2"** (2026-07-20): Configure run after: Failed/TimedOut/Skipped auf KI-Node. Rohtext + Hinweis „manuell prüfen" → Postfach D (fest). Nur `triggerBody()`-Werte, keine Variablen.
- [x] **Postfach-Adressen**: **Entschieden (2026-09-04, endgültig)** — dauerhaft eine Mailbox `service@airco-systems.de` für alle Kritikalitätsstufen, kein Routing auf 4 getrennte Postfächer mehr geplant.

### F. Kunden-/AIRCO-Input (Blocker für D/E bzw. Go-Live)
- [ ] **Störungs-/Anliegenliste** vom Kunden (Grounding für den KI-Schritt) — **auf spätere Stufe verschoben (2026-08-25)**, siehe oben.
- [x] **4 Postfach-Adressen** — Interim: alle → `Service@airco-systems.de` (2026-07-20); späteres Routing auf 4 Adressen offen
- [x] **Lizenz / Copilot-Guthaben** für AI Builder bzw. Flow-Ausführungen — **erledigt (2026-08-25)**: AIRCO hat den Pay-As-You-Go-Abrechnungsplan eingerichtet, Testbetrieb (~40 Anrufe/Tag) ist möglich.

### F2. Technische Pflichtprüfungen vor Go-Live (2026-08-10 ergänzt)
- [x] **LLM = GPT-4.1**: **Entscheidung (2026-08-25): bewusst nicht erneut geprüft.** Michael möchte die aktuell funktionierende Konfiguration nicht durch eine Kontrolle in den Copilot-Studio-Einstellungen riskieren. Bleibt unverändert, solange alles läuft — bei künftigen Problemen mit der Sprachausgabe als erste Stelle zum Prüfen im Hinterkopf behalten.
- [x] **`Respond to agent` direkt nach Trigger** im Flow: Fix für `FlowActionBadGateway`-Timeout — umgesetzt im Slim-Flow (2026-08-18) **und im Stufe-1-Flow (2026-08-25)**. Testanruf bestätigt: kein Hänger/Fehlercode mehr, E-Mail kommt zuverlässig an → behebt Testfeedback #15 und #18 (siehe „Testing-Feedback" unten).
- ~~**Re-Zusammenfassung `text`-Feld**~~ — **entfällt (2026-09-04)**: betraf ausschließlich das Stufe-1-Topic „Zusammenfassung" (nicht Slim). Dieses Topic ist abgelöst und wird nicht mehr umgesetzt — nur noch die Slim-Variante ist relevant, die den Punkt ohnehin nicht betrifft (siehe Topics.md).
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
  **Zusätzlich (2026-09-02) explizites Sicherheitsnetz implementiert** (Architektur.md
  Abschnitt 9, Punkt 21 erledigt): „Ende der Unterhaltung" ruft den Flow mit
  vorhandenen Daten auf, wenn `Global.Telefonnummer` gesetzt und `Global.FlowAufgerufen`
  noch false ist — deckt alle Abbruch-Szenarien ab (Caller hängt auf, Stille-Timeout,
  Laufzeitfehler via „Bei Fehler").
  **Nachbessserung (2026-09-02, Kundentermin-Feedback):** Lücke geschlossen: Wenn
  `InvokeFlowAction` in der Zusammenfassung fehlschlägt, war `Global.FlowAufgerufen`
  bereits `true` gesetzt (vor dem Flow-Aufruf) → Sicherheitsnetz griff nicht.
  Fix: In „Bei Fehler" `SetVariable Global.FlowAufgerufen = false` **vor** dem
  `BeginDialog → EndofConversation` eingefügt (außerhalb der ConditionGroup,
  greift in beiden Zweigen) — Sicherheitsnetz kann jetzt auch nach einem
  Flow-Fehler greifen. `speak`-Text angepasst: statt „Bitte versuchen Sie
  es später erneut" nun „Ihr Anliegen wurde gespeichert und ein Mitarbeiter wird sich
  bei Ihnen melden." **Nur im Produktionszweig (`elseActions`)** — der
  `InTestMode = true`-Zweig behält bewusst den alten `speak`-Text (Michael,
  2026-09-03: Testpanel-Ausgabe ist nicht produktionsrelevant, keine Angleichung nötig).
  Verifiziert per Datei-Review 2026-09-03 (`agent/AIRCO Telefon-Bot/topics/OnError.mcs.yml`,
  vormals `YAML/Bei Fehler.txt` — Datei entfernt, da veraltet gegenüber dem
  synchronisierten Agent-Ordner) — Platzierung und beide Textstände bestätigt.
  **MCP-Review (2026-09-03):** 0 Validierungsfehler, lokal synchron mit dem
  Draft in Dataverse. ✅
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
  Entschuldigung sauber beenden. **(2026-09-02, umgesetzt):** `CancelAllDialogs`
  ersetzt durch `BeginDialog → EndofConversation` — beendet den Anruf zuverlässig
  statt den Bot in der Luft hängen zu lassen. `speak`-Text in beiden Zweigen
  bereits korrekt: „Es tut uns leid, es ist ein technischer Fehler aufgetreten.
  Bitte versuchen Sie es später erneut. Auf Wiederhören." Telemetrie-Logging
  (`LogCustomTelemetryEvent`) bleibt erhalten. (YAML: `agent/AIRCO Telefon-Bot/topics/OnError.mcs.yml`) ✅
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
  vollständig durch. Bestätigt: Barge-in-Problem bei 0,2 behoben.
  **Nachjustiert (2026-09-04)**: Wert von 0,2 auf **0,3** angehoben (Gegenprobe
  auf leise sprechende Anrufer) — im laufenden Testbetrieb mit echten Anrufen
  über die neue Testnummer bisher **positive Ergebnisse**, keine neuen
  Barge-in- oder Erkennungsprobleme gemeldet. Weiterhin im Testbetrieb zu
  beobachten.

### F4. Absturz-Analyse 2026-09-04 — bestätigte Ursache + Guard-Konvention

Ausgangspunkt: Anruf vom 2026-09-04, 05:26 Uhr. Bot bricht direkt nach der
Anrufgrund-Antwort („Störung") mit „Es tut uns leid, es ist ein technischer
Fehler aufgetreten" ab. E-Mail kam trotzdem an (Sicherheitsnetz griff).

- [x] **Ursache bestätigt (Dialog-Trace `Tests/Test 4/dialog.json`)**: Knoten
  `pBAscP` in `Zusammenfassung-Slim`, `InvokeFlowAction`:
  > „Der erforderliche Parameter `text_5` im Flow `Anliegen weiterleiten - slim`
  > (`019885f0-…`) hat in der Aktion `Call Flow` einen leeren oder keinen Wert."

  `text_5` war roh an `=Global.Anlage` gebunden. Beim Robustheit-Fix vom
  2026-09-02 wurden nur Firmenname/Ansprechpartner/Telefonnummer abgesichert —
  `Anlage` kam als neues Feld später dazu und wurde übersehen.
  **Schweregrad:** Bei 3 von 5 Anrufgründen (Ersatzteil, Rückruf, Sonstiges)
  wird „Anlage erfassen" *by design* übersprungen → `Global.Anlage` leer →
  **jeder** solche Anruf crasht an der Zusammenfassung. Nur das Sicherheitsnetz
  verhinderte bisher Datenverlust.
  **Erklärt außerdem**, warum im Power-Automate-Ausführungsverlauf **kein Lauf**
  zu finden war: Der Fehler entsteht client-seitig in Copilot Studio, *bevor*
  der Flow gesendet wird (= Lektion 1 aus dem Session-Gedächtnis).

- [x] **Guard-Konvention eingeführt und umgesetzt (2026-09-04)**: Flow-Eingaben
  binden **nie** direkt eine Variable, sondern immer
  `=If(IsBlank(Global.X), "<Platzhalter>", Global.X)`. Platzhalter muss echter
  Text sein — `""` scheitert erneut, da `IsBlank("") = true`.
  Einzige Ausnahme: `=System.Conversation.Id` (Systemvariable, immer gesetzt).
  Platzhalter: Firmenname/Ansprechpartner/Telefonnummer/Anrufgrund →
  „nicht angegeben"; Anlage/Anliegen → „nicht erfasst"; KanalLabel → „Telefon".
  `Global.Anrufgrund` braucht zusätzlich `Text(...)` (ClosedList-Wert).
  **22 Bindungen in 5 Dateien** korrigiert: `Kundendatenerfassen`,
  `Anrufgrunderfassen`, `Anlagenerfassung`, `Anliegenerfassen`,
  `Zusammenfassung-Slim` (beide Flow-Knoten). Validierung: 33 Dateien,
  0 Fehler / 0 Warnungen. Push in den Draft erfolgt. **Veröffentlichen steht
  noch aus.**
  **Bewusst ausgenommen**: `Zusammenfassung.mcs.yml` (Stufe-1-Topic, laut
  Michael veraltet und nicht mehr erreichbar) — dort bleiben 14 Bindungen
  ungeschützt.

- [x] **GELÖST (2026-09-04) — Reihenfolge-Anomalie**: Nach „Störung" wird **„Anliegen" vor
  „Anlage"** gefragt, statt Anrufgrund → Anlage → Anliegen. Reproduziert im
  **Testpanel** (`Tests/Test 3`) *und* am Telefon — also **kein**
  Sprachkanal-Problem. Der Topic-Graph ist nachweislich korrekt (alle drei
  Topics `OnRedirect`, keine Trigger-Phrasen, IDs zeigen laut Trace auf die
  richtigen Inhalte). Im Trace fällt auf, dass `invokeFlowAction_nOBDGN` +
  `lzTpPJ` (Ende von „Kundendaten erfassen") **erst nach** der beantworteten
  Anliegen-Frage protokolliert werden und `question_anrufgrund` dort ein
  zweites Mal ohne Ausgabe läuft. **Arbeitshypothese**: Die asynchronen
  Staging-Flow-Aufrufe mitten in der Redirect-Kette führen zu einem
  Wiedereintritt in den Dialog-Stack. **Noch nicht getestet** —
  siehe Warnung unten.
  **Nachtest 2026-09-04 (`Tests/Test 5`, Chat, nach dem Guard-Fix)**: Absturz
  weg (0 Exceptions, Gespräch läuft bis `EndConversation` durch, Sicherheitsnetz
  nimmt korrekt den `else`-Zweig → keine Doppel-Mail). Reihenfolge weiterhin
  falsch, und der Trace zeigt das Muster eindeutig: Jede Anomalie sitzt
  unmittelbar hinter einem `InvokeFlowAction`. Beim ersten Durchlauf wertet die
  `ConditionGroup` `Global.Anrufgrund` noch als leer → `else` → Anliegen; nach
  Rückkehr des Flow-Aufrufs wird der Dialog erneut betreten, die Bedingung
  ergibt `true` → Anlage. Bereits gefüllte Questions laufen dabei stumm durch
  (Pos. 59000: `question_RZHeKB` ohne Ausgabe).
  **Vermutete Grundursache gefunden**: Der Flow **„Staging schreiben"**
  (`a0a99959`) antwortet dem Bot **erst nach** `Elemente_abrufen` (SharePoint)
  **und** `Bedingung` — im Gegensatz zum Slim-Flow (`019885f0`), bei dem
  `Respond_to_the_agent` seit 2026-08-18 **direkt nach dem Trigger** steht.
  Der Bot wartet also bei jedem der vier Staging-Aufrufe auf zwei
  SharePoint-Roundtrips. **Fix umgesetzt (2026-09-04, Michael im
  Power-Automate-Designer)**: `Respond_to_Copilot` steht jetzt direkt hinter
  dem Trigger; serverseitig per Pull verifiziert (`<TRIGGER> →
  Respond_to_Copilot → Elemente_abrufen → Bedingung`). Gleiche Medizin wie
  beim Slim-Flow. Unbedenklich
  bzgl. Daten, da die Response leer ist (`body: {}`) und alle vier
  `InvokeFlowAction`-Knoten `output: {}` haben.
  ⚠ **Nebenwirkung beachten**: Das synchrone Warten serialisiert aktuell die
  vier Staging-Upserts (get → if → update/create auf `Conversation.Id`).
  Ohne Warten können zwei Aufrufe theoretisch gleichzeitig „nicht gefunden"
  sehen und **doppelte SharePoint-Einträge** anlegen. Zwischen den Aufrufen
  liegt je ein Nutzer-Turn, das Risiko ist also gering — aber nach der
  Umstellung auf Duplikate in der Staging-Liste achten.

  ⚠ **Achtung bei diesem Test**: Ein Löschversuch des `InvokeFlowAction`-Knotens
  in „Anrufgrund erfassen" am 2026-09-04 hat **nicht** gegriffen — der Designer
  hat den Knoten lediglich umbenannt (`invokeFlowAction_stgAnr` → `2FRjW9`),
  er existiert mit identischen Bindungen weiter. Vor dem Nachtest per Pull
  verifizieren, dass die Löschung wirklich im Draft angekommen ist.

  ### ✅ Auflösung (2026-09-04, per Pull verifiziert)

  **Bestätigte Ursache**: die `InvokeFlowAction`-Aufrufe des Staging-Flows
  **im Frageablauf**. Michael hat sie in `Kundendaten erfassen`,
  `Anrufgrund erfassen`, `Anlage erfassen` und `Anliegen erfassen` entfernt und
  nur die in `Zusammenfassung - slim` und `Ende der Unterhaltung` belassen —
  **danach lief die Reihenfolge korrekt, im Chat-Test *und* am Telefon.**

  **Widerlegte Zwischenschritte** (nicht erneut verfolgen):
  - *Position des Knotens im Topic*: Verschieben an den Topic-Anfang half
    nicht, sondern verschob den Fehler nur — die Anrufgrund-Frage kam danach
    **doppelt** (echter Pfad kehrt zurück, `init:Global.Anrufgrund` setzt
    zurück, Frage wird erneut gestellt).
  - *`Respond_to_Copilot` als erste Aktion*: **kein** Unterscheidungsmerkmal —
    `Staging schreiben` **und** `Ticketerstellung` machen das beide, nur einer
    forkte.
  - *`flowKind: Stateless`*: im Export stehen **beide** Flows auf `Stateful`.
    Die entsprechende Notiz im Konzeptdokument ist veraltet.

  **Beweisführung für den `else`-Ast**: Die Anliegen-Frage ist nur über drei
  Wege erreichbar — `Fallback` (ausgeschlossen, würde „Entschuldigung…" ausgeben),
  `Anlage erfassen` (ausgeschlossen, Anlagenfrage kam nie) und den
  `elseActions`-Zweig von `Anrufgrund erfassen`. Damit war belegt, dass
  `Global.Anrufgrund` zum Auswertungszeitpunkt leer war.

  **Offene Hypothese für die Rückkehr des Stagings**: In
  `Zusammenfassung - slim` steht der Staging-Flow an der strukturell
  **identischen** Stelle (Flow → Question → ConditionGroup) und funktioniert.
  Einziger Unterschied: die Bedingung liest `Topic.korrekt`, nicht
  `Global.Anrufgrund`. Vermutung: **Global-Variablen werden über die
  Flow-Grenze hinweg nicht rechtzeitig persistiert, Topic-Variablen schon.**
  → Testbar, indem in `Anrufgrund erfassen` nach der Frage
  `SetVariable Topic.Grund = Global.Anrufgrund` gesetzt und die
  `ConditionGroup` auf `Topic.Grund` umgestellt wird; danach den Staging-Flow
  wieder an den Topic-Anfang setzen.

- [ ] **NEU/OFFEN (2026-09-04) — Fall 3 ist durch den Fix aktuell nicht mehr
  abgedeckt.** Es gibt nur noch **zwei** Staging-Aufrufpunkte
  (`Zusammenfassung - slim` Pos. 2 und das Safety-Net in
  `Ende der Unterhaltung`) statt der im Konzept beschriebenen fünf. Legt ein
  Anrufer während `Anrufgrund` / `Anlage` / `Anliegen` auf, existiert **kein
  SharePoint-Datensatz** → der Sweep findet nichts → **der Anruf geht
  verloren**. Das verletzt die Kundenforderung („sobald Firmenname,
  Ansprechpartner und Telefonnummer genannt sind, muss garantiert eine E-Mail
  raus"). Bei 5–10 Min Gesprächsdauer kein Randfall.
  → Priorität: vor der Hausmesse (16./17.09.). Lösungsweg siehe offene
  Hypothese oben.

- [ ] **OFFEN (2026-09-04) — Kein Heartbeat in der Korrekturschleife.**
  `GotoAction SLfuCH` zeigt weiterhin auf `question_Qt7GUF` statt auf den
  Staging-Knoten; `invokeFlowAction_4QAYoF` ist entfallen. Eine lange
  Korrekturschleife lässt `Modified` altern → der Sweep kann in ein noch
  laufendes Gespräch mailen. Bei 20 Min Karenz vertretbar, entspricht aber
  nicht P1 #2 des Konzepts.

- **Verworfene Hypothesen** (dokumentiert, damit sie nicht erneut verfolgt werden):
  1. ~~`Text(Global.Anrufgrund)` wirft zur Laufzeit, weil `Text()` keine
     Record-/Choice-Typen unterstützt~~ — **widerlegt**: In `Tests/Test 3` läuft
     `invokeFlowAction_stgAnr` mit gefülltem `Global.Anrufgrund`
     (`OptionDataValue`, `u9xatS`) mit leerem `exception`-Feld durch.
  2. ~~Das System-Topic „Fallback" (`OnUnknownIntent` → „Anliegen erfassen")
     unterbricht „Anrufgrund erfassen"~~ — **widerlegt**: Fallback gibt
     „Entschuldigung, das habe ich nicht richtig verstanden…" aus; dieser Satz
     taucht in keinem Protokoll auf, und im Trace gibt es keinen
     Fallback-Eintrag.

- **Merker Draft vs. Live**: Das Testpanel testet den **Draft**, jeder echte
  Anruf (Teams Phone) läuft gegen die **veröffentlichte** Version. Änderungen
  ohne Publish sind am Telefon nicht wirksam — mehrere Testanrufe dieser Session
  waren dadurch nicht aussagekräftig.

### F5. Leere E-Mail-Felder 2026-09-04 — Ursache: Stateless-Flow pollt KI nicht

**Symptom** (Chat-Test 04.09. 13:29, Momentaufnahme `Tests/Test 7`): E-Mail kam an,
aber Firmenname, Ansprechpartner, Telefonnummer, Anrufgrund, Kritikalität und
KI-Zusammenfassung waren **leer**. Gefüllt waren nur Anlage, Kanal und Rohtext.

**Trennlinie:** alles aus `triggerBody()?['text_N']` kam an, alles aus
`body('KI-Verarbeitung')?['structuredOutput/...']` war leer. Bot-Seite fehlerfrei —
`dialog.json` belegt korrekte Globals (mosaiic / michael / 14879 / Störung / PSAL 15).

**Ursache:** `Ticketerstellung` (834c8025) ist ein **Agent-Flow** mit
`flowKind: Stateless`. Der KI-Connector `shared_agentnode` ist long-running und
antwortet mit `HTTP 202` + `Location` + `Retry-After`; Body nur `{conversationId}`.
Stateful Cloud Flows pollen bis 200 — Stateless nicht. Die 202 wird als Endergebnis
gewertet: Aktionsstatus `SUCCEEDED`, `structuredOutput` nie vorhanden.
Der strukturell identische `Anliegen weiterleiten - slim` (019885f0, stateful)
lief deshalb korrekt. Details: Memory `reference-cps-stateless-async-ki`.

**Beweis erbracht 2026-09-04, 15:41 (echter Anruf, `Tests/Test 8`, Vorgang
`1848d2a6-c815-40b2-b5a8-d326534644db`):** Symptom identisch zu Test 7, diesmal
am Telefon. Rohdatenausgabe der Aktion `KI-Verarbeitung`:

```json
{ "statusCode": 202,
  "headers": { "Location": "https://.../agentnodes/conversations/fb30cf93-...",
               "Retry-After": "5" },
  "body": { "conversationId": "fb30cf93-0a06-4cae-848b-5f12d257f756" } }
```

Damit ist die Hypothese keine mehr. Weitere Belege desselben Laufs:
- `VarZusammenfassung` zeigt unter EINGABEN nur *Name* + *Typ*, **kein Feld
  „Wert"** → der Ausdruck wertete zu `null` aus. (`Keine Ausgaben` ist bei
  `InitializeVariable` normal und **kein** Befund.)
- Trigger 15:41:36, `VarZusammenfassung` 15:41:37, Start = Ende — der gesamte
  Flow inkl. „GPT-Analyse" lief in unter 1 s. Eine echte Auswertung dauert 5-20 s.
- Bot-Seite erneut entlastet: `Tests/Test 8/dialog.json` belegt korrekte Globals
  (mosaiic / Michael / 414981 / Störung / PSAL 14), `invokeFlowAction_s8uOdc`
  mit `exception: ""`; das Trigger-Schema in `workflow.json` deckt sich exakt
  mit der Bindung in `Zusammenfassung-Slim.mcs.yml:60`.
- **Kein Befund:** „Fehler beim Laden von Eingaben" am Trigger-Knoten. Ein
  reines Portal-/Anzeigeproblem (`InternalServerError` beim Blob-Abruf) —
  Aktionsknoten desselben Laufs zeigen ihre Eingaben normal. Nicht verfolgen.

**Nicht durch Ausdruckskorrektur behebbar** — das Ergebnis existiert zum
Zeitpunkt der E-Mail schlicht nicht.

**Offene Aufgaben:**
- [ ] Weg A testen (5 Min, Hypothese): `Respond to Copilot` ans Ende hinter
      `E-Mail senden bei Erfolg` verschieben. Falls es wirkt: Anrufer hört
      10–20 s Stille — nur als Diagnose brauchbar, nicht als Endlösung.
- [ ] Weg C (sicher): `Ticketerstellung` als **klassischen Cloud Flow** neu
      anlegen (Trigger „Wenn Copilot Studio den Flow aufruft"). Eingaben in
      exakter Reihenfolge: Firmenname, Ansprechpartner, Telefonnummer, Anliegen,
      KanalLabel, Anlage, Anrufgrund, ConversationId. Danach Tool in CPS
      aktualisieren + `invokeFlowAction_s8uOdc` in `Zusammenfassung-Slim`
      umhängen; alten Flow deaktivieren, nicht löschen.
      (Rückkonvertierung Agent → Cloud ist laut 03.09. nicht vorgesehen —
      vorher kurz im Portal prüfen.)
- [x] E-Mail-Template `YAML/Email Body.html` aktualisiert und im Flow eingesetzt:
      Kritikalität in Betreff **und** Body-Zelle wiederhergestellt (war in der
      Cloud-Version verschwunden), ConversationId als „Vorgang" in der Fußzeile.
- **Entscheidung 2026-09-04 (Nutzer):** Die KI-Ausgaben werden **bewusst
  beibehalten** — Firmenname, Ansprechpartner, Telefonnummer und Anrufgrund
  kommen weiter aus `structuredOutput`, nicht aus `triggerBody()`. Der zuvor
  erwogene Umbau auf Trigger-Werte ist damit **verworfen**; `VarFirmenname`
  und `VarAnsprechpartner` bleiben in Gebrauch, die `runAfter`-Kette bleibt
  unverändert. Folge: 6 von 7 E-Mail-Feldern hängen an der KI — der
  Stateless-Fix ist dadurch **blockierend**, nicht optional.
- [ ] **Fail-Safe schärfen (Priorität):** Nach `VarKritikalitaet` eine Bedingung
      `empty(variables('VarKritikalitaet'))` einziehen → wahr: `E-Mail senden bei
      KI fehlgeschlagen`, falsch: Erfolgszweig. Der bestehende `runAfter`-Zweig
      auf `[FAILED, TIMEDOUT, SKIPPED]` greift bei der 202 **nicht**, weil die
      Aktion `SUCCEEDED` meldet. Der Fehlerzweig selbst ist korrekt gebaut
      (zieht aus `triggerBody()`, Betreff `[UNBEKANNT - KI-Fehler]`) — er wird
      nur nie ausgelöst.
      **Umsetzung (minimal-invasiv, ohne Zweige zu verschieben):**
      1. Compose-Aktion `Pruefung_KI` zwischen `VarKritikalitaet` und
         `VarFirmenname` einfügen, Eingabe
         `@if(empty(variables('VarKritikalitaet')), int(variables('VarKritikalitaet')), 'ok')`
         → gefüllt = `ok`/SUCCEEDED, leer = `int('')` wirft Fehler/FAILED.
      2. `E-Mail senden bei KI fehlgeschlagen` → „Ausführen nach konfigurieren":
         von `KI-Verarbeitung` auf `Pruefung_KI` umhängen, Haken bei
         **Fehlgeschlagen + Übersprungen + Zeitüberschreitung**. „Übersprungen"
         ist nötig, damit ein harter Connector-Fehler (der sich als SKIPPED
         durch die Kette fortpflanzt) weiterhin die Fehler-Mail auslöst.
      3. `VarFirmenname` hängt dann automatisch an `Pruefung_KI`.
      Nebeneffekt solange der Stateless-Fix fehlt: **jeder** Test liefert die
      `[UNBEKANNT - KI-Fehler]`-Mail — zugleich die Gegenprobe zur Diagnose.
      Ausdruck ist ungetestet (Designer-Validierung ggf. prüfen); Fallback ist
      die klassische Condition mit `empty(variables('VarKritikalitaet'))`.
- [ ] `Global.KanalLabel` steht in `ConversationStart` hart auf „Telefon" —
      Chat-Tests werden dadurch als Telefonanruf protokolliert.
      **Zurückgestellt** (Nutzerentscheidung 2026-09-04).

### F6. Sweep abgebrochener Anrufe 2026-09-04 — Spaltenfalle `Anrufgrund0`

**Erster echter Abbruch-Test am Telefon erfolgreich:** Staging-Datensatz wurde
angelegt, der Flow `Sweep offene Anrufe` hat ihn gefunden und die Abbruch-Mail
versendet. Kernmechanik von Fall 3 damit am Telefon bestätigt.

**Befund in der Mail:** Zeile „Anrufgrund" enthielt Roh-JSON
`{"@odata.type":"…SPListExpandedReference","Id":0,"Value":"abgebrochen"}`.

**Ursache:** In der Liste `Telefonbot-Anrufe` wurde die Auswahl-Spalte zuerst
„Anrufgrund" genannt und später in **Abschlussart** umbenannt — SharePoint ändert
dabei nur den Anzeigenamen, der **interne Name bleibt `Anrufgrund`**. Die danach
angelegte Textspalte „Anrufgrund" heißt intern deshalb **`Anrufgrund0`**
(verifiziert: Listeneinstellungen → Spalte → URL `…&Field=Anrufgrund0`).
Der Dynamic-Content-Picker im Sweep zeigt beide als „Anrufgrund" an und hat die
falsche (Auswahl-)Spalte gebunden. Details:
[Konzept-Ausfallsichere-Weiterleitung.md](Konzept-Ausfallsichere-Weiterleitung.md),
Abschnitt „Spaltenfalle".

- [ ] **Fix im Sweep-Flow** (Power Automate, nicht im Bot): im HTML-Body der
      Abbruch-Mail den `fx`-Ausdruck der Zeile „Anrufgrund" ersetzen durch
      `if(empty(item()?['Anrufgrund0']),'nicht angegeben',item()?['Anrufgrund0'])`.
      Anlage/Anliegen bleiben unverändert (korrekt).
      **Regel:** Auswahl-Spalten immer als `item()?['Spalte']?['Value']` lesen,
      Textspalten direkt; im Sweep Ausdrücke statt Picker verwenden.
- [ ] **Nachtest:** Testanruf mit klarer Störungs-Formulierung, dann Sweep-Filter
      temporär `addMinutes(utcNow(),-10)` → `-0`, „Testen → Manuell" auslösen,
      Filter zurück auf `-10` stellen.
- [ ] **Folgefrage (hängt am Nachtest):** Im Test wurde „Die Anlage knallt ab und
      zu" **nicht** als Störung erkannt — Anlage blieb „nicht erfasst", der Bot
      lief also durch den else-Ast in `Anrufgrunderfassen.mcs.yml:57`. Sobald die
      Mail den echten Anrufgrund zeigt: prüfen, worauf die Closed List gematcht
      hat, und `entities/Anrufgrund.mcs.yml` ggf. um Synonyme erweitern
      („knallt", „Geräusch", „läuft nicht", „Alarm", „Fehler", …).
- [ ] **Kosmetik nach der Messe:** Telefonnummer wird als STT-Rohform gespeichert
      („Plus 49 760007") — Normalisierung auf `+49 …` prüfen.
- [ ] **Nach Go-Live aufräumen:** Auswahl-Spalte sauber als `Abschlussart` neu
      anlegen. **Nicht vorher** — es bricht die Bindings in `Staging schreiben`
      und `Anliegen weiterleiten – slim`.

### F7. Code-Review `Staging schreiben` + `Sweep offene Anrufe` (2026-09-04)

Vollständige Durchsicht beider `workflow.json` gegen
[Konzept-Ausfallsichere-Weiterleitung.md](Konzept-Ausfallsichere-Weiterleitung.md).
Anleitungen zu allen Punkten stehen dort im Abschnitt **"Betriebs-Runbook"**,
die Befundtabellen im Abschnitt **"Code-Review Staging + Sweep"**.
Ergänzt **F6** (Spaltenfalle `Anrufgrund0`) — der dort beschriebene Bug ist
im geprüften Flow-Stand bereits behoben, siehe „Im Review als korrekt
bestätigt" weiter unten.

**Am Montag (2026-09-08) hier weitermachen.** Reihenfolge: **F7-0 zuerst** (trifft
die Kundenforderung), dann F7-1 Monitoring (schützt vor Unbekanntem), dann messen.

#### OFFEN - vor der Hausmesse (16./17.09.)

- [ ] **F7-0 · ZUERST KLÄREN — Staging läuft zu spät, Fall 3 ist dadurch
      faktisch nicht mehr abgedeckt** (aufgefallen 2026-09-04 beim Review)

      *Ist-Stand (per YAML verifiziert):* Der einzige Staging-Aufruf steht am
      **Anfang von `Zusammenfassung - Slim`**, also **nachdem** Kundendaten,
      Anrufgrund, Anlage und Anliegen bereits erfasst sind.

      *Warum das die Kundenforderung bricht:* Das Konzept verlangt „sobald
      Firmenname, Ansprechpartner und Telefonnummer genannt sind, muss
      **garantiert** eine E-Mail ausgelöst werden". Legt der Anrufer **nach den
      Kundendaten, aber vor der Zusammenfassung** hart auf — genau der Fall aus
      dem Kundentermin vom 2026-09-03 — dann existiert **kein**
      SharePoint-Datensatz. Der Sweep findet nichts, es geht **keine** Mail
      raus. Das Safety-Net in `Ende der Unterhaltung` greift bei hartem
      Auflegen ebenfalls nicht (kein Ereignis vom Teams-Phone-Kanal, siehe
      Konzept, Abschnitt „Das Problem").

      *Der Zielkonflikt:* frühes Staging (Kundenforderung) ⟷ kein
      `InvokeFlowAction` im Frageablauf (Reihenfolge-Anomalie). Beides zugleich
      geht nur, wenn der Aufruf an einer Stelle sitzt, auf die **keine
      `ConditionGroup` über einer gerade gesetzten Variablen** folgt.

      *Lösungsansatz zum Prüfen:* Staging-Aufruf ans **Ende von
      `Kundendaten erfassen`**, unmittelbar **vor** dem Redirect auf
      `Anrufgrund erfassen`. Dort folgt als Nächstes eine **Frage**, keine
      Bedingung — die Anomalie träfe also nicht zu. Das muss am Telefon
      verifiziert werden (Testszenario 2 aus der Übergabedoku:
      Abbruch nach Telefonnummer ⇒ Datensatz vorhanden?).

      *Fallback, falls das nicht stabil läuft:* Kundenerwartung nachjustieren —
      Abdeckung erst ab erfasstem Anliegen. Das wäre eine **Rücknahme einer im
      Termin zugesagten Eigenschaft** und gehört dann ausdrücklich mit AIRCO
      besprochen, nicht stillschweigend.


- [ ] **F7-1 - Fehler-Monitoring einrichten** (empfohlen als Erstes, ~10 Min)
      Power Automate hat **keine Checkbox** dafür - nur eine wöchentliche
      Sammel-Mail an Flow-Besitzer, viel zu träge. Standardweg ist Try/Catch
      über `runAfter`:
      1. Ganz unten im Flow eine `E-Mail senden (V2)` anhängen (Empfänger: du;
         Betreff z. B. `Flow-Fehler: Staging schreiben`).
      2. Auf dem Node **`...` -> "Ausführen nach konfigurieren"** -> Haken bei
         **`ist fehlgeschlagen`**, **`Zeitüberschreitung`**, **`wurde
         übersprungen`**; Haken bei `ist erfolgreich` **entfernen**.

      | Flow | Node, hinter dem der Alarm hängt |
      |------|-----------------------------------|
      | `Staging schreiben` | `Bedingung` |
      | `Sweep offene Anrufe` | `Auf alle anwenden` |
      | `Ticketerstellung` | letzte Aktion des Erfolgs-Astes |

      In den Body gehören `@{workflow()?['run']?['name']}` (Lauf-ID) **und** die
      ConversationId - sonst ist der Lauf im Verlauf nicht wiederzufinden.
      **Besonders wichtig bei `Staging schreiben`:** Der Flow ist `Stateless`
      *und* antwortet dem Bot vor der eigentlichen Arbeit - ein Ausfall ist
      sonst vollständig unsichtbar (vgl. F5, gleiche Fehlerklasse).

- [ ] **F7-2 - Staging-Update monoton machen** (Ausdrücke fertig, nur einsetzen)
      **Priorität hängt an F7-0:** Bei nur *einem* Staging-Aufruf pro Gespräch
      läuft immer der Create-Zweig, der Update-Zweig ist toter Code und das
      Problem tritt nicht auf. **Sobald F7-0 wieder mehrere Aufrufe einführt,
      wird dieser Punkt zwingend.** Dann gilt:
      *Problem:* Die 5 Staging-Aufrufe laufen **asynchron ohne garantierte
      Reihenfolge**. Trifft ein früher Lauf verspätet ein, überschreibt sein
      Platzhalter (`"nicht erfasst"`) einen bereits echten Wert.
      **Das ist eine ernstzunehmende Alternativhypothese für den offenen Befund
      "Anlage nicht erfasst trotz gefülltem Anliegen"** - bisher auf einen
      Closed-List-Miss getippt. Der `Staging schreiben`-Ausführungsverlauf zum
      Testanruf entscheidet zwischen beiden Ursachen.

      *Wo:* `Staging schreiben` -> `Bedingung` -> Wahr-Ast -> `For each` ->
      **`Element aktualisieren`**. Bei den drei Feldern den Chip mit `x`
      entfernen und über den `fx`-Reiter ersetzen:

      | Feld | Ausdruck |
      |------|----------|
      | Anrufgrund | `if(or(empty(triggerBody()?['text_4']), equals(triggerBody()?['text_4'],'nicht angegeben')), coalesce(item()?['Anrufgrund0'],'nicht angegeben'), triggerBody()?['text_4'])` |
      | Anlage | `if(or(empty(triggerBody()?['text_5']), equals(triggerBody()?['text_5'],'nicht erfasst')), coalesce(item()?['Anlage'],'nicht erfasst'), triggerBody()?['text_5'])` |
      | Anliegen | `if(or(empty(triggerBody()?['text_6']), equals(triggerBody()?['text_6'],'nicht erfasst')), coalesce(item()?['Anliegen'],'nicht erfasst'), triggerBody()?['text_6'])` |

      `Firmenname`, `Ansprechpartner`, `Telefonnummer` bleiben **roh** gebunden -
      ab dem ersten Aufruf gefüllt, kein Platzhalter nötig.

      *Platzhalter projektweit verifiziert (2026-09-04, Grep über alle
      `topics/*.mcs.yml`, an allen 5 Aufrufpunkten identisch):*
      Anrufgrund -> `nicht angegeben` - Anlage -> `nicht erfasst` -
      Anliegen -> `nicht erfasst`. Ändert sich eine Bot-Bindung, **muss der
      `equals()`-Text mitgezogen werden** (Groß-/Kleinschreibung zählt) - sonst
      greift die Prüfung still nicht mehr.

      *Warum `item()` hier in zwei Rollen steht:* einmal als **Ziel**
      (`item()?['ID']` = welche Zeile patchen), einmal als **Vorher-Wert**
      (`item()?['Anlage']` = was steht schon drin). Der `GetItems`-Treffer ist
      gleichzeitig die Lese-Quelle; die Schleife hält beide Zugriffe automatisch
      auf derselben Zeile.

- [ ] **F7-3 - Testanruf mit bewusster Korrektur**
      Firmenname korrigieren, dann bis zum Ende durchsprechen. Prüft in *einem*
      Durchgang: monotones Update (F7-2), Korrekturzweig-Heartbeat und die
      20-Min-Karenz. Erwartung: genau **eine** Mail, Datensatz danach
      `gesendet`/`vollständig`, korrigierter Firmenname in der Mail.

- [ ] **F7-4 - Messung: doppelte `Titel` in `Telefonbot-Anrufe`?**
      Liste nach Spalte *Titel* gruppieren und nach Doubletten suchen.
      **Vorbedingung für die Entscheidung zu F7-7** - ohne Befund wäre der Fix
      dort unnötige Latenz im Gespräch.

#### OFFEN - nach der Messe

- [ ] **F7-5 - Aufräum-Flow `Telefonbot-Anrufe aufräumen`**
      SharePoint kann das nicht selbst (nur über Purview-Aufbewahrung -
      lizenzpflichtig, Overkill). Vierter kleiner Flow:

      | Node | Konfiguration |
      |------|---------------|
      | `Wiederholung` | wöchentlich, Sonntag nachts |
      | `Elemente abrufen` | Filterabfrage: `Status eq 'gesendet' and Modified lt '@{addDays(utcNow(),-90)}'`, `$top` 500 |
      | `Auf alle anwenden` -> `Element löschen` | ID = `@item()?['ID']` |

      Zwei Sicherungen: **nur `gesendet`** löschen (ein `offen` gebliebener
      Datensatz ist eine *unbearbeitete Anfrage* und darf nie stillschweigend
      verschwinden), und vorher per `CSV-Tabelle erstellen` + `Datei erstellen`
      archivieren, falls das Reporting historisch sein soll.
      **Kein Blocker mehr**, seit die drei Indizes gesetzt sind: ein Filter auf
      eine *indizierte* Spalte funktioniert auch über 5.000 Items hinaus,
      solange die Treffermenge dieser Klausel darunter bleibt - bei
      `Status eq 'offen'` immer der Fall.

- [ ] **F7-6 - Kosmetik** (Nutzerentscheidung 2026-09-04: später)
      - `Modified lt '@{addMinutes(...)}'` liefert 7 Nachkommastellen;
        funktioniert, ist aber fragil ->
        `formatDateTime(addMinutes(utcNow(),-20),'yyyy-MM-ddTHH:mm:ssZ')`
      - Kein HTML-Escaping im Mailbody (`&`, `<` im Firmennamen zerlegen die
        Tabelle)
      - `KanalLabel` ist das einzige Feld ohne `if(empty(...))`-Fallback
      - Betreffschema uneinheitlich: `[Abgebrochener Anruf] - Firma: X` vs.
        `[Kritikalität] - X` -> für Outlook-Regeln beim Kunden vereinheitlichen
      - `Staging schreiben` ggf. von `Stateless` auf `Stateful` umstellen
        (Trigger -> `Einstellungen`) - Stateless protokolliert Läufe nur
        eingeschränkt; fehlende Verlaufseinträge sind *kein* verlorener Aufruf

#### ZURÜCKGESTELLT - separat bearbeiten (Entscheidung 2026-09-04)

- [ ] **F7-7 - Race Condition beim Staging-Create -> doppelte Datensätze**
      **Ebenfalls von F7-0 abhängig:** mit nur einem Aufruf pro Gespräch gibt
      es keine konkurrierenden Läufe — der Punkt ist derzeit **latent**.
      Kehren mehrere Aufrufpunkte zurück, ist er wieder aktiv.
      *Mechanik:* `Respond to Copilot` steht als **2. Node**, also *vor*
      `Elemente abrufen`. Der Flow antwortet dem Bot, bevor der Datensatz
      existiert. Da Copilot Studio Staging-Aufrufe um 1-2 Turns verzögert,
      können zwei Läufe ihr `GetItems` machen, bevor der erste das Item angelegt
      hat => **zwei Items mit derselben ConversationId** => zwei Abbruch-Mails vom
      Sweep. SharePoint-Listen haben keine Unique-Constraint und kein atomares
      Upsert - "GetItems -> If -> PostItem" ist prinzipiell rennbar.

      *Lösungskandidaten (noch zu entscheiden):*
      1. `Respond to Copilot` ans **Ende** des Flows - serialisiert die Aufrufe
         zwangsläufig. Preis: Bot wartet ~1 s pro Staging-Aufruf (5x pro Anruf).
      2. Response vorn lassen, zweites `GetItems` unmittelbar vor
         `Element erstellen` - verkleinert das Fenster, schließt es nicht.
      3. Deduplizierung im Sweep - behandelt das Symptom, lässt doppelte
         Datensätze im Reporting stehen.

      *Vorbedingung:* **F7-4 zuerst** (erst messen, ob die Race real auftritt).

#### ERLEDIGT (2026-09-04)

- [x] **Sweep-Takt**: bleibt **stündlich** - als Entscheidung übernommen (nicht
      als Mangel). Konzepttext an vier Stellen nachgezogen (Diagramm,
      Flow-Tabelle, Erwartungsmanagement, Review-Tabelle). Versatz damit **bis
      zu ~80 Min**; unkritisch bei 24-h-SLA. **Für Demos auf der Messe den Sweep
      manuell auslösen** (`Testen -> Manuell`) statt die Stunde abzuwarten.
- [x] **Sweep-Karenz 10 -> 20 Min.** Kein eigenes Feld! Die Zahl steht in
      `Sweep offene Anrufe` -> `Elemente abrufen` -> **Filterabfrage**:
      `Status eq 'offen' and Modified lt '@{addMinutes(utcNow(),-20)}'`.
      *Grund:* Gespräche dauern 5-10 Min; bei 10 Min Karenz konnte der Sweep in
      ein **laufendes** Gespräch mailen und danach mailte `Ticketerstellung`
      regulär => zwei Mails. Der Dedup-Schutz greift dort nicht, weil
      `Ticketerstellung` nach `Title` filtert, nicht nach `Status`.
- [x] **Staging-`For each`: ID auf `item()?['ID']`.**
      *Stolperstein:* `items('foreach')` -> `InvalidTemplate` ("repetition
      action(s) 'foreach' ... not defined"). `foreach` ist der **Typ** der Aktion,
      nicht ihr Name - der lautet `For_each`. **Konvention: immer `item()`
      benutzen**, nicht `items('<Name>')`: bezieht sich immer auf die innerste
      Schleife, braucht keinen Namen, überlebt Umbenennungen und den
      Sprachwechsel der Oberfläche (im Sweep heißt dieselbe Schleife
      `Auf_alle_anwenden`). Ein falscher Name in `items()` wird teils
      anstandslos akzeptiert und liefert zur Laufzeit still `null`.
      *Vorher:* `first(...)?['ID']` innerhalb der Schleife - hätte bei mehreren
      Treffern n-mal dasselbe Item gepatcht und die Duplikate stehen lassen.
- [x] **Indizes** auf `Title`, `Status`, `Modified` gesetzt (SharePoint ->
      Listeneinstellungen -> Indizierte Spalten). Verhindert, dass die gefilterten
      `GetItems` ab ~5.000 Items (bei 40 Anrufen/Tag in ~4 Monaten) **still**
      am Listenschwellwert scheitern - still deshalb, weil der Bot den
      Flow-Ausgang gar nicht auswertet.
- [~] **Staging-Aufrufpunkte — Aussage im Lauf des 2026-09-04 überholt.**
      Ein Zwischenstand hatte fünf Aufrufe (Kundendaten, Anrufgrund je Ast,
      Anlage, Anliegen, Korrekturzweig). **Der aktuelle Pull-Stand hat genau
      einen**: `Zusammenfassung-Slim.mcs.yml:13` (`InvokeFlowAction 2exxIi`),
      ganz am Anfang des Topics. Grund ist die Reihenfolge-Anomalie (siehe
      Topics.md): ein `InvokeFlowAction` im Frageablauf spaltet die
      Dialogausführung, die folgende `ConditionGroup` sieht die gerade gesetzte
      Global-Variable noch leer und nimmt den falschen Ast. **Konsequenz siehe
      F7-0** — das ist kein Detail, sondern trifft die Kundenforderung.

#### IM REVIEW ALS KORREKT BESTÄTIGT (keine Aktion nötig)

- Staging setzt `Status`/`Abschlussart` **nur im Create-Zweig** - ein verspäteter
  Staging-Lauf kann kein `gesendet` zurück auf `offen` kippen.
- Sweep liest `Anrufgrund0` (Textspalte) statt `Anrufgrund` (Auswahl) - **der
  Roh-JSON-Bug aus der Übergabedoku ist behoben.**
- Sweep benutzt `item()?[...]` statt `items('Apply_to_each')?[...]` und ist damit
  immun gegen die Lokalisierungsfalle `Auf_alle_anwenden`.
- `Status eq 'offen'` im Filter ist korrekt: SharePoint **filtert** Choice-Spalten
  über den skalaren Textwert, **liefert** sie aber als Objekt zurück (deshalb beim
  *Lesen* `?['Value']`). Kein Widerspruch, sondern zwei verschiedene Schichten.
- **Race-Schutz-Reihenfolge:** Das Konzept forderte "Status *vor* dem Mailversand
  auf `gesendet` setzen"; implementiert ist Mail -> Patch. **Die Implementierung
  ist fachlich richtiger** (verlorene Mail = Verstoß gegen die Kundenforderung
  "kein Anruf geht verloren"; Doppelmail = bloßes Ärgernis). -> Konzept wurde
  angeglichen, nicht der Flow.

#### BINDUNGSFALLE Staging vs. Ticketerstellung - vollständig

Beide Flows haben generische Parameternamen, aber **unterschiedliche
Reihenfolgen**. Die Doku warnte bisher nur vor der ConversationId - tatsächlich
stimmt **nur `text_5`** überein:

| Slot | `Staging schreiben` | `Ticketerstellung` |
|------|---------------------|--------------------|
| `text` | **ConversationId** | **Firmenname** |
| `text_1` | Firmenname | Ansprechpartner |
| `text_2` | Ansprechpartner | Telefonnummer |
| `text_3` | Telefonnummer | **Anliegen** |
| `text_4` | **Anrufgrund** | **KanalLabel** |
| `text_5` | Anlage | Anlage (identisch) |
| `text_6` | **Anliegen** | **Anrufgrund** |
| `text_7` | **KanalLabel** | **ConversationId** |

Eine zwischen den Flows kopierte Bindung schreibt still ins falsche Feld -
alles Strings, der Bot meldet nichts. **Angleichen der Reihenfolgen erst nach
der Messe** (bräche die Bindungen in mehreren `InvokeFlowAction`-Nodes).

### G. Test / Go-Live
- [x] **End-to-End-Test**: `FlowActionBadGateway`-Timeout durch `Respond to the agent` direkt nach dem Trigger behoben (siehe F2, 2026-08-25) — Testanrufe laufen seither zuverlässig durch.
- [x] **Praxistest per echtem Anruf** (Teams Phone) — **erledigt (2026-09-04)**: Agent über vorläufige Testnummer **+4961712779569** (easybell) erreichbar, Teams-Phone-Einrichtung komplett abgeschlossen; laufende Testphase mit echten Anrufen (siehe Sprachempfindlichkeit-Update unten).

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
- ~~**4 Postfächer je Kritikalität**~~ — **Entschieden (2026-09-04): verworfen.**
  Es bleibt dauerhaft bei **einer** Ziel-Mailbox `service@airco-systems.de` für
  alle Kritikalitätsstufen (kein Interim mehr, sondern Zielzustand). Die
  inline-`if()`-Routing-Logik im Flow (Kritisch→A/Hoch→B/Mittel→C/Rest→D) kann
  entweder auf eine feste Adresse vereinfacht oder unverändert gelassen werden
  (alle 4 Branches zeigen ohnehin auf dieselbe Adresse) — funktional macht das
  keinen Unterschied mehr, aber zur Vereinfachung des Flows könnte der
  `if()`-Ausdruck durch die feste Adresse ersetzt werden. Kein offener Blocker
  mehr vor Produktivsetzung.
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

- ~~**Postfach-Adressen**~~ — **Endgültig entschieden (2026-09-04)**: eine Mailbox `service@airco-systems.de` für alle Kritikalitätsstufen. Keine Differenzierung auf 4 Postfächer mehr geplant.
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
