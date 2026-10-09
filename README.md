# RPT — Reverse Pyramid Training

Minimalistische Offline-App fürs iPhone. Kein App Store, kein Xcode, kein Mac.
Eine HTML-Datei plus Service Worker; alle Daten bleiben im Browser-Speicher des Geräts.

## Aufs iPhone bringen

1. Neues öffentliches GitHub-Repo anlegen, diese vier Dateien hochladen
   (`index.html`, `sw.js`, `manifest.webmanifest`, `icon-180.png`, `icon-512.png`).
2. Repo → Settings → Pages → Source: `main` / root. Nach ~1 Min. gibt es eine
   `https://<name>.github.io/<repo>/`-Adresse.
3. Auf dem iPhone in **Safari** öffnen (nicht Chrome), Teilen-Symbol →
   **Zum Home-Bildschirm**.
4. Einmal mit Netz starten, damit der Service Worker cached. Danach läuft alles offline.

HTTPS ist Pflicht, sonst registriert Safari keinen Service Worker — deshalb GitHub
Pages und nicht die Dateien-App.

## Was lokal heißt — und was nicht

Der Service Worker cached die App selbst, sie startet also dauerhaft ohne Netz.
Die **Daten** sind eine andere Frage: sie liegen in `localStorage`, und iOS
garantiert dafür keine Dauerhaftigkeit.

- **Sieben Tage Inaktivität.** Safari löscht scriptseitig geschriebenen Speicher
  nach sieben Tagen ohne Interaktion. Web-Apps auf dem Home-Bildschirm laufen mit
  einem eigenen Zähler, aber die Mechanik existiert. Bei drei Einheiten pro Woche
  spielt das keine Rolle — bei längeren Pausen schon.
- **Getrennte Speicher.** Die App im Home-Bildschirm und dieselbe Seite in Safari
  teilen sich den Speicher nicht. Immer über das Symbol öffnen, nie über Safari.
- **Symbol löschen löscht die Daten.** Ebenso „Verlauf und Websitedaten löschen"
  in den Einstellungen und Aufräumaktionen bei knappem Speicher.
- **Kein iCloud-Sync.** Handy weg heißt Daten weg.

Die App ruft beim Start `navigator.storage.persist()` auf und bittet iOS damit um
dauerhaften Speicher. Das ist eine Bitte, keine Garantie.

### Sicherung

Eine Web-App kann nicht still in die iCloud schreiben, dafür gibt es keine
Schnittstelle. Es bleibt ein Download, und der unterbricht. Deshalb passiert er
nicht mehr automatisch nach jeder Einheit.

Stattdessen zählt die App mit. Nach drei ungesicherten Einheiten erscheint oben auf
der Startseite eine Zeile mit einem Knopf. Die Zahl ist in den Einstellungen
änderbar.

Der Knopf ist ein echtes `<a download>` und kein im Code erzeugter Klick. Dadurch
ist der Tipp eine direkte Nutzeraktion auf einem Link, iOS fragt inline nach und die
App bleibt offen.

Einmal einrichten: Einstellungen > Apps > Safari > Downloads > **iCloud Drive**.
Ab Werk steht das schon so, Zielordner ist dann „Downloads" in iCloud Drive.

„Backup einlesen" spielt eine dieser Dateien zurück, inklusive Gewichten und
Habit-Raster. Die Dateien sind wenige Kilobyte groß; einmal im Jahr die alten
löschen reicht.

## Optik

Schwarz, Weiß und Graustufen, Poppins als Schrift, übernommen von
distinctplaces.de. Die drei Schriftschnitte stecken als base64 in der `index.html`,
zusammen rund 23 KB, damit die App ohne Netz gleich aussieht. Es gibt keine
Akzentfarbe: aktiv, erledigt und fertig unterscheiden sich über Rahmen, Helligkeit
und Umkehrung.

## Die Logik

Alles nach dem Leangains-Guide, abhängiges System:

- Satz 1 ist der Anker. Satz *n* = Satz 1 × (1 − Breakdown × (n−1)).
- Breakdown: 5 % für Bank, Overhead Press, Seal Row. 10 % für Kreuzheben, Squat,
  Rudern, Klimmzüge und Isolationsübungen.
- Doppelte Progression: mehr Wiederholungen als das Ziel in Satz 1 → beim nächsten
  Mal ein Schritt mehr Gewicht. Sätze 2 und 3 ziehen automatisch mit.
- Jeder Satz ist AMRAP. Die angezeigten Ziele sind Orientierung, keine Obergrenze.
- Bei Klimmzügen rechnet die App mit Körpergewicht + Zusatz, weil sich der
  Breakdown auf die Gesamtlast beziehen muss. Solange kein Zusatzgewicht
  eingetragen ist, gibt es keinen Breakdown — alle Sätze laufen mit Körpergewicht,
  und ab 10 sauberen Wiederholungen im ersten Satz schaltet die App auf +5 kg um.
  Unter 6 Wiederholungen empfiehlt der Guide erst Latzug oder Band.

## Habit Tracker

