# Hinweise für Claude

## Änderungen: erst Artefakt, dann push

1. Jede Änderung zuerst im Artefakt „Spieltage-Übersicht“
   (https://claude.ai/artifact/1kr6aesQgbHLjAwDDLobvF) zeigen, also die
   geänderte `index.html` dorthin veröffentlichen. Vorher nichts auf GitHub tun:
   kein Commit, kein Push, kein Branch, kein Pull Request.
2. Erst wenn der Nutzer „push“ schreibt: alles direkt auf `main` committen und
   pushen, `index.html` wird dabei überschrieben. Kein Feature-Branch, kein Pull
   Request.

Ungepushte Änderungen liegen nur in der Sitzungsumgebung und verschwinden mit
ihr — liegt länger etwas herum, daran erinnern.

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
