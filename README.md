# Certia

Managementsystem für ISO 9001 und ISO 27001. Ein Produkt der n'cloud.swiss AG.

Dieser Stand ist ein lauffähiger Prototyp zum Vorführen, nicht die
Produktivversion. Der Weg dorthin steht in `docs/Bauplan.md`.

## Aufbau

| Datei | Zweck |
|---|---|
| `index.html` | Die vollständige Anwendung, eine Datei, kein Build |
| `favicon.svg` | App-Icon, dunkles Quadrat mit der Marke |
| `logo.svg` | Marke freigestellt |
| `_headers` | Sicherheitsheader für Cloudflare Pages |
| `_redirects` | Alle Pfade auf `index.html` |
| `beispiel-dokumente/` | Drei PDF zum Ausprobieren des Imports |
| `CLAUDE.md` | Projektgedächtnis für Claude Code |
| `docs/Bauplan.md` | Was die Produktivversion braucht |

## Anmeldung

Benutzer `Matter`, Passwort steht im Quelltext bei `ZUGANG`. Das ist eine
Demoschranke, kein Schutz. Für einen echten Zugang gehört Cloudflare Access
davor, siehe unten.

**Dieses Repository muss privat bleiben.**

## Veröffentlichen über Cloudflare Pages

Einmalig einrichten:

1. Im Cloudflare Dashboard auf **Workers & Pages**, dann **Create**,
   **Pages**, **Connect to Git**.
2. GitHub verbinden und dieses Repository auswählen.
3. Build-Einstellungen:
   - Framework preset: **None**
   - Build command: leer lassen
   - Build output directory: `/`
4. **Save and Deploy**.

Danach löst jeder Push auf `main` eine neue Veröffentlichung aus. Jeder
Branch und jeder Pull Request bekommt eine eigene Vorschauadresse. Frühere
Versionen lassen sich im Dashboard mit einem Klick zurückholen.

Wichtig: Ein Projekt, das über Direct Upload angelegt wurde, lässt sich
nachträglich nicht auf Git umstellen. Falls schon ein solches Projekt
besteht, ein neues anlegen.

## Zugang schützen

Cloudflare Access, kostenlos bis fünfzig Personen:

1. **Zero Trust**, **Access**, **Applications**, **Add an application**,
   **Self-hosted**.
2. Als Domain die Pages-Adresse eintragen.
3. Policy anlegen, zum Beispiel Zugriff für alle Adressen mit der Endung
   `@ncloud.swiss`.

## Lokal ansehen

`index.html` im Browser öffnen genügt. Für den Import über die
Laufwerksanbindung wird Chrome oder Edge gebraucht.

## Claude Code

Im Repository `claude` starten. `CLAUDE.md` wird automatisch gelesen und
enthält die Sprach-, Gestaltungs- und Fachregeln.
