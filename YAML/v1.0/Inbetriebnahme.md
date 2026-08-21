kind: AdaptiveDialog
beginDialog:
  kind: OnRedirect
  id: main
  actions:
    - kind: Question
      id: question_PRuokc
      interruptionPolicy:
        allowInterruption: true

      variable: init:Global.Inbetriebnahme
      prompt:
        text:
          - Wurde die Anlage in den letzten 12 Monaten in Betrieb genommen?
        speak:
          - Wurde die Anlage in den letzten 12 Monaten in Betrieb genommen?

      entity: BooleanPrebuiltEntity

    - kind: BeginDialog
      id: Yz5HBW
      dialog: mosaiic_AIRCOTelefonBot.topic.Vertragsfrage

inputType: {}
outputType: {}