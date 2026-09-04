kind: AdaptiveDialog
beginDialog:
  kind: OnRedirect
  id: main
  actions:
    - kind: Question
      id: question_qMVPee
      interruptionPolicy:
        allowInterruption: false

      variable: Global.Firmenname
      prompt:
        text:
          - Für welche Firma kontaktieren Sie uns?
        speak:
          - Für welche Firma rufen Sie an?
        allowBargeIn: false

      entity: StringPrebuiltEntity
      voiceInputSettings:
        fallbackDialogOnSilence: mosaiic_AIRCOTelefonBot.topic.EndofConversation
        defaultValueMissingAction: GoToDialog

      fallbackDialogOnInvalidEntity: mosaiic_AIRCOTelefonBot.topic.EndofConversation

    - kind: Question
      id: question_H0ONs7
      interruptionPolicy:
        allowInterruption: false

      variable: init:Global.Ansprechpartner
      prompt:
        text:
          - Wie ist Ihr Name?
        speak:
          - Wie ist Ihr Name?
        allowBargeIn: false

      entity: StringPrebuiltEntity
      voiceInputSettings:
        fallbackDialogOnSilence: mosaiic_AIRCOTelefonBot.topic.EndofConversation
        defaultValueMissingAction: GoToDialog

      fallbackDialogOnInvalidEntity: mosaiic_AIRCOTelefonBot.topic.EndofConversation

    - kind: Question
      id: 9CvROV
      interruptionPolicy:
        allowInterruption: false

      variable: Global.Telefonnummer
      prompt:
        text:
          - Unter welcher Nummer können wir Sie erreichen?
        speak:
          - Wie ist Ihre Telefonnummer?
        allowBargeIn: false

      entity: StringPrebuiltEntity
      voiceInputSettings:
        fallbackDialogOnSilence: mosaiic_AIRCOTelefonBot.topic.EndofConversation
        defaultValueMissingAction: GoToDialog

      fallbackDialogOnInvalidEntity: mosaiic_AIRCOTelefonBot.topic.EndofConversation

    - kind: SetVariable
      id: 2X2Lbc
      variable: Global.KanalLabel
      value: Telefon

    - kind: InvokeFlowAction
      id: invokeFlowAction_nOBDGN
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
      id: lzTpPJ
      dialog: mosaiic_AIRCOTelefonBot.topic.Anrufgrunderfassen

inputType: {}
outputType: {}