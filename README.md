# kunde-produkt

Produktrepository fuer aufweiss, Produkt dac_template.

## Struktur
- docs/           - Quelldokumente (de-DE)
- translations/   - Uebersetzungen (je Sprache: Map + concepts/tasks/references/reuse)
- shared/         - Submodul: [kunde]-shared
- output/         - Generierte Ausgaben (nicht versioniert)

## Branching
- main            - Stabile Inhalte
- develop         - Laufende Bearbeitung
- feat/[thema]    - Neue Inhalte oder Funktionen
- fix/[thema]     - Fehlerkorrekturen
- partner/[name]_[thema] - Arbeitsbranches externer Partner

## Submodul aktualisieren
  git submodule update --remote shared
  git add shared
  git commit -m "chore: shared aktualisiert"

Einrichtungsanleitung: siehe SETUP.md
