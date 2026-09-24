# Spieltage-Übersicht

Online-Übersicht, welcher Tabletop-Playday beim Privateer-Poza-Boyz e.V. wann
stattfindet. **Kein Kalender** – eine feste Wochen-Vorlage (Mo–So), in der
Runden nach Rhythmus eingetragen werden, ergänzt um den jeweils nächsten Termin.

**Live:** https://otzmo.github.io/Playday-Uebersicht/

Lesen kann die Seite jeder. Ändern nur, wer das Vereinspasswort kennt.

## Aufbau

Eine einzige Datei, `index.html`, mit HTML, CSS und JavaScript darin. Kein
Build-Schritt, keine Abhängigkeiten zum Installieren. Bearbeiten, committen,
pushen – GitHub Pages veröffentlicht `main` automatisch.

Zwei Script-Blöcke am Dateiende:

- Der **klassische Block** enthält die gesamte Logik: Wochenraster, Regeln,
  Terminberechnung, Konfliktprüfung, Formular, Export.
- Der **Modul-Block** (`type="module"`) lädt Firebase und verbindet sich mit
  Datenbank und Anmeldung. Er ruft die Funktionen `ppbServerDaten`,
  `ppbVerbindung`, `ppbFehler` und `ppbFreischaltung` im klassischen Block auf
  und stellt umgekehrt `ppbSchreibe`, `ppbAnmelden` und `ppbAbmelden` bereit.

Die Trennung hat einen Grund: Lädt das Firebase-Modul nicht – offline, blockiert,
Dienst gestört –, läuft der klassische Block unverändert weiter und arbeitet mit
dem lokalen Zwischenspeicher. Die Seite bleibt lesbar und bedienbar.

## Slots und Regeln

Neun feste Zeit-Slots pro Woche, definiert im Array `TAGE`:

- Montag bis Freitag Abend, 16:00–22:00
- Samstag und Sonntag Vormittag, 09:30–15:30
- Samstag und Sonntag Nachmittag, 16:00–22:00

`TAGE` ist die **einzige** Quelle für Tage, Zeiten und Beschriftungen. Wochen-
ansicht, Checkbox-Auswahl im Formular, Labels und Export werden daraus
erzeugt; eine Zeitänderung passiert nur an dieser Stelle. Die Slot-Schlüssel
(`samstag_vormittag` usw.) stehen so auch in gespeicherten Regeln und dürfen
nicht umbenannt werden.

Eine Regel belegt einen oder mehrere Slots:

```json
{
  "system": "Kill Team",
  "slots": ["samstag_vormittag"],
  "rhythmus": "jede_woche",
  "farbe": "#8E7CC3",
  "startDatum": "2026-09-19"
}
```

Wählbare Rhythmen: `jede_woche`, `alle_2_wochen`, `alle_4_wochen`,
`alle_8_wochen`, `einmalig`.

### Nicht mehr wählbar: gerade und ungerade Wochen

`gerade_wochen` und `ungerade_wochen` stehen nicht mehr im Formular. Sie hingen
an der Kalenderwochen-Nummer, und nach einem Jahr mit 53 Wochen folgt auf KW 53
wieder KW 1 – zwei ungerade Wochen hintereinander. 2026 ist so ein Jahr: Eine
ungerade Runde hätte am 02.01.2027 (KW 53) *und* am 09.01.2027 (KW 1) gespielt,
eine gerade vom 26.12.2026 bis 16.01.2027 pausiert. `alle_2_wochen` zählt
stattdessen vom Startdatum aus und kommt nie aus dem Takt.

**Bestehende Runden mit diesen Rhythmen bleiben erhalten** und werden weiter
richtig berechnet (`VERALTETE_RHYTHMEN`). Würde die Seite sie verwerfen, wären
sie in der Datenbank zwar noch da, auf der Seite aber unsichtbar. In der
Regel-Liste tragen sie den Zusatz "bitte auf ‚alle 2 Wochen' umstellen". Beim
Bearbeiten ist dann kein Rhythmus vorausgewählt – gespeichert werden kann erst
nach einer Wahl –, und das Startdatum ist mit dem nächsten Termin nach dem alten
Takt vorbelegt. Mit "alle 2 Wochen" läuft die Runde so im selben Wechsel weiter,
Ausfälle bleiben erhalten. Erst jenseits des nächsten 53-Wochen-Jahres weichen
die Termine vom alten Verhalten ab – und genau das ist der Zweck.

Sind keine solchen Runden mehr in der Datenbank, können `VERALTETE_RHYTHMEN`,
die beiden Einträge in `SUFFIX` und die gerade/ungerade-Zweige in `terminAb`
und `grundTakt` entfernt werden.

