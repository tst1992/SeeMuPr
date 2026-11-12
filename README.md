# Seerecht Muster Prüfung

Trainer für 30 Musterprüfungsfragen im Fach **Schifffahrtsrecht** des
Sportseeschifferscheins: KVR, SeeSchStrO, Schiffspapiere und SUG. Die Fragen
kommen in zufälliger Reihenfolge, die Antwort wird frei formuliert und gegen
feste Schlüsselelemente geprüft. Danach erscheint immer die Musterlösung.

Falsche Antworten kommen in derselben Runde nochmals, höchstens zweimal.

Die App läuft ohne Server, ohne Anmeldung und nach dem ersten Aufruf auch
ohne Netz. Gestaltung und Programm stammen aus „SSS Seemannschaft 25 Fragen“
(Repository SeeMann25).

---

## Auf GitHub Pages veröffentlichen

Im Repository **Settings → Pages** öffnen. Unter **Build and deployment** als
Quelle **Deploy from a branch** wählen, Branch **main**, Ordner **/ (root)**,
speichern. Nach ein bis zwei Minuten ist die App erreichbar unter
`https://tst1992.github.io/SeeMuPr/`.

Alle Pfade sind relativ, die App läuft deshalb auch in einem Unterordner.

## Als App installieren

- **iPhone/iPad**: Adresse in Safari öffnen → Teilen → „Zum Home-Bildschirm“.
- **Android**: in Chrome öffnen → Menü ⋮ → „App installieren“.
- **Desktop (Chrome/Edge)**: Installations-Symbol rechts in der Adresszeile.

Nach dem ersten Aufruf liegt alles auf dem Gerät. Die App startet dann auch
im Flugmodus.

## Örtlich ausprobieren

`index.html` doppelklicken. Die Datei ist in sich geschlossen: Stylesheet,
Programm und alle 30 Fragen stecken darin. Installation und Offlinebetrieb
funktionieren erst über eine Webadresse (GitHub Pages oder z. B.
`python3 -m http.server 8080` in diesem Ordner).

---

## Aufbau

```
index.html             Die ganze App: Stylesheet, Programm, Fragen (JSON-Block)
manifest.webmanifest   Angaben für die Installation als App
sw.js                  Offlinebetrieb
icons/                 App-Symbole (192, 512, maskable, Apple)
.nojekyll              GitHub soll die Dateien unverändert ausliefern
```

## Fragen bearbeiten

Die Fragen stehen in `index.html` im Block
`<script id="fragen-daten" type="application/json">`. Je Frage:

```json
{
  "id": "f17", "nummer": 17,
  "frage": "Welche Schallsignale muss ein Maschinenfahrzeug geben, das Fahrt durchs Wasser macht?",
  "loesung": ["Mindestens alle 2 Minuten einen langen Ton (KVR-Regel 35 a)."],
  "anmerkung": "optional, erscheint unter der Musterlösung",
  "hinweis": "Ein Ton – wie lang, wie oft?",
  "mindestanzahl": 2,
  "elemente": [
    { "label": "Ein langer Ton", "pflicht": true, "treffer": ["lang", "langer ton"] }
  ]
}
```

- **`frage`**: `\n` erzeugt einen Zeilenumbruch, z. B. für Teilfragen.
- **`loesung`**: eine Zeile je Absatz. Zeilen, die mit `- ` beginnen, werden
  zu Aufzählungspunkten.
- **`elemente`**: die Schlüsselelemente. `pflicht: true` = muss vorkommen.
  `verneinend: true`, wenn das Element selbst eine Verneinung enthält
  („darf kein Fahrzeug führen“), dann entwertet ein „nicht“ den Treffer nicht.
- **`treffer`**: Wörter oder Wortfolgen, die das Element ausdrücken.
  Gross-/Kleinschreibung, Umlaute und ß spielen keine Rolle.
  - `wort` – unter 4 Zeichen exakt, 4 Zeichen am Wortanfang, 5–6 Zeichen auch
    als Wortteil, ab 7 Zeichen als Wortteil mit Tippfehler-Toleranz
  - `=wort` ganzes Wort exakt · `^wort` Wortanfang · `~wort` ganzes Wort mit
    Toleranz · `#100-150` Zahl im Bereich
  - Wortfolgen: Wörter in dieser Reihenfolge, bis zu zwei Wörter dazwischen
  - Signalfolgen wie „kurz, lang, kurz“ werden trotz Kommas als Folge erkannt.
