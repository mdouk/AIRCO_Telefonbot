kind: AdaptiveDialog
beginDialog:
  kind: OnRedirect
  id: main
  actions:
    - kind: Question
      id: question_SeHkg7
      interruptionPolicy:
        allowInterruption: true

      variable: init:Global.VertragVorhanden
      prompt:
        text:
          - Haben Sie einen Wartungsvertrag mit Garantieverlängerung mit uns?
        speak:
          - Haben Sie einen Wartungsvertrag mit Garantieverlängerung mit uns?

      entity: BooleanPrebuiltEntity

    - kind: BeginDialog
      id: Evl3va
      dialog: mosaiic_AIRCOTelefonBot.topic.Anliegenerfassen

inputType: {}
outputType: {}