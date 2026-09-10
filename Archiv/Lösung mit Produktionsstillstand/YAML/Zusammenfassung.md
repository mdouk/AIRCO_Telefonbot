kind: AdaptiveDialog
modelDescription: Gemeinsamer Schritt direkt vor dem Action Node. Liest die gesammelten Werte vor und lässt den Anrufer bestätigen oder korrigieren.
beginDialog:
  kind: OnRecognizedIntent
  id: main
  intent: {}
  actions:
    - kind: SetVariable
      id: setVariable_BaHx7i
      variable: Topic.Korrekturversuche
      value: =0

    - kind: ConditionGroup
      id: conditionGroup_AYYXWr
      conditions:
        - id: conditionItem_A1esZb
          condition: =Global.Inbetriebnahme = true && Global.VertragVorhanden = true
          actions:
            - kind: SendActivity
              id: sendActivity_yvsm7h
              activity:
                text:
                  - |-
                    Ich fasse Ihre Angaben zusammen: 
                    - Firma: {Global.Firmenname}
                    - Ansprechpartner: {Global.Ansprechpartner} 
                    - erreichbar unter: {Global.Telefonnummer}.
                    - Die Inbetriebnahme erfolgte in den letzten 12 Monaten und ein gültiger Wartungsvertrag liegt vor. 
                    - Ihr Anliegen: {Global.Anliegen}.
                speak:
                  - "Ich fasse Ihre Angaben zusammen: Firma {Global.Firmenname}, Ansprechpartner {Global.Ansprechpartner}, erreichbar unter {Global.Telefonnummer}. Die Inbetriebnahme erfolgte in den letzten 12 Monaten und ein gültiger Wartungsvertrag liegt vor. Ihr Anliegen: {Global.Anliegen}."

        - id: conditionItem_BruZdC
          condition: =Global.Inbetriebnahme = true && Global.VertragVorhanden = false
          actions:
            - kind: SendActivity
              id: irHxXz
              activity:
                text:
                  - |-
                    Ich fasse Ihre Angaben zusammen:
                    - Firma: {Global.Firmenname}
                    - Ansprechpartner: {Global.Ansprechpartner}
                    - erreichbar unter: {Global.Telefonnummer}.
                    - Die Inbetriebnahme erfolgte in den letzten 12 Monaten, ein Wartungsvertrag liegt jedoch nicht vor.
                    - Ihr Anliegen: {Global.Anliegen}.
                speak:
                  - "Ich fasse Ihre Angaben zusammen: Firma {Global.Firmenname}, Ansprechpartner {Global.Ansprechpartner}, erreichbar unter {Global.Telefonnummer}. Die Inbetriebnahme erfolgte in den letzten 12 Monaten, ein Wartungsvertrag liegt jedoch nicht vor. Ihr Anliegen: {Global.Anliegen}."

        - id: conditionItem_adv1Et
          condition: =Global.Inbetriebnahme = false && Global.VertragVorhanden = true
          actions:
            - kind: SendActivity
              id: 2AtQl7
              activity:
                text:
                  - |-
                    Ich fasse Ihre Angaben zusammen:
                    - Firma: {Global.Firmenname}
                    - Ansprechpartner: {Global.Ansprechpartner}
                    - erreichbar unter: {Global.Telefonnummer}.
                    - Die Inbetriebnahme liegt länger als 12 Monate zurück, ein gültiger Wartungsvertrag liegt vor.
                    - Ihr Anliegen: {Global.Anliegen}.
                speak:
                  - "Ich fasse Ihre Angaben zusammen: Firma {Global.Firmenname}, Ansprechpartner {Global.Ansprechpartner}, erreichbar unter {Global.Telefonnummer}. Die Inbetriebnahme liegt länger als 12 Monate zurück, ein gültiger Wartungsvertrag liegt vor. Ihr Anliegen: {Global.Anliegen}."

      elseActions:
        - kind: SendActivity
          id: mNf11f
          activity:
            text:
              - |-
                Ich fasse Ihre Angaben zusammen:
                - Firma: {Global.Firmenname}
                - Ansprechpartner: {Global.Ansprechpartner}
                - erreichbar unter: {Global.Telefonnummer}.
                - Die Inbetriebnahme liegt länger als 12 Monate zurück und ein Wartungsvertrag liegt nicht vor.
                - Ihr Anliegen: {Global.Anliegen}.
            speak:
              - "Ich fasse Ihre Angaben zusammen: Firma {Global.Firmenname}, Ansprechpartner {Global.Ansprechpartner}, erreichbar unter {Global.Telefonnummer}. Die Inbetriebnahme liegt länger als 12 Monate zurück und ein Wartungsvertrag liegt nicht vor. Ihr Anliegen: {Global.Anliegen}."

    - kind: Question
      id: question_2sag0a
      interruptionPolicy:
        allowInterruption: true

      variable: init:Topic.Bestaetigt
      prompt:
        text:
          - Ist das so korrekt?
        speak:
          - Ist das so korrekt?

      entity: BooleanPrebuiltEntity

    - kind: ConditionGroup
      id: conditionGroup_6Zuz0V
      conditions:
        - id: conditionItem_W20Ovj
          condition: =Topic.Bestaetigt = true
          displayName: Bedingung wahr

      elseActions:
        - kind: SetVariable
          id: setVariable_sGZhQP
          variable: Topic.Korrekturversuche
          value: =Topic.Korrekturversuche +1

        - kind: ConditionGroup
          id: conditionGroup_J3XrZ6
          conditions:
            - id: conditionItem_GiRCig
              condition: =Topic.Korrekturversuche <= 2
              displayName: Versuche =< 2
              actions:
                - kind: Question
                  id: question_cVAf0h
                  interruptionPolicy:
                    allowInterruption: true

                  variable: init:Topic.Korrekturfeld2
                  prompt:
                    text:
                      - "Welche Angabe war nicht korrekt: Firmenname, Ansprechpartner, Telefonnummer, Ihr Anliegen, die Angabe zur Inbetriebnahme oder zum Wartungsvertrag?"
                    speak:
                      - "Welche Angabe war nicht korrekt: Firmenname, Ansprechpartner, Telefonnummer, Ihr Anliegen, die Angabe zur Inbetriebnahme oder zum Wartungsvertrag?"

                  entity:
                    kind: ClosedListEntityReference
                    entityId: cr0d0_sdhfw.entity.Korrekturfeld2

                - kind: ConditionGroup
                  id: conditionGroup_CLQt9z
                  conditions:
                    - id: conditionItem_SOr90x
                      condition: =Topic.Korrekturfeld2 = 'cr0d0_sdhfw.entity.Korrekturfeld2'.aek9zI
                      actions:
                        - kind: Question
                          id: question_zXPl5y
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

                    - id: conditionItem_XogWuC
                      condition: =Topic.Korrekturfeld2 = 'cr0d0_sdhfw.entity.Korrekturfeld2'.r7NfRA
                      actions:
                        - kind: Question
                          id: question_jcimdr
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

                    - id: conditionItem_YLZnAS
                      condition: =Topic.Korrekturfeld2 = 'cr0d0_sdhfw.entity.Korrekturfeld2'.xOWdUB
                      actions:
                        - kind: Question
                          id: question_VGggey
                          interruptionPolicy:
                            allowInterruption: true

                          variable: Global.Telefonnummer
                          prompt:
                            text:
                              - Wie lautet die richtige Telefonnummer?
                            speak:
                              - Wie lautet die richtige Telefonnummer?

                          entity:
                            kind: StringPrebuiltEntity
                            sensitivityLevel: None

                    - id: conditionItem_Sdj5Sd
                      condition: =Topic.Korrekturfeld2 = 'cr0d0_sdhfw.entity.Korrekturfeld2'.iRVsqE
                      actions:
                        - kind: Question
                          id: question_y2xJsG
                          interruptionPolicy:
                            allowInterruption: true

                          variable: Global.Anliegen
                          prompt:
                            text:
                              - Wie würden Sie Ihr Anliegen richtig beschreiben?
                            speak:
                              - Wie würden Sie Ihr Anliegen richtig beschreiben?

                          entity:
                            kind: StringPrebuiltEntity
                            sensitivityLevel: None

                    - id: conditionItem_rMpi9T
                      condition: =Topic.Korrekturfeld2 = 'cr0d0_sdhfw.entity.Korrekturfeld2'.jNs2gh
                      actions:
                        - kind: SetVariable
                          id: setVariable_iidhbP
                          variable: Global.Inbetriebnahme
                          value: =Not(Global.Inbetriebnahme)

                    - id: conditionItem_FerqI1
                      condition: =Topic.Korrekturfeld2 = 'cr0d0_sdhfw.entity.Korrekturfeld2'.xQ0lDU
                      actions:
                        - kind: SetVariable
                          id: setVariable_eYtJjI
                          variable: Global.VertragVorhanden
                          value: =Not(Global.VertragVorhanden)

                - kind: SendActivity
                  id: sendActivity_949bzQ
                  activity:
                    text:
                      - |-
                        Ich fasse Ihre Angaben zusammen:
                        - Firma: {Global.Firmenname}
                        - Ansprechpartner: {Global.Ansprechpartner}
                        - erreichbar unter: {Global.Telefonnummer}
                        - Inbetriebnahme in den letzten 12 Monaten? {If(Global.Inbetriebnahme, "ja", "nein")}
                        - Wartungsvertrag vorhanden? {If(Global.VertragVorhanden, "ja", "nein")}
                        - Ihr Anliegen: {Global.Anliegen}
                    speak:
                      - "Ich fasse Ihre Angaben zusammen: Firma {Global.Firmenname}, Ansprechpartner {Global.Ansprechpartner}, erreichbar unter {Global.Telefonnummer}. Inbetriebnahme in den letzten 12 Monaten? {If(Global.Inbetriebnahme, \"ja\", \"nein\")}. Wartungsvertrag vorhanden? {If(Global.VertragVorhanden, \"ja\", \"nein\")}. Ihr Anliegen: {Global.Anliegen}"

                - kind: GotoAction
                  id: 6dTSPX
                  actionId: question_2sag0a

          elseActions:
            - kind: ConditionGroup
              id: conditionGroup_i5tKQY
              conditions:
                - id: conditionItem_jRaugp
                  condition: =Weekday(Now(), StartOfWeek.Monday) <= 5 && Hour(Now()) >= 8 && Hour(Now()) < 17
                  actions:
                    - kind: InvokeFlowAction
                      id: invokeFlowAction_rpWHlB
                      input:
                        binding:
                          boolean: =Global.VertragVorhanden
                          boolean_1: =Global.Produktionsstillstand
                          boolean_2: =Global.Inbetriebnahme
                          text: =Global.Firmenname
                          text_1: =Global.Ansprechpartner
                          text_2: =Global.Telefonnummer
                          text_3: =Global.Anliegen
                          text_4: =Global.KanalLabel

                      output: {}
                      flowId: 299a805a-9036-df8d-ed1d-ec8dbc0fcd99

                    - kind: SendActivity
                      id: sendActivity_QMQGLy
                      activity:
                        text:
                          - Ihre Angaben konnten leider nicht abschließend bestätigt werden. Ich verbinde Sie jetzt mit einem Mitarbeiter, der Ihnen weiterhilft.
                        speak:
                          - Ihre Angaben konnten leider nicht abschließend bestätigt werden. Ich verbinde Sie jetzt mit einem Mitarbeiter, der Ihnen weiterhilft.

                    - kind: TransferConversationV2
                      id: transferConversationV2_NhIOpC
                      transferType:
                        kind: TransferToPhoneNumber
                        phoneNumber: +4916090179359

              elseActions:
                - kind: InvokeFlowAction
                  id: invokeFlowAction_iEcaJK
                  input:
                    binding:
                      boolean: =Global.VertragVorhanden
                      boolean_1: =Global.Produktionsstillstand
                      boolean_2: =Global.Inbetriebnahme
                      text: =Global.Firmenname
                      text_1: =Global.Ansprechpartner
                      text_2: =Global.Telefonnummer
                      text_3: =Global.Anliegen
                      text_4: =Global.KanalLabel

                  output: {}
                  flowId: 299a805a-9036-df8d-ed1d-ec8dbc0fcd99

                - kind: SendActivity
                  id: sendActivity_AUu37E
                  activity:
                    text:
                      - Sie kontaktieren uns außerhalb der Geschäftszeiten. Ich habe Ihr Anliegen weitergeleitet. Ein Mitarbeiter wird sich zeitnah bei Ihnen melden. Auf Wiederhören!
                    speak:
                      - Sie rufen außerhalb der Geschäftszeiten. Ich habe Ihr Anliegen weitergeleitet. Ein Mitarbeiter wird sich zeitnah bei Ihnen melden. Auf Wiederhören!

                - kind: BeginDialog
                  id: ubu1M7
                  dialog: cr0d0_sdhfw.topic.EndofConversation

    - kind: InvokeFlowAction
      id: invokeFlowAction_n5Rq0K
      input:
        binding:
          boolean: =Global.VertragVorhanden
          boolean_1: =Global.Produktionsstillstand
          boolean_2: =Global.Inbetriebnahme
          text: =Global.Firmenname
          text_1: =Global.Ansprechpartner
          text_2: =Global.Telefonnummer
          text_3: =Global.Anliegen
          text_4: =Global.KanalLabel

      output: {}
      flowId: 299a805a-9036-df8d-ed1d-ec8dbc0fcd99

    - kind: SendActivity
      id: sendActivity_OWo9mN
      activity:
        text:
          - Vielen Dank, ich habe Ihr Anliegen weitergeleitet und ein Ticket eröffnet. Ein Mitarbeiter wird sich zeitnah bei Ihnen melden. Auf Wiederhören.
        speak:
          - Vielen Dank, ich habe Ihr Anliegen weitergeleitet und ein Ticket eröffnet. Ein Mitarbeiter wird sich zeitnah bei Ihnen melden. Auf Wiederhören.

    - kind: BeginDialog
      id: Wbgc8F
      dialog: cr0d0_sdhfw.topic.EndofConversation

inputType: {}
outputType: {}