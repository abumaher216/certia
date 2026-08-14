# Certia, Weg zur Produktivversion

Stand 14. August 2026. Grundlage ist der lauffähige Prototyp, der Import,
Volltextsuche, Normabdeckung, interne Audits, Massnahmen, Schulungen und das
Nachweispaket bereits zeigt. Dieses Dokument beschreibt, was zwischen dem
Prototyp und einer Version steht, die ein KMU produktiv nutzen und die n'cloud
gegen Lizenz verkaufen kann.

---

## 1. Entscheidungen, die vor der ersten Codezeile fallen müssen

Diese sechs Punkte bestimmen fast alles Weitere. Solange sie offen sind,
lohnt sich kein Bau.

**1.1 Wo läuft Certia?**
Der Prototyp liegt auf Cloudflare Pages. Für die Vorführung ist das ideal, für
den Betrieb nicht, denn das Werbeversprechen lautet Betrieb in der Schweiz.
Cloudflare verteilt Inhalte weltweit. Produktiv gehört Certia in die eigene
Infrastruktur der n'cloud, als Container in einem Schweizer Rechenzentrum.
Cloudflare kann davor bleiben als DNS, WAF und Zugangsschutz.

**1.2 Werden Dokumente kopiert oder nur verwiesen?**
Das ist die wichtigste Produktentscheidung. Zwei Betriebsarten sind sinnvoll
und schliessen sich nicht aus:

- *Verweis*: Die Datei bleibt auf SharePoint, OneDrive oder dem Fileserver.
  Certia speichert Pfad, Prüfsumme, Angaben und den extrahierten Text. Das ist
  das Fraktala-Versprechen, nur mit Nachweisführung. Vorteil: keine zweite
  Wahrheit, kein Datenschutzproblem. Nachteil: Certia hat keine Kontrolle über
  Versionen, die jemand direkt im Laufwerk überschreibt.
- *Ablage in Certia*: Die Datei wird hochgeladen und liegt in der Objektablage.
  Nötig für lückenlose Versionierung und echte Freigabe.

Empfehlung: beides anbieten, pro Ablage konfigurierbar, Standard ist Verweis.

**1.3 Wo läuft die KI?**
Die Anthropic-API läuft in den USA. Das verträgt sich nicht ohne Weiteres mit
der Zusage, dass keine Daten die Schweiz verlassen. Drei Wege:

- Claude über AWS Bedrock oder Google Vertex in einer europäischen Region
  betreiben und die Regionsverfügbarkeit vorher prüfen.
- KI-Funktionen pro Mandant abschaltbar machen. Ohne KI bleibt Certia voll
  nutzbar, die Erkennung läuft dann regelbasiert, so wie im Prototyp im
  Offline-Fall.
- Direkte Nutzung der Anthropic-API mit ausdrücklicher Einwilligung, sauberem
  Auftragsverarbeitungsvertrag und Eintrag im Bearbeitungsverzeichnis.

Diese Frage muss beantwortet sein, bevor Nora gebaut wird, weil sie das
Marketing bindet.

**1.4 Wie melden sich Menschen an?**
KMU in der Schweiz arbeiten überwiegend mit Microsoft 365. Anmeldung über
OpenID Connect gegen Entra ID sollte der Normalfall sein, lokale Konten nur
als Rückfallebene. Damit erbt Certia auch die Mehrfaktorauthentisierung.

**1.5 Welche Normen zum Start?**
ISO 9001 und ISO 27001 sind gesetzt. ISO 14001 und ISO 45001 kommen häufig
dazu, ISO 9001:2026 ohnehin. Jede weitere Norm ist Datenpflege, kein Code,
wenn das Modell von Anfang an mehrnormfähig ist.

Wichtig und oft übersehen: **Normtexte sind urheberrechtlich geschützt.** Die
Kapitelnummern und eigene Umschreibungen darf Certia führen, den Wortlaut der
ISO nicht mitliefern. Das muss im Datenmodell und in der Redaktion konsequent
durchgehalten werden.

