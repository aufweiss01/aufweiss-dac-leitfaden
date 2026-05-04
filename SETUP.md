# Einrichtungsanleitung und Referenz: kunde-produkt

Dieses Dokument beschreibt die Einrichtung eines neuen Produktrepos
und erlaeutert alle enthaltenen Dateitypen. Es dient als Grundlage
fuer den Redaktionsleitfaden und den Uebersetzerleitfaden.

---

## Teil A: Einrichtung

### 1  Ordner und Repo umbenennen

Benenne den Ordner und spaeter das GitHub-Repository um:
  kunde-produkt  ->  z.B. mustermann-geraet-x

Passe ausserdem an:
  README.md              -> Kundenname und Produktname eintragen
  .vscode/settings.json  -> [mapname] durch echten Map-Dateinamen ersetzen

### 2  Git initialisieren

  git init
  git checkout -b develop

### 3  Shared-Repo als Submodul einbinden

  git submodule add [URL des kunde-shared Repos] shared
  git submodule update --init --recursive

  Waehrend der Entwicklung zeigt das Submodul auf branch=develop.
  Nach Stabilisierung: .gitmodules auf branch=main umstellen.

### 4  Produktwerte eintragen

  docs/reuse/project_specific_names.ditamap
    -> Produktname, Version, Firmenname eintragen (siehe B.5)

  docs/projektname.ditamap
    -> Datei umbenennen: projektname.ditamap -> [mapname].ditamap
    -> doc-id eintragen (z.B. HB-001)
    -> doc-type eintragen (z.B. Handbuch)

  docs/filters/standard.ditaval
    -> Filterregeln fuer Produktvarianten eintragen (siehe B.9)

### 5  Schematron-Wertelisten anpassen

  shared/schematron/dita_rules.sch (im Submodul):
    -> Erlaubte Produktnamen (prodname) eintragen
    -> Erlaubte @product-Attributwerte eintragen
  Aenderungen im Submodul muessen im shared-Repo committet werden.

### 6  Uebersetzungsmaps anpassen

  translations/[lang]/projektname_[lang].ditamap
    -> Umbenennen: projektname_[lang] -> [mapname]_[lang]

  translations/[lang]/reuse/project_specific_names.ditamap
    -> Produktname und Version in Zielsprache eintragen

  translations/[lang]/reuse/reusables.ditamap
    -> conkeyref-Keys zeigen auf shared/translations/[lang]/reuse/
       (bereits vorkonfiguriert, nur bei Bedarf anpassen)

### 7  Terminologiedatenbank befuellen

  project_specific_termbase.tbx
    -> Projektspezifisch verbotene Benennungen eintragen
    -> Muster: siehe Abschnitt B.8 und SETUP_shared.md B.11

### 8  GitHub einrichten

  git remote add origin [URL]
  git add .
  git commit -m "chore: Produktrepo initialisiert"
  git push -u origin develop

  Repository in GitHub als privat markieren.

### 9  GitHub-Secret hinterlegen

  GitHub -> Repo -> Settings -> Secrets and variables -> Actions
    -> New repository secret
    Name:  SUBMODULE_PAT
    Wert:  [Personal Access Token mit Scope "repo"]

### 10  Branch Protection Rules einrichten

  GitHub -> Repo -> Settings -> Branches -> Add branch ruleset
    -> Enforcement status: Active
    -> Target branches: develop und main (je eine Regel)
    -> Require status checks to pass -> Status-Check: validierung

### 11  Ersten Inhalt anlegen

  Vorlagedateien kopieren, umbenennen, Prolog befuellen:
    docs/concepts/c_[thema].dita  -> z.B. c_einfuehrung.dita
    docs/tasks/t_[thema].dita     -> z.B. t_inbetriebnahme.dita
    docs/references/r_[thema].dita -> z.B. r_technische_daten.dita

  Topics in der Haupt-Map eintragen (siehe B.1).

---

## Teil B: Dateireferenz

### B.1  docs/[mapname].ditamap (Haupt-Map)

Zweck: Einstiegspunkt fuer die Ausgabe. Definiert Struktur und
Reihenfolge aller Topics des Dokuments.

Pflichtfelder in topicmeta:
  doc-id    -> Eindeutige Dokument-ID, z.B. HB-001, SM-001
  doc-type  -> Dokumenttyp, z.B. Handbuch, Service-Manual,
               Schnittstellenbeschreibung

Topics eintragen:
  <topicref href="concepts/c_einfuehrung.dita"/>
  <topicref href="tasks/t_inbetriebnahme.dita"/>
  <topicref href="references/r_technische_daten.dita"/>

Verschachtelung fuer Kapitelstruktur:
  <topicref href="concepts/c_kapitel.dita">
    <topicref href="tasks/t_unterabschnitt.dita"/>
  </topicref>

Die Map bindet per mapref die Reuse-Ressourcen ein:
  shared/reuse/reusables.ditamap  -> Keys aus dem Shared-Repo
  docs/reuse/reusables.ditamap    -> Projektspezifische Keys

### B.2  docs/concepts/c_[thema].dita

