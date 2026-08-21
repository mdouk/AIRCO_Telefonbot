kind: AdaptiveDialog
beginDialog:
  kind: OnRedirect
  id: main
  actions:
    - kind: SetVariable
      id: setVariable_IgcdCf
      variable: Topic.Korrekturversuche
      value: =0

    - kind: Question
      id: question_Qt7GUF
      interruptionPolicy:
        allowInterruption: true

      variable: init:Topic.korrekt
      prompt:
        text:
          - |-
            Ich fasse Ihre Angaben zusammen:
            - Firma: {Global.Firmenname}
            - Ansprechpartner: {Global.Ansprechpartner}
            - Telefonnummer: {Global.Telefonnummer}

            Ist das so korrekt?
        speak:
          - Ich fasse Ihre Angaben zusammen. Sie rufen für die Firma {Global.Firmenname} an. Ihr Name lautet {Global.Ansprechpartner} und Sie sind unter {Global.Telefonnummer} erreichbar. Ist das so korrekt?

      entity: BooleanPrebuiltEntity

    - kind: ConditionGroup
      id: conditionGroup_lwSRiq
      conditions:
        - id: conditionItem_3i56ds
          condition: =Topic.korrekt = true
          actions:
            - kind: SendActivity
              id: sendActivity_GiyFXt
              activity:
                text:
                  - Vielen Dank, ich habe Ihr Anliegen weitergeleitet. Ein Mitarbeiter wird sich zeitnah bei Ihnen melden. Auf Wiederhören.
                speak:
                  - Vielen Dank, ich habe Ihr Anliegen weitergeleitet. Ein Mitarbeiter wird sich zeitnah bei Ihnen melden. Auf Wiederhören.

            - kind: InvokeFlowAction
              id: invokeFlowAction_O1HpfY
              input:
                binding:
                  text: =Global.Firmenname
                  text_1: =Global.Ansprechpartner
                  text_2: =Global.Telefonnummer
                  text_3: =Global.Anliegen
                  text_4: =Global.KanalLabel
                  text_5: =Global.Anlage

              output: {}
              flowId: 019885f0-e29a-f111-b8db-7ced8d476627

            - kind: BeginDialog
              id: PWDKsD
              dialog: mosaiic_AIRCOTelefonBot.topic.EndofConversation

      elseActions:
        - kind: SetVariable
          id: setVariable_DQIEeM
          variable: Topic.Korrekturversuche
          value: =Topic.Korrekturversuche +1

        - kind: ConditionGroup
          id: conditionGroup_73xIxM
          conditions:
            - id: conditionItem_Ewz9wH
              condition: =Topic.Korrekturversuche <= 3
              displayName: Versuche =< 3
              actions:
                - kind: Question
                  id: question_ODxM04
                  interruptionPolicy:
                    allowInterruption: true

                  variable: init:Topic.Korrekturfeld
                  prompt:
                    text:
                      - Welche Angabe war nicht korrekt? Der Firmenname, der Ansprechpartnername oder die Telefonnummer?
                    speak:
                      - Welche Angabe war nicht korrekt? Der Firmenname, der Ansprechpartnername oder die Telefonnummer?

                  entity:
                    kind: ClosedListEntityReference
                    entityId: mosaiic_AIRCOTelefonBot.entity.Korrekturfeld

                - kind: ConditionGroup
                  id: conditionGroup_7MJ9bT
                  conditions:
                    - id: conditionItem_kLZWTj
                      condition: =Topic.Korrekturfeld = 'mosaiic_AIRCOTelefonBot.entity.Korrekturfeld'.KJwQ9x
                      actions:
                        - kind: SetVariable
                          id: setVariable_EmKb6d
                          variable: Global.Firmenname
                          value: =Blank()

                        - kind: Question
                          id: question_UrXver
                          interruptionPolicy:
                            allowInterruption: true

                          variable: Global.Firmenname
                          prompt:
                            text:
                              - Wie lautet der richtige Firmenname?
                            speak:
                              - Wie lautet der richtige Firmenname?

                          entity:
                            kind: StringPrebuiltEntity
                            sensitivityLevel: None

                    - id: conditionItem_33ZPed
                      condition: =Topic.Korrekturfeld = 'mosaiic_AIRCOTelefonBot.entity.Korrekturfeld'.md1vGC
                      actions:
                        - kind: SetVariable
                          id: setVariable_2nLyQe
                          variable: Global.Ansprechpartner
                          value: =Blank()

                        - kind: Question
                          id: question_sB1Ck1
                          interruptionPolicy:
                            allowInterruption: true

                          variable: Global.Ansprechpartner
                          prompt:
                            text:
                              - Wie ist Ihr richtiger Name?
                            speak:
                              - Wie ist Ihr richtiger Name?

                          entity:
                            kind: StringPrebuiltEntity
                            sensitivityLevel: None

                    - id: conditionItem_nV5Ax7
                      condition: =Topic.Korrekturfeld = 'mosaiic_AIRCOTelefonBot.entity.Korrekturfeld'.swLyyJ
                      actions:
                        - kind: SetVariable
                          id: setVariable_j62k3e
                          variable: Global.Telefonnummer
                          value: =Blank()

                        - kind: Question
                          id: question_Q8Yy2q
                          interruptionPolicy:
                            allowInterruption: true

                          variable: Global.Telefonnummer
                          prompt:
                            text:
                              - Wie ist Ihre richtige Telefonnummer?
                            speak:
                              - Wie ist Ihre richtige Telefonnummer?

                          entity:
                            kind: StringPrebuiltEntity
                            sensitivityLevel: None
                            elseActions:

                  elseActions:
                    - kind: SendActivity
                      id: sendActivity_zYNVAs
                      activity:
                        text:
                          - Das habe ich leider nicht verstanden.
                        speak:
                          - Das habe ich leider nicht verstanden.

                    - kind: GotoAction
                      id: rlmkpf
                      actionId: question_ODxM04

                - kind: GotoAction
                  id: SLfuCH
                  actionId: question_Qt7GUF

          elseActions:
            - kind: SendActivity
              id: sendActivity_5wGk52
              activity:
                text:
                  - Ihre Angaben konnten leider nicht abschließend bestätigt werden. Ich habe Ihr Anliegen weitergeleitet. Ein Mitarbeiter wird sich zeitnah bei Ihnen melden. Auf Wiederhören!
                speak:
                  - Ihre Angaben konnten leider nicht abschließend bestätigt werden. Ich habe Ihr Anliegen weitergeleitet. Ein Mitarbeiter wird sich zeitnah bei Ihnen melden. Auf Wiederhören!

            - kind: InvokeFlowAction
              id: invokeFlowAction_71Q8ZX
              input:
                binding:
                  text: =Global.Firmenname
                  text_1: =Global.Ansprechpartner
                  text_2: =Global.Telefonnummer
                  text_3: =Global.Anliegen
                  text_4: =Global.KanalLabel
                  text_5: =Global.Anlage

              output: {}
              flowId: 019885f0-e29a-f111-b8db-7ced8d476627

            - kind: BeginDialog
              id: Cs2X1Y
              dialog: mosaiic_AIRCOTelefonBot.topic.EndofConversation

inputType: {}
outputType: {}