**1.6 Welche Sprachen?**
Deutsch zum Start. Französisch ist für den Schweizer Markt fast Pflicht,
Italienisch und Englisch je nach Zielkunden. Mehrsprachigkeit nachträglich
einzubauen ist teuer, die Vorbereitung im Code kostet fast nichts.

---

## 2. Zielarchitektur

Ein Vorschlag, der zu einem IT-Dienstleister mit eigener Cloud passt und den
Claude durchgängig bauen kann.

| Baustein | Wahl | Begründung |
|---|---|---|
| Sprache | TypeScript durchgängig | Ein Ökosystem, ein Typmodell, Claude arbeitet darin sehr sicher |
| Frontend | React mit Vite, TanStack Query und Router | Der Prototyp ist bereits komponentenartig gedacht, die Übertragung ist geradlinig |
| Backend | Fastify mit tRPC oder OpenAPI | Schlank, schnell, gut testbar |
| Datenbank | PostgreSQL 16 mit Prisma | Volltextsuche und Vektorsuche im gleichen System |
| Suche | tsvector mit deutscher Konfiguration, pg_trgm, pgvector | Stichwort und Sinnverwandtschaft ohne zweite Infrastruktur |
| Objektablage | MinIO, S3-kompatibel | Läuft im eigenen Rechenzentrum |
| Textauszug | Eigener Dienst in Python mit PyMuPDF, python-docx, OCRmyPDF | Die reifsten Werkzeuge liegen in Python |
| Hintergrundarbeit | BullMQ mit Redis | Import, OCR, Fristen, Erinnerungen |
| Betrieb | Docker Compose zum Start, Kubernetes wenn es mehrere Mandanten werden | Nicht überdimensionieren |
| Beobachtung | strukturierte Logs, Prometheus, GlitchTip | Selbst betreibbar, keine Daten nach aussen |

Die Trennung in vier Container ist bewusst: Web, API, Extraktion, Arbeiter.
Die Extraktion ist der einzige Teil, der viel Rechenzeit braucht, und lässt
sich getrennt skalieren.

---

## 3. Datenmodell, die tragenden Tabellen

```
mandanten            id, name, einstellungen, lizenz_bis
benutzer             id, mandant_id, name, email, rolle, idp_subject
ablagen              id, mandant_id, typ (sharepoint|onedrive|smb|certia), konfig, status
register_knoten      id, mandant_id, eltern_id, art (phase|bereich|taetigkeit), name, reihenfolge
dokumente            id, mandant_id, nummer, titel, typ, knoten_id, ablage_id,
                     aktuelle_version_id, naechste_pruefung, status
versionen            id, dokument_id, nummer, datei_id, text, seiten, pruefsumme,
                     aenderungsnotiz, erstellt_von, erstellt_am
freigaben            id, version_id, rolle (pruefer|freigeber), person_id,
                     entscheid, kommentar, zeitpunkt
dateien              id, ablage_id, pfad_oder_key, groesse, mimetyp, pruefsumme
lesebestaetigungen   id, version_id, person_id, zeitpunkt, erinnert_am
normen               id, kuerzel, ausgabe, sprache
klauseln             id, norm_id, nummer, kurzbezeichnung   (kein Normwortlaut)
belege               id, klausel_id, dokument_id, guete (voll|teilweise), notiz
audits               id, mandant_id, nummer, titel, norm_id, bereich, termin, auditor, status
feststellungen       id, audit_id, klausel_id, art, text, massnahme_id
massnahmen           id, mandant_id, nummer, titel, ursache, auslöser_typ, auslöser_id,
                     verantwortlich, frist, status, wirksamkeit_geprueft_am
kurse                id, mandant_id, name, klausel_id, gueltigkeit_monate
schulungsnachweise   id, kurs_id, person_id, datum, nachweis_datei_id
pflichtmatrix        id, kurs_id, funktion_oder_person
pruefpfad            id, mandant_id, zeitpunkt, person_id, objekt, objekt_id,
                     aktion, vorher, nachher            (nur anfügen, nie ändern)
auftraege            id, art, nutzlast, status, versuche, fehler
```

