# Anleitung: Microsoft Teams Phone mit Copilot Studio Bot verbinden

**Kontext AIRCO:** Die Festnetznummer ist bereits vorhanden. Ziel ist es, eingehende
Anrufe auf dieser Nummer an den Copilot-Studio-Voice-Agent weiterzuleiten statt an
einen Menschen/Warteschleife.

**Admin-Zugang erforderlich:** Alle Schritte in Teams Admin Center und Microsoft 365
Admin Center erfordern globale Admin- oder Teams-Admin-Rechte. Falls Michael Doukas
diesen Zugang nicht hat, muss der IT-Verantwortliche bei AIRCO involviert werden.

---

## 1. Voraussetzungen (Checkliste vor dem Start)

| # | Voraussetzung | Wo prüfen |
|---|---------------|-----------|
| 1 | **Microsoft Teams Phone System Lizenz** vorhanden | M365 Admin Center → Billing → Licenses |
| 2 | **Calling Plan ODER Direct Routing** aktiv (für die Festnetznummer) | Teams Admin Center → Voice → Phone Numbers |
| 3 | **Festnetznummer** ist dem Tenant zugewiesen | Teams Admin Center → Voice → Phone Numbers |
| 4 | **Copilot Studio Agent** ist gebaut und als Voice Agent konfiguriert | Copilot Studio (✅ bereits erledigt) |
| 5 | **Teams Admin Center Zugang** (Admin-Rechte) | admin.teams.microsoft.com |
| 6 | **Microsoft 365 Admin Center Zugang** | admin.microsoft.com |

---

## 2. Schritt 1 — Resource Account erstellen

Ein Resource Account ist ein spezielles Konto in Azure AD/Entra ID, das einer
automatisierten Telefonfunktion (Bot, Auto Attendant, Call Queue) eine Rufnummer
zuordnet. Ohne Resource Account kann keine Nummer auf den Bot geleitet werden.

**Wo:** Teams Admin Center → Voice → Resource accounts → **+ Add**

| Feld | Wert für AIRCO |
|------|---------------|
| Display name | `AIRCO Service Bot` |
| Username | `airco-service-bot@[tenant].onmicrosoft.com` |
| Resource account type | **Auto attendant** |

Nach dem Anlegen: **Lizenz zuweisen**
- M365 Admin Center → Users → den neuen Resource Account suchen
- Lizenz zuweisen: **Microsoft Teams Phone Resource Account** (kostenlose Lizenz,
  separat von normalen Teams-Lizenzen)

---

## 3. Schritt 2 — Festnetznummer dem Resource Account zuweisen

**Wo:** Teams Admin Center → Voice → Phone numbers

1. Die AIRCO-Festnetznummer in der Liste suchen
2. Klick auf die Nummer → **Edit**
3. Feld „Assigned to": Resource Account `AIRCO Service Bot` auswählen
4. Speichern

Nach dieser Zuweisung leitet Teams eingehende Anrufe auf dieser Nummer an den
Resource Account weiter — der Resource Account muss dann noch mit dem Bot verknüpft
werden (Schritt 4).

**Hinweis Direct Routing:** Falls AIRCO eine eigene Telefonanlage/SBC (Session Border
Controller) nutzt statt eines Microsoft Calling Plans, muss die Nummer dort über eine
Voice Route dem Resource Account zugeordnet werden. Das erfordert zusätzliche
Konfiguration im Teams Admin Center → Voice → Direct Routing.

---

## 4. Schritt 3 — Copilot Studio Agent veröffentlichen

Bevor der Agent mit einem Kanal verbunden werden kann, muss er veröffentlicht sein.

**Wo:** Copilot Studio → oben rechts → **Publish**

- Klick auf „Publish" → „Publish latest content"
- Warten bis Status „Published" angezeigt wird

**Wichtig:** Jede spätere Änderung am Agent (Topics, Flows, etc.) erfordert ein erneutes
Publish, damit die Änderungen live gehen.

---

## 5. Schritt 4 — Teams Phone Kanal in Copilot Studio aktivieren

**Wo:** Copilot Studio → linke Sidebar → **Channels** (oder Settings → Channels)

1. Kanal **„Microsoft Teams"** suchen und öffnen
2. **„Turn on Teams"** klicken
3. Im Abschnitt **„Telephony"** oder **„Voice"**: Voice-Verbindung aktivieren
4. Es wird eine **App-ID / Bot-Endpunkt-URL** generiert — diese wird in Schritt 5 benötigt

**Hinweis:** Die genaue Bezeichnung der UI-Elemente variiert je nach Copilot-Studio-
Version. Suche nach Begriffen wie „Telephony", „Voice channel", „Phone" oder
„Teams Phone".

---

