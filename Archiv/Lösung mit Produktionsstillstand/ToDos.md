# Fortschrittsübersicht

Status je Topic/Baustein: **offen** / **in Bearbeitung** / **erledigt** /
**zurückgestellt** (= spätere Stufe).
Detailinhalte der bereits erarbeiteten Topics stehen in [Topics.md](Topics.md),
die Roadmap der Ausbaustufen in [Architektur.md](Architektur.md).

## Stufe 1 (aktuelle Stufe) — Status

| Baustein | Status |
|----------|--------|
| Agent-Konfiguration (Basic Voice Agent, Anweisungen, Wissen, Tools) | erledigt (siehe Architektur.md Abschnitt 2) |
| Conversation Start | erledigt (Praxistest Aussprache „GmbH" ausstehend, siehe unten) |
| Kundendaten erfassen | erledigt (YAML korrigiert 2026-07-14: Caller-ID-Check via `Not(IsBlank(System.Activity.From.Name))` statt Kanal-Check; `Global.KanalLabel` fest auf „Telefon" gesetzt; YAML in YAML/Kundendaten_erfassen.md dokumentiert) |
| Inbetriebnahme | erledigt (YAML geprüft 2026-07-13) |
| Vertragsfrage | erledigt (YAML geprüft und korrigiert 2026-07-13: Wortlaut „mit Garantieverlängerung" ergänzt, allowInterruption false) |
| Prio-Filter (Produktionsstillstand) | erledigt (YAML geprüft und korrigiert 2026-07-13: außerhalb Geschäftszeiten → Message + Anliegen erfassen; elseActions äußere Condition ergänzt; Message vor Transfer ergänzt) |
| Anliegen erfassen (Freitext) | erledigt (in Copilot Studio umgesetzt, bestätigt 2026-07-10) |
| Sicherheits-Eskalation | zurückgestellt (2026-07-10, Michael: bewusst nach hinten priorisiert zugunsten des Power Automate Flows; Risiko: Sicherheitsnotfall wird vorerst wie jeder andere Anruf behandelt, siehe unten) |
| Zusammenfassung & Bestätigung (Kurzfassung) | erledigt (YAML geprüft und korrigiert 2026-07-13; Synonymtabelle Korrekturfeld2-Entity noch ausstehend, siehe unten) |
| Eskalation / Unterhaltung übertragen | in Bearbeitung (in allen Topics modelliert; Zielrufnummer noch Platzhalter, siehe unten) |
| Power Automate Flow „Anliegen weiterleiten" (reines Routing, keine KI-Klassifizierung) | erledigt (manuell getestet 2026-07-13: Standard-Pfad ✅, KRITISCH-Pfad ✅; `Global.KanalLabel` als `text_4`/„Kanal"-Input ergänzt 2026-07-14; End-to-End-Test aus Copilot Studio noch ausstehend) |
| Teams Phone einrichten (Agent mit Festnetznummer verbinden) | offen (Anleitung erstellt 2026-07-13, siehe [Anleitung MS-Teams Phone.md](Anleitung%20MS-Teams%20Phone.md); erfordert M365/Teams-Admin-Zugang bei AIRCO) |

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

## Offene Detailfragen — Stufe 1

### Conversation Start

- **Praxistest ausstehend**: Aussprache von „GmbH" in der Begrüßung noch
  nicht per echtem Anruf geprüft — Copilot-Studio-Testpanel spielt bei
  Basic-Voice-Agents nur Text ab, kein echtes Audio (siehe Quelle in
  Architektur.md Abschnitt 9, Punkt 1). Blockiert, bis der Agent mit der
  AIRCO-Telefonnummer (Teams Phone) verbunden ist. Falls Aussprache
  schlecht: Sprache-Text umformulieren (z. B. ohne „GmbH" oder phonetisch).

### Kundendaten erfassen

- **Firmenname-Erfassung — Entity-Typ**: Getestet (2026-07-14): Die vortrainierte
  Entity `Organisation` kürzt die Utterance korrekt, übersetzt aber Teile des
  Firmennamens ins Englische (Beispiel: „xyzSystem Weltweit" → `xyzSystem Worldwide`).
  Firmennamen müssen verbatim erfasst werden — Übersetzung ist k.o.-Kriterium.
  **Entscheidung**: Identify bleibt auf „Gesamte Antwort des Benutzers"; stattdessen
  Frageformulierung optimieren (z. B. „Für welche Firma rufen Sie an?" statt „Wie ist
  Ihr Firmenname?"), um natürlich kürzere Antworten zu provozieren. In Stufe 1
  akzeptabel, da der Innendienst die E-Mail menschlich liest und parst.

- **Telefonnummer-Erfassung**: Node-Struktur jetzt entschieden (Caller-ID
  vorschlagen + Ja/Nein-Bestätigung + Fallback-Frage bei „Nein"), siehe
  [Topics.md](Topics.md). Offen bleiben:
  - Exakter Name der Caller-ID-Systemvariable im Editor noch nicht verifiziert
    (Kandidat `System.Activity.From.Name`; im Test-Panel bzw. an der Question
    Node „Antwort speichern in" nachsehen).
  - Zentrale-Erkennung per Abgleich gegen bekannte AIRCO-Zentralnummer(n) ist
    zurückgestellt (kein Blocker, siehe Begründung in Topics.md) — sobald eine
    Zentralnummer bekannt ist, als Optimierung nachrüstbar.

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
  max. 2 Korrekturversuche, danach `Unterhaltung übertragen`.
- **Noch offen**:
  - **Synonym-Tabelle für die `Korrekturfeld2`-Entity**: In Copilot Studio
    bereits als geschlossene Liste mit **6 Werten** (Firmenname,
    Ansprechpartner, Telefonnummer, Anliegen, **Inbetriebnahme**, **Vertrag**
    — beide neu, 2026-07-10) angelegt, aktuell aber nur mit dem Wertnamen
    selbst als Trigger (Platzhalter, kein Synonym hinterlegt). Michael
    liefert die Synonymtabelle nach (analog `Anlagentyp` in CLAUDE.md); dann
    in Copilot Studio bei der Entität ergänzen.
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

- ~~KI-Klassifizierung (Kategorie A–F + Entitäten aus Freitext)~~ —
  **entfällt komplett in Stufe 1** (Korrektur 2026-07-10, Michael): Es gab
  ein Missverständnis, KI-Klassifizierung war nie als Stufe-1-Feature
  gemeint. Der Flow leitet den Freitext unverändert weiter; der Innendienst
  sortiert die Kategorie manuell ein. Kommt erst mit Stufe 2 über die
  explizite Gesprächsfrage „Kategorie auswählen" zurück (Annahme — bitte
  gegenprüfen, falls „später" etwas anderes meint als Stufe 2, siehe
  Architektur.md Abschnitt 1/8).
- ~~E-Mail-Vorlage (Betreff-Schema mit Vertragsstatus, Body-Aufbau)~~ — **erledigt
  (2026-07-13)**: Betreff `[KRITISCH] Ticket – {Firmenname}` / `Ticket – {Firmenname}`;
  Body mit Datum/Uhrzeit, allen 7 Variablen, boolean→Ja/Nein-Expressions,
  Anliegen-Leerprüfung. Manuell getestet, E-Mail korrekt angekommen.
- **⚠ VOR PRODUKTIVSETZUNG PFLICHT — E-Mail-Empfänger konkretisieren**:
  - **Standard-E-Mail** (Produktionsstillstand = Nein): Empfänger = Service-Innendienst.
    Aktueller Platzhalter: `michael.doukas@mosaiic.com`. Korrekte Adresse(n) von Michael
    bereitstellen (Gruppen-Adresse oder Einzeladressen Constantin Metzler, Robert Wallner,
    Thorsten Schröder).
  - **[KRITISCH]-E-Mail** (Produktionsstillstand = Ja): Empfänger = Falk Recknagel +
    Constantin Metzler. Aktueller Platzhalter: `michael.doukas@mosaiic.com`. Echte
    E-Mail-Adressen von Michael bereitstellen.
- **Neu (2026-07-09)**: Copilot Studio zeigt beim Verknüpfen des Flows als
  Tool die Warnung „In dieser Umgebung ist nicht genügend Copilot-Guthaben
  vorhanden" (Flowprüfung) — betrifft die Laufzeit-Kapazität für
  Flow-Ausführungen, nicht die Konfiguration. Bei ~40 Anrufen/Tag
  potenziell ein Lizenz-/Kostenthema. Michael muss das mit AIRCO/dem
  M365-Lizenzverantwortlichen vor Produktivbetrieb klären (Copilot-Guthaben
  pro Umgebung, Kosten pro Flow-Ausführung).
- **⚠ Trigger-Input-Reihenfolge nicht verändern**: Der Copilot-Studio-Trigger in Power
  Automate benennt Inputs generisch nach Typ + Reihenfolge (`text`, `text_1`, …,
  `boolean`, `boolean_1`, …). Aktuelle Zuordnung (Stand 2026-07-13):
  `text`=Firmenname, `text_1`=Ansprechpartner, `text_2`=Telefonnummer,
  `text_3`=Anliegen, `boolean`=VertragVorhanden, `boolean_1`=Produktionsstillstand,
  `boolean_2`=Inbetriebnahme. Reihenfolge der Inputs im Trigger **niemals ändern**,
  sonst müssen alle Flow-Ausdrücke manuell angepasst werden.
- ~~Eigene Sparring-Session erforderlich~~ — **erledigt (2026-07-13)**.

### Eskalation / Unterhaltung übertragen

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
