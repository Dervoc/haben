# Hostmint-Fork von Haben

Dieser Fork ist eine Werkbank, keine eigene Linie. Ziel ist ein Patch-Satz, der gegen null geht: Was wir ändern, geht
als Pull Request nach [firsttris/haben](https://github.com/firsttris/haben) und verschwindet aus unserem Branch, sobald
es dort gemergt ist.

## Branches

| Branch | Inhalt | Regel |
| --- | --- | --- |
| `main` | exakter Spiegel von upstream | nie darauf committen; der nächtliche Abgleich spult ihn nur vor |
| `hostmint` | upstream plus unser Patch-Satz | Standard-Branch; aus ihm wird gebaut und ausgerollt |
| `feat/…`, `fix/…` | ein Thema für einen Pull Request nach upstream | immer von `upstream/main` abzweigen, nie von `hostmint` |

## Regeln für den Patch-Satz

1. **Upstream zuerst.** Jeder Fix und jede Erweiterung, die anderen auch nützt, geht als Pull Request nach upstream.
   Dort gelten die Regeln aus `CONTRIBUTING.md` und [Mitwirken](https://haben.app/mitwirken.html), unter anderem die
   Selbstprüfung mit dem Skill `beitrag-pruefen`.
2. **Keine eigenen Migrationen in `apps/web/drizzle/`.** Die Kette ist linear und wächst um mehrere Migrationen am
   Tag; eine eigene Nummer kollidiert beim nächsten Abgleich. Braucht etwas wirklich eigene Tabellen, gehören sie in ein
   eigenes Postgres-Schema `hostmint` mit eigenem Migrationswerkzeug.
3. **GoBD-Trigger und Steuerlogik nur über upstream ändern**, nie nur hier. Upstream rechnet jede fachliche Änderung
   an einem typischen und einem Randfall gegen die Primärquelle nach; diese zweite Prüfung wollen wir haben.
4. **Eigene Oberflächen gehören nach Hostmint**, nicht in diesen Code. In Haben hängen sie über
   **Einstellungen → Eigene Seiten** ohne Codeänderung im Menü.
5. Bleibt etwas länger als zwei Wochen im Patch-Satz, ohne dass es upstream gehen kann: in Frage stellen.

## Automatik

| Workflow | Wann | Was |
| --- | --- | --- |
| `hostmint-abgleich.yml` | täglich 02:41 UTC, oder von Hand | spult `main` auf upstream vor, führt upstream in `hostmint` zusammen; Konflikt → Issue mit Label `hostmint-abgleich` |
| `hostmint-ci.yml` | bei jedem Push auf `hostmint` | ruft die unveränderte `ci.yml` von upstream auf; rot → Issue mit Label `hostmint-ci` |
| `ci.yml` (upstream) | bei jedem Push auf `main` | zeigt, ob upstream selbst grün ist |
| `hostmint-pruefen.yml` | von Hand, mit Branch und Testpfaden | Lint, Typecheck und ausgewählte Tests eines Feature-Branches, bevor er als Pull Request nach upstream geht; zeigt bei Lint-Fehlern den Formatierungs-Diff von Biome |

Läuft `ci.yml` auf `main` grün und `hostmint-ci.yml` rot, liegt es an unserem Patch-Satz.

Die übrigen Workflows von upstream sind in den Actions-Einstellungen des Forks **abgeschaltet**, nicht gelöscht, damit
`main` ein exakter Spiegel bleibt: `board`, `bump`, `docs`, `gobd-kern-release`, `jahreswerte`,
`pr-regeln-kommentar`, `pr-regeln`, `release`, `screenshots`, `uebernehmen`. Sie brauchen Secrets von upstream,
veröffentlichen Images oder Doku, oder arbeiten an upstreams Projektboard. Nach dem Abgleich prüfen, ob upstream einen
neuen Workflow hinzugefügt hat, und ihn gegebenenfalls ebenfalls abschalten.

## Token für den Abgleich

Der Abgleich braucht das Secret `ABGLEICH_TOKEN`. Der eingebaute `GITHUB_TOKEN` reicht nicht: Er darf keine Dateien
unter `.github/workflows` ändern, und Pushes mit ihm starten keine weiteren Workflows.

1. GitHub → Settings → Developer settings → Personal access tokens → **Fine-grained tokens** → Generate new token.
2. Name `haben-abgleich`, Resource owner `Dervoc`, Ablauf höchstens ein Jahr (Erinnerung eintragen).
3. Repository access: **Only select repositories** → `Dervoc/haben`. Nur dieses eine Repository.
4. Repository permissions: **Contents** Read and write, **Workflows** Read and write. Sonst nichts.
5. Im Terminal: `gh secret set ABGLEICH_TOKEN -R Dervoc/haben` und das Token einfügen.
6. Probelauf: `gh workflow run hostmint-abgleich.yml -R Dervoc/haben`.

## Von Hand abgleichen

```sh
git fetch upstream
git switch hostmint
git merge upstream/main
git push origin hostmint
```

`upstream` ist lokal nur zum Lesen eingerichtet (`git remote set-url --push upstream …`); nach upstream geht etwas
ausschließlich über Pull Requests.

## Lizenz

Haben steht unter der AGPL-3.0. Dieser Fork ist öffentlich, damit ist der Quellcode jeder Instanz, die wir betreiben,
für ihre Nutzer verfügbar (etwa für die Steuerberatung). Hostmint spricht Haben nur über die REST-API an und bleibt ein
eigenständiges Programm.
