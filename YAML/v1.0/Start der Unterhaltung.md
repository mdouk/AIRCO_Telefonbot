kind: AdaptiveDialog
beginDialog:
  kind: OnConversationStart
  id: main
  actions:
    - kind: SendActivity
      id: sendMessage_M0LuhV
      activity:
        text:
          - Willkommen beim technischen Service der Firma AIRCO Systems GmbH. Sie sprechen mit unserem digitalen Service-Assistenten. Zur Bearbeitung Ihres Anliegens wird dieses Gespräch transkribiert und verarbeitet.
        speak:
          - Willkommen beim technischen Service der Firma AIRCO Systems GmbH. Sie sprechen mit unserem digitalen Service-Assistenten. Zur Bearbeitung Ihres Anliegens wird dieses Gespräch transkribiert und verarbeitet.

    - kind: BeginDialog
      id: y3pHxU
      dialog: mosaiic_AIRCOTelefonBot.topic.Kundendatenerfassen