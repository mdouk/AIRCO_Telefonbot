# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **⚠ ARCHITEKTUR v2 (2026-07-14):** Auf Kundenwunsch umgestellt — **kein
> Produktionsstillstand-Prio-Filter, keine Eskalation/Transfer an einen
> Menschen**. Das Freitext-Anliegen wird **im Power-Automate-Flow per KI**
> zusammengefasst und mit einer **Kritikalität** (Kritisch/Hoch/Mittel/Niedrig)
> versehen, die das E-Mail-Routing an **4 Postfächer** steuert. Die komplette
> v1-Lösung (mit Prio-Filter + Transfer) liegt im Ordner
> `Lösung mit Produktionsstillstand` (**nicht bearbeiten**). Bot + Flow sind als
> Solution 1.1.0.0 dupliziert. Die Aussagen unten sind bereits auf v2
> aktualisiert.

## Projektkontext

Telefon-Bot für den **1st Level Support der AIRCO Systems GmbH** (https://airco-systems.de/).
AIRCO verkauft und wartet Industriekompressoren, Stickstoffanlagen und Druckluftaufbereitung.

**Ausgangslage:** ~40 Anrufe/Tag, 5–10 Min Gesprächsdauer, externer Hotline-Dienstleister soll durch einen KI-Bot ersetzt/entlastet werden.

**Vorgehen: Stufenweiser Aufbau** (Details + Nutzen je Stufe in [Architektur.md](Architektur.md), Abschnitt 0):

| Stufe | Kurzbeschreibung |
|-------|------------------|
| **0 „Slim"** ⬅ **die aktiv gebaute Variante** (Stand 2026-09-05) | **Ohne** Inbetriebnahme/Vertragsfrage. Kette: Kundendaten (Firma, Ansprechpartner, Telefon, **Anrufgrund** als Closed List) → Verzweigung → bei Störung/Wartung **Anlagenbeschreibung** (Freitext → `Global.Anlage`) → Anliegen (Freitext) → Zusammenfassung (bestätigt nur die 3 Kontaktfelder) → Flow **„Ticketerstellung"** (flowId `834c8025-3da8-f111-b8dd-70a8a52f67fc`, Agent-Flow; löst `019885f0-…` ab). Zusätzlich **Ausfallsicherung Fall 1–3**: Safety-Net in „Ende der Unterhaltung", Flow „Staging schreiben" (`a0a99959-…`) + stündlicher Sweep. Details: [Konzept-Ausfallsichere-Weiterleitung.md](Konzept-Ausfallsichere-Weiterleitung.md). Seit 2026-09-08 zusätzlich **Zweisprachigkeit DE/EN** in Umsetzung (Sprach-Fork am Gesprächsanfang + duplizierte englische Kette) — alles dazu in [Konzept-Englisch.md](Konzept-Englisch.md). |
| **1** (v2, 2026-07-14) | Vertragsfrage + Inbetriebnahme (je ja/nein, Selbstauskunft), Anliegen als Freitext erfassen; **kein** Prio-Filter, **keine** Eskalation. Der Power-Automate-Flow verarbeitet den Freitext **per KI**: Zusammenfassung + Kritikalität (Kritisch/Hoch/Mittel/Niedrig) anhand einer Kunden-Störungsliste → E-Mail-Betreff + Routing an 4 Postfächer. **Keine** Kategorie-Klassifizierung A–F (erst Stufe 2) |
| 1.5 | Ticketanlage in Planner / SharePoint-Liste |
| 1.6 | Ticketanlage in Odoo |
| 2 | Hauptkategorien im Gespräch + kategoriespezifische Datenerfassung |
| 2.5 | Automatische Angebotsgenerierung, Eintrag in Odoo |
| 3 | Interaktiver Fehlerbehebungsbot (intern / Vertragskunden / buchbare Zusatzleistung) |

**Technologiestack:**
- Microsoft Teams Phone (Festnetznummer via **Operator Connect** portiert; Anbindung über **Agents & Queues**, nicht Auto Attendant)
- Microsoft Copilot Studio (Voice-fähiger Agent; **LLM muss GPT-4.1 von OpenAI sein** — einziges Modell mit Sprachunterstützung, Stand 21.07.2026)
- Power Platform Umgebung: **EMEA-Region + Dataverse erforderlich** (sonst kein Sprachkanal); **Pay-As-You-Go** Abrechnungsplan (Azure + Copilot Chat + SharePoint Agents)
- Power Automate (Stufe 1/v2: KI-Verarbeitung des Anliegens — Zusammenfassung + Kritikalität — und E-Mail-Routing an 4 Postfächer; ab 1.5: Ticketerstellung)
- AI Builder / GPT-4.1-Prompt (KI-Baustein innerhalb des Flows; Structured Output mit JSON-Schema)
- Planner / SharePoint-Liste (Ticketablage ab Stufe 1.5)
- ERP-System (Serviceticket-Erstellung ab Stufe 1.6; Kandidat **Odoo**, Connector noch zu definieren — siehe [ToDos.md](ToDos.md))

**Deployment-Weg (Copilot Studio → Teams):**
Lösung als **nicht verwaltete Lösung** exportieren → `.zip` herunterladen →
in Teams Admin Center als Teams App hochladen → in **Agents & Queues** als
Copilot Studio Agent registrieren. Nach jeder Änderung am Agent: erneutes
Publish + neuer Export + erneuter Upload nötig.

**Projektsprache:** Deutsch (alle Bot-Dialoge, Topics, Variablen auf Deutsch)

**Sparring-Modus:** Claude agiert als Lehrer/Sparring-Partner. Keine fertige Lösung liefern — stattdessen den Nutzer durch Konzepte führen, Aufgaben stellen, Feedback geben. Copilot Studio Skills werden iterativ eingesetzt.

---

## Arbeitsweise über mehrere Sessions

Jedes Copilot-Studio-Topic wird in einer **eigenen Claude-Session** erarbeitet. Da eine
neue Session keinen Verlauf aus vorherigen Sessions hat, dienen drei Dateien als
Gedächtnis zwischen den Sessions:

- **`Architektur.md`** — das stabile Gesamtbild: Roadmap der Ausbaustufen,
  Systemkomponenten, Workflow der aktuellen Stufe (inkl. Diagramme), Zielbild
  Stufe 2+, Entscheidungspunkte, Variablen-Architektur. Ändert sich nur bei
  Designentscheidungen auf Architekturebene, nicht pro Session.
- **`Topics.md`** — der aktuelle, final abgestimmte Inhalt jedes Topics (Nodes,
  Fragen, Variablenbindungen, Begründungen für Design-Entscheidungen), mit
  Stufen-Zuordnung. Quelle der Wahrheit für das, was bereits erarbeitet wurde.
- **`ToDos.md`** — Fortschrittsübersicht, gruppiert nach Stufen (Status: offen /
  in Bearbeitung / erledigt / zurückgestellt) sowie alle offenen Detailfragen.

**Zu Beginn jeder neuen Session**: Zuerst `ToDos.md` lesen (Status-Überblick), dann
den relevanten Abschnitt in `Topics.md` (Kontext zum konkreten Topic) und bei
Bedarf `Architektur.md` (Gesamtzusammenhang), bevor weitergearbeitet wird.

**Am Ende jeder Session**: Alle drei Dateien aktuell halten — neue Detailentscheidungen
in `Topics.md`, Änderungen am Gesamtdesign in `Architektur.md`, Status in `ToDos.md`.

---

## Fachdomäne: Anrufkategorien

In **Stufe 1 (v2)** werden die **Kategorien A–F nicht** erkannt. Der KI-Schritt
im Flow erzeugt lediglich eine **freie Zusammenfassung + Kritikalität**, kein
Kategorieschema. Die explizite Kategorie-Erkennung im Gespräch und die
Pflichtfelder-Erfassung kommen mit **Stufe 2**. Die Prioritätsmatrix unten
liefert die Kriterien für die KI-Kritikalität.

| Kategorie | Priorität | Pflichtfelder |
|-----------|-----------|---------------|
| **A. Störung/Ausfall** | Kritisch bis Niedrig | Firmenname, Ansprechpartner, Tel, Standort, Anlagentyp, Seriennummer, Fehlerbeschreibung, Alarmcode, Priorität, Störungsdauer, Maßnahmen |
| **B. Wartungsanfrage** | — | Ansprechpartner, letzte Wartung, Betriebsstunden, Zeitraum, Dringlichkeit |
| **C. Ersatzteilanfrage** | — | Ansprechpartner, Artikelnummer, Modell, Seriennummer, Menge, Lieferadresse, Expressversand |
| **D. Angebotsanfrage** | — | Ansprechpartner, Anwendung, Leistung, bestehende Anlage, Zeitrahmen |
| **E. Rückrufbitte** | — | Ansprechpartner, Telefonnummer, Thema, Erreichbarkeit |
| **F. Allgemeine Anfrage** | — | Freitext |

**Prioritätsmatrix (nur Störung):**

| Priorität | Bedingung |
|-----------|-----------|
| Kritisch | Produktionsstillstand |
| Hoch | Anlage eingeschränkt, Produktion läuft |
| Mittel | Problem vorhanden, kein Einfluss auf Produktion |
| Niedrig | Sporadisches Problem |

**Top-Fehlerbilder Kompressor/Anlagen:**
1. Übertemperatur Kompressor (Öl, Kühler, Umgebungstemperatur)
2. Übertemperatur Trockner
3. Drucktaupunkt-Alarm
4. Stickstoffanlage spült dauerhaft
5. Hochdruckverdichter läuft nicht an
6. Druck am Laser bricht zusammen
7. Anlage füllt Bündel nicht

---

## Copilot Studio Architektur (erarbeitetes Design)

### Topic-Übersicht

Der vollständige Ende-zu-Ende-Workflow (Diagramme, Systemkomponenten, alle
Entscheidungspunkte inkl. Eskalationsreihenfolge) steht in
**[Architektur.md](Architektur.md)** — dort auch als Quelle der Wahrheit
gepflegt, um Diagramm-Duplikate mit widersprüchlichem Stand zu vermeiden.

Kurzfassung **Stufe 0 „Slim" — die aktiv gebaute Kette** (Stand 2026-09-08,
per Pull verifiziert):
`Conversation Start` → `Kundendaten erfassen` (Firmenname, Ansprechpartner,
Telefonnummer, **Anrufgrund** als Closed List, dann `SetVariable KanalLabel`
und der Staging-Flowaufruf `a0a99959-…` mit `IsBlank()`-Guards auf allen
8 Parametern) → `Anrufgrund erfassen` (**nur noch Verzweigung, keine Frage**) → bei
Störung/Wartung `Anlage erfassen`, sonst direkt → `Anliegen erfassen`
(Freitext) → `Zusammenfassung - Slim` (Staging-Flow, Bestätigung der
3 Kontaktfelder, Korrekturschleife ≤ 3) → Flow `Ticketerstellung` →
`Ende der Unterhaltung` (Safety-Net, falls `Global.FlowAufgerufen` = false).
Inaktiv: `Inbetriebnahme`, `Vertragsfrage`, `Zusammenfassung` (Nicht-Slim).

