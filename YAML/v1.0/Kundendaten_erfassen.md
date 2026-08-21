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

      entity:
        kind: StringPrebuiltEntity
        sensitivityLevel: None

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

      entity: StringPrebuiltEntity

    - kind: ConditionGroup
      id: conditionGroup_ebCmRL
      conditions:
        - id: conditionItem_HqRFLM
          condition: =!IsBlank(System.Activity.From.Name)
          displayName: Wie ist Ihr Name?
          actions:
            - kind: Question
              id: question_XtVNZK
              interruptionPolicy:
                allowInterruption: true

              variable: init:Topic.richtigeTelefonnumer
              prompt:
                text:
                  - Ich sehe, Ihre Telefonnummer lautet {System.Activity.From.Name}. Ist das die richtige Rückrufnummer für Sie?
                speak:
                  - Ich sehe, Sie rufen von der Nummer {System.Activity.From.Name} an. Ist das die richtige Rückrufnummer für Sie?

              entity: BooleanPrebuiltEntity

            - kind: ConditionGroup
              id: conditionGroup_7ywK32
              conditions:
                - id: conditionItem_3qHgH9
                  condition: =Topic.richtigeTelefonnumer = true
                  displayName: Bedingung Wahr
                  actions:
                    - kind: SetVariable
                      id: setVariable_1WwEJg
                      variable: Global.Telefonnummer
                      value: =System.Activity.From.Name

              elseActions:
                - kind: Question
                  id: question_DOF847
                  interruptionPolicy:
                    allowInterruption: true

                  variable: Global.Telefonnummer
                  prompt:
                    text:
                      - Wie lautet die Telefonnummer, unter der wir Sie erreichen können?
                    speak:
                      - Wie lautet die Telefonnummer, unter der wir Sie erreichen können?

                  entity:
                    kind: StringPrebuiltEntity
                    sensitivityLevel: None

            - kind: SetVariable
              id: setVariable_dT0Z8k
              variable: Global.KanalLabel
              value: Telefon

      elseActions:
        - kind: Question
          id: question_yLatcn
          interruptionPolicy:
            allowInterruption: true

          variable: Global.Telefonnummer
          prompt:
            text:
              - Unter welcher Nummer können wir Sie erreichen?
            speak:
              - Unter welcher Nummer können wir Sie erreichen?

          entity:
            kind: StringPrebuiltEntity
            sensitivityLevel: None

        - kind: SetVariable
          id: setVariable_tuk9d9
          variable: Global.KanalLabel
          value: Chat

    - kind: BeginDialog
      id: lzTpPJ
      dialog: mosaiic_AIRCOTelefonBot.topic.Inbetriebnahme

inputType: {}
outputType: {}