### Einzeltermine

`einmalig` ist ein einzelner Termin statt eines Takts – etwa ein Turnier an
einem bestimmten Samstag. `startDatum` ist dann **Pflicht** und benennt den
Termin selbst.

Dabei gibt das Datum den Wochentag vor: Ein Termin am Samstag kann keinen
Mittwoch-Slot belegen. Das Formular weist eine solche Kombination ab und nennt
den Wochentag des gewählten Datums.

Ist der Termin vorbei, verschwindet er aus der Wochenansicht – eine
Wochenvorlage soll nicht zeigen, was nicht mehr ansteht. Im Export
erscheint er, wenn der gewählte Zeitraum sein Datum einschließt.
In der Regel-Liste unter "Bearbeiten" bleibt er sichtbar und ist mit "vorbei"
gekennzeichnet, damit er nicht unbemerkt verschwindet und gelöscht werden kann.
Ein Datum in der Vergangenheit lässt sich eintragen, die Seite fragt aber
einmal nach – sonst sähe ein Tippfehler im Jahr wie ein Fehlschlag aus.

### Ausfälle

Wiederkehrende Runden können einzelne Termine absagen. Dazu trägt die Regel ein
optionales Feld `ausfaelle` mit ISO-Datumswerten:

```json
{ "system": "Kill Team", "slots": ["samstag_vormittag"],
  "rhythmus": "jede_woche", "ausfaelle": ["2026-09-26", "2026-10-03"] }
```

Eingetragen wird das beim **Bearbeiten** einer Regel: Dort erscheinen die
nächsten acht Termine des Takts als Kacheln zum Abhaken. Beim Anlegen einer
neuen Regel fehlt die Auswahl – da gibt es noch nichts abzusagen – und bei
`einmalig` ebenfalls, dort löscht man stattdessen die Regel.

Ein Ausfall hängt an einem konkreten Tag, nicht an einer Woche: Belegt eine
Regel Montag und Freitag, streicht ein Ausfall am Montag nicht den Freitag
derselben Woche. Die Konfliktprüfung berücksichtigt das und bekommt deshalb den
Wochentag des jeweiligen Slots mitgegeben.

Vergangene Ausfälle werden beim Speichern verworfen. Sie wirken sich auf nichts
mehr aus und würden sich über die Jahre ansammeln.

In der Wochenansicht steht bei einer Regel, deren nächster Takttermin
ausfällt, "26.09. fällt aus, nächster Termin 03.10." – die Absage ist für den
Leser die wichtigere Information als das Datum danach.

### Startdatum

`startDatum` ist bei **jedem** Rhythmus Pflicht: Das Formular speichert keine
Regel ohne. Es legt fest, ab wann die Runde läuft – vor diesem Tag gibt es
keinen Termin, und die Konfliktprüfung zählt die Wochen davor nicht mit. Damit
lässt sich "ab Oktober jeden Samstag" abbilden.

Regeln, die aus der Zeit stammen, als das Feld noch freiwillig war, bleiben
gültig: Eine wöchentliche Runde ohne Startdatum läuft einfach von Anfang an,
ein Takt ohne Startdatum zeigt "Datum unbekannt". In der Regel-Liste tragen
beide den Zusatz "Startdatum fehlt". Beim Bearbeiten schlägt das Formular den
nächsten Termin vor, wenn er sich berechnen lässt (wöchentlich), und verlangt
sonst eine Eingabe; gespeichert wird so oder so nur mit Datum.

### Zusätzliche Rolle bei den langen Takten

Bei den 2-, 4- und 8-Wochen-Rhythmen ist `startDatum` nicht nur Beginn, sondern
auch Taktgeber, und benennt einen echten
Termin der Runde. Der Takt wird von dort aus absolut weitergerechnet, nicht über
Kalenderwochen-Nummern – sonst würde er am Jahreswechsel springen, weil ein Jahr
auch 53 Wochen haben kann. Fehlt das Startdatum, lässt sich weder der nächste
Termin berechnen noch ein Konflikt sicher feststellen; die Seite schreibt dann
"Datum unbekannt" beziehungsweise "möglicher Konflikt".

Die Takte stehen im Code an einer Stelle, in `TAKT_WOCHEN`
(`{ alle_2_wochen: 2, alle_4_wochen: 4, alle_8_wochen: 8 }`). Ein weiterer
Takt ist dort ein Eintrag, dazu ein Text in `SUFFIX` und ein Knopf im Formular.

Konflikte werden nicht über Perioden-Kongruenz bestimmt, sondern indem für die
nächsten 104 Wochen Woche für Woche verglichen wird, ob zwei Regeln im selben
Slot zusammentreffen. Grund: Die Kalenderwochen-Parität von "gerade/ungerade
Wochen" lässt sich nicht zuverlässig mit einem absoluten Wochentakt verrechnen.