## 6. Schritt 5 — Auto Attendant erstellen und mit Bot verknüpfen

Der Auto Attendant verbindet den Resource Account (= Rufnummer) mit dem
Copilot-Studio-Bot.

**Wo:** Teams Admin Center → Voice → **Auto attendants** → **+ Add**

### Allgemein
| Feld | Wert |
|------|------|
| Name | `AIRCO Service Bot Auto Attendant` |
| Time zone | `(UTC+01:00) Amsterdam, Berlin, Bern, Rome, Stockholm, Vienna` |
| Language | `Deutsch (Deutschland)` |
| Operator | Optional: Falk Recknagel (Serviceleiter) |

### Anrufverhalten (Call flow)
- **Greeting**: Keine (der Bot begrüßt selbst)
- **Route call to**: **Bot** → Copilot Studio Agent auswählen
  - Hier erscheint der in Schritt 4 erstellte Teams-Kanal des Agents

### Geschäftszeiten
- Geschäftszeiten definieren: Mo–Fr 08:00–17:00 Uhr
- Außerhalb der Geschäftszeiten: Weiterleitung an Voicemail ODER andere Nummer
  (der Bot selbst prüft Geschäftszeiten intern per Power Fx — diese Einstellung
  ist eine zusätzliche Sicherheitsebene auf Teams-Ebene)

### Resource Account verknüpfen
- Am Ende des Wizards: Resource Account `AIRCO Service Bot` zuordnen

---

## 7. Schritt 6 — Testen mit echtem Anruf

Nach vollständiger Einrichtung:

1. Vom Mobiltelefon die AIRCO-Festnetznummer anrufen
2. Bot sollte sich mit der Begrüßung melden (Conversation Start Topic)
3. Gespräch bis zum Ende durchführen → prüfen ob E-Mail ankommt
4. Prüfpunkte:
   - Aussprache „GmbH" (offener Praxistest aus ToDos.md)
   - Caller-ID korrekt erkannt (System.Activity.From.Name)
   - „Unterhaltung übertragen" funktioniert mit echter Zielrufnummer
   - Anliegen-leer-Fall (Sofort-Transfer bei Produktionsstillstand)

---

## 8. Bekannte Probleme / Troubleshooting

| Problem | Ursache | Lösung |
|---------|---------|--------|
| Nummer kann nicht zugewiesen werden | Keine Phone System Lizenz | M365 Admin → Lizenzen prüfen |
| Bot antwortet nicht | Agent nicht published | Copilot Studio → Publish |
| Bot antwortet auf Englisch | Language-Setting im Auto Attendant falsch | Teams Admin → Auto Attendant → Language auf Deutsch |
| Caller-ID wird nicht erkannt | System.Activity.From.Name verhält sich anders als im Test-Panel | Alternativen: `System.Activity.From.Id` testen |
| Transfer (Unterhaltung übertragen) funktioniert nicht | Zielrufnummer ist Platzhalter | Echte Durchwahl eintragen (siehe ToDos.md) |
| Anruf kommt nicht beim Bot an | Auto Attendant nicht mit Resource Account verknüpft | Schritt 5 wiederholen |

---

## 9. AIRCO-spezifische Offene Punkte (vor Produktivsetzung)

Diese Punkte sind auch in [ToDos.md](ToDos.md) markiert:

| Punkt | Status |
|-------|--------|
| ⚠ Zielrufnummer für „Unterhaltung übertragen" | Michael muss Durchwahl von AIRCO erfragen |
| ⚠ E-Mail-Empfänger (Service-Innendienst + Recknagel/Metzler) | Echte Adressen einsetzen statt Platzhalter |
| ⚠ Praxistest Aussprache „GmbH" | Erst nach Teams-Phone-Verbindung möglich |
| ⚠ Praxistest Caller-ID unter echten Telefonie-Bedingungen | Erst nach Teams-Phone-Verbindung möglich |
| ⚠ Praxistest „Unterhaltung übertragen" | Erst nach Teams-Phone-Verbindung und echter Zielrufnummer |

---

## 10. Lizenz-Übersicht (Kostencheck)

| Lizenz | Wofür | Kosten |
|--------|-------|--------|
| Microsoft Teams Phone System | Grundlage für Teams-Telefonie | In M365 E3/E5 enthalten oder Add-on |
| Microsoft Calling Plan | Nationale/internationale Anrufe über Microsoft | Pro Nutzer/Monat — ODER durch Direct Routing ersetzt |
| Teams Phone Resource Account | Für den Resource Account des Bots | Kostenlos |
| Copilot Studio | Bot-Plattform | Bereits vorhanden |
| Power Automate | Flow-Ausführungen | Copilot-Guthaben-Warnung beachten (siehe ToDos.md) |