Zweck: Erklaerende und beschreibende Inhalte.
Benennungskonvention: Praefix c_, z.B. c_systemuebersicht.dita

Struktur:
  concept -> Wurzelelement mit id und xml:lang
    title  -> Titel des Topics
    prolog -> Metadaten (siehe B.10)
    conbody -> Inhalt: p, note, fig, table, ul, ol, dl etc.

Wann Concept verwenden:
  - Systemuebersichten, Einfuehrungen
  - Erklaerungen von Konzepten, Prinzipien, Funktionen
  - Hintergrundinformationen

### B.3  docs/tasks/t_[thema].dita

Zweck: Handlungsanweisungen Schritt fuer Schritt.
Benennungskonvention: Praefix t_, z.B. t_inbetriebnahme.dita

Struktur:
  task -> Wurzelelement
    title
    prolog -> Metadaten (siehe B.10)
    taskbody
      prereq   -> Voraussetzungen und Warnhinweise (vor steps)
      context  -> Optionaler Kontext (warum diese Aufgabe)
      steps    -> Schritt-fuer-Schritt-Anweisungen
        step -> cmd (Pflicht), info, substeps, stepresult
      result   -> Erwartetes Ergebnis
      postreq  -> Nachfolgende Aktionen

Warnhinweise aus shared einbinden (immer in prereq):
  <hazardstatement type="warning"
    conref="../../shared/reuse/warnings/warnings.dita
            #warnings/warn_[id]"/>

### B.4  docs/references/r_[thema].dita

Zweck: Nachschlage- und Referenzinformationen.
Benennungskonvention: Praefix r_, z.B. r_technische_daten.dita

Struktur:
  reference -> Wurzelelement
    title
    prolog -> Metadaten (siehe B.10)
    refbody -> Inhalt:
      table      -> Tabellarische Daten, z.B. Parameterlisten
      section    -> Abschnitte mit Titel
      properties -> Eigenschaftslisten (Attribut/Wert-Paare)

Beispiel Tabelle:
  <table>
    <tgroup cols="2">
      <thead>
        <row><entry>Parameter</entry><entry>Wert</entry></row>
      </thead>
      <tbody>
        <row><entry>Spannung</entry><entry>230 V</entry></row>
        <row><entry>Frequenz</entry><entry>50 Hz</entry></row>
      </tbody>
    </tgroup>
  </table>

Beispiel properties:
  <properties>
    <prophead>
      <proptypehd>Eigenschaft</proptypehd>
      <propvaluehd>Wert</propvaluehd>
    </prophead>
    <property>
      <proptype>Schutzklasse</proptype>
      <propvalue>IP54</propvalue>
    </property>
  </properties>

### B.5  docs/reuse/project_specific_names.ditamap

Zweck: Projektspezifische Variablen als Keys.
Pflegestelle fuer Produktname, Version und Firmenname.

Enthaltene Keys:
  produktname  -> Vollstaendiger Produktname
  version      -> Aktuelle Version
  firmenname   -> Firmenname (ueberschreibt ggf. den Wert aus shared)

Verwendung im Topic (Fliesstext):
  <ph keyref="produktname"/>
  <ph keyref="version"/>

Hinweis: prodname im Prolog wird als direkter Wert eingetragen,
nicht per keyref (DTD-Einschraenkung). Der Wert muss mit dem
Eintrag in shared/schematron/dita_rules.sch uebereinstimmen,
sonst schlaegt die Schematron-Validierung an.

### B.6  docs/reuse/reuse_concepts.dita
### B.7  docs/reuse/reuse_tasks.dita
### B.8  docs/reuse/reuse_references.dita

Zweck: Projektspezifische wiederverwendbare Bausteine.
Entsprechen den gleichnamigen Dateien im Shared-Repo, enthalten
aber nur projektspezifische Inhalte.

Fuer Erlaeuterungen zu Struktur, Beispielen und conkeyref:
  siehe SETUP_shared.md B.5, B.6, B.7

### B.9  docs/filters/standard.ditaval

Zweck: Filterregeln fuer Produktvarianten (Conditional Processing).

Ermoeglicht, Topics oder Abschnitte nur fuer bestimmte Produkt-
varianten einzuschliessen oder auszuschliessen.

Verwendung: @product-Attribut am zu filternden Element eintragen,
z.B.:
  <p product="geraet_x">Dieser Abschnitt gilt nur fuer Geraet X.</p>

Filterregel in standard.ditaval:
  <prop action="include" att="product" val="geraet_x"/>
  <prop action="exclude" att="product" val="geraet_y"/>

Die @product-Werte muessen in shared/schematron/dita_rules.sch
in der Variable erlaubte-products eingetragen sein.

### B.10  Prolog (Metadaten in jedem Topic)

