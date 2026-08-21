# Anleitung: Microsoft Teams Phone mit Copilot Studio Bot verbinden

**Kontext AIRCO:** Die Festnetznummer wird per Operator Connect portiert.
Ziel ist es, eingehende Anrufe auf dieser Nummer direkt an den
Copilot-Studio-Voice-Agent weiterzuleiten — ohne Auto Attendant, stattdessen
über den neueren **Agents & Queues**-Ansatz in Teams Admin Center.

**Stand:** 2026-08-10 — aktualisiert auf Basis der tatsächlich implementierten
Lösung (PPTX „AIRCO Lösungskonzept für den Support", Folien 5 / 11–14).
Vorgänger-Ansatz mit Auto Attendant ist durch diesen Ansatz ersetzt.

**Admin-Zugang erforderlich:** Schritte 1–2 erfordern **SuperAdmin**-Rechte
(insbesondere Billing-Einrichtung). Steps 3–4 erfordern Teams-Admin-Rechte.

---

## 0. Voraussetzungen (Checkliste vor dem Start)

| # | Voraussetzung | Wo prüfen |
|---|---------------|-----------|
| 1 | **Microsoft Teams Phone System Lizenz** vorhanden | M365 Admin Center → Billing → Licenses |
| 2 | **Telephonieprovider** über Operator Connect verfügbar | Teams Admin Center → Voice → Operator Connect |
| 3 | **Festnetznummer** ist portierbar (oder Microsoft-Nummer wird gekauft) | Telephonieprovider bestätigen |
| 4 | **Copilot Studio Agent** gebaut, Flow verbunden, LLM = **GPT-4.1** (einziges Modell mit Sprachunterstützung, Stand 21.07.2026) | Copilot Studio → Settings → AI |
| 5 | **Power Platform Umgebung** in EMEA-Region mit Dataverse (Sprachfähigkeit ist regions- und datenbankabhängig) | Power Platform Admin Center → Environments |
| 6 | **Pay-As-You-Go Abrechnungsplan** aktiv (für Voice-Preview und Produktivbetrieb des Agenten) | M365 Admin Center → Billing |
| 7 | **Resource Account** `airco-service-bot@airco-systems.de` erstellt und lizenziert | ✅ erledigt (2026-07-20) |
| 8 | **SuperAdmin-Zugang** für Billing-Einrichtung | M365 Admin Center |

---

## 1. Schritt 1 — Power Platform Admin Center einrichten

**Wo:** [admin.powerplatform.microsoft.com](https://admin.powerplatform.microsoft.com)

### Umgebung konfigurieren

Die Copilot-Studio-Umgebung muss folgende Bedingungen erfüllen — sonst ist die
Voice-/Anruffähigkeit nicht verfügbar:

- **Region: EMEA** (Europa) — andere Regionen unterstützen keinen Sprachkanal
- **Dataverse-Speicher** aktiv — ohne Dataverse wird die Umgebung für den
  Vorschau-Kanal nicht berechtigt

**Einstellungen** (Settings → Product → Features):

| Einstellung | Wert |
|-------------|------|
| Verhalten der Umgebungseinstellungen | **Deaktivieren** |
| Gelöschte Datensätze | **Deaktivieren** |
| Vorschau- und experimentelle KI-Modelle | **Deaktivieren** |

### Pay-As-You-Go Abrechnungsplan einrichten

Unter **Billing → Pay-as-you-Go**:
1. Azure-Dienste verknüpfen
2. Unter Copilot: richtige Umgebung auswählen, Kapazitätsüberschreitungen
   festlegen, Services auswählen (insbesondere Copilot Chat + SharePoint Agents)

---

## 2. Schritt 2 — M365 Admin Center: Resource Account + Billing

**Wo:** [admin.microsoft.com](https://admin.microsoft.com)

> **Hinweis:** Dieser Schritt erfordert SuperAdmin-Rechte.

### Resource Account User anlegen (bereits erledigt 2026-07-20)

Der Resource Account `airco-service-bot@airco-systems.de` ist bereits angelegt
und lizenziert. Zur Dokumentation der Schritte:

1. **Users → Add a User**: neuen User als Resource Account anlegen
2. Lizenz zuweisen: **„Microsoft Teams Phone Resource Account"** (kostenlose
   Sonderlizenz — unter Billing → Licenses suchen)

### Pay-As-You-Go Billing einrichten (SuperAdmin erforderlich)

**Billing → Pay-As-You-Go** — folgende Dienste aktivieren:

| Dienst | Wofür |
|--------|-------|
| **Azure (allgemein)** | Grundlage für KI-Dienste |
| **Microsoft 365 Copilot Chat** | Voice-Kanal in Copilot Studio |
| **SharePoint Agents** | Knowledge-Anbindung im Flow |
| **Copilot** | Agent-Ausführungen |

### Frontier Features aktivieren

**Settings → Frontier Features → aktivieren**

---

## 3. Schritt 3 — Copilot Studio: Agent konfigurieren und veröffentlichen

**Wo:** [copilotstudio.microsoft.com](https://copilotstudio.microsoft.com)

> **Wichtig:** Sicherstellen, dass die richtige **EMEA-Umgebung** ausgewählt ist
> (oben rechts in der Umgebungsauswahl — Funktionalität ist stark davon abhängig).

### Einstellungen (Settings)

| Bereich | Einstellung |
|---------|-------------|
| **Generative KI** | Klassische Orchestrierung (kein generatives Routing) |
| **Sicherheit** | Keine Authentifizierung |
| **Sprachanruf** | Sprache einschalten |
| **LLM** | GPT-4.1 von OpenAI (einziges Modell mit Sprachunterstützung, Stand 21.07.2026) |

### Entitäten

- **Korrekturfeld**: als geschlossene Liste (Closed List) mit Synonymen gepflegt
  (Firmenname / Ansprechpartner / Telefonnummer / Inbetriebnahme / Wartungsvertrag)

### Teams-Telefon-Kanal aktivieren

**Channels → Teams-Telefon-Agent**:
1. Kanal hinzufügen
2. Sprache: **Ein**
3. Status: **Verbunden**

> **Hinweis:** Um den Kanal auf „Vorschau" zu aktivieren, muss das
> **Pay-As-You-Go**-Abrechnungsmodell sowohl für Azure-Dienste als auch für
> Copilot Chat und SharePoint Agents eingerichtet sein (Schritt 2).

### Flow verbinden

- Agent → Tools → Power Automate Cloud Flow „Anliegen weiterleiten (KI)"
  verbinden
- In Aktion-Nodes: Ausgabevariablen selektieren, die an den Flow übergeben
  werden (text=Firmenname, text\_1=Ansprechpartner, text\_2=Telefonnummer,
  text\_3=Anliegen, text\_4=KanalLabel, boolean=VertragVorhanden,
  boolean\_1=Inbetriebnahme)

### Lösung veröffentlichen und herunterladen

1. **„Publish" → „Publish latest content"** — warten bis Status „Published"
2. Lösung als **nicht verwaltete Lösung** exportieren
3. **Lösung als .zip herunterladen** — wird in Schritt 4 als Teams App
   hochgeladen

> **Hinweis:** Jede spätere Änderung am Agent erfordert erneutes Publish +
> neuen Export, damit die Änderungen live gehen.

---

## 4. Schritt 4 — Teams Admin Center: App hochladen + Telefonie konfigurieren

**Wo:** [admin.teams.microsoft.com](https://admin.teams.microsoft.com)

### 4a. Agent als Teams App hochladen

**Actions → Upload new App**:
1. Die in Schritt 3 heruntergeladene `.zip`-Datei hochladen
2. App erscheint danach unter den verfügbaren Teams-Apps

### 4b. Voice / Telefonie einrichten

**Voice → Service Configuration**:
- Unternehmen als Telefonie-Nutzer authentifizieren lassen

**Voice → Operator Connect**:
- Bestehenden Telephonieprovider anbinden (AIRCO-Festnetznummer ist beim
  Provider portierbar)

**Voice → Phone Numbers**:
- Telefonnummer des Telephonieproviders portieren
- Alternativ: eine oder mehrere Microsoft-Telefonnummern kaufen

### 4c. Resource Account mit Nummer und Agent verknüpfen

**Voice → Resource Accounts** → Resource Account `AIRCO Service Bot` öffnen:

| Schritt | Aktion |
|---------|--------|
| 1 | **Telefonnummer zuordnen**: portierte AIRCO-Festnetznummer zuweisen |
| 2 | **Lizenz**: in M365 User Management die Lizenz „Microsoft Teams Phone Resource Account" zugewiesen (✅ bereits erledigt) |
| 3 | **Copilot Agent verbinden**: den in 4a hochgeladenen Agent verknüpfen |

### 4d. Agent in Agents & Queues registrieren

**Templates and Resources → Agents and Queues → Copilot Studio Agent**:
- Den hochgeladenen Copilot-Studio-Agenten hier als Queue-Agent registrieren
- Dies ist der Schritt, der den eingehenden Anruf auf der Resource-Account-Nummer
  an den Bot weiterleitet

---

## 5. Schritt 5 — Testen mit echtem Anruf

Nach vollständiger Einrichtung:

1. Vom Mobiltelefon die AIRCO-Festnetznummer anrufen
2. Bot sollte sich mit der Begrüßung melden (Conversation Start Topic)
3. Gespräch bis zum Ende durchführen → prüfen ob E-Mail ankommt
4. Prüfpunkte:
   - Aussprache „GmbH" in der Begrüßung (erster Praxistest)
   - Caller-ID korrekt erkannt (`System.Activity.From.Name` trägt Rufnummer)
   - Ja/Nein-Fragen (Inbetriebnahme, Vertrag) korrekt erkannt
   - Korrektur-Schleife funktioniert (bis 3 Versuche)
   - E-Mail kommt an `Service@airco-systems.de` an (Interim-Postfach)
   - KI-Zusammenfassung + Kritikalität im E-Mail-Betreff korrekt

---

## 6. Bekannte Probleme / Troubleshooting

| Problem | Ursache | Lösung |
|---------|---------|--------|
| Sprachkanal nicht verfügbar | Umgebung nicht in EMEA-Region | Power Platform Admin Center → neue EMEA-Umgebung anlegen |
| Voice-Vorschau nicht aktivierbar | Pay-As-You-Go fehlt | M365 Admin Center → Billing → Pay-As-You-Go für alle 3 Dienste aktivieren |
| Bot antwortet nicht | Agent nicht published oder Flow nicht verbunden | Copilot Studio → Publish + Flow-Verbindung prüfen |
| Bot antwortet auf Englisch | Spracheinstellung im Agenten falsch | Copilot Studio → Settings → Language auf Deutsch |
| Caller-ID nicht erkannt | `System.Activity.From.Name` leer | Erst unter echten Telefoniebedingungen testbar (Test-Panel liefert andere Werte) |
| KI-Ausfall → E-Mail kommt trotzdem an | Fallback-Node feuert (Configure run after: Failed/TimedOut/Skipped) | Rohtext + „manuell prüfen" an `Service@airco-systems.de` |
| Bot-Timeout (`FlowActionBadGateway`) | `Respond to agent` steht zu spät im Flow (nach KI-Schritten) | `Respond to agent` direkt nach Trigger platzieren; KI-/Mail-Teil läuft dann als fire-and-forget |
| Anruf kommt nicht beim Bot an | Resource Account nicht mit Agent verknüpft | Schritt 4c wiederholen |
| `.zip`-Upload fehlgeschlagen | Falsche Lösung exportiert | In Copilot Studio „nicht verwaltete Lösung" wählen |

---

## 7. AIRCO-spezifische offene Punkte (vor Produktivsetzung)

| Punkt | Status |
|-------|--------|
| ✅ Resource Account `airco-service-bot@airco-systems.de` erstellt + lizenziert | erledigt (2026-07-20) |
| ⚠ Festnetznummer portieren (Operator Connect) | offen |
| ⚠ Telefonnummer dem Resource Account zuweisen | offen |
| ⚠ Agent als Teams App hochladen + in Agents & Queues registrieren | offen |
| ⚠ Praxistest Aussprache „GmbH" | erst nach Verbindung möglich |
| ⚠ Praxistest Caller-ID unter echten Telefoniebedingungen | erst nach Verbindung möglich |
| ⚠ `Respond to agent`-Position im Flow prüfen (Bot-Timeout-Fix) | offen |
| ⚠ Störungs-/Anliegenliste von AIRCO → in KI-Node einbinden | wartet auf Kundenlieferung |
| ⚠ 4 Postfach-Adressen je Kritikalität (interim: alle → `Service@airco-systems.de`) | wartet auf AIRCO |
| ⚠ Pay-As-You-Go Billing + Copilot-Guthaben (SuperAdmin, ~40 Anrufe/Tag) | wartet auf AIRCO-Admin |

---

## 8. Lizenz-Übersicht (Kostencheck)

| Lizenz / Dienst | Wofür | Kosten |
|-----------------|-------|--------|
| Microsoft Teams Phone System | Grundlage für Teams-Telefonie | In M365 E3/E5 enthalten oder Add-on |
| Operator Connect | Bestehende Festnetznummer portieren | Beim Telephonieprovider klären |
| Teams Phone Resource Account | Für den Resource Account des Bots | **Kostenlos** |
| Pay-As-You-Go (Azure + Copilot Chat + SharePoint Agents) | Voice-Kanal + KI-Ausführungen | Nutzungsbasiert (bei ~40 Anrufen/Tag klären) |
| Copilot Studio | Bot-Plattform | Bereits vorhanden |
| Power Automate + AI Builder (GPT-4.1) | Flow-Ausführungen + KI-Schritt | Copilot-Guthaben-Warnung beachten |

---

## 9. Prozessübersicht (Kurzreferenz)

```
Power Platform Admin Center
  └── EMEA-Umgebung + Dataverse + Pay-As-You-Go + Frontier Features
      ↓
M365 Admin Center (SuperAdmin)
  └── Resource Account User + Lizenz + Pay-As-You-Go Billing + Frontier Features
      ↓
Copilot Studio
  └── Einstellungen + Teams-Kanal + Flow verbinden + Lösung (nicht verwaltet) → .zip
      ↓
Teams Admin Center
  ├── App hochladen (.zip)
  ├── Operator Connect (Telephonieprovider anbinden)
  ├── Phone Numbers (Nummer portieren)
  ├── Resource Accounts (Nummer + Agent verknüpfen)
  └── Agents & Queues (Copilot Studio Agent registrieren)
      ↓
Praxistest (echter Anruf)
```