Auf der Startseite, ohne Zutun: jede abgeschlossene Einheit setzt automatisch einen
Punkt im 16-Wochen-Raster (eine Spalte je Woche, Montag oben).

Die Serie zählt **Wochen, nicht Tage**. Bei drei Einheiten pro Woche wäre eine
Tagesserie das falsche Maß — hier zählt eine Woche ab drei Einheiten. Die laufende
Woche geht erst in die Serie ein, wenn sie voll ist, bricht sie aber auch nicht ab,
solange sie läuft.

## Startseite

Jede Tageskarte listet ihre Übungen mit dem Gewicht, das dort aktuell als Satz 1
anläge, gerundet auf das eingestellte Raster. Rechts oben steht, wann der Tag zuletzt
dran war, bei einer laufenden Einheit stattdessen der Stand in Sätzen.

Gerechnet wird ohne Deload, auch wenn gerade eine Deload-Einheit läuft: die
Übersicht zeigt den Stand, nicht den Tag.

## Freiwilliger Block

Unter den drei Trainingstagen steht eine Karte mit den Übungen für die
trainingsfreien Tage. Ein Tipp auf die Karte setzt den Haken für heute, ein zweiter
nimmt ihn zurück.

Diese Tage zählen **nicht** in die Wochenserie und nicht in die Gesamtzahl der
Einheiten. Es ist kein Pflichttraining, nur ein Haken für die Übersicht. Im Raster
erscheinen sie grau, Trainingstage weiß. Fällt beides auf denselben Tag, gewinnt das
Training.

Name und Übungsliste sind in den Einstellungen änderbar, eine Übung je Zeile.

## Satzzeilen lesen

Der schmale Streifen am unteren Rand jeder Satzzeile zeigt die Last im Verhältnis zu
Satz 1, exakt proportional: 100 %, dann 95 und 90 bei 5 % Breakdown. Er sagt nichts
über den Fortschritt innerhalb der Einheit.

Rechts steht, was zu erreichen ist. Satz 1 zeigt das Ziel aus dem Programm, darunter
das Ergebnis vom letzten Mal. Sätze 2 und 3 haben programmseitig kein Ziel, dort
steht das letzte Ergebnis als Marke und darunter der Hinweis AMRAP. Hat sich das
Gewicht seitdem geändert, wird die alte Last mitgenannt, damit die Zahl vergleichbar
bleibt.

Nach dem Eintragen steht dort die Differenz zum letzten Mal, aber nur wenn das
Gewicht identisch war.

Ob ein Satz erledigt ist, steht am Rahmen, am Häkchen und an der grünen
Wiederholungszahl. Beides lag früher in derselben Fläche, wodurch ein erledigter
leichterer Satz halb gefüllt aussah.

## Laufende Einheit und Pausenuhr

Beides liegt im gespeicherten Zustand, nicht in der Ansicht. Die Uhr klebt oben am
Bildschirm und bleibt sichtbar, während du durch Verlauf oder Einstellungen
blätterst. Gespeichert wird der Endzeitpunkt, nicht die Restzeit: du kannst die App
verlassen, WhatsApp beantworten und zurückkommen, die Uhr steht auf dem richtigen
Stand. Auch ein kompletter Neustart der App ändert daran nichts.

Angefangene Sätze bleiben ebenso erhalten. Auf der Startseite ist der laufende
Trainingstag orange markiert.

Solange die Pause läuft, hält die App über die Wake-Lock-Schnittstelle den
Bildschirm an, sofern die App im Vordergrund ist. Nach Ablauf, beim Wegtippen und
beim Abschluss der Einheit wird die Sperre wieder freigegeben. iOS entzieht sie
beim Wegschalten von selbst, deshalb fordert die App sie bei Rückkehr neu an.

Eine Anzeige in der Dynamic Island ist nicht möglich. Die läuft über ActivityKit,
ein natives Framework mit eigener Widget-Extension in Xcode, und hat keine
Web-Schnittstelle.

Am Pausenende kommen zwei kurze Töne. Der Ton steckt als WAV direkt in der
`index.html`, es wird also nichts nachgeladen und er funktioniert offline.
Abgespielt wird er über ein `<audio>`-Element und nicht über die Web Audio API,
weil iOS letztere bei aktivem Stummschalter blockiert. Der Session-Typ steht auf
`transient`, dem Typ für Benachrichtigungstöne: der Ton legt sich über laufende
Musik und senkt sie höchstens kurz ab. `playback` wäre ein exklusiver Typ und hält
Spotify an. Voraussetzung bleibt, dass die App im
Vordergrund ist; im Hintergrund lässt iOS eine Web-App nicht tönen. Abschalten
lässt sich der Ton in den Einstellungen.

## Fortschritt

Unter „Fortschritt" je Übung auswählbar. Die obere Kurve zeigt das Arbeitsgewicht von
Satz 1, die untere das nach Epley geschätzte 1RM aus Gewicht und Wiederholungen von
Satz 1. Letzteres steigt auch, wenn das Gewicht gleich bleibt und die Wiederholungen
zunehmen, macht also Fortschritt sichtbar, den die reine Gewichtskurve verschluckt.
Deload-Einheiten sind aus beiden Kurven ausgenommen.

