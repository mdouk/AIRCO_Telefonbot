kind: AdaptiveDialog
beginDialog:
  kind: OnRedirect
  id: main
  actions:
    - kind: Question
      id: question_RZHeKB
      interruptionPolicy:
        allowInterruption: true

      variable: init:Global.Anliegen
      prompt:
        text:
          - Bitte beschreiben Sie nun Ihr Anliegen.
        speak:
          - Bitte beschreiben Sie nun Ihr Anliegen.

      entity: StringPrebuiltEntity

    - kind: BeginDialog
      id: uFP7SX
      dialog: mosaiic_AIRCOTelefonBot.topic.Zusammenfassung-Slim

inputType: {}
outputType: {}