## Firebase

Projekt **spieltageuebersicht**. Die Konfiguration steht offen im Quelltext von
`index.html` – das ist bei Firebase-Web-Apps normal und unkritisch, die Werte
benennen nur das Projekt. Der Schutz kommt aus den Datenbankregeln.

### Realtime Database

Region europe-west1:

```
https://spieltageuebersicht-default-rtdb.europe-west1.firebasedatabase.app
```

Ablage unter `/regeln/<id>`, ein Knoten pro Regel. Die ID steht im Schlüssel,
nicht noch einmal im Datensatz.

Geschrieben wird gezielt nur die geänderte, neue oder gelöschte Regel, nicht die
ganze Liste. Damit überschreiben sich zwei Leute nicht gegenseitig, wenn sie
gleichzeitig verschiedene Runden bearbeiten.

Regeln der Datenbank:

```json
{
  "rules": {
    "regeln": {
      ".read": true,
      ".write": "auth != null"
    }
  }
}
```

Alles außerhalb von `regeln` bleibt gesperrt.

### Anmeldung

Anmeldeverfahren **E-Mail/Passwort**, ein einziger Nutzer als gemeinsames
Bearbeiter-Konto:

```
philip.scherer85@gmail.com
```

Die Adresse steht als `BEARBEITER_KONTO` im Modul-Block und wird auf der Seite
nie angezeigt. Für die Nutzer gibt es kein Konto und keine Registrierung: Ein
Klick auf "Bearbeiten" zeigt ein einzelnes Passwortfeld. Das Passwort wird von
Firebase geprüft, nicht in der Seite – ein clientseitiger Vergleich wäre
wirkungslos, weil der Quelltext öffentlich ist.

Passwort ändern oder Adresse wechseln: Firebase-Konsole → Authentication →
Nutzer. Bei einer neuen Adresse zusätzlich `BEARBEITER_KONTO` in `index.html`
anpassen.

Firebase liefert für "Konto existiert nicht" und "falsches Passwort" absichtlich
denselben Fehlercode. Schlägt die Anmeldung fehl, lohnt darum zuerst der Blick
in die Nutzerliste, ob die Adresse dort exakt so steht wie im Code.

### Autorisierte Domains

Unter Authentication → Einstellungen muss `otzmo.github.io` eingetragen sein,
sonst verweigert die Anmeldung von der Seite aus den Dienst.

### Analytics

Bewusst **nicht** eingebunden, obwohl das Firebase-Projekt eine
`measurementId` hat. Für einen Spielplan bringt es nichts, und Analytics ohne
Einwilligungsbanner ist in Deutschland heikel. Die Seite setzt keine Cookies und
braucht kein Banner.

## Lokaler Zwischenspeicher

Unter dem Schlüssel `ppb-playday-regeln` im `localStorage`. Er ist **nicht** die
Wahrheit, sondern Zwischenspeicher: Er zeigt die Übersicht sofort beim Laden und
trägt sie weiter, wenn die Datenbank nicht erreichbar ist. Sobald Daten vom
Server ankommen, gilt der Server.

Sind beim ersten Verbinden lokale Regeln vorhanden und die Datenbank leer,
bietet die Seite einmalig an, sie zu übernehmen.

Die Statuszeile im Fuß zeigt den Zustand an. "Verbunden" meldet sie erst, wenn
tatsächlich Daten angekommen sind – eine bestehende Verbindung allein genügt
nicht, sonst würde ein abgelehnter Lesezugriff übertüncht.

## Bedienung

Browser-Dialoge (`alert`, `confirm`, `prompt`) werden **nicht** verwendet.
Bestätigungen laufen über einen zweiten Klick auf denselben Knopf, der nach fünf
Sekunden verfällt. Grund: In eingebetteten Ansichten (Sandbox-iframes) ignoriert
der Browser solche Dialoge kommentarlos und liefert `false` zurück – Löschen und
Zurücksetzen waren dadurch wirkungslos, ohne jede Fehlermeldung.

Freitext von Nutzern (Systemnamen, Farben) wird nie über `innerHTML` eingefügt,
sondern über `textContent` beziehungsweise geprüfte Farbwerte – auch nicht in
der Druckansicht des Exports. Im CSV-Export bekommen Zellen mit führendem `=`,
`+`, `-` oder `@` ein vorangestelltes Apostroph, damit Tabellenprogramme sie
nicht als Formel ausführen. In der Excel-Datei stellt sich das Problem nicht:
Texte stehen dort als reiner Text, nie als Formel.

