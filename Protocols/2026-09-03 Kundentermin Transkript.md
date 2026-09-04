# Kundentermin — Transkript
**Datum:** 2026-09-03  
**Teilnehmer:**
- **Constantin** (Sprecher 1) — Service-Innendienst
- **Jens** (Sprecher 2) — Geschäftsführung
- **Robin** (Sprecher 3) — 3rd Level
- **Michael** (Sprecher 4) — Entwickler (mosaiic)
- **Thomas** (Sprecher 5)

---

**Constantin:** Nach ich glaube fünf Sekunden sagt er dann von wegen es ist Fehler aufgetreten, obwohl er vorher gesagt hat, er schickt eine E-Mail raus, es wird sich jemand melden.

**Jens:** Wie ist es denn an sich? Ich habe jetzt mal vor längerer Zeit angerufen, jetzt die letzten Zeit nicht. Ich weiß nicht, ob was verstellt ist. Wie ist denn die Durchführung im Menü und für unsere Definition des Telefonbot 1? Ist das für euch Robin aus eurer operativen Erfahrung, Consti, ist das so in Ordnung oder hättet ihr insgesamt an die Art und Weise jetzt neben den Fehlern, die die eben besprochen ist, da muss sich der Michael ja noch mal in Ruhe drum kümmern, deswegen machen wir das ja. Ist der Ablauf so jetzt OK oder müssen wir da noch mal richtig ran?

**Robin:** Ich glaub für den Bot 1.0 ist der Ablauf soweit erstmal völlig fein. Das Hauptproblem sind gerade echt die ganzen Bugs, die noch drin sind. Dass wir die rausbekommen sollten. Was, finde ich, auch noch dazugehört für den Bot 1.0. Wir haben ja auch viele Kunden aus dem Ausland. Der Bot erkennt leider gar kein Englisch. Da haben wir das letzte Mal gesagt, das ist nicht möglich in Deutschland aktuell. Aber das kann halt ein Problem darstellen. Ich habe es heute Morgen auf Englisch versucht einmal, habe mich dann irgendwie typisch gekämpft, bis zuletzt, weil die meisten Wörter Deutsch-Englisch sollte ja auch ein englischsprachiger Mensch verstehen, was der Bot gerade von einem will, so ungefähr. Dann hatte ich es sogar auch geschafft bis am Ende, dass er sagt, eine E-Mail wird ausgelöst und die E-Mail kam dann auch nicht an, weil er gesagt hat, Fehler am Ende. Wie seht ihr das mit der Sprache, Jens, was meinst du?

**Jens:** Ja, das ist schon wichtig. Gebe ich dir vollkommen recht.

**Constantin:** Der Bot erkennt auch nicht, wenn man bei der Telefonnummer sagt +49 oder +41, dann lässt er das immer weg, das Plus.

**Robin:** Das ist aber nicht so wichtig. Das können wir uns nicht denken, dass es +41 ist.

**Constantin:** Wenn der Name vom Kunden jetzt richtig hinterlegt ist und wir wissen, er ist aus der Schweiz.

**Jens:** Wie ist denn die Spracherkennung, also dort, wo er dann eine E-Mail schickt, ist der Inhalt, den ihr reingesprochen habt, auch zu 100% abgebildet oder fehlen da Inhalte?

**Constantin:** Die Inhalte haben bisher immer gepasst. Es sind halt ein paar sprachliche Kleinigkeiten, wie jedes Mal, wenn wir Psal gesagt haben, also PSAL, dass er dann TZ geschrieben hat und nicht PS, aber da wissen wir eigentlich, was damit gemeint ist.

**Robin:** Also es wird grob erkannt. Eine ganz wichtige Funktion, die wir das letzte Mal besprochen haben, ist auch noch nicht umgesetzt. Wir haben ja gesagt, völlig egal, ob der Anrufer bis zum Ende kommt oder ob er es nur bis zum Namen schafft oder whatever, sollte ein Fehler auftauchen, es muss zwingend immer eine E-Mail ausgelöst werden. Der Anrufer weiß nicht, ob bei uns eine E-Mail ankommt. Er geht fest davon aus, er hat jetzt mit dem Bot gequatscht. Er hat mit dem vielleicht noch rumgezankert, aber er weiß, okay, ich habe jetzt irgendwas gemacht, ich werde schon einen Rückruf bekommen. Und da passieren mir viel zu viele Anrufe noch, wo es einen Fehler gibt, wo keine E-Mail ausgelöst wird.

**Michael:** Ich habe an zwei, drei Stellen das gemacht, aber nicht überall. Wenn zum Beispiel nach der Kundendatenerfassung ganz vorne ein Fehler passiert, dann sollte eine E-Mail geschickt werden. Aber danach, bei der Anlagebeschreibung nicht mehr, erst dann später bei der Anliegenbeschreibung. Ja, also ich kann überall.

**Constantin:** Angekommen ist bisher keine einzige Fehlermeldung, Fehler-E-Mail.

**Michael:** Wenn es direkt nach der Kundendatenerfassung, nach der Telefonnummererfassung ein Fehler passiert, dann soll es eine E-Mail. Ich hab das selber getestet. Das hat geklappt, aber nicht nach der Anlagenbeschreibung.

