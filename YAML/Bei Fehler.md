kind: AdaptiveDialog
startBehavior: UseLatestPublishedContentAndCancelOtherTopics
beginDialog:
  kind: OnError
  id: main
  actions:
    - kind: SetVariable
      id: setVariable_timestamp
      variable: init:Topic.CurrentTime
      value: =Text(Now(), DateTimeFormat.UTC)

    - kind: ConditionGroup
      id: condition_1
      conditions:
        - id: bL4wmY
          condition: =System.Conversation.InTestMode = true
          actions:
            - kind: SendActivity
              id: sendMessage_XJBYMo
              activity:
                text:
                  - |-
                    Fehlermeldung: {System.Error.Message}
                    Fehlercode: {System.Error.Code}
                    Unterhaltungs-ID: {System.Conversation.Id}
                    Zeit (UTC): {Topic.CurrentTime}
                speak:
                  - Es tut uns leid, es ist ein technischer Fehler aufgetreten. Bitte versuchen Sie es später erneut. Auf Wiederhören.

      elseActions:
        - kind: SendActivity
          id: sendMessage_dZ0gaF
          activity:
            text:
              - |-
                Ein Fehler ist aufgetreten.
                Fehlercode: {System.Error.Code}
                Unterhaltungs-ID: {System.Conversation.Id}
                Zeit (UTC): {Topic.CurrentTime}.
            speak:
              - Es tut uns leid, es ist ein technischer Fehler aufgetreten. Ihr Anliegen wurde gespeichert und ein Mitarbeiter wird sich bei Ihnen melden. Auf Wiederhören.

    - kind: LogCustomTelemetryEvent
      id: 9KwEAn
      eventName: OnErrorLog
      properties: "={ErrorMessage: System.Error.Message, ErrorCode: System.Error.Code, TimeUTC: Topic.CurrentTime, ConversationId: System.Conversation.Id}"

    - kind: SetVariable
      id: resetFlowFlag
      variable: Global.FlowAufgerufen
      value: =false

    - kind: BeginDialog
      id: NW7NyY
      dialog: mosaiic_AIRCOTelefonBot.topic.EndofConversation