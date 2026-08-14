# Certia, Projektgedächtnis

Diese Datei liest Claude Code bei jeder Sitzung. Sie enthält die
Festlegungen, die sonst jedes Mal neu erklärt werden müssten.

## Was das hier ist

Certia ist ein Managementsystem für ISO 9001 und ISO 27001, gedacht für
Schweizer KMU von rund zehn bis zweihundertfünfzig Personen. Es ist ein
Produkt der n'cloud.swiss AG.

Der Nutzen in einem Satz: Certia liest bestehende Dokumente ein, erkennt
welche Normanforderung sie belegen, zeigt die Lücken und gibt am Ende ein
prüffähiges Nachweispaket aus.

**Abgrenzung.** Certia ist ausdrücklich keine GRC-Plattform. Kein
quantitatives Risikomanagement, keine Kontrolltests über mehrere Regelwerke,
kein Vertragsmanagement, keine Konzernstrukturen. Wer das braucht, ist bei
Swiss GRC richtig. Diese Grenze hält das Produkt einfach genug für die eine
Person im KMU, die den Zertifizierungshut trägt. Feature-Vorschläge, die
diese Grenze überschreiten, bitte hinterfragen statt umsetzen.

## Aktueller Stand

`index.html` ist ein vollständiger, lauffähiger Prototyp in einer einzigen
Datei, ohne Build. Er zeigt Register, Dokumentenimport mit Texterkennung,
Volltextsuche, Normbezug, interne Audits, Massnahmen, Schulungen,
Audit-Cockpit und die Assistentin Nora. Ablage ist IndexedDB im Browser.

Der Prototyp ist Referenz, nicht Wegwerfware. Die Erkennungsregeln, die
Abläufe und die Gestaltung werden in die Produktivversion übernommen.

Der Bauplan für die Produktivversion liegt in `docs/Bauplan.md`.

## Sprache und Schreibweise

Alle Oberflächentexte, Kommentare und Dokumente auf Deutsch, in Schweizer
Schreibweise.

- Kein Eszett, immer `ss`. Also Massnahme, Grösse, Schliessen.
- Keine langen Gedankenstriche, weder Halbgeviert noch Geviert. Stattdessen
  Komma, Punkt oder Doppelpunkt.
- Umlaute korrekt, niemals `ae`, `oe`, `ue` als Ersatz. Das gilt auch in
  Bezeichnern, Prompts und Testdaten.
- Tausendertrennzeichen ist der Apostroph, also `1'234`.
- Datum im Format `14.08.2026`, in der Datenbank ISO `2026-08-14`.
- Anrede in der Oberfläche: Sie.
- Aktive Verben auf Knöpfen, die Aktion benennen. Also "Freigabe erteilen",
  nicht "Absenden".

## Fachliche Regeln, die nicht verhandelbar sind

1. **Normwortlaut ist urheberrechtlich geschützt.** Certia speichert
   Kapitelnummern und eigene Kurzbezeichnungen. Niemals den Originaltext der
   ISO-Normen einbetten, weder in Datenbank noch in Prompts.
2. **Der Prüfpfad wird nur angefügt.** Kein Ändern, kein Löschen. Jede
   Änderung an Dokumenten, Freigaben, Rechten und Massnahmen wird
   protokolliert mit Person, Zeitpunkt, Vorher und Nachher.
3. **Jede Abfrage filtert auf die Mandanten-ID.** Ohne Ausnahme.
4. **Freigabe ist ein Vorgang, kein Feld.** Entwurf, Prüfung, Freigabe,
   Inkraftsetzung, mit benannten Personen und Zeitstempel. Das ist die
   Kernanforderung aus ISO 9001 Kapitel 7.5.
5. **Nichts wird sofort endgültig gelöscht.** Papierkorb mit Frist.

## Gestaltung

- Marke: Wortmarke Certia, Bildmarke ist das C-Sechseck in `logo.svg`.
  Immer mit dem Zusatz "Ein Produkt von n'cloud.swiss AG".
- Farben: Akzent Blau `#1F6FCC`, helle Stufe Cyan `#3FD8FC`, dunkle Stufe
  `#175BAA`, dunkle Flächen Navy `#14193F`, Hintergrund `#F2F5FA`, Text
  `#131A33`, Linien `#E3E9F2`.
- Violett ist raus. Hintergründe bleiben hell und neutral, Blau ist Akzent.
- Schriften: Sora für Titel, Plus Jakarta Sans für Text, JetBrains Mono für
  Dokumentnummern, Versionen und Kapitel.
- Status: freigegeben grün, in Prüfung gelb, überfällig rot, Entwurf grau.
- Zurückhaltung bei Animation. Nichts bewegt sich von selbst, alles reagiert
  auf eine Handlung der Person.
- Mobil muss funktionieren. Kein horizontaler Überlauf, Spalten stapeln sich.

## Arbeitsweise

- Ein Arbeitspaket umfasst einen Bereich vollständig: Datenmodell,
  Schnittstelle, Oberfläche und Test zusammen. Nicht nach Schichten trennen.
- Tests sind bei Freigabe, Versionierung, Rechten und Prüfpfad Pflicht, nicht
  Kür. Ein Werkzeug, das Nachweise führt, muss seine eigenen Nachweise
  beweisen können.
- Vor jeder Lieferung: mobil und Desktop ansehen, Konsole auf Fehler prüfen.
- Keine erfundenen Zahlen in Beispieldaten, die nach echten Kundendaten
  aussehen. Der Mustermandant heisst Alpine Präzision AG.

## Betrieb

Aktuell: statische Auslieferung über Cloudflare, Anmeldung ist eine
Demoschranke im Browser mit Benutzer `Matter`. Das ist kein Schutz. Für einen
geschützten Zugang gehört Cloudflare Access davor.

Produktiv: Container in der Schweizer Infrastruktur der n'cloud. Die
Anthropic-API läuft in den USA, deshalb müssen KI-Funktionen entweder über
eine europäische Region laufen oder pro Mandant abschaltbar sein. Siehe
Bauplan, Abschnitt 1.

## Was Menschen entscheiden, nicht Claude

Betriebsort, Region für die KI, ob Dokumente kopiert oder nur verwiesen
werden, Anmeldeverfahren, Normen und Sprachen zum Start, Preismodell,
Markenrecht. Diese Punkte nicht eigenmächtig festlegen, sondern nachfragen.