**Constantin:** Wo geht die E-Mail denn hin? Also im Service sind sie nicht gelandet.

**Michael:** Nee, wenn ich teste, ich mach das, ich entferne die Service-Atako-Adresse und nutze meine eigene, um euch nicht zu bombardieren mit E-Mails.

**Constantin:** Naja, aber wie gesagt, bei den Fehlermeldungen, die der Robin heute Morgen hatte, sind wir gestern — wo wir nicht weiterkamen, obwohl er gesagt hat, es geht eine E-Mail raus, ist ja dann auch ein Fehler aufgetreten, da kam nichts bei uns an.

**Michael:** Ja, nee, das sollte bei euch ankommen. Ich hab die E-Mail wieder am Donnerstag aufgenommen. Heißt, morgen und am Montag oder am Freitag sollte es auch bei euch ankommen.

**Jens:** Das hab ich jetzt nicht verstanden. Wo ist jetzt — aus den Telefonanrufen heute Morgen und gestern sind E-Mails rausgegangen, trotz Fehler, ja oder nein?

**Michael:** Hattet ihr heute Fälle gehabt? Ja, sagtest du, Robin oder?

**Constantin:** Ja.

**Michael:** Und Moment. Heute — also es sind drei Anrufe.

**Robin:** Drei E-Mails haben wir erhalten im Service.

**Michael:** Drei Anrufe sind durch. Ich sehe jetzt den Ausführungsverlauf der letzten 28 Tage. Heute sehe ich die drei erfolgreichen. Und man kann auch einen Drilldown machen.

**Robin:** Siehst du nicht fehlgeschlagene?

**Michael:** Nein, es gibt keine fehlgeschlagenen. Das ist das. Alle sind jetzt erfolgreich seit Mitte August.

**Robin:** Ja, das entspricht ja ein, zwei Minuten.

**Michael:** Wenn man den Anruf abbricht, manuell, wenn ich zum Beispiel die Kundendaten sage und dann auf „Anruf beenden" klicke, dann wird nichts transkribiert. Das ist nicht möglich. Es gibt keinen Trigger. Ich muss weitermachen. Entweder muss ein Netzwerkfehler passieren oder der Empfang muss schlecht sein oder so was, damit da eine E-Mail geschickt wird. Wenn ich manuell sage „Nee, beenden", weil der Erste, der die Kundendatenerfassung nicht geklappt hat, dann wird keine E-Mail geschickt.

**Jens:** Aber das hatten wir doch eben in der Erläuterung, Michael.

**Michael:** Ja, das ist technisch nicht möglich, weil der Agent weiß nicht—

**Jens:** Das ist etwas anderes, Michael. Eben haben Robin und Konstantin berichtet, die haben die letzten, gestern und heute, drei Abbrüche ganz am Ende, wo das System gesagt hat, ich habe einen Fehler. Und das ist nicht sichtbar.

**Michael:** Genau.

**Jens:** Das ist ein Bug.

**Michael:** Also Bug ist es nicht. Ich muss herausfinden, wo diese fehlgeschlagenen Anrufe landen, weil—

**Jens:** Er transkribiert ja nicht. Er schickt ja nicht eine E-Mail. Das, was er da alles aufgenommen hat, kommt zumindest mal nicht in eine Transkription und landet nicht in einem E-Mail-Postfach mit den Informationen.

**Robin:** Richtig. Und das ist ja wie gesagt das, was wir in unserem letzten Meeting auch schon besprochen haben, dass das essenziell ist, dass egal, was der Kunde sagt und eigentlich auch egal, wie weit er kommt, wir eine E-Mail brauchen. Wir brauchen die Info, da ist jemand da draußen, der hat ein Problem, der hat mit unserem Antwort gesprochen und erwartet Hilfe von uns.

**Michael:** Ja, also ich denke, das ist technisch nicht möglich. Ich muss das nochmal prüfen.

**Jens:** Was ist technisch nicht möglich?

**Michael:** Egal, wann ein Abbruch passiert, selbst wenn der Kunde nur den Namen gesagt hat, dass eine E-Mail geschickt wird, dass irgendein Trigger ausgelöst wird mit zwei Prozent der erforderlichen Angaben. Das ist der Punkt, weil im Normalfall wird am Ende acht Variablen gespeichert: Kundenname, Anliegen, Anlagen, Beschreibung und so weiter. Wenn diese acht Variablen nicht vollständig erfasst sind, es gibt Zwischenpunkte, wann E-Mails verschickt werden mit fehlenden Angaben, aber ganz ganz vorne, wenn der Bot sagt, wie ist Ihr Name? „Mein Name ist Michael." Und dann klickt er auf „Anruf beenden". Das ist nicht gespeichert. Oder wenn der Empfang schlecht ist oder wenn der Bot hängt, keine Ahnung. Dieser Zeitpunkt ist nicht erfasst. Und deswegen wird keine E-Mail geschickt. Ich muss deswegen—