Kurzfassung **Stufe 1 (v2, 2026-07-14)** — Topics vorhanden, aber nicht aktiv:
`Conversation Start` → `Kundendaten erfassen` → `Inbetriebnahme` („Wurde die
Anlage in den letzten 12 Monaten in Betrieb genommen?", Ja/Nein,
Selbstauskunft) → `Vertragsfrage` (Ja/Nein, Selbstauskunft) →
`Anliegen erfassen` (Freitext) → `Zusammenfassung & Bestätigung` → Flow
`Anliegen weiterleiten (KI)` (KI-Zusammenfassung + Kritikalität → Routing an
4 Postfächer). **Kein Prio-Filter, kein Transfer**, kein Ticket / keine
Ticketnummer in Stufe 1.

Zielbild **Stufe 2+** (zurückgestellt, siehe Architektur.md Abschnitt 8):
`Kategorie auswählen` → kategoriespezifische Topics (`Störung` /
`Wartungsanfrage` / `Ersatzteilanfrage` / `Angebotsanfrage` / `Rückrufbitte` /
`Allgemeine Anfrage`) → `Zusammenfassung & Bestätigung` → `Ticket erstellen`
(Ticketnummer wird vorgelesen).

### Node-Typen (erarbeitet)

| Node-Typ | Verwendung |
|----------|-----------|
| `Question Node` | Daten sammeln (Anlagentyp, Seriennummer, etc.) |
| `Condition Node` | Verzweigen (z.B. Produktionsausfall = Ja?) |
| `Message Node` | Informieren / Zusammenfassung mit Power Fx Variablen |
| `Redirect to Topic` | Intern zwischen Topics navigieren |
| ~~`Transfer to Agent`~~ | **entfällt in v2** (kein Live-Transfer mehr); v1-Node „Unterhaltung übertragen" nur noch im Backup |
| `Action Node` | Power Automate Flow aufrufen |

