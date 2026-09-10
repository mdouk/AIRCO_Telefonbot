# Konzept: Ausfallsichere Anfrageerfassung („kein Anruf geht verloren")

**Stand:** 2026-09-03
**Betrifft:** AIRCO Telefon-Bot, Stufe 0 „Slim"
**Kundenforderung (Termin 2026-09-03):** Sobald ein Anrufer **Firmenname,
Ansprechpartner und Telefonnummer** genannt hat, muss **garantiert** eine E-Mail
an den Service ausgelöst werden – egal wie weit das Gespräch kommt oder ob es
abbricht.

---

## Das Problem

Bricht ein Anruf **mitten im Gespräch** ab (Kunde legt hart auf, Netzabbruch,
Bot hängt), sendet der Teams-Phone-Kanal **kein Ereignis** an den Bot. Der Bot
wartet auf eine Eingabe, die nie kommt – er ist „eingefroren" und kann in diesem
Moment **nichts** mehr auslösen (keine E-Mail, kein Flow).

→ Jede Rettung, die *im Bot* laufen müsste, ist damit technisch ausgeschlossen.
Die Lösung muss **außerhalb** des Bots und **zeitbasiert** sein.

### Drei Abbruch-Fälle

| Fall | Situation | Lösung |
|------|-----------|--------|
| 1 | Voller Ablauf, aber Fehler am Ende → keine Mail | `Bei Fehler` → `Ende der Unterhaltung` → Safety-Net (E-Mail) |
| 2 | Auflegen **während** der Verabschiedung | Reihenfolge gedreht: **erst** E-Mail, **dann** Verabschiedung |
| 3 | Auflegen **mitten** im Gespräch (nach den 3 Feldern) | **Dieses Konzept:** Fortlaufendes Speichern + Sweep |

---

## Die Lösung (Fall 3): Fortlaufendes Speichern + zeitgesteuertes Nachfassen

Kerngedanke: **„Daten speichern" und „E-Mail senden" werden getrennt.** Der Bot
speichert fortlaufend und *präventiv* – nicht erst am Ende. Ein unabhängiger
Zeitplan-Dienst fasst nach, was liegenbleibt.

> **Wichtig:** Der Speicher-Schritt reagiert **nicht** auf den Abbruch (das ginge
> nicht). Er läuft **proaktiv, solange der Anrufer noch spricht**. Beim Abbruch
> ist deshalb nichts zu „retten" – die Daten liegen längst gesichert vor.

### Ablauf

```
Anruf läuft ──► "Staging schreiben" (mehrfach) ──► SharePoint-Datensatz [offen]
                                                          │
        ┌──────────────────────────────────────────────────┤
        │                                                    │
   Anruf endet SAUBER                              Anruf bricht HART ab
   (Zusammenfassung / Safety-Net)                  (kein Ereignis!)
        │                                                    │
   "Anliegen weiterleiten – slim"                      Bot macht nichts
   → E-Mail + Status = [gesendet]                            │
        │                                     "Sweep" (stuendlich) findet
   Sweep ignoriert ihn                        [offen] & inaktiv → E-Mail
```

### Drei beteiligte Flows

| Flow | Aufgabe | Auslöser |
|------|---------|----------|
| **Staging schreiben** (neu) | Zwischenstand in SharePoint ablegen/aktualisieren (Upsert per ConversationId), Status `offen`. Kein KI, keine Mail. | Bot. ⚠ **Stand 2026-09-04 nur noch am Anfang von `Zusammenfassung - slim`** — die Aufrufe im Frageablauf mussten entfernt werden, siehe „Regression 2026-09-04" unten. |
| **Anliegen weiterleiten – slim** (bestehend, erweitert) | KI-Zusammenfassung + Kritikalität + E-Mail. **Neu:** setzt danach Datensatz auf `gesendet`. | Bot, am **Abschluss** / im Safety-Net |
| **Sweep offene Anrufe** (neu) | Findet `offen` + seit >10 Min inaktiv → **Abbruch-Mail** → Status `gesendet`. | **Zeitplan, stündlich** (Entscheidung 2026-09-04), unabhängig vom Bot |

---

## Datenspeicher: SharePoint-Liste `Telefonbot-Anrufe`

| Spalte | Typ | Zweck |
|--------|-----|-------|
| `Titel` (intern `Title`) | Text | = ConversationId (Schlüssel für Upsert) |
| `Firmenname`, `Ansprechpartner`, `Telefonnummer`, `KanalLabel`, `Anrufgrund` | Einzeltext | Kontaktdaten |
| `Anliegen`, `Anlage` | Mehrere Zeilen (Nur-Text) | Freitext |
| `Status` | Auswahl `offen` / `gesendet` (Default `offen`) | Steuert den Sweep |
| `Abschlussart` | Auswahl `abgebrochen` / `vollstaendig` (Default `abgebrochen`) | Reporting |
| `Geändert` (`Modified`) | (SharePoint-Standard) | Basis für „inaktiv seit >10 Min" |

### ⚠ Spaltenfalle: interne Namen weichen von den Anzeigenamen ab

Beim Anlegen der Liste wurde die Auswahl-Spalte zuerst „Anrufgrund" genannt und
später in **Abschlussart** umbenannt. SharePoint ändert dabei nur den
Anzeigenamen — der **interne Name bleibt `Anrufgrund`**. Die danach angelegte
Textspalte „Anrufgrund" bekam deshalb den internen Namen **`Anrufgrund0`**.

