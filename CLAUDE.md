# Hinweise für Claude

## Nicht ungefragt veröffentlichen

**Kein `git push` ohne ausdrückliche Aufforderung.** Ändern, testen und lokal
committen ist in Ordnung; der Push wartet auf ein klares "push" vom Nutzer.

Nach einem lokalen Commit kurz melden, dass etwas zum Pushen bereitliegt.
Wichtig, weil ungepushte Commits nur in der Sitzungsumgebung liegen und mit ihr
verschwinden — liegt länger etwas herum, daran erinnern.

Ein Push auf `main` geht hier direkt live: GitHub Pages veröffentlicht den
Stand automatisch unter https://otzmo.github.io/Playday-Uebersicht/

## Zusammenarbeit

Sprache ist Deutsch, knapp und direkt. Keine Tabellen — weder in Antworten noch
in der Oberfläche; die Seite ist für Handys gebaut, dort sind Tabellen schlecht
lesbar.

Bei Unklarheit vor der Umsetzung nachfragen, statt eine Annahme einzubauen.
Getroffene Entscheidungen und ihre Gründe gehören in die Commit-Nachricht, nicht
nur in den Chat — der Chat ist in der nächsten Sitzung nicht mehr da.

## Vor dem Ändern lesen

`README.md` beschreibt Aufbau, Datenmodell, die Firebase-Einrichtung und die
Entscheidungen, die sonst wieder aufgerollt werden. Zwei Fallen, die dort
ausführlicher stehen und schon einmal Zeit gekostet haben:

- In der Seite werden **keine** Browser-Dialoge verwendet (`alert`, `confirm`,
  `prompt`). In eingebetteten Ansichten ignoriert der Browser sie kommentarlos
  und liefert `false` — Löschen und Zurücksetzen waren dadurch wirkungslos,
  ohne Fehlermeldung. Bestätigungen laufen über einen zweiten Klick.
- Freitext von Nutzern nie über `innerHTML` einfügen, sondern über
  `textContent`; Farben und Datumswerte vorher prüfen.

Beim Testen im Browser die Seite in einem Sandbox-iframe laden, nicht direkt als
Datei — sonst verhält sie sich anders als im echten Einsatz.