**Robin:** Das wäre ja auch in Ordnung für uns, dass man den nicht erfasst, wenn der nur seinen Namen sagt. Dann ist er selbst dran schuld. Aber wir sind ja viel weiter fortgeschritten. Constantin und ich, wir waren ja wirklich, wir haben das Prozedere komplett durchgelaufen, mehrfach. Und es sind immer wieder Fehler aufgetaucht und es sind keine E-Mails entstanden. Oder wir haben, nachdem der Bot gesagt hat, es ist alles super, alles tutti, du kriegst eine E-Mail, wir melden uns, habe ich dann aufgelegt vor dem Bot, weil der Bot — ich brauche ja auch zehn Sekunden, bis der auflegt. Es wird kein Kunde warten. Der wird einfach auflegen, wenn der Bot ihm sagt, E-Mail geht raus. Warum soll er weiter warten? Er hat es ja schon durchgestanden, das Prozedere. Und dann kommt auch keine E-Mail.

**Constantin:** Also ich habe jetzt gerade mal geguckt. Ich hatte gestern um 7.22 Uhr angerufen, wo ich dann hängengeblieben bin, wo keine E-Mail kam.

**Michael:** Gestern um — Moment.

**Constantin:** 7.22 Uhr.

**Robin:** Und ich hatte heute sechs Anrufe und davon sind drei durchgekommen. Kann man das irgendwie nachvollziehen, wann die durch sind und wann nicht?

**Michael:** Ja, die sind natürlich alle Anrufe gespeichert.

**Robin:** Ja, genau. 7:19 Uhr heute, da ging drei Minuten, habe ich mit dem hin und her diskutiert.

**Constantin:** Es kam nichts.

**Robin:** Und wenn ich mich jetzt in einen Kunden hineinversetze, der drei Minuten mit einem Bot hin und her diskutiert, und dann kommt nicht mal eine E-Mail: „Boah, da wäre ich richtig sauer. Da wäre ich wirklich richtig sauer." Das kann so nicht sein.

**Michael:** Mhm. Ja, ja. Du sagtest heute um 7:22 Uhr kurz angerufen, oder?

**Robin:** Gestern.

**Michael:** Also gestern, ja genau, gestern 07:00 Uhr. Mhm, gut, ich—

**Jens:** Ich glaub, das ist jetzt für diese Runde, müssen wir jetzt nicht alle fünf die Analyse von Michael abwarten.

**Michael:** Nee, das ist wichtig, weil die Daten sind, die brauche ich, wann zum Beispiel ihr angerufen habt und wann, wie lange ihr gesprochen habt, weil die sind ja Erkenntnisse, die ich gebrauchen kann. Wir machen einen Deep-Dive.

**Robin:** Immer übers Handy, ja.

**Thomas:** Vielleicht, dass er dann Michael kurz mal so — man kann sich ja anzeigen lassen, was man angerufen hat. Ihm einen kurzen Screenshot schickt. Dass man diese Fehlerzuordnung hat, weil das Problem müssen wir lösen. Michael, kann man auf diese Telefonanlage mit diesem Easy irgendwas auch drauf zugreifen? Wir müssen eigentlich auch alle Telefonate aufschlagen, ne?

**Michael:** Ja.

**Thomas:** Müssen wir mit dem Oliver nochmal reden, was wir da—

**Jens:** Habt ihr die Zugangsdaten?

**Michael:** Wie bitte?

**Thomas:** Ich nicht.

**Jens:** Hast du die Zugänge zu der Telefonanlage und deine Telefonnummer?

**Michael:** Hab sie nicht, aber ich — wir müssen mal ganz prüfen in den Microsoft-Portalen, ob was—

**Thomas:** Ja, weil der Oliver ist nächste Woche zwei Wochen Urlaub. Das heißt, da müssen wir auf jeden Fall — dann können wir schauen, was da alles hängen bleibt.

**Jens:** Ja, lasst euch aber auch die Zugangsdaten bitte geben, dass ihr da unabhängig diese Probleme lösen könnt.

**Michael:** Noch eine Frage. Bei den fehlgeschlagenen Anrufen, gibt es irgendeinen Namen oder Firma oder keine Ahnung, was ich in den Logs suchen kann. Habt ihr einen bestimmten Firmennamen genutzt?

**Constantin:** Ramonta Ronald war der Name, den ich benutzt habe.

**Michael:** Okay, das habe ich notiert.

**Robin:** Hier war es einmal auf jeden Fall die Firma Prematik.

**Michael:** Prematik, okay. Rematik oder Prematik? Ich sehe hier—

**Robin:** Prematik.

**Michael:** Prematik. Und Vogt Sommer GmbH.

**Robin:** Ja, genau, der ist ja durchgekommen.

**Michael:** Er ist durchgekommen, okay.

**Robin:** Also jetzt muss ich noch mal gucken. Genau, die Vogt Sommer GmbH ist durchgekommen. Die Prematik ist einmal durchgekommen, da hatte ich aber vorher auch schon mal angerufen und die anderen — Schneider Gruppe ist auch durchgekommen.

**Thomas:** Was auch eine Frage wäre, Michael, ob du, dass du dir einrichten, dass du die E-Mails irgendwie im Web Outlook oder sowas zumindest sehen kannst?