| Anzeigename | Interner Name | Typ | Zugriff im Flow |
|-------------|---------------|-----|-----------------|
| Anrufgrund | `Anrufgrund0` | Einzeltext | `items('Apply_to_each')?['Anrufgrund0']` |
| Abschlussart | `Anrufgrund` | Auswahl | `items('Apply_to_each')?['Anrufgrund']?['Value']` |
| Status | `Status` | Auswahl | `items('Apply_to_each')?['Status']?['Value']` |

**Zwei Regeln daraus:**
1. **Auswahl-Spalten immer mit `?['Value']` lesen.** Ohne `Value` liefert der
   Connector das ganze Objekt
   (`{"@odata.type":"…SPListExpandedReference","Id":0,"Value":"…"}`) und genau
   das landet dann im Mailtext.
2. **Im Sweep nicht den Dynamic-Content-Picker benutzen**, sondern Ausdrücke mit
   internem Namen — der Picker zeigt Anzeigenamen und greift hier die falsche
   Spalte.

**Internen Namen einer Spalte prüfen (2 Wege):**
- SharePoint → *Listeneinstellungen* → Spalte anklicken → in der Adresszeile
  steht `…&Field=Anrufgrund0` (bestätigt am 2026-09-04).
- Oder in einem exportierten Flow: `workflows/…/workflow.json` nach
  `"item/…"` durchsuchen — dort stehen ausschließlich interne Namen.

*Aufräumen (Spalte sauber neu anlegen) erst **nach** Go-Live — es würde die
Bindings in `Staging schreiben` und `Anliegen weiterleiten – slim` brechen.*

**Warum die Defaults pessimistisch sind:** Ein Datensatz entsteht früh, wenn das
Gespräch noch nicht abgeschlossen ist. Im Zweifel ist er ein Abbruch (`offen` /
`abgebrochen`). Erst der saubere Abschluss setzt ihn aktiv auf `gesendet` /
`vollstaendig`.

---

## ⚠ Regression 2026-09-04: Staging-Aufrufe zerlegen den Dialog

**Symptom:** Nach „Störung" fragte der Bot **„Bitte beschreiben Sie nun Ihr
Anliegen"** statt nach der Anlage — reproduzierbar in allen acht Testläufen,
im Testpanel *und* am Telefon.

**Bestätigte Ursache:** Ein `InvokeFlowAction` im Frageablauf spaltet die
Dialogausführung. Die nachfolgende `ConditionGroup` wertet `Global.Anrufgrund`
noch als **leer** aus und nimmt den `else`-Ast (→ Anliegen). Erst wenn der
Flow-Aufruf zurückkommt, läuft der echte Pfad nach und nimmt den richtigen Ast.

**Nachweis:** Entfernen der Staging-Aufrufe in `Kundendaten erfassen`,
`Anrufgrund erfassen`, `Anlage erfassen`, `Anliegen erfassen` — Reihenfolge
sofort korrekt, im Chat und am Telefon. Nicht die Position im Topic war das
Problem: ein Verschieben an den Topic-Anfang machte es schlimmer (doppelte
Anrufgrund-Frage, weil `init:Global.Anrufgrund` beim Nachlauf zurücksetzt).

**Widerlegt:** `Respond_to_Copilot` als erste Aktion (machen beide Flows) und
`flowKind: Stateless` (beide Flows stehen im Export auf `Stateful` — die Notiz
im P3-Abschnitt ist veraltet).

**Preis dafür:** Fall 3 ist derzeit **nicht** abgedeckt. Ein Auflegen zwischen
Kundendaten und Zusammenfassung hinterlässt keinen Datensatz; der Sweep hat
nichts zu finden. Das ist ein Verstoß gegen die Kundenforderung und muss vor
der Hausmesse (16./17.09.) gelöst sein.

**Offene Hypothese für die Rückkehr:** In `Zusammenfassung - slim` steht der
Staging-Flow strukturell **identisch** (Flow → Question → ConditionGroup) und
funktioniert. Einziger Unterschied: die Bedingung liest `Topic.korrekt` statt
`Global.Anrufgrund`. Vermutung: Global-Variablen werden über die Flow-Grenze
nicht rechtzeitig persistiert, Topic-Variablen schon. Test: in
`Anrufgrund erfassen` nach der Frage `SetVariable Topic.Grund =
Global.Anrufgrund`, `ConditionGroup` auf `Topic.Grund` umstellen, Staging-Flow
wieder an den Topic-Anfang.

**Nachtrag — was danach geschah, damit dieser Abschnitt nicht in die Irre
führt:** Die „Rückkehr" fand tatsächlich statt, aber anders als hier
spekuliert. Am **2026-09-05** wurde der Staging-Aufruf zurück in
`Kundendaten erfassen` gezogen (direkt vor die Anliegen-Frage, unmittelbar
nach der zeitgleich hochgezogenen Anrufgrund-Frage) — ohne die
Global-vs-Topic-Hypothese oben zu testen; siehe `Topics.md`, Abschnitt
„Warum die Anrufgrund-Frage hier steht". Dieser Zustand wurde nie in dieses
Dokument nachgetragen. Am **2026-09-10** wurde der Aufruf im Portal erneut
entfernt — siehe „Regression 2026-09-10" unten. Die Checkliste unter
„Umsetzungsstand" ist entsprechend veraltet.

