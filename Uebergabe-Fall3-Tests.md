# Übergabe: Fall 3 (Staging + Sweep) — Weitertest

**Erstellt:** 2026-09-03 · **Für Session:** ab 2026-09-04
**Thema:** „Kein Anruf geht verloren" — ausfallsichere E-Mail bei Gesprächsabbruch.
Konzept-Details in [Konzept-Ausfallsichere-Weiterleitung.md](Konzept-Ausfallsichere-Weiterleitung.md).

---

## Wo wir stehen

**Fall 1 + 2 erledigt** (Safety-Net bei Fehler/Stille + Reorder „E-Mail vor Verabschiedung").

**Fall 3 (Abbruch mitten im Gespräch) — gebaut und im Kern verifiziert:**

| Baustein | Status |
|----------|--------|
| SharePoint-Liste `Telefonbot-Anrufe` (Site `aircodruckluft/SharePoint`) | ✅ |
| Flow **`Staging schreiben`** (`a0a99959-80a7-f111-b8de-7ced8d476627`) — Upsert per ConversationId | ✅ |
| 4 Staging-Aufrufe im Bot (Kundendaten, Anrufgrund, Anlage, Anliegen) | ✅ |
| **`Anliegen weiterleiten – slim`** (`019885f0-e29a-f111-b8db-7ced8d476627`) → setzt Datensatz auf `gesendet`/`vollstaendig` in **beiden** Ästen; neue Eingabe `text_7 = ConversationId` | ✅ |
| Flow **`Sweep offene Anrufe`** (stündlich; `Status eq 'offen' and Modified lt now-10min` → Abbruch-Mail + `gesendet`) | ✅ gebaut, **noch nicht getestet** |

**Heute gefixter Bug:** Staging crashte mit `FlowActionBadRequest` — leere **Pflicht**-Parameter (`text_5` = Anlage) beim frühen Aufruf. Fix: alle 4 Staging-Aufrufe binden `text_4/5/6` jetzt als
`=If(IsBlank(Global.X), "nicht angegeben"/"nicht erfasst", …)` statt roh.
Nachweis im Trace `Tests/Test 2/dialog.json`: `invokeFlowAction_nOBDGN` und `invokeFlowAction_stgAnr` mit `exception: ""` (Erfolg); Anlage-Frage wird nach „Störung" korrekt gestellt (der frühere Skip war eine Crash-Folge).

---

## Wichtige Eigenheiten (nicht vergessen!)

- **Staging läuft asynchron/verzögert:** Der Flow-Aufruf erscheint im Trace erst 1–2 Turns später und blockt das Gespräch nicht. → Nach einem Test **kurz warten**, dann SharePoint/Ausführungsverlauf prüfen. „Direkt danach" ist zu früh.
- **Test-Panel testet die Draft:** Nach einem MCP-Push im Browser **F5 / Agent neu laden** und **frische Unterhaltung** starten, sonst läuft der alte Stand.
- **Sweep-Staleness 10 Min:** Zum Sofort-Test entweder ~10 Min warten **oder** im Sweep-Filter temporär `addMinutes(utcNow(),-10)` auf `-0` setzen (danach zurückstellen). Sweep manuell via „Testen → Manuell" auslösen statt die Stunde abzuwarten.
- **Empfänger:** Sweep- und slim-Mail gehen aktuell auch an `service@airco-systems.de` + `michael.doukas@mosaiic.com`. **Vor Go-Live:** michael raus, Service rein. Für Tests ggf. Service entfernen.
- **Platzhalter statt leer:** Staging speichert `"nicht erfasst"`/`"nicht angegeben"` statt leer. Die `if(empty(...))`-Prüfung im Sweep löst dadurch praktisch nie aus (harmlos).

## Arbeitsweise MCP (bewährt)

Lokale Quelle der Wahrheit für Push: `agent/AIRCO Telefon-Bot/topics/*.mcs.yml`.
`YAML/*.md` sind Arbeitskopien (parallel pflegen).
**Immer erst Pull, dann Edit, dann Push** (sonst wird Cloud-Stand überschrieben).
Skill: `copilot-studio:manage-agent` mit `pull` / `push` (Browser-Login nötig).
Push macht nur die **Draft** — für echten Anruf ist **Publish** nötig.

---

## Nächste Schritte: Testszenarien (Phase 1 im Test-Panel, kein Publish nötig)

Zuerst offene Verifikation von heute abschließen:
- [ ] **Test 2 nachprüfen:** Steht der Datensatz `7bf24e65-…` in `Telefonbot-Anrufe`? Läufe im `Staging schreiben`-Verlauf erfolgreich?

Dann die Szenarien:

| # | Szenario | Erwartung |
|---|----------|-----------|
| 1 | **Normaler Volldurchlauf** (bis Zusammenfassung bestätigt) | 1 slim-Mail; Datensatz `gesendet`/`vollstaendig`; Sweep ignoriert ihn |
| 2 | **Abbruch nach Kundendaten** (Panel schließen nach Telefon) | Datensatz `offen` mit 3 Feldern → Sweep manuell → Abbruch-Mail, Datensatz `gesendet`/`abgebrochen` |
| 3 | **Abbruch nach Anliegen** (weiter erfasst, dann schließen) | Abbruch-Mail enthält Anlage/Anliegen |
| 4 | **Dedup-Kontrolle** | Nach Szenario 1 Sweep laufen lassen → **keine** zweite Mail |
| 5 | **Unter Schwelle** (nur Name, dann schließen) | **kein** Datensatz, **keine** Mail (bewusst) |

**Phase 2 (echter Anruf, braucht Publish):** dieselben Abbruch-/Stille-Szenarien am Telefon — v. a. hartes Auflegen nach den 3 Feldern (der Originalfall aus dem Kundentermin).

---

## Nach erfolgreichem Test

- [ ] `Architektur.md`, `ToDos.md`, `Topics.md` auf Fall-3-Stand fortschreiben (bislang nur Konzeptdoku aktualisiert).
- [ ] Empfänger im Sweep/slim für Go-Live umstellen (Service rein, michael raus).
- [ ] Deadline: **Hausmesse 16./17. September** — Bot muss live sein.

## Testergebnis 2026-09-04 (echter Anruf, Phase 2)

Szenario „Abbruch mitten im Gespräch" **erfolgreich**: Staging-Datensatz angelegt,
Sweep hat ihn gefunden und die Abbruch-Mail versendet. Damit ist die Kernmechanik
von Fall 3 am Telefon bestätigt.

Zwei Nacharbeiten aus der Mail (Details in
[Konzept-Ausfallsichere-Weiterleitung.md](Konzept-Ausfallsichere-Weiterleitung.md),
Abschnitt „Spaltenfalle"):

1. **Sweep-Mail, Zeile „Anrufgrund"** zeigt Roh-JSON
   (`…SPListExpandedReference…"Value":"abgebrochen"`), weil der Dynamic-Content-
   Picker die umbenannte Auswahl-Spalte (intern `Anrufgrund` = Abschlussart)
   erwischt hat. Fix: Ausdruck `items('Apply_to_each')?['Anrufgrund0']`;
   Auswahl-Spalten grundsätzlich mit `?['Value']`.
2. **Anlage „nicht erfasst" trotz gefülltem Anliegen** ⇒ der Bot lief durch den
   else-Ast in `Anrufgrunderfassen.mcs.yml:57`, der Anrufgrund wurde also nicht
   als Störung/Wartung erkannt. Nach Fix 1 in der Mail verifizieren und ggf. die
   Closed List `Anrufgrund` um Synonyme erweitern.

## Referenzdateien

- Konzept: `Konzept-Ausfallsichere-Weiterleitung.md`
- Test-Traces: `Tests/Test 1/`, `Tests/Test 2/` (`dialog.json` = voller Dialog-Trace mit Exceptions)
- Bot-Topics: `agent/AIRCO Telefon-Bot/topics/` (+ Arbeitskopien in `YAML/`)
- Kundentermin (Ursprung der Anforderung): `Protocols/2026-09-03 Kundentermin Transkript.md`

## Code-Review Staging + Sweep (2026-09-04)

Beide `workflow.json` vollständig gegen das Konzept geprüft — Befunde und Fixes in
[Konzept-Ausfallsichere-Weiterleitung.md](Konzept-Ausfallsichere-Weiterleitung.md),
Abschnitt „Code-Review Staging + Sweep".

**Vor der Messe abzuarbeiten (P1):**
- [x] ~~Sweep-Takt~~ — **stündlich bleibt** (Entscheidung 2026-09-04), Doku nachgezogen
- [x] Sweep-Karenz 10 → 20 Min (2026-09-04). Kein eigenes Feld — die Zahl steht
      in `Sweep` → `Elemente abrufen` → **Filterabfrage**: `addMinutes(utcNow(),-20)`
- [x] Staging-`For each`: ID auf `item()?['ID']` (2026-09-04). *Stolperstein war*
      `items('foreach')` → `InvalidTemplate`: `foreach` ist der **Typ** der Aktion,
      nicht ihr Name (der lautet `For_each`). `item()` braucht gar keinen Namen und
      ist deshalb die robustere Form — siehe Sweep.
- [ ] ⏸ **Zurückgestellt:** Staging `Respond to Copilot` ans Ende (Race →
      Doppel-Datensätze) — wird separat bearbeitet, siehe Todo-Abschnitt unten
- [ ] **Prüfen:** doppelte `Titel` in `Telefonbot-Anrufe`? (belegt die Race)

**P2:**
- [x] Indizes auf `Title`, `Status`, `Modified` gesetzt (2026-09-04)
- [ ] Aufräum-Flow `Telefonbot-Anrufe aufräumen` (siehe Konzept, Abschnitt
      „Aufräum-Policy“) — nach den Indizes nur noch Listenhygiene, kein Blocker
- [ ] Fehlerbenachrichtigung für beide Flows abonnieren
- [ ] Staging-Update monoton machen (Platzhalter überschreiben keine echten Werte)
      — Platzhalter am 2026-09-04 verifiziert: Anrufgrund `nicht angegeben`,
      Anlage + Anliegen `nicht erfasst`; fertige Ausdrücke im Konzept-Runbook

Der Sweep-Bug „Roh-JSON in der Zeile Anrufgrund" ist im geprüften Stand
**bereits behoben** (`item()?['Anrufgrund0']`).

Die Hypothese „Anlage nicht erfasst" ist offen zwischen **Closed-List-Miss** und
**Race im Staging-Update** (P2 #7) — der `Staging schreiben`-Ausführungsverlauf
zum Testanruf entscheidet.

---

## ⏸ Zurückgestellt: Race beim Staging-Create (separat bearbeiten)

**Beschlossen 2026-09-04:** wird *nicht* vor der Messe angefasst.

**Problem:** `Respond to Copilot` steht als 2. Node, also *vor* `Elemente abrufen`.
Der Flow antwortet dem Bot, bevor der Datensatz existiert. Da Copilot Studio
Staging-Aufrufe um 1–2 Turns verzögert, können zwei Läufe ihr `GetItems` machen,
bevor der erste das Item angelegt hat ⇒ **zwei Items mit derselben ConversationId**
⇒ zwei Abbruch-Mails vom Sweep.

**Lösungskandidaten (noch zu entscheiden):**
1. `Respond to Copilot` ans **Ende** des Flows — serialisiert die Aufrufe
   zwangsläufig. Preis: Bot wartet ~1 s pro Staging-Aufruf (4× pro Anruf).
2. Response vorn lassen, zweites `GetItems` unmittelbar vor `Element erstellen`
   als Last-Minute-Check — verkleinert das Fenster, schließt es nicht.
3. Deduplizierung im Sweep (nach ConversationId gruppieren) — behandelt das
   Symptom, lässt doppelte Datensätze im Reporting stehen.

**Vorbedingung für die Entscheidung:** erst messen, ob die Race real auftritt —
Liste nach `Titel` gruppieren und nach Doubletten suchen. Ohne Befund ist
Option 1 unnötige Latenz im Gespräch.