**Michael:** Dann sehe ich alles.

**Thomas:** Dann siehst du auch, was kommt.

**Michael:** Ja, ja, das sehe ich alles.

**Jens:** Gut, dann holt ihr euch die Zugangsdaten und guckt mal, ob das irgendwo zwischen Telefonanlage und sonstigen Themen irgendwo hängt.

**Thomas:** Aber das hat jetzt mitgenommen, dass das Hauptproblem ist, dass da quasi egal wer anruft, dass man das immer mitkriegt. Das muss man hinkriegen, dass man immer sieht, dass zumindest jemand angerufen hat. Und das zweite Thema wäre das mit dem Englisch.

**Michael:** Ja, vorab die Info, das ist nur möglich, wenn man die Kundendaten erfasst hat. Also Mindestanforderung ist Kundenname, Firmenname und Telefonnummer. Ab dem Punkt—

**Thomas:** Du bist gerade abgehakt, Michael.

**Michael:** Jetzt geht's wieder. Ja. Genau, die Mindestanforderungen sind diese drei. Kundenname, Firmenname und Telefonnummer. Wenn nach diesem Teil—

**Jens:** Das ist okay, ich meine, ich glaube, das ist okay, wenn ein Kunde nicht bereit ist, seinen Firmennamen zu sagen, seinen Namen und wo man ihn zurückruft und er vorher auflegt, dann kann man ihm nicht helfen. Da gebe ich Robin recht. Aber wenn diese drei Attribute gefallen sind, dann muss der Bot definitiv eine E-Mail schicken. Weil dann entsteht eine Erwartungshaltung, die wir erfüllen müssen.

**Thomas:** Und die anderen Sachen, da kann man noch mal schauen, ob man aus der Telefonanlage zumindest mal zyklisch mal die Daten zieht und abgleicht. Dann kann man sagen, ok, so oft wurde angerufen und dann wird er abgelehnt.

**Robin:** Meine Anruferliste habe ich jetzt gerade an den Michael schon per E-Mail weitergeleitet.

**Thomas:** Nee, ich meine bei euch nicht. Im Telefonsystem ist es ja trotzdem wichtig, dass wir sagen, wie viel legen auf oder gehen gar nicht durch. Das müsste man trotzdem mehr erfassen.

**Jens:** Mhm.

**Thomas:** Also, dass wir wirklich jede auch irgendeine Übersicht haben, wer hat überhaupt alles angerufen?

**Jens:** Das wäre dann aber eine Analyse, die müsste man auch reinschalten, Thomas, dass wir so ein tägliches Report bekommen, automatisiert. Da müssen wir uns aber auch an die Telefonanlage andocken. Weil wenn dann dort jemand auflegt und es nichts passiert, ist das ja in der ersten Instanz letztendlich weggebrochen.

**Thomas:** Genau, entweder weggebrochen oder jemand hat aufgelegt, war ja nicht gut oder wie auch immer. Aber trotzdem wäre es, glaube ich, wichtig, dass wir das erfassen.

**Jens:** Ok. Habt ihr auch schon mal darüber geredet, wie so ein Reporting dann aussieht, wenn wir jetzt live gehen, darüber möchte ich auch mal reden, wann streben wir das an, ganz konkret, dass wir jetzt auch in der Organisation und für uns auch jetzt diesem Telefonbot 1.0 eine Zielgröße geben. Und wenn der dann live ist, brauchen wir ja auch so Tagesberichte, wie an Robin oder Consti zu sehen: Okay, heute haben X angerufen, drei haben aufgelegt, acht haben draufgesprochen und wir haben acht E-Mails bekommen, sodass wir uns da quasi tracken können, um zu sehen, wie die Performance läuft. Gibt es da Ideen von euch, wie so ein Berichtswesen aussehen kann?

**Michael:** Ja, es gibt auf jeden Fall solche Protokolle oder Logs, die man verarbeiten kann und zentral irgendwie oder per E-Mail am Ende der Woche verschicken kann. Aber ich — das Problem sind diese fehlgeschlagenen Anrufe. Denn den Rest — erfolgreiche Anrufe, Dauer, durchschnittliche Dauer und so weiter — das kann man alles ableiten. Aber ich muss einen Weg finden, diese abgebrochenen Anrufe auch zu erfassen.

**Jens:** Ja, das ist ein Ergebnis aus heute, Michael, weil du brauchst jetzt deine Zeit. Dafür testen wir ja auch. Ist ja auch gut, dass es jetzt hier passiert und nicht woanders.

**Michael:** Nee, diesen Bericht können wir auf jeden Fall konzipieren. Mit fünf Metriken, wichtigen Metriken für euch.

**Jens:** Das wäre schon mal gut.

**Michael:** Ich sag mal fünf wichtigen Metriken für euch. Ja.

**Jens:** Okay, weil das wäre wichtig, dass der Robin da sieht, wie wir da performen oder wie das auch genutzt wird, damit wir auch mal wirklich ein Gefühl haben, wie viele rufen eigentlich im Service an?

**Constantin:** Mhm. Ja.

