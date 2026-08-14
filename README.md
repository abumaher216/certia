# Certia

Managementsystem für ISO 9001 und ISO 27001. Ein Produkt der n'cloud.swiss AG.

Dieser Stand ist ein lauffähiger Prototyp zum Vorführen, nicht die
Produktivversion. Der Weg dorthin steht in `docs/Bauplan.md`.

**Dieses Repository muss privat bleiben.** Die Zugangsdaten der Demoschranke
stehen im Quelltext.

## Aufbau

```
public/                    <- nur dieser Ordner wird veröffentlicht
  index.html               die vollständige Anwendung, eine Datei, kein Build
  favicon.svg              App-Icon
  logo.svg                 Marke freigestellt
  _headers                 Sicherheitsheader
  .assetsignore            hält .git und .wrangler aus dem Upload
  beispiel-dokumente/      drei PDF zum Ausprobieren des Imports
docs/Bauplan.md            was die Produktivversion braucht
CLAUDE.md                  Projektgedächtnis für Claude Code
wrangler.jsonc             Cloudflare-Konfiguration
```

Alles ausserhalb von `public` wird nicht ausgeliefert. Das ist Absicht:
`CLAUDE.md`, `docs` und die Git-Historie gehören nicht ins Netz.

## Veröffentlichen über Cloudflare

Beim Anlegen des Projekts:

- Framework preset: **None**
- Build command: leer lassen
- Build output directory: **`public`**

Die Datei `wrangler.jsonc` setzt dasselbe noch einmal fest, damit auch ein
Deploy über die Kommandozeile dasselbe Verzeichnis nimmt.

Danach löst jeder Push auf `main` eine neue Veröffentlichung aus, jeder
Branch bekommt eine eigene Vorschauadresse, frühere Stände lassen sich im
Dashboard zurückholen.

Über die Kommandozeile:

```
npx wrangler deploy
```

## Zugang schützen

Die eingebaute Anmeldung ist eine Demoschranke, kein Schutz. Für einen
echten Zugang Cloudflare Access davorlegen, kostenlos bis fünfzig Personen:
Zero Trust, Access, Applications, Add an application, Self-hosted, als Domain
die veröffentlichte Adresse eintragen, dann eine Policy für die erlaubten
E-Mail-Adressen anlegen.

## Anmeldung

Benutzer `Matter`. Das Passwort steht in `public/index.html` im Block
`ZUGANG`.

## Lokal ansehen

`public/index.html` im Browser öffnen genügt. Für die Laufwerksanbindung im
Import wird Chrome oder Edge gebraucht.

## Claude Code

Im Repository `claude` starten. `CLAUDE.md` wird automatisch gelesen.
