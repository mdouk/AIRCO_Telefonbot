kind: AdaptiveDialog
beginDialog:
  kind: OnRedirect
  id: main
  actions:
    - kind: Question
      id: question_RZHeKB
      interruptionPolicy:
        allowInterruption: false

      variable: init:Global.Anliegen
      prompt:
        text:
          - Bitte beschreiben Sie nun Ihr Anliegen.
        speak:
          - Bitte beschreiben Sie nun Ihr Anliegen.
        allowBargeIn: false

      entity: StringPrebuiltEntity
      voiceInputSettings:
        fallbackDialogOnSilence: mosaiic_AIRCOTelefonBot.topic.EndofConversation
        defaultValueMissingAction: GoToDialog

      fallbackDialogOnInvalidEntity: mosaiic_AIRCOTelefonBot.topic.EndofConversation

    - kind: InvokeFlowAction
      id: invokeFlowAction_stgAli
      displayName: Staging
      input:
        binding:
          text: =System.Conversation.Id
          text_1: =If(IsBlank(Global.Firmenname), "nicht angegeben", Global.Firmenname)
          text_2: =If(IsBlank(Global.Ansprechpartner), "nicht angegeben", Global.Ansprechpartner)
          text_3: =If(IsBlank(Global.Telefonnummer), "nicht angegeben", Global.Telefonnummer)
          text_4: =If(IsBlank(Global.Anrufgrund), "nicht angegeben", Text(Global.Anrufgrund))
          text_5: =If(IsBlank(Global.Anlage), "nicht erfasst", Global.Anlage)
          text_6: =If(IsBlank(Global.Anliegen), "nicht erfasst", Global.Anliegen)
          text_7: =If(IsBlank(Global.KanalLabel), "Telefon", Global.KanalLabel)

      output: {}
      flowId: a0a99959-80a7-f111-b8de-7ced8d476627

    - kind: BeginDialog
      id: uFP7SX
      dialog: mosaiic_AIRCOTelefonBot.topic.Zusammenfassung-Slim

inputType: {}
outputType: {}