**Jens:** D.h. es spricht auch derjenige drauf, der einfach ein Wartungsangebot oder einen Wartungstermin haben möchte oder der fünf Liter Öl bestellen will. Der spricht dann jetzt auch erst mal drauf. Und diese Idee damals ist die umgesetzt, zu sagen, es gibt erkenntliche Prioritäten der E-Mails. Wenn jemand die Begriffe Störung und sowas nennt, dass wir sehen, okay, das sind gehighlightete oder mit Prio hinterlegte Mails. Ist das umgesetzt?

**Robin:** Ja, das ist umgesetzt und es funktioniert auch tatsächlich am besten eigentlich. Das erkennt er schon ganz gut, wenn man jetzt sagt: „Meine Anlage macht laute Geräusche." „Mein Bündel ist leer." „Produktionsstillstand", das sind alles so Wörter. Da erkennt er, das hat eine hohe Prio.

**Jens:** Sehr gut.

**Robin:** Das ist natürlich, wenn man das als Ganzes betrachtet, weil du eben auch gesagt hast, Warenbestellung kommt ja auch über diesen Servicebot. Das ist halt ein bisschen unpraktisch für den Kunden aufgebaut, der halt wirklich nur eine Ware bestellen möchte per Telefon bei uns, weil er halt gefragt wird, was haben Sie für eine Anlage? Und das ist ja in dem Fall völlig irrelevant für den Kunden. Aber gut, muss er halt durch, muss er halt sagen. Ich habe keine Anlage, ich will Ware bestellen, dann wird das aufgenommen. Müssen wir mal schauen, wie man das dann im 2.0 anders da umsetzen kann. Oder ob man jetzt schon mit reinnimmt, dass man unterscheidet, man relativ früh fragt, Kundenname, Firma, Telefonnummer und dann, wie ist Ihr Anliegen? Und wenn er dann sagt, Warenbeschaffung, dass es dann in eine andere Richtung geleitet wird, als wenn er sagt, ich habe eine Störung. Ich weiß nicht, ob das jetzt schon möglich wäre.

**Michael:** Ja, das hatten wir gehabt, aber wir haben das wieder ausgebaut, weil wir hatten am Anfang diese fünf Richtungen gehabt: Problemmeldung, Ersatzteilbestellung, Wartung, Sonstiges und was noch. Ja, wir hatten noch eine Kategorie gehabt. Aber das hatten wir — wir hatten damals anders entschieden. Und wir hatten das vereinheitlicht und einen einzigen Ablauf für alles.

**Jens:** Und aus technischer Sicht, Michael, wäre das aber möglich. Wir haben es aus anderen Gründen jetzt—

**Michael:** Aus technischer Sicht kann man ganz, ganz vorne eine Frage ergänzen. Was ist der Grund des Anrufs? Ersatzteilbestellungen, Problemmeldung und so weiter und so fort. Diese fünf Kategorien, sag ich mal. Und je nachdem, was der Kunde sagt, dann wird er weitergeleitet. Und weitergeleitet heißt, es wird in diesem Baum eine Verzweigung definiert. Wenn der Kunde sagt, Ersatzteilbestellung, dann fragt der Bot: „Ja, welche Materialnummer wollen Sie bestellen beispielsweise, wie viele davon?" Und genau.

**Robin:** Das wäre ja der sauberste Weg, wenn man das eigentlich von Anfang an so aufbauen würde. Weil es ist ja letztendlich jeder KI-Bot funktioniert ja relativ ähnlich, sag ich mal. Gerade wenn man in einem Serviceunternehmen anruft, wird man zu Beginn ja erst mal gefragt: Was will man eigentlich? Bevor man gefragt wird, was für eine Anlage haben Sie, welche Seriennummer hat die, welches Baujahr hat die? Weil den Anrufer, der einfach nur Teile bestellen hat, interessiert das relativ wenig und der hat auch vielleicht gar keine Anlage, so wie wir es jetzt uns denken aktuell.

**Jens:** Dann würdest du Robin das begrüßen, dass wir das vorziehen. Dass wir den Kunden fragen, was ist Ihr Anrufgrund? Sie können wählen zwischen erstens Ersatzteilbestellung, zweites Servicetermin vereinbaren, drittens Störungsmanagement, viertens Sonstiges. Und dann sagt er, bitte zwei und dann kommt die Frage hinter zwei, Frage hinter drei.

**Robin:** Das würde ich tatsächlich begrüßen, dass wir es von Anfang an so strukturieren, dass wir tatsächlich darauf aufbauen können, weil wenn wir jetzt sagen, wir haben uns für den einen Weg im Einserbot entschieden, müssen aber im Zweierbot müssen wir einen ganz anderen Baum aufbauen, macht das ja eigentlich keinen Sinn.

**Jens:** Wie siehst du das, Michael?

**Michael:** Wir müssen viele neuere Themen erfassen, und zwar mit euch, die diesen Ablauf — wir müssen den Ablauf klären. Wenn der Kunde sagt, Ersatzteil, dann müssen wir definieren, welche Fragen werden gestellt und das für alle vier, fünf Verzweigungen des Baumes.