- **`mindestanzahl`**: so viele Elemente braucht „Richtig“ (zusätzlich zu allen
  Pflichtelementen). „Teilweise“ ab der Hälfte davon.

Nach einer Änderung in `sw.js` die Zeile `const BESTAND = "seemupr-v5";`
hochzählen (`v2`, `v3` …), damit installierte Apps die neue Fassung laden.
Nach einer Änderung an `index.html` ebenfalls.

## Bewertung und Wiederholung

| Ergebnis  | Bedingung                                           |
| --------- | --------------------------------------------------- |
| Richtig   | alle Pflichtelemente und mindestens `mindestanzahl` |
| Teilweise | mindestens die Hälfte von `mindestanzahl`           |
| Falsch    | weniger                                             |

Die Automatik erkennt Begriffe, keine Begründungen. Mit „Doch richtig“ lässt
sich die Bewertung überstimmen.

Falsch, teilweise, gelöst oder übersprungen: Die Frage wird frühestens nach
drei anderen Fragen wieder gestellt, höchstens zweimal. Danach bleibt sie
offen und erscheint im Ergebnis. „Fehler nochmals üben“ startet eine Runde nur
mit diesen Fragen.

Richtig beantwortet: Mit **„Nochmals fragen“** kommt die Frage nach drei bis
sechs anderen Fragen nochmals, so oft man möchte.

## Lernstand und Auswahl der Fragen

Die App speichert im Browser (`localStorage`, Schlüssel `seemupr.lernstand`)
für jede Frage, wie oft und in welcher Runde sie zuletzt gestellt wurde, und
die Punkte der letzten fünf **ersten** Versuche je Runde: richtig und „Doch
richtig“ 1, teilweise 0,5, falsch, gelöst und übersprungen 0.

Runden mit 10, 15 oder 20 Fragen wählen gewichtet aus:

- Noch nie gestellte Fragen kommen zuerst. Nach drei Runden à 10 Fragen war
  jede Frage einmal dran.
- Danach gilt: Priorität = Runden seit dem letzten Stellen × Gewicht. Das
  Gewicht reicht von 1 (zuletzt immer richtig) bis 4 (immer falsch). In der
  Praxis kommen schwache Fragen dadurch etwa dreimal so oft wie sichere.
- Keine Frage setzt länger aus als 3 × (30 ÷ Rundengrösse, aufgerundet)
  Runden: bei 10 Fragen 9 Runden, bei 15 und 20 Fragen 6 Runden.

„Alle 30“ und „Fehler nochmals üben“ wählen wie bisher, ihre Ergebnisse
zählen aber für den Lernstand.

Das Balkensymbol oben rechts öffnet die Ansicht **Fortschritt**: eine Kachel
je Frage (grün sicher ab 80 %, gelb teilweise ab 40 %, rot darunter, grau
neu). Darunter lässt sich der Lernstand als Datei ausgeben, wieder einlesen
oder nach einer Rückfrage löschen. Auf der Startseite steht eine Kurzübersicht.

Stellschrauben im Programm: `VERLAUF_MAX` (5), `FEHLER_FAKTOR` (3),
`maxPause()`.

## Datengrundlage

Eigene Sammlung von Musterprüfungsfragen zum Schifffahrtsrecht. Fragen und
Musterlösungen sind sprachlich vereinheitlicht; zusammengehörige Teilfragen
(Unterelbe im Nebel, Ankerlieger, Grundsitzer 13,5 m) stehen als eine Frage.
Frage 27 enthält eine Anmerkung zur Abgrenzung von KVR-Regel 30 b und
SeeSchStrO Anlage II Nr. 11.

Vor der Prüfung mit den aktuellen Fassungen von KVR und SeeSchStrO abgleichen.

## Datenschutz

Keine Übertragung, kein Tracking. Gespeichert wird nur die zuletzt gewählte
Rundengrösse im `localStorage` des Browsers.
