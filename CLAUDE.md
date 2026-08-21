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
| **0 „Slim"** ⬅ neue Variante (2026-08-18) | **Ohne** Inbetriebnahme/Vertragsfrage; stattdessen **Anlagenbeschreibung** (Bezeichnung + Seriennummer + Baujahr, Freitext → `Global.Anlage`). Kundendaten + Anlage + Anliegen → 6 Text-Inputs an Flow „Anliegen weiterleiten – slim" (flowId: `019885f0-e29a-f111-b8db-7ced8d476627`). Zusammenfassung bestätigt nur Kontaktdaten (Anlage + Anliegen unbestätigt in den Flow). Läuft parallel zu Stufe 1. |
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

Kurzfassung **Stufe 1 (v2, 2026-07-14)**:
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