**Thomas:** Will man das schon zu kompliziert machen oder sagt man, man hat die Kategorien und trägt dann einfach nur eine Frage: Welche Teile willst du bestellen? Welche Wartungsthemen hast du? Was ist dein Problem?

**Robin:** Ich würde eine Einser relativ einfach lassen, so wie du sagst, Thomas. Und dann brauchst du eine Frage.

**Thomas:** Man sagt einfach, ich will was bestellen.

**Robin:** Richtig.

**Thomas:** Und dann kommt nur die Frage, welches Ersatzteil willst du bekommen?

**Robin:** Und dann fängt der an zu erzählen. Ja, ganz genau.

**Thomas:** Also mich sehen noch keine komplexen, noch keine komplexen Workflows.

**Michael:** Ich meine, das haben wir schon. Wir können die Fragen Anlageerfassung und Anliegenerfassung umdrehen, dass zunächst das Anliegen beschrieben wird. Und dann, wenn der Kunde eine Maschine bereits im Einsatz hat, dann wird die Frage Anlageerfassung gestellt. Wenn er sagt, ich will Ersatzteile bestellen oder Wartung oder habe eine allgemeine Frage, dann wird die Frage Anlage bezüglich Anlage nicht gestellt. Wir gehen davon aus, dass der Kunde keine Anlage hat.

**Robin:** Das mit der Anliegenbeschreibung ist mir trotzdem zu offen. Ich würde es trotzdem direkt unterteilen in Hauptgrund-Gruppen, wie wir es eben gesagt haben, dass du halt ganz klar den Kunden fragst: Was willst du von uns eigentlich? Geht es dir nur um eine Teilebestellung? Hast du eine Störung? Möchtest du einen Wartungstermin vereinbaren? Das würde ich schon gliedern.

**Michael:** Das ist ganz einfach zu implementieren, dass man sagt: Wir haben aktuell den Sprechertext „Bitte beschreiben Sie nun Ihr Anliegen." Das können wir erweitern. Brauchen Sie Ersatzteile? Möchten Sie ein Problem melden? Wir können den Text erweitern.

**Robin:** Du machst ja diese Anliegenbeschreibung zweimal. Wenn du zu Beginn die Offenheit bestellst — nee, nee, davor. Und danach kommt der Kunde sagt: „Ich möchte Wartungsteile bestellen." Oder „Nee, ich habe eine Störung." Dann fragst du: „Okay, nennen Sie mir bitte Baujahr, Seriennummer und Typ." Und danach fragst du dann wieder: „Nennen Sie mir jetzt nochmal Ihr Anliegen, theoretisch." Weißt du, deswegen würde ich vorher die Frage nicht offen stellen, sondern schon sagen: Möchten Sie warten? Möchten Sie eine Störung melden? Oder was auch immer. Dass man das schon gliedert von Anfang an.

**Jens:** Das wäre ja dann der Fall, wenn ich anrufe und wenn er fragt: Hier ist der Telefonbot hin und her. Sie können auswählen zwischen 1. Ersatzteilbeschaffung, 2. Terminvereinbarung für einen Vor-Ort-Termin, 3. Störungsmanagement, 4. Sonstiges. Bitte wählen Sie zwischen 1 und 4. Dann drückt er 2 und dann sagt er: „Bitte nennen Sie Ihre Ersatzteilbestellung." Dann redet er. Wenn er sagt Störungsmanagement, fragt er an: Bitte nennen Sie mir Ihren Anlagentyp, Seriennummer und beschreiben Sie Ihr Anliegen.

**Michael:** Das meine ich so. Ja, aber das ist jetzt, was ich davor beschrieben habe. Wir haben einen Baum und je nachdem, Option 1, 2, 3, 4, 5. Es werden danach andere abhängige Fragen gestellt. Wenn der Kunde Ersatzteile braucht, dann fragt der Bot: Welche Materialnummer, welche Artikelnummer?

**Jens:** Wie ist die — Michael. Da hat der Thomas recht. Diese Tiefe würde ich jetzt in diesen Workflow noch nicht reinbauen. Das wäre dann 2.0. Wenn er dann angekommen ist und hat sich für die Ersatzteilbeschaffung entschieden, dann würde ich in diesem Baum sagen, beschreiben Sie ein Anliegen. Dann schreibt er, ich hätte gern fünf Liter Öl von dem Hersteller des Hochdruckverdichters. Dann ist er — weil vielleicht hat er auch die Artikelnummer gerade nicht zur Hand — das reicht uns aber auch dann, glaub ich, im ersten Schritt, Robin, mit dieser Information sich dann zurückzumelden oder ne Bestellung auszulösen. Und wenn wir dann selbst mal Artikeln eigene Artikelnummer haben, Bot 2.0, kann man das natürlich konkretisieren und kann immer konkretere Details abfragen.