Zwei Regeln, die nicht verhandelbar sind: Jede Tabelle trägt die `mandant_id`
und jede Abfrage filtert darauf. Der `pruefpfad` wird ausschliesslich
angefügt, niemals geändert oder gelöscht, mit Datenbankrechten abgesichert.

---

## 4. Was funktional noch fehlt

Sortiert nach Wichtigkeit für ein bestandenes Audit.

**Muss vor dem ersten Kunden da sein**

1. **Freigabeworkflow.** Entwurf, Prüfung, Freigabe, Inkraftsetzung, mit
   benannten Personen, Kommentar, Zeitstempel und E-Mail-Benachrichtigung.
   Das ist die Kernanforderung aus ISO 9001 Kapitel 7.5 und heute nur
   angedeutet.
2. **Lesebestätigung als Vorgang.** Verteilung an einen Personenkreis,
   Erinnerung nach Frist, Auswertung, wer noch fehlt.
3. **Fristen und Wiedervorlage.** Nächtlicher Lauf, der überfällige Prüfungen,
   Massnahmen und Schulungen erkennt und Erinnerungen verschickt.
4. **Revisionssicherer Prüfpfad.** Jede Änderung nachvollziehbar. Ohne das
   fällt jedes Audit auf das Werkzeug selbst zurück.
5. **OCR für gescannte Dokumente.** In der Praxis liegt ein spürbarer Teil des
   Altbestands als Bildscan vor.
6. **Papierkorb und Wiederherstellung.** Nichts wird sofort endgültig gelöscht.
7. **Rechte je Bereich.** Nicht jede Person darf jedes Dokument sehen, gerade
   im Personalbereich.
8. **Nachweispaket als PDF.** Heute HTML, für die Übergabe an eine
   Zertifizierungsstelle gehört ein PDF mit Deckblatt und Inhaltsverzeichnis
   hin.

**Sollte bald folgen**

9. Anbindung Microsoft Graph mit Änderungserkennung, damit Certia merkt, wenn
   jemand die Datei direkt im Laufwerk austauscht.
10. Vorlagenbibliothek mit eigenen Musterdokumenten, klar getrennt von
    Normtexten.
11. Managementbewertung als geführter Ablauf statt als reines Dokument.
12. Risiken und Chancen als eigenes, schlankes Register, getrennt geführt wie
    es ISO 9001:2026 verlangt.
13. Kennzahlen mit Zeitreihe und Zielwert.
14. Mehrsprachigkeit der Oberfläche.
15. Onboarding-Assistent für den ersten Tag eines neuen Mandanten.

**Später, wenn Kunden es verlangen**

16. Lieferantenbewertung als eigener Bereich.
17. Kalenderanbindung für Audits und Termine.
18. Mobile Erfassung von Feststellungen direkt im Audit vor Ort.
19. Mandantenübergreifende Auswertung für Beratende, die mehrere KMU betreuen.

---

## 5. Was der Betrieb braucht

- **Sicherung und Wiederherstellung.** Nächtliche Datenbanksicherung,
  Objektablage gespiegelt, mindestens halbjährlicher dokumentierter
  Wiederherstellungstest. Genau das, was Certia von seinen Kunden verlangt.
- **Mandantentrennung** technisch belegt, nicht nur behauptet.
- **Verschlüsselung** im Transport und im Ruhezustand.
- **Datenschutz.** Bearbeitungsverzeichnis, Auftragsverarbeitungsvertrag als
  Vorlage für Kunden, technische und organisatorische Massnahmen beschrieben,
  Löschkonzept mit Fristen. Revidiertes DSG und DSGVO.
- **Certia im eigenen Managementsystem der n'cloud führen.** Wer ein
  ISO-Werkzeug verkauft, sollte selbst zertifiziert sein. Das ist zugleich das
  beste Verkaufsargument und der beste Test.
