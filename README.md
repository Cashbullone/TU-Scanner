# TU-Screener (Turtle-Strategie)

Eigenständiger Screener, der nur die Regeln der Turtle-Händler (Richard Dennis, William Eckhardt) auf US-Aktien anwendet.
GitHub holt jeden Börsentag nach US-Börsenschluss die Kurse von **S&P 500 und Nasdaq-100**, rechnet die Turtle-Kennzahlen
und veröffentlicht sie zusammen mit der App als Webseite. Du öffnest nur den Link.

Was die App zeigt:

- **Orderzettel für morgen:** die Stop-Orders, die du heute Abend setzt (Verkaufs-Stops, Zukäufe, neue Einstiege) mit Marke und Stückzahl.
- **Depot:** deine Positionen, nachgespielt Tag für Tag. Die App meldet Stop und Ausstieg, die nächste Zukauf-Marke und das Risiko bis zum Stop.
- **Neue Einstiege:** Aktien, die kurz vor dem Ausbruch über das 20- oder 55-Tage-Hoch stehen, mit Limits (je Aktie, Sektor, Richtung).
- **Heute ausgelöste Ausbrüche:** was am letzten Handelstag ausgelöst wurde, samt Skip-Regel von System 1.

## Die Regeln in Kürze

| Baustein | Regel |
| --- | --- |
| N | Wilder-Durchschnitt der True Range über 20 Tage |
| Einheit | Stück = Depot × Risiko ÷ N (Original: 1 % Risiko, hier Standard 0,25 %, siehe unten) |
| System 1 | Einstieg über dem 20-Tage-Hoch, Ausstieg unter dem 10-Tage-Tief. Nach einem Gewinner-Trade wird der nächste Ausbruch übersprungen |
| System 2 | Einstieg über dem 55-Tage-Hoch, Ausstieg unter dem 20-Tage-Tief, ohne Überspringen |
| Stop | 2 N unter dem letzten Kaufkurs |
| Zukauf | alle ½ N über dem letzten Kaufkurs, bis zu 4 Einheiten je Aktie |
| Limits | 4 Einheiten je Markt, 6 je Gruppe verwandter Märkte (hier: Sektor), 12 je Richtung |

Die Rechenwege, alle Annahmen und die nicht bestätigten Punkte stehen in der App unter „So rechnet die App“.

**Wichtig für Aktien:** Die Turtles handelten gehebelte Futures. Mit 1 % Risiko je Einheit wäre bei einem N von 2 % des Kurses
eine Einheit schon 50 % des Depots. Die App startet deshalb mit 0,25 % Risiko, höchstens 25 % des Depots je Aktie und höchstens
100 % Investitionsgrad. Alles ist unter „Einstellungen“ änderbar.

## Einrichtung (einmalig, ca. 5 Minuten)

Die Dateien liegen schon im Repository. Du musst nur noch die Veröffentlichung einschalten und den ersten Lauf starten:

1. **Veröffentlichung einschalten:** im Repository „Settings“ → „Pages“ → bei „Source“ **GitHub Actions** wählen.
2. **Ersten Lauf starten:** Reiter „Actions“ → „Kurse aktualisieren und Seite veröffentlichen“ → „Run workflow“.
   Der erste Lauf dauert etwa 10 bis 25 Minuten. Danach läuft die Automatik Montag bis Freitag von selbst
   (22:30 UTC, also 23:30 oder 00:30 Uhr österreichischer Zeit).
3. **App öffnen:** `https://cashbullone.github.io/TU-Scanner/`. Auf dem Handy kannst du die Seite
   über das Teilen-Menü zum Home-Bildschirm hinzufügen.

## Was wo gespeichert wird

- Im Repository liegen nur App, Skript und Tests. Die Kursdaten entstehen bei jedem Lauf neu und gehen direkt auf die Webseite.
- Dein Depot und deine Einstellungen bleiben **nur in deinem Browser**. Auf einem zweiten Gerät übertragst du sie mit
  „Depot exportieren“ und „Depot importieren“.
- Die Webseite ist öffentlich erreichbar, wer den Link kennt, sieht die Kurse (aber nicht dein Depot).
  Die Kursdaten stammen von Yahoo Finance und sind für den privaten Gebrauch gedacht. Gib den Link deshalb nicht weiter.

## Einstellungen der Automatik

In `update.yml` unter „Kursdaten holen“ kannst du die auskommentierten Zeilen aktivieren:

- `MIN_PRICE` Mindestkurs in USD (Standard 5)
- `MIN_AVG_VOL` Mindest-Durchschnittsvolumen der letzten 50 Tage in Aktien (Standard 100000)
- `INDEXES` Standard ist `SP500,NDX`. Mit `SP500,NDX,R2000` kommt der Russell 2000 dazu (der Abruf dauert dann deutlich länger)

Den Sektor liest das Skript aus den Indexlisten. Fehlt er, ergänzt es ihn aus `universe/sectors.csv` und sonst von Yahoo
(Zeile in der Datei: `TICKER,Sektor`).

## Was das Skript an die App liefert

Für jede Aktie: N, die Marken der letzten 10, 20 und 55 Tage, die heute ausgelösten Ausbrüche, den Zustand der Skip-Regel
für Long und Short (aus einer Nachspielung der letzten rund zwei Jahre) und die Kursreihe der letzten 140 Handelstage
(Hoch, Tief, Schluss, N) für die Depotprüfung.

## Tests

`python tests/test_core.py` prüft N, Ausbruchsmarken, Stops, Lücken, die Skip-Regel und die Spiegelung für Short mit handgebauten
Kursverläufen. `python tests/test_assemble.py` prüft Aufbereitung und Hauptablauf mit künstlichen Daten ohne Netzwerk.
`python tests/make_fake.py ORDNER` erzeugt eine künstliche `prices.json` zum Ausprobieren der App.

## Wenn etwas nicht klappt

- **Lauf schlägt fehl (roter Haken unter „Actions“):** den Lauf öffnen und die Meldung bei „Kursdaten holen“ ansehen.
  Der Lauf bricht absichtlich ab, wenn weniger als 300 Aktien herauskommen. Die Seite zeigt dann weiter die Daten vom letzten
  erfolgreichen Lauf.
- **Yahoo blockiert Abrufe:** passiert gelegentlich. Einfach „Run workflow“ erneut starten.
- **Nach 60 Tagen ohne Änderung im Repository** pausiert GitHub geplante Läufe. Dann unter „Actions“ wieder aktivieren.

Das ist ein Regelwerk aus der Literatur und keine Anlageberatung. Die App macht keinen Backtest und sagt nichts darüber,
ob die Regeln heute bei Aktien noch funktionieren.