**Michael:** Ja, ja, aber genau das ist genau der Punkt. Du hast davor gesagt, Welche Ersatzteile wollen Sie bestellen? Und danach hast du gesagt: Bitte beschreiben Sie Ihr Anliegen. Das ist ganz unterschiedlich, weil das Anliegen, bitte beschreiben Sie Ihr Anliegen, ist allgemein, kann man überall nutzen, egal ob es sich um Ersatzteilbestellung handelt oder Anliegensbeschreibung oder Wartung oder sonstige Fragen. Das ist eine allgemeine Frage, die immer gestellt wird. Das ist einfach zu implementieren. Nein, das spielt eine Rolle. Das ist einfach zu implementieren. Das haben wir schon mehr oder weniger implementiert. Es muss ein bisschen angepasst werden, damit wir davor die Frage stellen: Was wollen Sie von uns? Wenn wir ganz—

**Jens:** Ich glaube, wir meinen das Gleiche, aber jeder denkt da in seiner eigenen Struktur, du, Michael, in der Art, die ich gleich umsetze. Für uns ist wichtig, dass wir, wenn wir eine E-Mail bekommen, wissen, um was es geht. Will der ein Ersatzteil oder will der eine Störung? Und dass der Kunde auch ein Gefühl hat, er läuft nicht immer das Gleiche durch, wo er seinen Anlagentyp nennen muss, hin und her, wenn er einfach nur einen Termin mit uns vereinbaren will. Da wollen wir eine Schleife vorziehen.

**Michael:** Genau, das ist die einzige Änderung, dass die Frage bezüglich Anlage — nennen Sie Anlagenbezeichnung, Seriennummer, Baujahr — die wird nur dann gestellt, wenn der Kunde ein Problem melden möchte oder wenn er eine Wartung vereinbaren möchte. Wenn die zwei Themen genannt wurden von dem Kunden, dann wird die Frage zur Anlage gestellt. Wenn er Ersatzteile braucht, sonstige Fragen hat — egal, wir finden das raus. Genau. Nur in den zwei Fällen wird die Frage zur Anlage gestellt, in den anderen Fällen nicht. Aber die Frage „Bitte beschreiben Sie Ihr Anliegen" bleibt immer gleich. Die wird immer gestellt, egal ob der Kunde Wartung braucht oder Ersatzteile.

**Thomas:** Ja. Vielleicht, Michael, das müssen wir einfach mal kurz skizzieren. Dann schauen wir auch, was diese vier oder fünf Kriterien sind. Und da muss man auch, glaube ich, schauen, wann — dürft nicht vergessen, wir müssen nur den Bug noch rauskriegen.

**Michael:** Das ist prima. Das mach ich zunächst.

**Thomas:** Das wär auch der Trigger, sobald jemand sagt „Ersatzteil", da muss auf jeden Fall das irgendwie gelockt werden. Und das wär dann der erste Trigger, dass was rausgeschickt wird, ne.

**Michael:** Ja.

**Jens:** Okay. Für uns wäre ganz wichtig, einen Termin zu haben. Wir haben am 16. — korrigiert mich, Jungs, ich weiß nicht, oder 17.

**Thomas:** Nee, Hausmesse.

**Jens:** September die Hausmesse.

**Michael:** Mhm.

**Jens:** Und da muss der Telefonbot live sein.

**Michael:** Also, ja, ich mach mich an die Arbeit.

**Robin:** Ich denke, wir brauchen jetzt auch mal engere Termine, dass wir da nicht immer mehr, zwei Wochen Zeit haben, dass wir uns da mal richtig dahinter klären.

**Michael:** Genau, vielleicht können wir auch live testen zusammen. Also—

**Jens:** Ja, müssen wir einfach mal machen, dass jeder sieht, wie der Ablauf ist.

**Michael:** Also wie sieht's bei euch, also jetzt Robin und Konstantin — morgen um zehn. Morgen um zehn. Ja. Ich melde mich, also wir können das so machen. Wir treffen uns morgen um zehn.

**Robin:** Ja, nee, das war jetzt nur so gesagt. Ich hab morgen um zehn einen Termin.

**Michael:** Okay.

**Robin:** Um elf könnten wir uns morgen treffen.

**Michael:** Um elf.

**Jens:** Meinst du, ich bin so schnell mit dir fertig, in einer Stunde?

**Robin:** Ich denke, so schnell ist sie mit uns fertig, Jens.

**Jens:** Wenn es nur um uns gehen würde, ja.

**Michael:** Machen wir morgen um elf aus und wenn ich das nicht schaffe heute, diesen Bug zu fixen, dann schieben wir den Termin auf. Was wäre für dich ein alternativer Zeitpunkt? Und dann schieb doch den Termin gleich auf Freitag.

**Jens:** Da brauchen wir jetzt Zeit, um das Geld zu bearbeiten. Dann lass uns Freitag, 10 Uhr.

**Robin:** Machen, wenn das für dich in Ordnung ist, Michael. Und dann hast du da noch einen Tag los.

**Michael:** Okay. Und dann versuche ich auch beide Termine — auch die Frage zum Thema — zu implementieren. Gut, Freitag um 10. Super, du kriegst die Einladung von mir. Konstantin, bist du auch dabei?

**Constantin:** Ja.

---

