kind: AdaptiveDialog
beginDialog:
  kind: OnRedirect
  id: main
  actions:
    - kind: Question
      id: question_qMVPee
      interruptionPolicy:
        allowInterruption: true

      variable: Global.Firmenname
      prompt:
        text:
          - Für welche Firma kontaktieren Sie uns?
        speak:
          - Für welche Firma rufen Sie an?

      entity: StringPrebuiltEntity

    - kind: Question
      id: question_H0ONs7
      interruptionPolicy:
        allowInterruption: true

      variable: init:Global.Ansprechpartner
      prompt:
        text:
          - Wie ist Ihr Name?
        speak:
          - Wie ist Ihr Name?

      entity: PersonNamePrebuiltEntity

    - kind: Question
      id: 9CvROV
      interruptionPolicy:
        allowInterruption: true

      variable: Global.Telefonnummer
      prompt:
        text:
          - Unter welcher Nummer können wir Sie erreichen?
        speak:
          - Unter welcher Nummer können wir Sie erreichen?

      entity: PhoneNumberPrebuiltEntity

    - kind: SetVariable
      id: 2X2Lbc
      variable: Global.KanalLabel
      value: Telefon

    - kind: BeginDialog
      id: lzTpPJ
      dialog: mosaiic_AIRCOTelefonBot.topic.Anlagenerfassung

inputType: {}
outputType: {}