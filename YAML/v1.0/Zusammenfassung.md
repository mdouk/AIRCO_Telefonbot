kind: AdaptiveDialog
beginDialog:
  kind: OnRedirect
  id: main
  actions:
    - kind: SetVariable
      id: setVariable_IgcdCf
      variable: Topic.Korrekturversuche
      value: =0

    - kind: ConditionGroup
      id: conditionGroup_ZJ5vX7
      conditions:
        - id: conditionItem_WblSrn
          condition: =Global.Inbetriebnahme = true && Global.VertragVorhanden = true
          actions:
            - kind: SendActivity
              id: sendActivity_CGuX1G
              activity:
                text:
                  - |-
                    Ich fasse Ihre Angaben zusammen:
                    - Firma: {Global.Firmenname}
                    - Ansprechpartner: {Global.Ansprechpartner}
                    - erreichbar unter: {Global.Telefonnummer}
                    - Die Inbetriebnahme erfolgte in den letzten 12 Monaten und ein gültiger Wartungsvertrag liegt vor.
                speak:
                  - "Ich fasse Ihre Angaben zusammen: Firma {Global.Firmenname}, Ansprechpartner {Global.Ansprechpartner}, erreichbar unter {Global.Telefonnummer}. Die Inbetriebnahme erfolgte in den letzten 12 Monaten und ein gültiger Wartungsvertrag liegt vor."

        - id: conditionItem_YJKEsX
          condition: =Global.Inbetriebnahme = true && Global.VertragVorhanden = false
          actions:
            - kind: SendActivity
              id: sendActivity_WpsA6l
              activity:
                text:
                  - |-
                    Ich fasse Ihre Angaben zusammen:
                    - Firma: {Global.Firmenname}
                    - Ansprechpartner: {Global.Ansprechpartner}
                    - erreichbar unter: {Global.Telefonnummer}
                    - Die Inbetriebnahme erfolgte in den letzten 12 Monaten, ein Wartungsvertrag liegt jedoch nicht vor.
                speak:
                  - "Ich fasse Ihre Angaben zusammen: Firma {Global.Firmenname}, Ansprechpartner {Global.Ansprechpartner}, erreichbar unter {Global.Telefonnummer}. Die Inbetriebnahme erfolgte in den letzten 12 Monaten, ein Wartungsvertrag liegt jedoch nicht vor."

        - id: conditionItem_lvyiWi
          condition: =Global.Inbetriebnahme = false && Global.VertragVorhanden = true
          actions:
            - kind: SendActivity
              id: sendActivity_xbsS7K
              activity:
                text:
                  - |-
                    Ich fasse Ihre Angaben zusammen:
                    - Firma: {Global.Firmenname}
                    - Ansprechpartner: {Global.Ansprechpartner}
                    - erreichbar unter: {Global.Telefonnummer}
                    - Die Inbetriebnahme liegt länger als 12 Monate zurück, ein gültiger Wartungsvertrag liegt vor.
                speak:
                  - "Ich fasse Ihre Angaben zusammen: Firma {Global.Firmenname}, Ansprechpartner {Global.Ansprechpartner}, erreichbar unter {Global.Telefonnummer}. Die Inbetriebnahme liegt länger als 12 Monate zurück, ein gültiger Wartungsvertrag liegt vor."

      elseActions:
        - kind: SendActivity
          id: sendActivity_v8BQUR
          activity:
            text:
              - |-
                Ich fasse Ihre Angaben zusammen:
                - Firma: {Global.Firmenname}
                - Ansprechpartner: {Global.Ansprechpartner}
                - erreichbar unter: {Global.Telefonnummer}
                - Die Inbetriebnahme liegt länger als 12 Monate zurück und ein Wartungsvertrag liegt nicht vor.
            speak:
              - "Ich fasse Ihre Angaben zusammen: Firma {Global.Firmenname}, Ansprechpartner {Global.Ansprechpartner}, erreichbar unter {Global.Telefonnummer}. Die Inbetriebnahme liegt länger als 12 Monate zurück und ein Wartungsvertrag liegt nicht vor."

    - kind: Question
      id: question_Qt7GUF
      interruptionPolicy:
        allowInterruption: true

      variable: init:Topic.korrekt
      prompt:
        text:
          - Ist das so korrekt?
        speak:
          - Ist das so korrekt?

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
              id: invokeFlowAction_ykYqsj
              input:
                binding:
                  boolean: =Global.VertragVorhanden
                  boolean_1: =Global.Inbetriebnahme
                  text: =Global.Firmenname
                  text_1: =Global.Ansprechpartner
                  text_2: =Global.Telefonnummer
                  text_3: =Global.Anliegen
                  text_4: =Global.KanalLabel

              output: {}
              flowId: 299a805a-9036-df8d-ed1d-ec8dbc0fcd99

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
              displayName: Versuche =< 2
              actions:
                - kind: Question
                  id: question_ODxM04
                  interruptionPolicy:
                    allowInterruption: true

                  variable: init:Topic.Korrekturfeld
                  prompt:
                    text:
                      - Welche Angabe war nicht korrekt? Firmenname, Ansprechpartner, Telefonnummer, die Angabe zur Inbetriebnahme oder zum Wartungsvertrag?
                    speak:
                      - Welche Angabe war nicht korrekt? Firmenname, Ansprechpartner, Telefonnummer, die Angabe zur Inbetriebnahme oder zum Wartungsvertrag?

                  entity:
                    kind: ClosedListEntityReference
                    entityId: mosaiic_AIRCOTelefonBot.entity.Korrekturfeld

                - kind: ConditionGroup
                  id: conditionGroup_7MJ9bT
                  conditions:
                    - id: conditionItem_HQGMfo
                      condition: =Topic.Korrekturfeld = 'mosaiic_AIRCOTelefonBot.entity.Korrekturfeld'.YvfEJQ
                      actions:
                        - kind: SetVariable
                          id: setVariable_naAopU
                          variable: Global.VertragVorhanden
                          value: =Not(Global.VertragVorhanden)

                    - id: conditionItem_Xmxluw
                      condition: =Topic.Korrekturfeld = 'mosaiic_AIRCOTelefonBot.entity.Korrekturfeld'.gjJ4ZJ
                      actions:
                        - kind: SetVariable
                          id: setVariable_65aVAp
                          variable: Global.Inbetriebnahme
                          value: =Not(Global.Inbetriebnahme)

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
                          - Das habe ich leider nicht verstanden. Sie können Firmenname, Ansprechpartner, Telefonnummer, Inbetriebnahme oder Wartungsvertrag eintippen. Ich fasse Ihre Angaben noch einmal zusammen.
                        speak:
                          - Das habe ich leider nicht verstanden. Sie können Firmenname, Ansprechpartner, Telefonnummer, die Angabe zur Inbetriebnahme oder zum Wartungsvertrag sagen. Ich fasse Ihre Angaben noch einmal zusammen.

                - kind: SendActivity
                  id: sendActivity_OMhM43
                  activity:
                    speak:
                      - "Ich fasse Ihre Angaben zusammen: Firma {Global.Firmenname}, Ansprechpartner {Global.Ansprechpartner}, erreichbar unter {Global.Telefonnummer}. Inbetriebnahme in den letzten 12 Monaten? {If(Global.Inbetriebnahme, \"Ja\", \"Nein\")}. Wartungsvertrag vorhanden? {If(Global.VertragVorhanden, \"Ja\", \"Nein\")}."

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
              id: invokeFlowAction_w4pP3b
              input:
                binding:
                  boolean: =Global.VertragVorhanden
                  boolean_1: =Global.Inbetriebnahme
                  text: =Global.Firmenname
                  text_1: =Global.Ansprechpartner
                  text_2: =Global.Telefonnummer
                  text_3: =Global.Anliegen
                  text_4: =Global.KanalLabel

              output: {}
              flowId: 299a805a-9036-df8d-ed1d-ec8dbc0fcd99

            - kind: BeginDialog
              id: Cs2X1Y
              dialog: mosaiic_AIRCOTelefonBot.topic.EndofConversation

inputType: {}
outputType: {}