**Jens:** Also, ich glänze jetzt mal mit gefährlichem Halbwissen. Ich hab einen Artikel gelesen vor 3–4 Wochen und hab mir ein paar Notizen dazu geschrieben. Ich weiß nicht mehr genau, wie ich da drauf kam, aber das waren mehrere Seiten irgendwo in einem Magazin. Da geht es um spezialisierte Voice-KI-Firmen. Und da gibt es ein in Europa führendes Unternehmen, das ist, glaube ich, was weiß ich, deutsch, irisch oder sitzt auf jeden Fall in Berlin. Eleven Labs.

**Michael:** Ja. Wir machen Stimmerkennung, Transkribierung, Speech-to-Text, Text-to-Speech, also solche Sachen.

**Jens:** Genau, spezialisiert auf diese Themen, wo du mit Kunden kommunizierst, ja, diese Abläufe. Und wenn ich das richtig verstanden habe, haben wir ja jetzt den technischen Hintergrund, Michael, über Copilot Studio laufen, oder?

**Michael:** Das weiß ich nicht. Sie haben ein eigenes Tool auf jeden Fall.

**Thomas:** Nein, wir, was wir haben—

**Jens:** Wir, was wir haben.

**Michael:** Das ist jetzt Copilot Studio, ja.

**Jens:** Könnt ihr mal gucken, was da der Unterschied ist? Wenn ich so an den Telefonbot 3.0 denke, also ich denke jetzt an das Endziel. Mit welcher technischen Struktur und mit welcher KI kommen wir dahin, am besten nach unseren Bedingungen? Und wer bietet da die bestmögliche Lösung an? Weil wir wollen ja mit 3.0 den Kunden, im schlimmsten Fall hängt der eine Viertelstunde mit dem Bot, lässt sich das erklären, lässt sich vom Bot erklären wie eine Abstellmaßnahme, weil er das Problem sauber beschrieben hat, dass der Bot ihm die Standardabstellmaßnahme erklärt, was er an der Anlage zu tun hat. Das ist ja eine richtige Kommunikation, ein Hin und Her. Und mich interessiert einfach, wer am Markt kann uns das dem Telefonbot 3.0 denn von der technischen Gesamtlösung am besten darstellen. Mal unabhängig davon, was es kostet. Einfach mal, wo steht da der Markt und wo stehen da die Anbieter? Jetzt für den Telefonbot 1.0 reicht uns das. Wir haben ja gesagt, wir wollen jetzt erst mal eine Entlastung im Service und dass wir aktiver reagieren können und nicht mehr durch Anrufe überrascht werden, sondern wir können uns vorbereiten.

**Thomas:** Und der Anspruch war ja auch eine günstige Lösung.

**Jens:** Richtig.

**Michael:** Ja, genau. Wir machen es in der Schlüssel.

**Jens:** Ich wollte es jetzt nur noch platzieren, dass wir uns bitte da Richtung Weihnachten Gedanken machen und wir sollten uns vorher überlegen, was wir uns da unter den Weihnachtsbaum legen. Das sollte dann ein, zwei Jahre funktionieren.

**Michael:** Ja, super. Eleven Labs habe ich auch genutzt vor anderthalb Jahren oder so. Die sind immer noch die besten in dem Bereich, was Speech-to-Text oder Text-to-Speech zu tun hat. Sie machen auch — ich denke, sie werden — sie sind auch in Hollywood genutzt, weil sie die beste Maschine anbieten für sowas.

**Jens:** Das ist ein Text-to-Speech.

**Michael:** Text-to-Speech ist, ich lasse einen Text vorlesen.

**Jens:** Und das andere, das muss ich — das wäre ja dann zum Beispiel ganz wichtig, wenn wir sagen, wenn der Agent erkennt, okay, der in 3.0 — ich denke jetzt laut, kann auch sein, dass ich komplett falsch liege — dann korrigiert mich bitte. „Bei mir sind da zwei Lichter rot, das und das und das." „Okay, jetzt weiß ich, wo du bist." Dafür gibt es eine Abstellmaßnahme. Dann greift der Bot quasi auf einen Text zurück, weil wir diese Abstellmaßnahme definiert haben. Dann liest er ihm den Text vor und sagt: In einem ersten Schritt gehst du hin und drückst jetzt auf den Knopf 1. Dann drückst du auf den Knopf 2. Dann stellst du den Wert wieder zurück auf 0 oder so irgendetwas. Er muss ihm ja Antworten geben. Jetzt haben wir ja einen passiven Bot. Der nimmt auf und schreibt es weg. Der kommuniziert ja nicht.

**Michael:** Genau.

**Jens:** Okay, also. Das heißt, wenn wir am 16. oder 17. die Messe haben, haben wir schon einen Tag live hinter uns. Die Kollegen dann auch im Vertrieb, alle wissen Bescheid und dann kann man damit offen kommunizieren.

**Michael:** Cool. Super.

**Jens:** Gut.

**Thomas:** Okay.

**Jens:** Haben wir noch was?

**Thomas:** Nee, die Bugs müssen jetzt raus.

**Jens:** Ja. Gut.

**Thomas:** Genau.

**Jens:** Okay.

**Michael:** Danke.

**Robin:** Dann hören wir uns dazu am Freitag wieder.

**Michael:** Genau. Danke sehr und bis dann.

**Robin:** Bis Freitag.

**Thomas:** Bis dann.

**Michael:** Ciao.
