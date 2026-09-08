kind: AdaptiveDialog
beginDialog:
  kind: OnRedirect
  id: main
  actions:
    - kind: Question
      id: question_anrufgrund
      interruptionPolicy:
        allowInterruption: false

      variable: init:Global.Anrufgrund
      prompt:
        text:
          - |-
            Was ist der Grund Ihres Anrufs?
            1 - Störung oder Problem
            2 - Wartungstermin vereinbaren
            3 - Ersatzteilbestellung
            4 - Rückrufbitte
            5 - Sonstiges
        speak:
          - "Was ist der Grund Ihres Anrufs? Sie können sagen: Störung oder Problem, Wartungstermin vereinbaren, Ersatzteilbestellung, Rückrufbitte, oder Sonstiges."
        allowBargeIn: false

      entity:
        kind: ClosedListEntityReference
        entityId: mosaiic_AIRCOTelefonBot.entity.Anrufgrund

      voiceInputSettings:
        fallbackDialogOnSilence: mosaiic_AIRCOTelefonBot.topic.EndofConversation
        defaultValueMissingAction: GoToDialog

      fallbackDialogOnInvalidEntity: mosaiic_AIRCOTelefonBot.topic.EndofConversation

    - kind: ConditionGroup
      id: conditionGroup_DklHkA
      conditions:
        - id: conditionItem_8IyZXO
          condition: =Global.Anrufgrund = 'mosaiic_AIRCOTelefonBot.entity.Anrufgrund'.u9xatS || Global.Anrufgrund = 'mosaiic_AIRCOTelefonBot.entity.Anrufgrund'.gShHbV
          displayName: Störung oder Wartung > Anlage fragen
          actions:
            - kind: InvokeFlowAction
              id: iKjeHG
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
              id: beginDialog_anlage
              dialog: mosaiic_AIRCOTelefonBot.topic.Anlagenerfassung

      elseActions:
        - kind: InvokeFlowAction
          id: O2iwrd
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
          id: beginDialog_anliegen_direkt
          dialog: mosaiic_AIRCOTelefonBot.topic.Anliegenerfassen

inputType: {}
outputType: {}