---

## ⚠ Regression 2026-09-10: Staging-Aufruf in `Kundendaten erfassen` erneut entfernt

**Symptom:** Anliegen-Frage wurde doppelt gestellt, erste Antwort verworfen —
identisches Muster zur Regression 2026-09-04, diesmal mit sichtbarem
Datenverlust statt reiner Fragenverdrehung. Nachgewiesen per Trace-Export in
`Tests/Test 18` und `Test 19` (siehe `ToDos.md`, F9, für den vollständigen
Ablauf).

**Fix (direkt im Portal, 2026-09-10):** Der `InvokeFlowAction`-Knoten „Staging
schreiben" (`a0a99959-…`, id `m8Ujcv`/`bJtc5e`) wurde aus
`Kundendaten erfassen`/`…EN` **ersatzlos entfernt** — dieses Mal nicht als
Zwischenstand, sondern als bewusste Entscheidung gegen eine erneute Rückkehr.
`init:` vor `Global.Anliegen` (DE) entfiel damit ebenfalls.

**Fall 3 ist damit wieder eingeschränkt — aber anders als 2026-09-04:**
- Der einzige verbleibende Staging-Aufruf liegt jetzt am **Anfang von
  `Zusammenfassung - slim`/`ZusammenfassungEN`**, also technisch **nach** der
  Anliegen-Frage statt davor.
- **Ungeschützt ist nur ein einzelner Frage-Turn:** Bricht der Anruf **hart**
  ab (kein `OnSystemRedirect`, siehe unten) **während** die Anliegen-Frage
  offen ist — also nach der Anlage-Frage, bevor der Anrufer geantwortet hat —
  existiert kein Staging-Datensatz, nicht einmal mit den bereits vorliegenden
  Feldern (Firma, Ansprechpartner, Telefon, Anrufgrund, Anlage). Sobald die
  Anliegen-Frage beantwortet ist, läuft die Ausführung ohne weiteren
  Nutzer-Turn direkt in `Zusammenfassung`, deren eigener Staging-Aufruf dann
  sofort feuert — dieses Zeitfenster ist also klein, nicht die ganze restliche
  Unterhaltung wie 2026-09-04.
- **Das synchrone Safety-Net in `Ende der Unterhaltung` ist davon nicht
  betroffen** — es hängt nicht am Staging-Flow, sondern ruft bei
  `Not(IsBlank(Global.Telefonnummer)) && Not(Global.FlowAufgerufen)` die
  Ticketerstellung direkt auf. Es greift aber nur bei einem **sauberen**
  Abbruch (`OnSystemRedirect` — z. B. Stille-Erkennung, reguläres Auflegen über
  das Teams-Phone-Signal), nicht beim harten Verbindungsabbruch ohne jedes
  Ereignis, für den der stündliche Sweep gedacht ist.

**Entscheidung noch offen:** Lücke hinnehmen (Fenster ist ein einzelner
Frage-Turn, geringe Eintrittswahrscheinlichkeit) oder einen erneuten
Zwischen-Staging-Aufruf an einer nachweislich unschädlichen Stelle suchen.
Nicht ohne Test wieder an dieselbe Stelle zurückschieben — das ist exakt der
Kreislauf, der bereits zweimal (2026-09-04 → 2026-09-05 → 2026-09-10)
durchlaufen wurde.

---

## Zusammenspiel / Doppelversand-Schutz

- **Genau eine** E-Mail pro Anruf: Entweder der Abschluss-Flow (sauberes Ende)
  **oder** der Sweep (Abbruch) – nie beide, weil der Sweep nur Datensätze mit
  Status `offen` erfasst und der Abschluss-Flow auf `gesendet` setzt.
- Bot-Flag `Global.FlowAufgerufen` verhindert zusätzlich, dass Abschluss **und**
  Safety-Net in derselben Session doppelt mailen.
- Race-Schutz im Sweep: Status **vor** dem Mailversand auf `gesendet` setzen.

---

## Erwartungsmanagement / Grenzen

- **Verzögerung:** Abbruch-Mails kommen mit **bis zu ~70 Min** Versatz
  (Sweep läuft **stündlich**, Karenz 10 Min). **Bewusste Entscheidung vom
  2026-09-04:** unkritisch bei 24-h-SLA, der Anrufer merkt nichts davon, und der
  stündliche Takt hält die Zahl der Power-Platform-Aufrufe niedrig. Wer den
  Abbruch-Fall **vorführen** will, löst den Sweep im Designer manuell aus
  („Testen → Manuell“) statt die Stunde abzuwarten.
- **Bewusst nicht abgedeckt:** Auflegen, **bevor** die drei Mindestdaten
  (Firmenname, Ansprechpartner, Telefonnummer) genannt wurden – vom Kunden im
  Termin ausdrücklich akzeptiert.
- **Nebeneffekt (positiv):** Die Liste `Telefonbot-Anrufe` ist zugleich die
  Datenbasis für das gewünschte Reporting (Anrufe gesamt, vollständig vs.
  abgebrochen, Mails versendet).

---

## Umsetzungsstand

