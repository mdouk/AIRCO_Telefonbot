kind: AdaptiveDialog
beginDialog:
  kind: OnConversationStart
  id: main
  actions:
    - kind: SendActivity
      id: sendMessage_M0LuhV
      activity:
        text:
          - Willkommen beim technischen Service der Firma AIRCO Systems GmbH. Ich bin der digitale Service-Assistent. Zur Bearbeitung Ihres Anliegens wird dieses Gespräch mittels KI verarbeitet.
        speak:
          - Willkommen beim technischen Service der Firma <sub alias="Airko">AIRCO</sub> Systems GmbH. Sie sprechen mit unserem digitalen Service-Assistenten. Zur Bearbeitung Ihres Anliegens wird dieses Gespräch transkribiert und verarbeitet.
        allowBargeIn: false

    - kind: BeginDialog
      id: y3pHxU
      dialog: mosaiic_AIRCOTelefonBot.topic.Kundendatenerfassen