Als SVG direkt im Code gezeichnet, keine Bibliothek.

## Daten korrigieren

Während einer Einheit: auf eine bereits gespeicherte Satzzeile tippen. Dort lassen
sich **Gewicht und Wiederholungen** ändern. Bei Satz 1 verschiebt sich damit der
Anker, die folgenden Sätze rechnen sich neu. Bei den übrigen Sätzen gilt die
Änderung nur für diesen einen Satz. Die Pausenuhr läuft dabei weiter.

Nachträglich: Verlauf öffnen, Einheit antippen, Satz antippen. Korrigierst du dort
Satz 1 der zuletzt absolvierten Einheit einer Übung, rechnet die App die Progression
neu und passt das aktuelle Arbeitsgewicht an. Bei älteren Einheiten wird nur der
Eintrag berichtigt, weil spätere Einheiten die Steigerung bereits fortgeschrieben
haben. Ganze Einheiten lassen sich dort ebenfalls löschen.

## Updates ohne Datenverlust

`localStorage` hängt an der Adresse, nicht an der Datei. Neue Fassungen von
`index.html` oder `sw.js` hochzuladen lässt Verlauf und Gewichte unangetastet.

Zwei Dinge löschen den Stand trotzdem:

- Die Zeile `v: 2` in den `DEFAULTS` hochzuzählen. Das ist als Notausgang gedacht
  und verwirft absichtlich alles.
- Das Repository umzubenennen oder auf eine andere Adresse umzuziehen.

## Abgleich mit dem Guide

- Abhängiges System: alle Sätze leiten sich von Satz 1 ab, keine unabhängige
  Progression je Satz.
- Doppelte Progression: mehr Wiederholungen als das Ziel in Satz 1 erhöht die Last
  in der nächsten Einheit.
- Breakdowns 5 % für Bank, Overhead Press und Seal Row, 10 % für Kreuzheben, Squat,
  Rudern, Klimmzüge und Isolationsübungen.
- Nur Satz 1 hat ein Ziel. Sätze 2 und 3 sind reine AMRAP-Sätze und werden auch so
  angezeigt. Der Vorschlagswert stammt aus der letzten Einheit, nicht aus
  "vorheriger Satz plus eins".
- Startlast: die App warnt, wenn Satz 1 unter Ziel−1 liegt oder deutlich darüber.
- Aufwärmen: 2–5 Sätze à 3–6 Wdh. von 40 bis 67 % der ersten Arbeitslast, mit
  konkreten Zahlen für die erste Übung des Tages.
- Klimmzüge: Körpergewicht bis 10 saubere Wiederholungen, dann +5 kg. Unter 6
  Wiederholungen Hinweis auf Latzug oder Band.
- Satzpausen mindestens 3 Minuten, bei Kreuzheben und Squat mehr; Isolation 90
  Sekunden für abwechselndes Arbeiten.
- Deload-Tage statt Deload-Wochen: Schalter oben am Trainingstag, senkt alle Lasten
  um 15 % und nimmt die Einheit aus der Progression heraus.

## Ausrüstung

Ein Profil, kein Umschalten: Stangengewicht, kleinste Scheibe je Seite, Schritt an
Maschinen und der Scheibensatz. Steht in den Einstellungen.

Einzutragen ist das **gröbste** Raster, das in einem der genutzten Studios vorkommt.
Dann lässt sich jedes Gewicht überall auflegen und es gibt nie eine Last, die vor
Ort nicht geht. Satz 1 wird auf dieses Raster abgerundet, nie aufgerundet.

Übungen an Maschinen oder Kurzhanteln, die anders abgestuft sind, bekommen im
Übungsmenü einen eigenen Schritt. Leer heißt Standard.

Ist der Schritt grob, kann der Breakdown einebnen: bei 15 kg je Seite und 5 % fallen
Satz 1 und 2 beide auf 15 kg, weil die Differenz unter 2,5 kg liegt. Die App weist an
der betroffenen Übung darauf hin. Gleiches Gewicht heißt nicht gleicher Satz, beide
bleiben AMRAP.

## Anpassen

Zahnrad neben jeder Übung: Gewicht, Ziel, Sätze, Breakdown, Ladeart
(je Seite / gesamt / Körpergewicht + Zusatz), Stangengewicht, kleinste Scheibe,
Steigerungsschritt, Satzpause.

Alle Übungen starten ohne Gewicht. Beim ersten Öffnen fragt jede danach: nimm eine
Last, mit der du im ersten Satz das Ziel oder eine Wiederholung darunter schaffst.
Ab da rechnet die App weiter.

Der Startzustand steht als `DEFAULTS` oben im `<script>`-Block von `index.html`.
Wer den Plan grundsätzlich ändern will, ändert ihn dort und zählt `v` eine Nummer
hoch — dann verwirft die App den alten Stand aus `localStorage` beim nächsten Start.