Keine `<table>`-Elemente: Tabellen sind auf dem Handy schlecht lesbar. Unter
380 px Breite stapeln sich Tag, Zeit und Inhalt untereinander. Es gibt eine
Druckansicht (`@media print`) für den Aushang am Vereinsbrett; ist die
Druckansicht des Exports offen, wird stattdessen nur sie gedruckt.

## Export

"Exportieren" öffnet ein Feld mit **Von** und **Bis** (vorbelegt: heute bis in
vier Wochen) und der Wahl des Formats. Inhalt ist in allen drei Formaten
derselbe, eine Tagesübersicht: je Tag, je Slot, je Runde ein Eintrag, nach
Datum sortiert, mit Datum, Wochentag, Zeit, System, Rhythmus und Hinweis
(gebaut in `tagesEintraege()`).

- Abgesagte Termine stehen mit Hinweis "fällt aus" drin – so bleibt sichtbar,
  dass dort sonst gespielt würde.
- Treffen an einem Tag im selben Slot zwei Runden aufeinander, die beide
  stattfinden, steht bei beiden "Konflikt". Fällt eine davon aus, ist es kein
  Konflikt.
- Mit "Freie Slots mit aufführen" bekommt jeder unbelegte Slot einen Eintrag
  "frei" – auch einer, in dem alle Runden des Tages ausfallen.
- Runden mit langem Takt ohne Startdatum lassen sich keinem Tag zuordnen. Sie
  stehen einmal am Ende, ohne Datum, mit "Termine unbekannt".

Der Zeitraum ist auf 366 Tage begrenzt. Dateiname:
`spieltage_<von>_bis_<bis>.xlsx` bzw. `.csv`.

**Excel-Datei** (Voreinstellung): formatiert und weiter bearbeitbar. Kopfzeile
fett auf Gold, fixiert und mit Filter; Datum als echtes Datum (sortier- und
filterbar); Wochen im Wechsel leicht hinterlegt, neue Woche mit Goldlinie,
neuer Tag mit feiner Linie; Konflikte rot hinterlegt, Ausfälle grau
durchgestrichen, freie Slots grau kursiv. Beim Drucken aus Excel: A4 hoch,
auf Seitenbreite, Kopfzeile auf jeder Seite, Zeitraum oben, Seitenzahl unten.

Die Datei wird in der Seite selbst erzeugt (`xlsxDatei()`, `zipArchiv()`),
ohne Bibliothek: die gängigen bringen rund ein Megabyte mit und hängen an einem
fremden Server. Eine .xlsx ist ein ZIP mit einigen XML-Dateien; das ZIP wird
unkomprimiert geschrieben, dafür reicht eine CRC-32-Prüfsumme. Geprüft mit
openpyxl, ExcelJS und SheetJS. Wer daran etwas ändert: Excel ist bei der
Reihenfolge der XML-Elemente streng und meldet sonst eine "beschädigte" Datei.

**Druckansicht / PDF**: ersetzt die Seite durch ein helles Blatt, gegliedert
nach Kalenderwochen und Tagen, mit Farbpunkt je System, Zählung oben
(Termine, Konflikte, Ausfälle) und Stand-Datum. Wochen ohne Termin stehen mit
"Keine Spieltage in dieser Woche." drin, damit die Lücke auffällt. "Drucken /
als PDF speichern" öffnet den Druckdialog des Browsers; "Zurück" oder Esc
schließt die Ansicht. Freie Slots lassen sich auch hier einblenden, machen die
Liste aber lang.

**CSV (Rohdaten)**: ohne Formatierung, zum Weiterverarbeiten. Beginnt mit einer
UTF-8-Kennung und trennt mit Semikolon, damit Excel sie auf deutschen Systemen
direkt richtig öffnet.

Im Artefakt (der Vorschau in Claude) lösen Excel und CSV keinen Download aus –
der Viewer blockiert Downloads –, und der Druckdialog kann dort ausbleiben. Auf
der echten Seite funktioniert beides.

## Offene Punkte

- **Kein Backup.** Alle Daten liegen an einer Stelle. "Alle Regeln löschen"
  trifft mit zwei Klicks den Spielplan für alle, ohne Rückholmöglichkeit. Der
  kostenlose Firebase-Tarif kennt keine automatischen Sicherungen. Der
  Export ist **keine** Sicherung: Er enthält Termine, nicht die Regeln, und
  lässt sich nicht wieder einlesen.
- Die Datenbankregeln prüfen nicht, ob eingehende Daten die richtige Form haben.
  Unkritisch, solange nur Leute mit Passwort schreiben.
- Es ist nicht nachvollziehbar, wer wann was geändert hat.
