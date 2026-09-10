kind: AdaptiveDialog
modelDescription: Erkennt, ob es sich um eine kritische Produktionsstörung mit potenziellem Produktionsstillstand handelt. Produktionsstillstände sofort herausfiltern, bevor weitere Daten erfragt werden - kritische Anrufe dürfen nicht in der Freitext-Erfassung warten.
beginDialog:
  kind: OnRecognizedIntent
  id: main
  intent: {}
  actions:
    - kind: Question
      id: question_NazfRB
      interruptionPolicy:
        allowInterruption: true

      variable: init:Global.Produktionsstillstand
      prompt:
        text:
          - Steht Ihre Produktion aktuell still?
        speak:
          - Steht Ihre Produktion aktuell still?

      entity: BooleanPrebuiltEntity

    - kind: ConditionGroup
      id: conditionGroup_T8hE0z
      conditions:
        - id: conditionItem_T4R5Xs
          condition: =Global.Produktionsstillstand = true
          displayName: Bedingung wahr
          actions:
            - kind: SetVariable
              id: setVariable_7pGzbd
              variable: Topic.AktuelleZeit
              value: =Now()

            - kind: SetVariable
              id: setVariable_JQWBP7
              variable: Topic.Geschaeftszeit
              value: =Weekday(Topic.AktuelleZeit, StartOfWeek.Monday) <= 5 &&   Hour(Topic.AktuelleZeit) >= 8 && Hour(Topic.AktuelleZeit) < 17

            - kind: ConditionGroup
              id: conditionGroup_zQwBLU
              conditions:
                - id: conditionItem_DtVOWQ
                  condition: =Topic.Geschaeftszeit = true
                  displayName: Geschäftszeiten Bedingung wahr
                  actions:
                    - kind: SendActivity
                      id: sendActivity_K6SQdq
                      activity:
                        text:
                          - Aufgrund des Produktionsstillstands verbinde ich Sie sofort mit einem Mitarbeiter.
                        speak:
                          - Aufgrund des Produktionsstillstands verbinde ich Sie sofort mit einem Mitarbeiter.

                    - kind: InvokeFlowAction
                      id: invokeFlowAction_zPPm8f
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

                    - kind: TransferConversationV2
                      id: transferConversationV2_ZP1Zo6
                      transferType:
                        kind: TransferToPhoneNumber
                        phoneNumber: +4916090179359

                    - kind: BeginDialog
                      id: ctSr9S
                      dialog: cr0d0_sdhfw.topic.EndofConversation

              elseActions:
                - kind: SendActivity
                  id: sendActivity_Ly4TXd
                  activity:
                    text:
                      - Sie rufen außerhalb unserer Geschäftszeiten (Mo–Fr 8–17 Uhr) an. Bitte schildern Sie trotzdem kurz Ihr Anliegen und wir melden uns schnellstmöglich bei Ihnen.
                    speak:
                      - Sie rufen außerhalb unserer Geschäftszeiten (Mo–Fr 8–17 Uhr) an. Bitte schildern Sie trotzdem kurz Ihr Anliegen und wir melden uns schnellstmöglich bei Ihnen.

                - kind: BeginDialog
                  id: Nc9gWS
                  dialog: cr0d0_sdhfw.topic.Anliegenerfassen

      elseActions:
        - kind: BeginDialog
          id: lDqfL6
          dialog: cr0d0_sdhfw.topic.Anliegenerfassen

inputType: {}
outputType: {}