Jedes Topic enthaelt im prolog-Element folgende Metadaten:

  audience
    @type:            Zielgruppe
      user              -> Anwender
      servicepersonnel  -> Servicetechniker
      administrator     -> Administrator
    @job:             Taetigkeit
      operating, maintaining, installing, programming,
      troubleshooting, other
    @experiencelevel: Erfahrungsniveau
      novice, general, expert

  prodinfo
    prodname:  Produktname (direkter Wert, kein keyref)
    vrmlist:   Version

  othermeta name="component"
    Komponentenname, z.B. "Netzteil", "Steuereinheit"
    Pflicht wenn der Titel allein nicht eindeutig ist.

  othermeta name="lifecycle-stage"
    Lebenszyklusphase(n), leerzeichen-getrennt:
    installation, commissioning, operation,
    maintenance, decommissioning

  othermeta name="status"
    Redaktionsstatus:
    draft -> in-review -> approved -> outdated -> deprecated

  othermeta name="assignee"
    Zustaendige Person; leer lassen wenn nicht vergeben.

Nicht im Prolog eintragen (wird aus Git gelesen):
  author, created, revised

Nur in translations/[lang]/ (nie im Ausgangsmodul):
  othermeta name="translation-status"
    translation-requested -> in-translation ->
    translation-review -> translation-approved

### B.11  docs/links/external_links.ditamap

Zweck: Projektspezifische externe URLs als Keys.
Konvention: Praefix "link_" fuer alle Link-Keys.

Beispiel:
  <keydef keys="link_datenblatt"
          href="https://beispiel.de/datenblatt.pdf"
          format="pdf" scope="external"/>

Verwendung:
  <xref keyref="link_datenblatt">Datenblatt</xref>

Vorteil: URL-Aenderungen zentral pflegbar.

### B.12  docs/links/internal_links.ditamap

Zweck: Interne Querverweise zwischen Topics als Keys.

Vorteil: Pfadaenderungen (z.B. nach Umbenennung einer Datei)
muessen nur hier gepflegt werden, nicht in jedem Topic das
diesen Verweis enthaelt. Die Datei gibt ausserdem einen Ueberblick
ueber alle definierten internen Querverweise.

Konvention: Praefix "link_" fuer alle Link-Keys.

Beispiel:
  <keydef keys="link_c_einfuehrung"
          href="../concepts/c_einfuehrung.dita"/>

Verwendung im Topic:
  <xref keyref="link_c_einfuehrung"/>

### B.13  docs/links/relationship_table.ditamap

Zweck: Thematische Beziehungen zwischen Topics definieren,
ohne diese in den Topics selbst oder in der Haupt-Map zu
verdrahten.

DITA-OT generiert aus der Relationship Table automatisch
"Verwandte Themen"-Links in der Ausgabe.

Aufbau: relrow enthaelt eine Zeile, relcell je eine Spalte
(concept, task, reference). Topics in derselben Zeile werden
als verwandt markiert.

Beispiel:
  <relrow>
    <relcell>
      <topicref href="../concepts/c_funktion.dita"/>
    </relcell>
    <relcell>
      <topicref href="../tasks/t_bedienung.dita"/>
    </relcell>
    <relcell/>
  </relrow>

### B.14  project_specific_termbase.tbx

Zweck: Projektspezifisch verbotene Benennungen.
Ergaenzt customer_specific_termbase.tbx aus dem Shared-Repo.

Fuer Erlaeuterungen zu Format, Struktur und Beispielen:
  siehe SETUP_shared.md B.11

---

## Teil C: Uebersetzungsworkflow

### C.1  Struktur der Uebersetzungen

Jede Zielsprache hat einen eigenen Ordner unter translations/[lang]/.
Vorkonfiguriert: de-DE, en-GB, en-US, fr-FR.

Struktur je Sprache:
  [mapname]_[lang].ditamap        -> Sprachspezifische Haupt-Map
  reuse/project_specific_names.ditamap -> Uebers. Produktvariablen
  reuse/reusables.ditamap         -> conkeyref-Keys (auf shared/translations)
  reuse/reuse_concepts.dita       -> Uebers. Concept-Bausteine
  reuse/reuse_tasks.dita          -> Uebers. Task-Bausteine
  reuse/reuse_references.dita     -> Uebers. Reference-Bausteine

### C.2  Ablauf bei der Uebersetzung eines neuen Topics

1. Redakteur legt uebers. Topic-Geruest an (kopiert Ausgangstopic
   in translations/[lang]/[type]/).
2. Redakteur setzt xml:lang auf Zielsprache.
3. Redakteur setzt translation-status auf "translation-requested".
4. Uebersetzer ersetzt Textinhalte (nicht Attribute oder Pfade).
5. Uebersetzer setzt translation-status auf "in-translation".
6. Review: translation-status -> "translation-review".
7. Freigabe: translation-status -> "translation-approved".

### C.3  Hinweise fuer Uebersetzer

- Keine Pfade aendern: conkeyref, href, keyref sind strukturell
  und werden vom Redakteur vorgegeben.
- xml:lang am Wurzelelement muss der Zielsprache entsprechen
  (wird vom Redakteur gesetzt).
- Attribute generell nicht uebersetzen (type, id, product etc.).
- Nur Texte zwischen XML-Tags uebersetzen sowie alt-Texte.
- translation-status-Werte sind festgelegt (s. B.10).
- Nur Topics uebersetzen, deren Ausgangsmodul status="approved" hat.
