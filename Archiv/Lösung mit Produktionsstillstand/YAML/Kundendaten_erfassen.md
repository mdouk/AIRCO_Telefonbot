# YAML: Topic „Kundendaten erfassen" — Stufe 1

**Stand:** 2026-07-14
**Status:** Finalisiert.

**Hinweis Caller-ID-Check:** Statt des Kanals wird geprüft, ob `System.Activity.From.Name`
befüllt ist (`Not(IsBlank(...))`). Funktioniert für echte Anrufe, unterdrückte Nummern
und Test-Panel gleichermaßen.

---

```yaml
kind: AdaptiveDialog
modelDescription: "Erfasst die Kontaktdaten des Anrufers: Firmenname, Ansprechpartner und Rückrufnummer."
beginDialog:
  kind: OnRecognizedIntent
  id: main
  intent: {}
  actions:

    # Node 1 — Firmenname
    - kind: Question
      id: question_MSHPje
      interruptionPolicy:
        allowInterruption: false
      variable: Global.Firmenname
      prompt:
        text:
          - Bitte nennen Sie zuerst Ihren Firmennamen.
        speak:
          - Bitte nennen Sie zuerst Ihren Firmennamen.
      entity: StringPrebuiltEntity

    # Node 2 — Ansprechpartner
    - kind: Question
      id: question_Y2w3or
      interruptionPolicy:
        allowInterruption: false
      variable: init:Global.Ansprechpartner
      prompt:
        text:
          - Wie ist Ihr Name?
        speak:
          - Wie ist Ihr Name?
      entity: StringPrebuiltEntity

    # Node 2.5 — Caller-ID vorhanden?
    - kind: ConditionGroup
      id: conditionGroup_CallerID
      conditions:
        - id: conditionItem_CallerID
          condition: =Not(IsBlank(System.Activity.From.Name))
          displayName: Caller-ID vorhanden
          actions:

            # Node 3a — Caller-ID vorschlagen und bestätigen lassen
            - kind: Question
              id: question_nQmanw
              interruptionPolicy:
                allowInterruption: false
              variable: init:Topic.richtigeTelefonnummer
              prompt:
                text:
                  - Ich sehe, Sie rufen von der Nummer {System.Activity.From.Name} an. Ist das die richtige Rückrufnummer für Sie?
                speak:
                  - Ich sehe, Sie rufen von der Nummer {System.Activity.From.Name} an. Ist das die richtige Rückrufnummer für Sie?
              entity: BooleanPrebuiltEntity

            # Node 3b — Auswertung der Bestätigung
            - kind: ConditionGroup
              id: conditionGroup_Q0Egwx
              conditions:
                - id: conditionItem_0bJgHf
                  condition: =Topic.richtigeTelefonnummer = true
                  displayName: Bedingung Wahr
                  actions:
                    # Ja: Caller-ID direkt übernehmen
                    - kind: SetVariable
                      id: setVariable_9ECvxH
                      variable: Global.Telefonnummer
                      value: =System.Activity.From.Name
              elseActions:
                # Nein: Telefonnummer manuell erfragen
                - kind: Question
                  id: question_KKDsSs
                  interruptionPolicy:
                    allowInterruption: false
                  variable: Global.Telefonnummer
                  prompt:
                    text:
                      - Wie lautet dann die Telefonnummer, unter der wir Sie erreichen können?
                    speak:
                      - Wie lautet dann die Telefonnummer, unter der wir Sie erreichen können?
                  entity:
                    kind: StringPrebuiltEntity
                    sensitivityLevel: None

      elseActions:
        # Keine Caller-ID (Test-Panel, unterdrückte Nummer): direkt fragen
        - kind: Question
          id: question_DirectPhone
          interruptionPolicy:
            allowInterruption: false
          variable: Global.Telefonnummer
          prompt:
            text:
              - Unter welcher Nummer können wir Sie erreichen?
            speak:
              - Unter welcher Nummer können wir Sie erreichen?
          entity:
            kind: StringPrebuiltEntity
            sensitivityLevel: None

    # Node 3c — KanalLabel fest setzen (Stufe 1 = ausschließlich Telefon)
    - kind: SetVariable
      id: setVariable_KanalTelefon
      variable: Global.KanalLabel
      value: Telefon

    # Node 4 — Weiter zu Topic "Inbetriebnahme"
    - kind: BeginDialog
      id: WcV779
      dialog: cr0d0_sdhfw.topic.Inbetriebnahme

inputType: {}
outputType: {}
```

---

## Variablen dieses Topics

| Variable | Typ | Gesetzt in |
|----------|-----|-----------|
| `Global.Firmenname` | Text | Node 1 |
| `Global.Ansprechpartner` | Text | Node 2 |
| `Global.Telefonnummer` | Text | Node 3b (Ja: Caller-ID / Nein: manuell) oder elseActions (kein Caller-ID) |
| `Global.KanalLabel` | Text | Node 3c (fest: `"Telefon"`) |
| `Topic.richtigeTelefonnummer` | Boolean | Node 3a (nur wenn Caller-ID vorhanden) |

## Verhalten je Szenario

| Szenario | `From.Name` | Gesprächspfad |
|---|---|---|
| Echter Anruf mit Nummer | `+4989...` | Caller-ID-Frage → Bestätigung oder manuelle Korrektur |
| Unterdrückte Rufnummer | leer | Direkt: „Unter welcher Nummer können wir Sie erreichen?" |
| Test-Panel | leer | Direkt: „Unter welcher Nummer können wir Sie erreichen?" |

## Offene Punkte

- **Caller-ID unter echtem Anruf testen**: `System.Activity.From.Name` liefert
  die Rufnummer nur bei echtem Teams-Phone-Anruf (bestätigt 2026-07-08).
- **`Global.KanalLabel` als Flow-Input übergeben**: In „Zusammenfassung &
  Bestätigung" beim Action Node als `text_4` ergänzen. Reihenfolge der
  bestehenden Inputs nicht verändern (siehe ToDos.md).