### ⚠ Bauregel: kein `InvokeFlowAction` im Frageablauf

**Ein Power-Automate-Aufruf zwischen zwei Fragen zerlegt die Fragereihenfolge.**
Die Copilot-Studio-Engine stellt den Aufruf zurück, springt zur nächsten Frage,
die sie im Graphen findet, und überspringt alles dazwischen; erst mit der
nächsten Nutzereingabe wird der Flow nachgeholt. Sichtbar als Frage aus einem
späteren Topic, die zu früh kommt — und bei `init:`-Variablen zusätzlich als
doppelt gestellte Frage. Reproduzierbar, nicht sporadisch, im Testpanel **und**
am Telefon (`Tests/Test 6`, `Test 8`, `Test 9`).

Widerlegt und **nicht erneut zu verfolgen**: Position des Knotens im Topic,
**Auslagerung des Aufrufs in ein eigenes Redirect-Topic** (am 2026-09-07 als
Topic `Staging` gebaut, am 2026-09-08 wieder gelöscht — brachte keine
Verbesserung; der Aufruf sitzt wieder in `Kundendaten erfassen`, die
`IsBlank()`-Guards auf allen 8 Parametern wurden aus dem Versuch behalten),
`Respond to Copilot` als erste Flow-Aktion, `flowKind: Stateless`, Umstellung
der Bedingung auf eine Topic-Variable, Expressmodus.

