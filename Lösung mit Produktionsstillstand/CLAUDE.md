# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Projektkontext

Telefon-Bot für den **1st Level Support der AIRCO Systems GmbH** (https://airco-systems.de/).
AIRCO verkauft und wartet Industriekompressoren, Stickstoffanlagen und Druckluftaufbereitung.

**Ausgangslage:** ~40 Anrufe/Tag, 5–10 Min Gesprächsdauer, externer Hotline-Dienstleister soll durch einen KI-Bot ersetzt/entlastet werden.

**Vorgehen: Stufenweiser Aufbau** (Details + Nutzen je Stufe in [Architektur.md](Architektur.md), Abschnitt 0):

| Stufe | Kurzbeschreibung |
|-------|------------------|
| **1** ⬅ aktuell | Prio-Anrufe (Produktionsstillstand) rausfiltern, Vertragsfrage + Inbetriebnahme (je ja/nein), Anliegen als Freitext per E-Mail weiterleiten (unverändert, **keine** KI-Klassifizierung — kommt erst mit Stufe 2 als explizite Gesprächsfrage) |
| 1.5 | Ticketanlage in Planner / SharePoint-Liste |
| 1.6 | Ticketanlage in Odoo |
| 2 | Hauptkategorien im Gespräch + kategoriespezifische Datenerfassung |
| 2.5 | Automatische Angebotsgenerierung, Eintrag in Odoo |
| 3 | Interaktiver Fehlerbehebungsbot (intern / Vertragskunden / buchbare Zusatzleistung) |

**Technologiestack:**
- Microsoft Teams Phone (Festnetznummer, bereits vorhanden)
- Microsoft Copilot Studio (Voice-fähiger Agent)
- Power Automate (Stufe 1: reines E-Mail-Routing, keine KI-Klassifizierung; ab 1.5: Ticketerstellung)
- Planner / SharePoint-Liste (Ticketablage ab Stufe 1.5)
- ERP-System (Serviceticket-Erstellung ab Stufe 1.6; Kandidat **Odoo**, Connector noch zu definieren — siehe [ToDos.md](ToDos.md))

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

In **Stufe 1** werden die Kategorien **gar nicht** erkannt — weder im
Gespräch noch nachgelagert per KI im Flow. Der Innendienst sortiert die
Kategorie manuell beim Bearbeiten der weitergeleiteten E-Mail ein. Die
Kategorie-Erkennung (als explizite Gesprächsfrage) und die
Pflichtfelder-Erfassung im Gespräch kommen mit **Stufe 2**.

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

Kurzfassung **Stufe 1** (aktueller Bauplan, Reihenfolge seit 2026-07-10):
`Conversation Start` → `Kundendaten erfassen` → `Inbetriebnahme` („Wurde die
Anlage in den letzten 12 Monaten in Betrieb genommen?", Ja/Nein,
Selbstauskunft) → `Vertragsfrage` (Ja/Nein, Selbstauskunft) → `Prio-Filter`
(„Steht Ihre Produktion still?"; Ja → `Transfer to Agent` **sofort**,
[KRITISCH]-E-Mail parallel) → `Anliegen erfassen` (Freitext) →
`Zusammenfassung & Bestätigung` → Flow `Anliegen weiterleiten` (reines
E-Mail-Routing, **keine KI-Klassifizierung** in Stufe 1). Kein Ticket, keine
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
| `Transfer to Agent` | Echten Mitarbeiter verbinden (Teams Phone Call Transfer) |
| `Action Node` | Power Automate Flow aufrufen |

### Variablen-Strategie

```
Stufe 1: alles Global Variables (keine Kategorie-Topics vorhanden)
  → Firmenname, Ansprechpartner, Telefonnummer,
    Produktionsstillstand, VertragVorhanden, Anliegen

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

- Produktionsstillstand (Kritisch) → **sofort** `Transfer to Agent` (innerhalb Geschäftszeiten), Flow läuft parallel
- Unerkannte Eingabe → Bot wiederholt Frage einmal → zweites Mal → Eskalation
- Sicherheitsrelevantes Problem (Brand, Rauch) → sofort `Transfer to Agent` — **in Stufe 1 zurückgestellt (2026-07-10)**, Topic noch nicht gebaut, Regel bis dahin inaktiv (siehe ToDos.md)

---

## Power Automate Flow (Stufe 1): "Anliegen weiterleiten"

### Inputs vom Bot

`Firmenname`, `Ansprechpartner`, `Telefonnummer`, `Inbetriebnahme` (Ja/Nein),
`VertragVorhanden` (Ja/Nein), `Produktionsstillstand` (Ja/Nein),
`Anliegen` (Freitext).

### Ablauf

**E-Mail-Routing** (kein KI-Aufruf, keine Kategorie-Ermittlung in Stufe 1):

| Bedingung | E-Mail Empfänger | Betreff |
|-----------|-----------------|---------|
| Produktionsstillstand = Ja | Falk Recknagel + Constantin Metzler | `[KRITISCH]` + Vertragsstatus |
| sonst | Service-Innendienst | Vertragsstatus |

**Prinzip:** Gesprächslogik im Bot, Geschäftslogik (Routing) im Flow.

### Output an Bot

- Keiner in Stufe 1 (kein Ticket, keine Ticketnummer). Ab Stufe 1.5/1.6 liefert
  der Flow „Ticket erstellen" die `Ticketnummer` (Text) zurück → Bot liest sie
  dem Anrufer vor. Routing dann per Switch auf `Priorität`
  (Prioritätsmatrix, siehe Zielbild in [Architektur.md](Architektur.md)).

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
