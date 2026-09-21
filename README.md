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
  Terminberechnung, Konfliktprüfung, Formular, CSV-Export.
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
ansicht, Checkbox-Auswahl im Formular, Labels und CSV-Export werden daraus
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

Mögliche Rhythmen: `jede_woche`, `gerade_wochen`, `ungerade_wochen`,
`alle_4_wochen`, `alle_8_wochen`, `einmalig`.

### Einzeltermine

`einmalig` ist ein einzelner Termin statt eines Takts – etwa ein Turnier an
einem bestimmten Samstag. `startDatum` ist dann **Pflicht** und benennt den
Termin selbst.

Dabei gibt das Datum den Wochentag vor: Ein Termin am Samstag kann keinen
Mittwoch-Slot belegen. Das Formular weist eine solche Kombination ab und nennt
den Wochentag des gewählten Datums.

Ist der Termin vorbei, verschwindet er aus der Wochenansicht und aus dem
CSV-Export – eine Wochenvorlage soll nicht zeigen, was nicht mehr ansteht.
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

`startDatum` ist bei jedem Rhythmus möglich und legt fest, **ab wann** die Runde
läuft: Vor diesem Tag gibt es keinen Termin, und die Konfliktprüfung zählt die
Wochen davor nicht mit. Damit lässt sich "ab Oktober jeden Samstag" abbilden.

Ohne Startdatum läuft eine Runde von Anfang an – für die wöchentlichen und die
geraden/ungeraden Rhythmen ist das Feld also rein optional.

### Zusätzliche Rolle bei den langen Takten

Bei den 4- und 8-Wochen-Rhythmen ist `startDatum` nicht nur Beginn, sondern auch
Taktgeber, und benennt einen echten
Termin der Runde. Der Takt wird von dort aus absolut weitergerechnet, nicht über
Kalenderwochen-Nummern – sonst würde er am Jahreswechsel springen, weil ein Jahr
auch 53 Wochen haben kann. Fehlt das Startdatum, lässt sich weder der nächste
Termin berechnen noch ein Konflikt sicher feststellen; die Seite schreibt dann
"Datum unbekannt" beziehungsweise "möglicher Konflikt".

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
sondern über `textContent` beziehungsweise geprüfte Farbwerte. Im CSV-Export
bekommen Zellen mit führendem `=`, `+`, `-` oder `@` ein vorangestelltes
Apostroph, damit Tabellenprogramme sie nicht als Formel ausführen.

Keine `<table>`-Elemente: Tabellen sind auf dem Handy schlecht lesbar. Unter
380 px Breite stapeln sich Tag, Zeit und Inhalt untereinander. Es gibt eine
Druckansicht (`@media print`) für den Aushang am Vereinsbrett.

## Offene Punkte

- **Kein Backup.** Alle Daten liegen an einer Stelle. "Alle Regeln löschen"
  trifft mit zwei Klicks den Spielplan für alle, ohne Rückholmöglichkeit. Der
  kostenlose Firebase-Tarif kennt keine automatischen Sicherungen; bis dahin ist
  der CSV-Export die einzige Sicherung, und die muss jemand von Hand anstoßen.
- Die Datenbankregeln prüfen nicht, ob eingehende Daten die richtige Form haben.
  Unkritisch, solange nur Leute mit Passwort schreiben.
- Es ist nicht nachvollziehbar, wer wann was geändert hat.
