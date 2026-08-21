kind: AdaptiveDialog
beginDialog:
  kind: OnRedirect
  id: main
  actions:
    - kind: Question
      id: question_Xnm8Nm
      interruptionPolicy:
        allowInterruption: true

      variable: Global.Anlage
      prompt:
        text:
          - Bitte teilen Sie uns die Anlagenbezeichnung, Serielnummer und das Baujahr mit.
        speak:
          - Bitte teilen Sie uns die Anlagenbezeichnung, Serialnummer und das Baujahr mit.

      entity:
        kind: StringPrebuiltEntity
        sensitivityLevel: None

    - kind: BeginDialog
      id: k1uiUZ
      dialog: mosaiic_AIRCOTelefonBot.topic.Anliegenerfassen

inputType: {}
outputType: {}