> **Nicht verwechseln (2026-09-07):** `flowKind: Stateless` / Expressmodus sind
> **nur für die Reihenfolge-Anomalie** widerlegt. Für den **anderen** Bug —
> leere KI-Felder in der E-Mail — sind sie die **bewiesene Ursache**; siehe die
> Bauregel direkt darunter.

Einzige nachweislich unschädliche Position: **direkt vor der Frage, die ohnehin
als nächste kommt, im selben Topic** (so gelöst in `Zusammenfassung - Slim`).
Vollständige Herleitung in `Architektur.md` §3a und `ToDos.md`
(„Reihenfolge-Anomalie").

**Diagnose-Werkzeug:** Der Trace-Export des Testpanels (`dialog.json`,
`valueType: DialogTracingInfo`) liefert je Aktion `topicId`, `actionId`,
`conditionItemExit` und `variableState.globalState` mit Millisekunden-
Zeitstempeln. Damit ist ein solcher Sprung direkt sichtbar — deutlich schneller
als jede Herleitung aus den YAMLs.

### ⚠ Bauregel: kein Expressmodus in Flows mit KI-Node

**Der Schalter „Expressmodus (Vorschau)" am Trigger „Wenn ein Agent den Flow
aufruft" setzt den Flow von `Stateful` auf `Stateless` — und bricht damit jede
long-running Connector-Aktion.** Betroffen ist der KI-Baustein
`shared_agentnode` in `Ticketerstellung`: Er antwortet mit `HTTP 202` +
`Location` + `Retry-After`, das Ergebnis muss nachgepollt werden. Stateful-Flows
tun das automatisch, Stateless-Flows nicht — sie werten die 202 als Endergebnis.
Folge: `structuredOutput` leer, Aktion trotzdem `SUCCEEDED`, E-Mail mit leeren
Feldern (Firmenname, Ansprechpartner, Telefonnummer, Anrufgrund, Kritikalität,
KI-Zusammenfassung), während die reinen `triggerBody()`-Felder (Anlage, Rohtext,
Kanal) gefüllt bleiben.

**Regel:** Expressmodus nur für Flows **ohne** KI-/Langläufer-Aktionen. Reine
SharePoint-/E-Mail-Flows (`Staging schreiben`) vertragen ihn.

> **Stand 2026-09-08 (Pull):** `Staging schreiben` steht in der Cloud trotzdem
> auf `flowKind: Stateful` — der Expressmodus wurde dort am 2026-09-07 wieder
> abgeschaltet. Die Regel oben bleibt richtig (der Flow *verträgt* ihn), aber
> genutzt wird er nicht mehr. Wer den Flow anfasst: nicht „zurück auf Express"
> optimieren, ohne den Grund zu kennen.

**Fallen bei der Fehlersuche (alle am 2026-09-07 durchlaufen):**
- Der Schalter sitzt am Trigger und wird von der **Versionshistorie nicht mit
  zurückgesetzt** — „Wiederherstellen" einer alten Flow-Version behebt es
  **nicht** und führt in die Irre.
- Der **Trigger-Typ ist nicht die Ursache**: derselbe Agent-Flow-Trigger
  funktioniert mit deaktiviertem Expressmodus einwandfrei.
- Das Symptom wirkt sporadisch („lief mehrfach, dann plötzlich nicht mehr"),
  ist aber deterministisch — es kippt exakt mit dem Umlegen des Schalters.

**Prüfweg:** Rohdatenausgabe der Aktion `KI-Verarbeitung` — `202` + Body nur
`{conversationId}` = Expressmodus aktiv. Gegenprobe im Export:
`metadata.flowSystemMetadata.flowKind` im `workflow.json`. Vollständige
Herleitung in `ToDos.md` (F5).

### ⚠ Bauregel: `init:`-Präfix ist Pro-Variable, nicht Pro-Topic

Der `init:`-Präfix vor einer globalen Variable (z. B. `init:Global.Anliegen`)
darf für dieselbe Variable **nur an einer einzigen Stelle im gesamten Agenten**
stehen — nicht einmal pro Topic, das die Variable befüllt. Zwei Topics, die
dieselbe `Global.`-Variable je mit `init:` deklarieren (typischer Fall bei
duplizierten Sprachzweigen, siehe `Konzept-Englisch.md`), erzeugen den
Validierungsfehler `DuplicateVariableInitializer`. Fix: `init:` nur in der
zuerst geschriebenen Deklaration behalten, in allen weiteren Vorkommen der
Variable schlicht `variable: Global.…` ohne Präfix setzen — die Bedeutung
(Variable existiert bereits global) bleibt gleich. Entdeckt 2026-09-08 beim
Befüllen der englischen Pendants zu `Kundendatenerfassen` und
`Anliegenerfassen`.

### Variablen-Strategie

```
Stufe 1 (v2): alles Global Variables (keine Kategorie-Topics vorhanden)
  → Firmenname, Ansprechpartner, Telefonnummer, KanalLabel,
    Inbetriebnahme, VertragVorhanden, Anliegen
    (Produktionsstillstand entfällt; KI-Zusammenfassung + Kritikalität
     entstehen im Flow, sind keine Bot-Variablen;
     KanalLabel = „Telefon" wenn Caller-ID erkannt, „Chat" sonst —
     wird an den Flow übergeben, damit die E-Mail den Kanal ausweist)

Stufe 2+ (Zielbild):
Global Variables  → Firmenname, Ansprechpartner, Telefonnummer
                    (einmal erfasst, überall verfügbar)
Topic Variables   → Anlagentyp, Seriennummer, Fehlerbeschreibung, Priorität
                    (kategoriespezifisch, leben nur im jeweiligen Topic)
```

Power Fx Syntax für Message Nodes: `{Topic.Anlagentyp}`, `{Global.Firmenname}`

**Namenskonvention**: Variablennamen deutsch, PascalCase, ohne Umlaute/ß
(z. B. `Fehlerbeschreibung`, `Ruecklaufnummer` statt `Rücklaufnummer`) —
vermeidet Encoding-Probleme beim Export/Import und in Power Fx-Ausdrücken.

### Entities (Closed List)

Für Anlagentyp-Auswahl:

| Wert | Synonyme |
|------|---------|
| kompressor | "Kompressor", "1", "Druckluft" |
| stickstoff | "Stickstoff", "N2", "2", "Stickstoffanlage" |
| trockner | "Trockner", "3" |
| sonstiges | "andere", "4", "weiß nicht" |

### Eskalationslogik

**v2: keine Eskalation / kein Transfer an einen Menschen mehr.** Die
Priorisierung erfolgt nachgelagert per KI-Kritikalität im Flow (Routing an
4 Postfächer). Folgen (Default vorläufig, offen — siehe Architektur.md Abschnitt 9):
- Unerkannte Eingabe → Bot wiederholt Frage einmal → zweites Mal → **kein Transfer**, sondern Entschuldigung + ggf. weiter zum Flow / Gesprächsende
- Sicherheitsrelevantes Problem (Brand, Rauch) → ohne Transfer neu zu konzipieren (dringlicher Hinweis + höchste Kritikalität?), Topic weiterhin nicht gebaut
- (v1-Regel „Produktionsstillstand → sofort Transfer" entfällt vollständig)

---

## Power Automate Flow (Stufe 1, v2): "Anliegen weiterleiten (KI)"

### Inputs vom Bot

`Firmenname`, `Ansprechpartner`, `Telefonnummer`, `Inbetriebnahme` (Ja/Nein),
`VertragVorhanden` (Ja/Nein), `Anliegen` (Freitext).
**v2:** `Produktionsstillstand` entfällt.

### Ablauf (implementiert 2026-07-15/16)

**Schritt 1a — Agent-Node (GPT-4.1):** Erhält `Anliegen` + Störungsliste
(Knowledge, SharePoint-Datei, wartet auf Kundenlieferung). Liefert eine
**Kurz-Zusammenfassung** (max. 3 Sätze, sachlich-technisch, Deutsch).

**Schritt 1b — Classify-Node (GPT-4.1 mini):** Erhält die Zusammenfassung
als Input und leitet die **Kritikalität** ab:
Kritisch / Hoch / Mittel / Niedrig / Other (automatischer Fallback).

**Schritt 2 — E-Mail bauen:** Betreff = `[Kritikalität] - Firmenname`
(z.B. `[Kritisch] - Musterfirma GmbH`). HTML-Body: Tabelle Kundendaten
(Firmenname, Ansprechpartner, Telefonnummer, Inbetriebnahme Ja/Nein,
Wartungsvertrag Ja/Nein) → KI-Zusammenfassung → Zeitstempel + Kanal →
Trennlinie → Rohtext Anliegen (Testperiode-Kontrolle).

**Schritt 3 — Routing** (Switch auf Classify-Output):

| Kritikalität | E-Mail Empfänger |
|--------------|------------------|
| Kritisch | Postfach A |
| Hoch | Postfach B |
| Mittel | Postfach C |
| Niedrig | Postfach D |
| Other | Postfach D (Fallback) |

Postfach-Adressen noch offen — wartet auf AIRCO (siehe ToDos.md).
**Prinzip:** Gesprächslogik im Bot, Geschäftslogik (KI-Priorisierung + Routing) im Flow.

### Output an Bot

- Keiner in Stufe 1 (kein Ticket, keine Ticketnummer). Ab Stufe 1.5/1.6 liefert
  der Flow „Ticket erstellen" die `Ticketnummer` (Text) zurück → Bot liest sie
  dem Anrufer vor.

---

## Quelldokumente (in /Daten)

- `1.Level Support.docx` — Telefonablaufplan, Top-Fehlerbilder, FAQ-Katalog, Entscheidungsbaum
- `KI Lösung Servicetelefonate.docx` — Gesprächsstruktur, Kategorien, Ticketfelder, Beispieldialog
- `Prozessdokumentation_Kundenanfrage_Service_260119_final.pdf` — Organisationsstruktur, 1./2./3. Level Support, Zuständigkeiten, SLA (24h Rückmeldung, 3 Werktage Beseitigung)

## AIRCO-interne Ansprechpartner

| Funktion | Person |
|----------|--------|
| 1st Level / Service-Innendienst | Constantin Metzler, Robert Wallner, Thorsten Schröder |
| Serviceleiter | Falk Recknagel |
| 3rd Level Vorsitzender | Robin Lang |
| Geschäftsführung | Thorsten Schröder, Jens Stolze |