- [x] Fall 1 + 2 gelöst (Safety-Net + Reorder „E-Mail vor Verabschiedung")
- [x] SharePoint-Liste `Telefonbot-Anrufe` angelegt
- [x] Flow `Staging schreiben` (flowId `a0a99959-80a7-f111-b8de-7ced8d476627`) angelegt
- [x] ⚠ **Überholt — siehe „Regression 2026-09-10" oben.** Zwischenzeitlich
      (2026-09-05) auf zwei Aufrufpunkte erweitert (`Kundendaten erfassen` +
      `Zusammenfassung - slim`), hier nie nachgetragen. Am 2026-09-10 wieder auf
      **einen** reduziert (nur `Zusammenfassung - slim`/`ZusammenfassungEN`),
      diesmal als bewusste Entscheidung gegen die Reihenfolge-Anomalie, nicht
      als Zwischenstand. Fall 3 dadurch **erneut eingeschränkt** — Ausmaß siehe
      oben, deutlich kleiner als am 2026-09-04.
- [x] `Anliegen weiterleiten – slim` um `Status = gesendet`/`Abschlussart = vollstaendig` erweitert (beide Äste) + neue Eingabe `text_7 = ConversationId`
- [x] Flow `Sweep offene Anrufe` (stündlich; Filter `Status eq 'offen' and Modified lt now-10min`; Abbruch-Mail + `Status = gesendet`)
- [x] Publish + erster End-to-End-Test am Telefon (2026-09-04): Abbruch erkannt,
      Sweep hat Abbruch-Mail ausgelöst — **Kernmechanik bestätigt**
- [ ] **Bug (offen):** Zeile „Anrufgrund" in der Sweep-Mail zeigt Roh-JSON der
      Abschlussart → auf `Anrufgrund0` umbiegen (siehe Spaltenfalle oben)
- [ ] Danach prüfen: Wurde „Die Anlage knallt ab und zu" als **Störung** erkannt?
      (Anlage blieb „nicht erfasst" ⇒ Bot lief durch den else-Ast ⇒ vermutlich
      auf „Sonstiges" gematcht — Closed-List-Synonyme nachschärfen)
- [ ] Kosmetik nach der Messe: Telefonnummer-Normalisierung („Plus 49 …" → „+49 …")
- [x] **Neuaufbau des Abschluss-Flows als Agent-Flow „Ticketerstellung"**
      (2026-09-04, siehe Abschnitt unten) — ersetzt `Anliegen weiterleiten – slim`;
      Bug „`id` leer" dabei behoben.

---

## Bug 2026-09-04: `Element aktualisieren` schlägt fehl (`id` leer) — **behoben 2026-09-04**

**Symptom (Lauf 07:35:48, `Anliegen weiterleiten – slim`):**
`WorkflowOperationParametersRuntimeMissingValue` — `'id'` darf nicht leer sein.
Vorher: `E-Mail senden` ✅, `Elemente abrufen` ✅ (aber **0 Treffer**).

**Mechanik:** `Elemente abrufen` mit 0 Zeilen ist kein Fehler. Erst das
nachfolgende `first(...)?['ID']` liefert leer → Update crasht → Lauf rot.

**Fachliche Folge:** Mail ist raus (Anruf nicht verloren), **aber** der
Datensatz bleibt auf `Status = offen` → der Sweep schickt 10 Min später eine
**zweite (Abbruch-)Mail**. Der Doppelversand-Schutz hängt genau an diesem Update.

**Mögliche Ursachen (zu verifizieren):**
1. Kein Staging-Datensatz vorhanden — Lauf kam vermutlich aus dem Safety-Net
   `Ende der Unterhaltung`, das als `fallbackDialogOnSilence` schon **während**
   `Kundendaten erfassen` feuern kann. Der Staging-Aufruf steht erst *nach* der
   Telefonnummer-Frage (`Kundendatenerfassen.mcs.yml:72`) und läuft **asynchron**
   → Zeitfenster ohne Datensatz.
2. ConversationId/Filter passt nicht (Test-Panel vs. Telefon).

**Prüfung:** Rohdatenausgaben von `Elemente abrufen` (`body.value == []`?),
`text_7` im Trigger, Suche dieser Id in Spalte *Titel* der Liste, Verlauf von
`Staging schreiben` zur selben Uhrzeit.

**Fix:**
- *Robustheit:* `Element aktualisieren` in ein **Apply to each** über
  `outputs('Elemente_abrufen')?['body/value']` mit `items('Apply_to_each')?['ID']`
  (bei 0 Zeilen passiert nichts) — **in beiden Ästen**, auch
  `Element aktualisieren 1` im KI-Fehler-Ast.
- *Ursache:* slim-Flow legt bei „nicht gefunden" den Datensatz **selbst an**
  (`Status = gesendet`), also echtes Upsert wie in `Staging schreiben`.
- *Race beachten:* `Staging schreiben` darf `Status` nur im **Create**-Zweig
  setzen, nicht im Update — sonst kippt ein verspäteter Staging-Lauf einen
  bereits als `gesendet` markierten Datensatz zurück auf `offen`.

---

## Neuaufbau 2026-09-04: Agent-Flow „Ticketerstellung"

**Anlass:** Der Cloud-Flow `Anliegen weiterleiten – slim`
(`019885f0-e29a-f111-b8db-7ced8d476627`) liess sich wegen eines
Microsoft-seitigen Designer-Bugs nicht mehr bearbeiten. Er wurde deshalb als
**Agent-Flow** (Tool direkt am Agenten angelegt, Trigger „Wenn ein Agent den
Flow aufruft") unter dem Namen **`Ticketerstellung`** neu gebaut und wird
danach geloescht. **flowId der Neufassung: `834c8025-3da8-f111-b8dd-70a8a52f67fc`.**

**Trigger-Inputs (Reihenfolge = Bindung, nicht der Anzeigename):**

| # | Feld | Ausdruck |
|---|------|----------|
| 1 | Firmenname | `triggerBody()?['text']` |
| 2 | Ansprechpartner | `triggerBody()?['text_1']` |
| 3 | Telefonnummer | `triggerBody()?['text_2']` |
| 4 | Anliegen | `triggerBody()?['text_3']` |
| 5 | KanalLabel | `triggerBody()?['text_4']` |
| 6 | Anlage | `triggerBody()?['text_5']` |
| 7 | Anrufgrund | `triggerBody()?['text_6']` |
| 8 | ConversationId | `triggerBody()?['text_7']` |

Der Agent-Flow-Trigger vergibt dieselben generischen internen Namen wie der
alte Copilot-Studio-Trigger — alle bestehenden `triggerBody()`-Ausdrücke
konnten 1:1 übernommen werden.

**Bewusste Abweichungen vom alten Flow:**

1. **Variablen weiterhin einstufig** (`InitializeVariable` im Erfolgs-Ast
   hinter dem KI-Node, wie im alten Flow — bestätigt am gespeicherten
   `workflow.json`). Beim Speichern trat einmalig
   `InvalidVariableOperation` auf (*„`VarKritikalitaet` must be
   initialized"*) — Ursache war schlicht, dass die Variable noch fehlte, **nicht**
   die parallele Verzweigung. Power Automate prüft Variablen statisch beim
   Speichern; sind alle fünf angelegt, akzeptiert der Validator die
   Platzierung im Zweig. Ein Umbau auf `InitializeVariable` vor dem KI-Node +
   `SetVariable` danach ist also **nicht** nötig (bliebe aber die robustere
   Variante, falls der Fehler wiederkehrt).
2. **`Element aktualisieren` in einer `Foreach`-Schleife** (beide Äste) —
   siehe Bugfix oben.
3. **Fehler-Ast setzt ebenfalls `gesendet` / `vollstaendig`.** Logik:
   `Status` steuert den Sweep und beschreibt, **ob eine Mail raus ist** — im
   Fehler-Ast ist sie das (Fallback-Mail). Bliebe er auf `offen`, schickte der
   Sweep 10 Min später eine zweite Mail. `Abschlussart` beschreibt den
   **Gesprächsverlauf**, nicht die Flow-Qualität: der Anrufer hat regulär
   durchgesprochen, also `vollstaendig`. `abgebrochen` bleibt Anrufen
   vorbehalten, bei denen der Sweep einspringen musste — sonst ist die
   Abbruchquote im Reporting mit Technikfehlern verunreinigt. Die
   KI-Störung ist bereits am Betreff `[UNBEKANNT – KI-Fehler]` erkennbar;
   soll sie in der Liste sichtbar werden, braucht es eine eigene Spalte.

**Namensfalle (bestätigt 2026-09-04):** Bei deutscher Designer-Oberfläche heißt
die Schleife intern **`Auf_alle_anwenden`** / **`Auf_alle_anwenden_1`**, *nicht*
`Apply_to_each`. `items()` erwartet den internen Namen; ein falscher Name wird
vom Editor anstandslos akzeptiert und liefert zur Laufzeit `null` (gleiche
Klasse wie die `outputs()`-vs-`body()`-Falle). Internen Namen immer über den
`fx`-Reiter „Dynamischer Inhalt" holen oder in der Codeansicht verifizieren.

**Restarbeiten:**

- [ ] Speichern/Publish des neuen Flows, Testanruf (genau **eine** Mail,
      Datensatz danach `gesendet`/`vollstaendig`)
- [x] flowId in den **drei** `InvokeFlowAction`-Nodes umgehängt (2026-09-04,
      per Pull verifiziert): `Zusammenfassung-Slim` Z. 64 + 237, `EndofConversation`
      Z. 37 — Bindung stimmt mit dem Trigger-Schema überein, alle 8 Parameter
      sind `required` und über `If(IsBlank(...))` gegen Leerwerte abgesichert
- [!] **Staging im Korrekturzweig / mehrere Aufrufpunkte — Stand überholt.**
      Zwischenzeitlich gab es fünf Staging-Aufrufe. Der **aktuelle Pull-Stand
      (2026-09-04) hat genau einen**: `Zusammenfassung-Slim.mcs.yml:13`
      (`InvokeFlowAction 2exxIi`), am Anfang des Topics — also **nach** der
      kompletten Datenerfassung. Ursache: die Reihenfolge-Anomalie
      (`InvokeFlowAction` im Frageablauf spaltet die Dialogausführung, die
      folgende `ConditionGroup` liest die Variable noch leer).
      **⚠ Damit ist Fall 3 faktisch nicht mehr abgedeckt:** bei hartem Auflegen
      nach den Kundendaten existiert kein Datensatz, der Sweep findet nichts,
      es geht keine Mail raus — genau der Fall aus dem Kundentermin.
      → Offener Punkt **F7-0** in `ToDos.md`, vor allem anderen zu klären.
- [ ] Verwaisten Tool-Eintrag `actions/Anliegenweiterleiten-slim.mcs.yml`
      (`019885f0-…`) aus dem Agenten entfernen — wird von keinem Topic mehr
      aufgerufen
- [ ] Erst nach grünem End-to-End-Test: `Anliegen weiterleiten – slim` löschen
- [ ] Optional (Ursachenbehebung statt Symptom): Upsert im Abschluss-Flow —
      bei „Datensatz nicht gefunden" selbst anlegen mit `Status = gesendet`

**Bindungsfalle Staging vs. Ticketerstellung:** Beide Flows haben generische
Parameternamen, aber **unterschiedliche Reihenfolgen** — `Staging schreiben`
erwartet die ConversationId als **`text`** (erstes Feld), `Ticketerstellung` als
**`text_7`** (letztes). Eine zwischen den Flows kopierte Bindung schreibt die
ConversationId still in das Firmennamen-Feld; beide sind Strings, der Bot meldet
nichts.

---

## Code-Review Staging + Sweep (2026-09-04)

Vollständige Durchsicht der `workflow.json` beider Flows gegen dieses Konzept.
**Bestätigt korrekt:** Staging setzt `Status`/`Abschlussart` nur im Create-Zweig;
der Sweep liest `Anrufgrund0` (Bug aus der Übergabedoku ist **behoben**); der
Sweep nutzt `item()?[…]` statt `items('Apply_to_each')?[…]` und ist damit immun
gegen die Lokalisierungsfalle `Auf_alle_anwenden`.

### P1 — vor der Hausmesse (16./17.09.)

| # | Befund | Fix |
|---|--------|-----|
| 1 | ~~Sweep läuft stündlich statt alle 5 Min~~ | **Erledigt 2026-09-04 – als Entscheidung übernommen.** Stündlich ist gewollt; Konzepttext oben nachgezogen. Für Demos den Sweep manuell auslösen. |
| 2 | ✅ **Erledigt 2026-09-04: Karenz auf 20 Min gesetzt** (+ Staging im Korrekturzweig als Heartbeat vorhanden). *Ursprünglicher Befund:* Karenz 10 Min < Gesprächsdauer (5–10 Min). Nach dem letzten Staging (Anliegen) folgen Zusammenfassung + Korrekturschleife → `Modified` altert, während der Anrufer spricht. Sweep mailt „abgebrochen", danach mailt Ticketerstellung regulär ⇒ **zwei Mails**. Der Dedup-Schutz greift nicht, weil Ticketerstellung nach `Title` filtert, nicht nach `Status`. | Karenz auf `addMinutes(utcNow(),-20)` **und** Staging im Korrekturzweig ergänzen (Heartbeat) |
| 3 | ✅ **Erledigt 2026-09-04** — Staging-`For each` nutzte `first(...)` statt `item()` für die `id` → bei mehreren Treffern wird n-mal dasselbe Item gepatcht, Duplikate bleiben. Latent wegen `$top:1`, aktiv genau im Duplikat-Fall. | `"id": "@item()?['ID']"` |
| 4 | ⏸ **Zurückgestellt (Entscheidung 2026-09-04, separat bearbeiten)** — **Race beim Create → doppelte Datensätze.** `Respond to Copilot` steht als 2. Node, also *vor* `Elemente abrufen`. Da CPS Staging-Aufrufe um 1–2 Turns verzögert, können zwei Läufe ihr `GetItems` machen, bevor der erste das Item anlegt ⇒ beide gehen in den Else-Zweig. Folge: zwei Abbruch-Mails. | **`Respond to Copilot` ans Ende verschieben** (serialisiert die Aufrufe, Preis ~1 s). *Prüfauftrag:* Liste nach doppelten `Titel` gruppieren. |

### P2 — Zeitbomben

| # | Befund | Fix |
|---|--------|-----|
| 5 | **Keine Indizes** auf `Title`, `Status`, `Modified`. Bei 40 Anrufen/Tag ist die 5.000er-Listenschwelle in ~4 Monaten erreicht → gefilterte `GetItems` schlagen fehl, **still** (Bot wertet den Flow-Ausgang nicht aus). | Indizierte Spalten anlegen + Aufräum-Policy (>90 Tage) |
| 6 | **Kein Fehler-Monitoring.** Staging antwortet vor der Arbeit, der Sweep berichtet niemandem → der ganze Fall-3-Schutz kann unbemerkt ausfallen (genau der Bug vom 04.09.). | Für beide Flows „Fehlerbenachrichtigung senden" abonnieren |
| 7 | **Staging-Update ist nicht monoton.** Async-Aufrufe haben keine garantierte Reihenfolge → ein später eintreffender früher Lauf überschreibt echte Werte mit `"nicht erfasst"` / `"nicht angegeben"`. **Alternativhypothese für den offenen Punkt „Anlage nicht erfasst trotz gefülltem Anliegen"** (bisher auf Closed-List-Miss getippt) — der `Staging schreiben`-Ausführungsverlauf entscheidet. | Im Update-Zweig Platzhalter nicht schreiben: `@{if(or(empty(triggerBody()?['text_5']), equals(triggerBody()?['text_5'],'nicht erfasst')), first(outputs('Elemente_abrufen')?['body/value'])?['Anlage'], triggerBody()?['text_5'])}` — analog `text_4`/`text_6` |

### Bindungsfalle vollständig (ergänzt die Notiz oben)

Zwischen `Staging schreiben` und `Ticketerstellung` stimmt **nur `text_5`** überein:

| Slot | `Staging schreiben` | `Ticketerstellung` |
|------|---------------------|--------------------|
| `text` | **ConversationId** | **Firmenname** |
| `text_1` | Firmenname | Ansprechpartner |
| `text_2` | Ansprechpartner | Telefonnummer |
| `text_3` | Telefonnummer | **Anliegen** |
| `text_4` | **Anrufgrund** | **KanalLabel** |
| `text_5` | Anlage | Anlage ✅ |
| `text_6` | **Anliegen** | **Anrufgrund** |
| `text_7` | **KanalLabel** | **ConversationId** |

Jede kopierte Bindung schreibt still ins falsche Feld (alles Strings, keine
Fehlermeldung). Angleichen der Reihenfolgen erst **nach** der Messe.

### P3 — Kosmetik

- **Race-Schutz-Reihenfolge:** Dieses Konzept fordert „Status *vor* dem Mailversand
  auf `gesendet` setzen"; implementiert ist Mail → Patch. Die Implementierung ist
  fachlich **richtiger** (verlorene Mail = Verstoß gegen die Kundenforderung,
  Doppelmail = Ärgernis). → **Konzept angleichen, nicht den Flow.**
- `Modified lt '@{addMinutes(...)}'` liefert 7 Nachkommastellen; funktioniert, ist
  aber fragil → `formatDateTime(addMinutes(utcNow(),-20),'yyyy-MM-ddTHH:mm:ssZ')`.
- Kein HTML-Escaping im Mailbody (`&`, `<` im Firmennamen zerlegen die Tabelle).
- `KanalLabel` ist das einzige Feld ohne `if(empty(...))`-Fallback (inkonsistent).
- Betreffschema uneinheitlich: `[Abgebrochener Anruf] - Firma: X` vs.
  `[Kritikalität] - X` → für Outlook-Regeln beim Kunden vereinheitlichen.
- `flowKind: Stateless` beim Staging: Läufe werden nur eingeschränkt protokolliert
  — fehlende Einträge im Verlauf sind kein verlorener Aufruf.

---

## Betriebs-Runbook (Anleitungen zu den Review-Punkten)

### Karenzfenster ändern — es gibt kein Feld dafür

Das „Karenzfenster" ist **keine Einstellung**, sondern die Zahl `-10` im
OData-Filter des Sweeps. Zu finden unter:

`Sweep offene Anrufe` → Node **`Elemente abrufen`** → Feld **`Filterabfrage`**
(engl. *Filter Query*, liegt unter *Erweiterte Parameter*):

```
Status eq 'offen' and Modified lt '@{addMinutes(utcNow(),-10)}'
```

`-10` → `-20` setzen. Semantik: „nur Datensätze, deren letzte Änderung mindestens
20 Minuten zurückliegt". Größere Zahl = geduldiger = weniger Gefahr, in ein noch
laufendes Gespräch zu mailen; kleinere Zahl = schnellere Abbruch-Mail.

Zum Sofort-Testen temporär auf `-0` setzen (danach zurückstellen) und den Sweep
über *Testen → Manuell* auslösen.

---

### Aufräum-Policy für `Telefonbot-Anrufe`

SharePoint hat für Listen **keine eingebaute Zeitsteuerung** („nach 90 Tagen
löschen") ohne Purview-Aufbewahrungsrichtlinien (lizenzpflichtig, Overkill).
Der pragmatische Weg ist ein vierter, kleiner Flow:

**Flow `Telefonbot-Anrufe aufräumen`**

| Node | Konfiguration |
|------|---------------|
| Trigger `Wiederholung` | wöchentlich, Sonntag nachts |
| `Elemente abrufen` | Liste `Telefonbot-Anrufe`, Filterabfrage:<br>`Status eq 'gesendet' and Modified lt '@{addDays(utcNow(),-90)}'`<br>`$top` = 500 |
| `Auf alle anwenden` → `Element löschen` | ID = `@item()?['ID']` |

Zwei Sicherungen dabei:

1. **Nur `gesendet`** löschen — ein `offen` gebliebener Datensatz ist eine
   unbearbeitete Anfrage und darf nie stillschweigend verschwinden.
2. **Vor dem Löschen archivieren**, falls das Reporting historisch sein soll:
   `Elemente abrufen` → `CSV-Tabelle erstellen` → `Datei erstellen` in einer
   SharePoint-Bibliothek. Sonst geht die Abbruchquote der alten Monate verloren.

> **Nach dem Setzen der drei Indizes (erledigt 2026-09-04) ist das kein
> Blocker mehr**, sondern Listenhygiene: Ein Filter auf eine *indizierte* Spalte
> funktioniert auch jenseits von 5.000 Items, solange die Treffermenge dieser
> Klausel unter 5.000 bleibt — bei `Status eq 'offen'` immer der Fall.

---

### Fehler-Monitoring — es gibt keine Checkbox

Power Automate schickt Flow-Besitzern nur eine **wöchentliche Sammel-Mail** über
fehlgeschlagene Läufe. Zu träge für einen Schutzmechanismus, der still ausfallen
kann. Der lizenzfreie Standardweg ist das **Try/Catch-Muster über `runAfter`**:

1. Im Flow eine letzte Aktion **`E-Mail senden (V2)`** ganz unten anhängen
   (Empfänger: du; Betreff z. B. `⚠ Flow-Fehler: Staging schreiben`).
2. Auf diesem Node: **`…` → `Ausführen nach konfigurieren`** (engl. *Configure
   run after*) → Häkchen bei **`ist fehlgeschlagen`**, **`Zeitüberschreitung`**
   und **`wurde übersprungen`** setzen, Häkchen bei `ist erfolgreich`
   **entfernen**.

Damit läuft der Node ausschließlich, wenn der Vorgänger gescheitert ist.
Sinnvolle Platzierung:

| Flow | Node, hinter dem der Alarm hängt |
|------|----------------------------------|
| `Staging schreiben` | `Bedingung` |
| `Sweep offene Anrufe` | `Auf alle anwenden` |
| `Ticketerstellung` | letzte Aktion des Erfolgs-Astes |

In den Alarm-Body gehören `@{workflow()?['run']?['name']}` (Lauf-ID) und die
ConversationId — sonst ist der Lauf im Verlauf nicht wiederzufinden.

> `Staging schreiben` ist **Stateless** — dort werden Läufe nur eingeschränkt
> protokolliert. Der Alarm-Node ist deshalb dort besonders wichtig; alternativ
> den Flow auf *Stateful* umstellen (`Einstellungen` des Triggers).

---

### Monotones Update im Staging (Platzhalter überschreiben keine echten Werte)

**Wo:** `Staging schreiben` → `Bedingung` → Wahr-Ast → `For each` →
**`Element aktualisieren`**. Die drei Felder `Anrufgrund`, `Anlage`, `Anliegen`
sind dort per Dynamischem Inhalt gebunden. Jeweils das `×` am Chip klicken und
über den **`fx`**-Reiter durch einen Ausdruck ersetzen:

| Feld | Ausdruck |
|------|----------|
| `Anrufgrund` | `if(or(empty(triggerBody()?['text_4']), equals(triggerBody()?['text_4'],'nicht angegeben')), coalesce(item()?['Anrufgrund0'],'nicht angegeben'), triggerBody()?['text_4'])` |
| `Anlage` | `if(or(empty(triggerBody()?['text_5']), equals(triggerBody()?['text_5'],'nicht erfasst')), coalesce(item()?['Anlage'],'nicht erfasst'), triggerBody()?['text_5'])` |
| `Anliegen` | `if(or(empty(triggerBody()?['text_6']), equals(triggerBody()?['text_6'],'nicht erfasst')), coalesce(item()?['Anliegen'],'nicht erfasst'), triggerBody()?['text_6'])` |

**Platzhalter — projektweit verifiziert am 2026-09-04** (Grep über alle
`topics/*.mcs.yml`; an allen fünf Staging-Aufrufpunkten identisch):

| Feld | Bot-Bindung | Platzhalter im `equals()` |
|------|-------------|---------------------------|
| Anrufgrund | `If(IsBlank(Global.Anrufgrund), "nicht angegeben", Text(Global.Anrufgrund))` | `nicht angegeben` |
| Anlage | `If(IsBlank(Global.Anlage), "nicht erfasst", Global.Anlage)` | `nicht erfasst` |
| Anliegen | `If(IsBlank(Global.Anliegen), "nicht erfasst", Global.Anliegen)` | `nicht erfasst` |

Wird eine Bot-Bindung später geändert, **muss der `equals()`-Text im Flow
mitgezogen werden** — sonst greift die Prüfung still nicht mehr und das Update
ist wieder nicht monoton.

**Achtung — Platzhaltertext muss exakt stimmen.** Die Bindungen im Bot lauten
`If(IsBlank(Global.X), "nicht angegeben"/"nicht erfasst", …)`; weicht der Text im
`equals()` auch nur in der Groß-/Kleinschreibung ab, greift die Prüfung nicht.
Vor der Änderung in `Kundendatenerfassen.mcs.yml` / `Anrufgrunderfassen.mcs.yml`
gegenprüfen, welcher Platzhalter je Feld tatsächlich gesendet wird.

`Firmenname`, `Ansprechpartner`, `Telefonnummer` bleiben roh gebunden — sie sind
ab dem ersten Aufruf gefüllt und haben keinen Platzhalter.

**Wirkung:** Ein Staging-Lauf kann Werte nur noch **füllen**, nie leeren. Damit
ist die Reihenfolge der asynchronen Aufrufe egal.

---

### Interner Schleifenname (Fehler `InvalidTemplate`)

`items('foreach')` ist falsch — `foreach` ist der *Typ* der Aktion, nicht ihr
Name. Der interne Name der Schleife in `Staging schreiben` ist **`For_each`**
(Leerzeichen → Unterstrich, aus der Codeansicht), im Sweep dagegen
**`Auf_alle_anwenden`**.

**Deshalb grundsätzlich `item()?['ID']` verwenden** statt `items('<Name>')` — es
bezieht sich immer auf die innerste Schleife, braucht keinen Namen und überlebt
Umbenennungen und Sprachwechsel der Oberfläche. Genau so ist es im Sweep bereits
gelöst.