- **Wartungsfenster, Statusseite, Support-Weg** und ein Versprechen zur
  Verfügbarkeit, das ihr auch halten könnt.
- **Marken- und Namensprüfung** für Certia sowie Domainsicherung, bevor das
  Marketing anläuft.

---

## 6. Meilensteine

| Nr | Inhalt | Ergebnis |
|---|---|---|
| M0 | Entscheidungen aus Abschnitt 1, Repository, CLAUDE.md, CI, Container | Ein leeres, aber lauffähiges Gerüst |
| M1 | Datenmodell, Anmeldung, Mandanten, Rollen, Prüfpfad | Anmelden und Rechte greifen |
| M2 | Register, Dokumente, Versionen, Upload, Objektablage | Der Prototyp, aber serverseitig und mehrbenutzerfähig |
| M3 | Extraktion, OCR, Erkennung der Angaben, Volltext- und Vektorsuche | Import in Produktionsqualität |
| M4 | Freigabe, Lesebestätigung, Fristen, Benachrichtigungen | Kapitel 7.5 vollständig erfüllt |
| M5 | Normbezug, Audits, Massnahmen, Schulungen, Nachweispaket als PDF | Auditfähig |
| M6 | Anbindung SharePoint und Fileserver, Onboarding, Mehrsprachigkeit | Erster Fremdkunde möglich |

Interner Einsatz bei n'cloud ist ab M4 sinnvoll, der externe Verkauf ab M6.

---

## 7. Wie das mit Claude gebaut wird

**Werkzeug.** Claude Code im Repository, nicht dieses Chatfenster. Der
Unterschied ist gross: Claude Code liest und schreibt echte Dateien, führt
Tests aus, sieht Fehlermeldungen und kann in Schleifen arbeiten.

**Projektgedächtnis.** Eine `CLAUDE.md` im Wurzelverzeichnis mit den
Festlegungen, die sonst in jeder Sitzung neu erklärt werden müssen:
Architektur, Namenskonventionen, Sprachregeln wie kein Eszett, keine langen
Gedankenstriche, korrekte Umlaute, Schweizer Tausendertrennzeichen, sowie das
Verbot, Normwortlaut zu speichern.

**Hausstil als Skill.** Die Design-Vorgaben, die im Prototyp stecken, gehören
in einen Skill, damit jede neue Ansicht ohne Nachjustieren passt.

**Zuschnitt der Aufgaben.** Ein Arbeitspaket sollte einen Bereich vollständig
umfassen, also Datenmodell, Endpunkt, Oberfläche und Test zusammen. Das
funktioniert deutlich besser als getrennte Schichten, weil Claude dann das
ganze Bild hat.

**Tests sind hier nicht optional.** Bei einem Werkzeug, das Nachweise führt,
muss automatisiert geprüft sein, dass Freigaben, Versionen, Rechte und
Prüfpfad tun, was sie sollen. Claude schreibt diese Tests gerne mit, wenn man
es von Anfang an verlangt.

**Was Menschen tun müssen.** Die Entscheidungen aus Abschnitt 1, die Zugänge
zu Entra ID, Rechenzentrum und Registry, die Abnahme jedes Meilensteins, eine
unabhängige Sicherheitsprüfung vor dem ersten Fremdkunden, und alles
Rechtliche. Den Rest kann Claude bauen.

---

## 8. Was der Prototyp bereits belegt

Nicht neu bauen, sondern übernehmen: die Drei-Klick-Logik des Registers, die
Erkennungsregeln aus dem Import inklusive der Zeilenrekonstruktion aus PDF und
der satzweisen Normbezugserkennung, die Abdeckungsmatrix, den Ablauf des
internen Audits von der Checkliste zur Massnahme, die Kompetenzmatrix, den
Aufbau des Nachweispakets und die gesamte Gestaltung. Diese Arbeit ist getan
und getestet, sie muss nur in die neue Architektur wandern.
