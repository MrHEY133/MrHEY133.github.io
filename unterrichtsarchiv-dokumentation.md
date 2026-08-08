# Das Unterrichtsarchiv — Aufbau, Funktionsweise, Abhängigkeiten

**Autor:** Fabian Heyne · **Stand:** 09.08.2026 · **Bezugsversion:** Archiv-Schema v1.9

Dieses Dokument beschreibt vollständig, wie mein Unterrichtsarchiv aufgebaut ist, welche Regeln
darin gelten und wie die Teile voneinander abhängen. Es richtet sich an Leserinnen und Leser,
die es genau wissen wollen — es ist bewusst kein Überblickstext.

Alle Beispiele darin sind schematisch. Konkrete Reihen, Lerngruppen, Prüfungsinhalte und
Reflexionstexte aus dem laufenden Betrieb sind nicht Gegenstand dieser Dokumentation; beschrieben
ist durchgehend die Funktion, nicht der Inhalt.

---

## 1 Grundidee

Das Archiv beruht auf einer einzigen Grundannahme: **Unterricht ist nie fertig.** Jede Stunde
wird gehalten, und beim Halten zeigt sich, was trägt und was nicht. Wenn dieses Wissen nur im
Kopf bleibt, ist es beim nächsten Durchgang ein Jahr später verblasst.

Daraus folgen drei Konstruktionsentscheidungen, die alles Weitere bestimmen:

**Planung und Durchführung sind getrennt.** Das geplante Konzept einer Stunde — der *Blueprint* —
bleibt unverändert stehen. Was in einer konkreten Lerngruppe tatsächlich passiert ist, wird
getrennt davon als *Durchlauf* erfasst. So entsteht über die Jahre eine Historie statt einer
überschriebenen Datei.

**Alles ist Text.** Reine Markdown-Dateien mit YAML-Frontmatter, keine Datenbank, kein
proprietäres Format. Jede Datei ist ohne Spezialsoftware lesbar und bleibt es auch in zehn
Jahren.

**Struktur ist maschinenprüfbar.** Eine zentrale Schemadatei legt Ordneraufbau, Benennung und
Pflichtfelder verbindlich fest; ein Linter prüft bei jedem Arbeitsbeginn, ob das Archiv dem
Schema entspricht.
Ohne diese Prüfung driftet eine gewachsene Ablage innerhalb eines Schuljahres auseinander.

---

## 2 Ordnerstruktur

Das Archiv hat zwei Bereiche: **Systemordner** mit führendem Unterstrich und **Fachordner**.

```
Unterrichtsarchiv/
│
├── _Vorlagen/          Kopiervorlagen für alle Dokumenttypen
├── _Quellen/           zentrale Quellensammlung, nach Einsatzzweck sortiert
├── _Grundsaetze/       Didaktische Grundsätze, Methodenkatalog, Operatoren
├── _Konfiguration/     Schema, Betriebsdaten, Stoffverteilungen
├── _Referenz/          Kerncurricula, Leitfäden, Aufbauanleitung, Spezifikation
├── _Skills/            KI-Skills und Werkzeuge
├── _Quarantaene/       fehlerhafte Bestände, aus dem Lint ausgenommen
│
├── PH/                 Physik
│   └── PH-JG/          Jahrgang
│       └── PH-JG.REIHE/            Reihe
│           ├── PH-JG.REIHE_Reihe.md
│           ├── PH-JG.REIHE_Lerngruppen_Tracker.md
│           └── PH-JG.REIHE-B1/     Block
│               ├── PH-JG.REIHE-B1_Blueprint.md
│               └── (Materialien)
├── CH/                 Chemie
└── MA/                 Mathematik
```

Die Hierarchie ist immer dieselbe: **Fach → Jahrgang → Reihe → Block.** Innerhalb eines
Blockordners wird nicht weiter verschachtelt; Materialien liegen flach neben dem Dokument, zu
dem sie gehören.

Zwei Ausnahmen von der Blockordner-Regel: Die Reihenübersicht und der Lerngruppen-Tracker liegen
auf Reihenebene, weil sie die ganze Reihe betreffen. Lernkontrollen ebenfalls — sie prüfen die
Reihe, nicht einen Block.

---

## 3 Kürzelsystematik

Jedes Dokument trägt seine Verortung im Namen. Das ist der Grund, warum Volltextsuche im Archiv
funktioniert.

| Ebene | Muster | Schematisches Beispiel |
|---|---|---|
| Jahrgang | `FACH-JG` | `PH-08` |
| Reihe | `FACH-JG.REIHE` | `PH-08.XYZ` |
| Block | `FACH-JG.REIHE-Typ+Nr` | `PH-08.XYZ-B1` |

Reihenkürzel sind sprechende Kurzformen, die beim Planen festgelegt und aus der Stoffverteilung
abgeleitet werden.

**Block-Präfixe** sind ein abschließendes Vokabular — es gibt genau diese acht:

| Präfix | Bedeutung |
|---|---|
| `B` | Blueprint / Einführung |
| `UE` | Übungsplan (nur Mathematik) |
| `PL` | Problemlöseaufgabe (nur Mathematik, optional) |
| `E` | Einschub |
| `WH` | Wiederholung |
| `LZ` | Lernzeit / LZF-Phase |
| `LKR` | LK-Rückgabe |
| `SPH` | Selbststudium / Vertretung |

Alternativvarianten einer Reihe bekommen ein angehängtes `a` ohne Bindestrich; beide Varianten
verweisen über das Frontmatter-Feld `bezug` aufeinander.

**Umlaute werden in Dateinamen vermieden** (`oe`, `ae`, `ue`) — sie verursachen bei der
Synchronisation zwischen Betriebssystemen Probleme.

---

## 4 Dokumenttypen

### 4.1 Reihe (`*_Reihe.md`)

Die Reihenübersicht ist die Landkarte. Sie enthält den inhaltlichen Schwerpunkt, den Bezug zum
Kerncurriculum und die **Blockfolge** als Tabelle:

| Nr | Typ | Std | Kompetenzziel (kurz) | Geplant | Tatsächlich | Dokument |
|---|---|---|---|---|---|---|

Entscheidend sind die beiden Datumsspalten. `Geplant` bleibt stehen, `Tatsächlich` wird
nachgeführt. Zeitverschiebungen werden ausschließlich hier erfasst — nie im Durchlauf. Damit
bleibt die Frage „Wo bin ich zeitlich?" getrennt von der Frage „Wie lief es?".

Das Feld `Dokument` verweist als Wiki-Link auf den zugehörigen Blueprint. So entsteht das
navigierbare Netz.

### 4.2 Blueprint (`*_Blueprint.md`)

Das Kernstück: die Planung einer Unterrichtsstunde. Pflichtabschnitte sind

- **Didaktisches Zentrum** — der zentral zu fördernde Kompetenzbereich, gefolgt von Indikatoren,
  die nach Anforderungsbereichen gestaffelt sind (AFB I / II / III). AFB III ist grundsätzlich
  didaktische Reserve, also nur bei verbleibender Zeit.
- **Kompetenzziel** — in Ich-kann-Form
- **Stundenfrage** — die Frage, die die Stunde beantwortet, aus Schüleräußerungen entwickelt
- **Ablauf** — in einer von zwei Varianten (siehe unten)
- **Materialien** — inklusive einer Pflichtzeile zur Bereitstellung
- **Methoden-Check** — Nachweis, dass die Methodenwahl bewusst getroffen wurde
- **Durchläufe** — anfangs leer, wird nach dem Unterricht gefüllt

Optional kommen hinzu: Differenzierung, Vorbereitung (Materiallogistik, Experimente), Eintrag
für das digitale Klassenbuch, Realexperiment-Option, externe Quellen, Notizen.

**Zwei Ablaufvarianten** stehen zur Wahl:

*Standard:* Einstieg → Erarbeitung → Sicherung → Festigung (optional) → Abschluss

*Gestuft:* Einstieg → Erarbeitung I → Zwischensicherung → Erarbeitung II → Sicherung →
Festigung (optional) → Abschluss

Die gestufte Variante wird gewählt, wenn eine vollständige Erarbeitung im Alleingang eine
Überforderung wäre und eine Vorabeinsicht nötig ist. Erarbeitung II beantwortet dann die
Stundenfrage.

### 4.3 Lerngruppen-Tracker (`*_Lerngruppen_Tracker.md`)

Eine flache Tabelle pro Reihe mit den Spalten *Block, Lerngruppe, Datum, Status, Notiz*. Der
Status stammt aus einem festen Vokabular: `✅ durchgeführt`, `❌ ausgefallen`, `— ausgelassen`,
`geplant`.

Der Tracker beantwortet die Frage, die bei Parallelgruppen sonst regelmäßig verloren geht:
Welche Gruppe ist wo? Zwei Klassen desselben Jahrgangs laufen nie synchron. Erfasst wird die
Gruppe als Kürzel — nie einzelne Lernende.

### 4.4 Übungsplan und Problemlöseaufgabe (Mathematik)

Mathematik hat einen eigenen Dreiphasenrhythmus: Einführung (`B`) → Übungsplan (`UE`) →
optional Problemlöseaufgabe (`PL`). Der Übungsplan ist kompetenzgebunden und arbeitet mit einem
Schwellenwert statt mit einer festen Stundenzahl. Die Problemlöseaufgabe ist eine offene
Transferaufgabe mit gestuften Tippkarten, ohne Benotung, und wird nur angelegt, wenn sie
eigenständigen didaktischen Mehrwert hat.

Aufgabenreferenzen für Mathematik stammen ausschließlich aus einem Lehrwerksindex — eine Datei
pro Jahrgang mit Kapiteln, Seitenzahlen, Aufgabennummern, Typen und Niveaus. Der Grund ist banal
und wichtig: Aufgabennummern aus dem Gedächtnis sind zuverlässig falsch.

### 4.5 Lernkontrolle und Erwartungshorizont

Beide sind **paarpflichtig** — eine Lernkontrolle ohne Erwartungshorizont ist ein Fehler, und
umgekehrt. Sie liegen im Reihenordner. Im Frontmatter stehen die Punktzahl und die prozentuale
Verteilung über die drei Anforderungsbereiche; nach der Rückgabe kommt ein Auswertungsabschnitt
hinzu, der auf Gruppenebene festhält, welche Aufgaben auffällig waren und welche Fehler häufig
auftraten.

Für diese Auswertung gilt bewusst eine niedrige Schwelle: Grobe Angaben — Tendenz statt Zählung —
sind ausdrücklich zulässig. Eine Erfassung, die mehr als eine Nachricht kostet, findet im
Schulalltag nicht statt.

---

## 5 Frontmatter und Tags

Jedes Dokument trägt YAML-Frontmatter mit Pflichtfeldern je Dokumenttyp. Für den Blueprint sind
das `title`, `tags`, `fach`, `jahrgang`, `reihe`, `nr`, `typ`, `kompetenzbereich`.

Das Feld `fach` nimmt ausschließlich die Kürzel `PH`, `CH`, `MA` — „Physik" ausgeschrieben ist
ein Lint-Fehler. Solche Verengungen sind Absicht: Ein Feld mit zwei erlaubten Schreibweisen ist
ein Feld, nach dem sich nicht mehr zuverlässig filtern lässt.

**Tags** folgen dem Format `[FACH, Dokumenttyp, THEMA]`. Der Dokumenttyp stammt aus einem
geschlossenen Vokabular, das THEMA aus einem eigenen Kanon-Dokument, das seinerseits aus den
Stoffverteilungsplänen abgeleitet ist. Freie Themen-Tags sind unzulässig — sie sind der
klassische Weg, auf dem eine Tag-Struktur unbrauchbar wird.

---

## 6 Durchläufe — das Herzstück

Ein Durchlauf ist kein eigenes Dokument, sondern ein Abschnitt am Ende des Blueprints. Pro
Lerngruppe und Termin entsteht ein Eintrag nach diesem Muster:

```markdown
## Durchläufe

### TT.MM.JJJJ (Gruppenkürzel)
- **Ändern:** [was beim nächsten Mal anders laufen soll]
- **Fehlkonzept:** [fachliche Fehlvorstellung, die aufgetreten ist]
- **Notiz:** [organisatorische Beobachtung]
```

Drei Felder, unter fünf Minuten Aufwand. Das ist die Obergrenze dessen, was nach einem
Unterrichtstag realistisch noch passiert — jede aufwendigere Erfassung wird nach zwei Wochen
stillschweigend eingestellt.

Festgehalten wird die **Sache**, nicht die Person: was didaktisch nicht funktioniert hat, welche
fachliche Fehlvorstellung auftrat, wo die Zeit nicht reichte. Einzelne Lernende kommen darin
nicht vor.

In Physik und Chemie ist die Schulzweig-Annotation Pflicht, und G- und R-Zweig werden getrennt
eingetragen, auch wenn die Stunde parallel lief. Derselbe Blueprint verhält sich in beiden
Zweigen unterschiedlich; ein gemeinsamer Eintrag würde genau die Information vernichten, wegen
der man ihn schreibt.

Ein Tracker-Status `✅ durchgeführt` ohne zugehörigen Durchlauf-Eintrag erzeugt eine
Lint-Warnung. Das ist die einzige Stelle, an der das System aktiv mahnt.

---

## 7 Das Schema als Single Source of Truth

`Archiv_Schema.yaml` ist die verbindliche Referenz. Sie legt fest:

- Pfadmuster und Pflichtdateien je Ebene
- reguläre Ausdrücke für Reihen- und Blockkürzel
- das abschließende Vokabular der Block-Präfixe
- Schreibweisen und Gültigkeitsregeln für Lerngruppenkürzel
- Pflicht- und Optionalfelder im Frontmatter je Dokumenttyp
- Pflicht- und Optionalabschnitte im Blueprint
- Statusvokabular des Trackers, zeichengenau inklusive Emoji
- verbotene Muster (Synchronisationsduplikate wie ` (1).pdf`, Office-Tempdateien)
- welche Bereiche vom Lint ausgenommen sind

Zwei Konsumenten lesen dieselbe Datei: der Linter und jeder schreibende KI-Skill. Damit können
Automatik und Assistenz nicht auseinanderlaufen — es gibt keine zweite Wahrheit.

Das Schema ist versioniert (aktuell 1.9). Skills tragen die Version, gegen die sie geschrieben
wurden, im Kopf und melden eine Abweichung, statt stillschweigend weiterzuarbeiten. Dieser
Mechanismus heißt intern **Selbstalterung**: Ein Werkzeug, das nicht mehr zum Schema passt, soll
das selbst bemerken.

---

## 8 Der Linter

`archiv_lint.py` ist ein reines Python-Skript ohne KI-Anteil, rund 670 Zeilen, einzige
Abhängigkeit ist PyYAML. Es liest das Schema, durchläuft das Archiv und schreibt einen Report in
die Konfigurationsebene.

Drei Schweregrade: **Fehler**, **Warnung**, **Hinweis**. Der Exit-Code unterscheidet zwischen
sauberem Lauf, gefundenen Fehlern und Konfigurationsabbruch.

Betrieb: automatisch bei jeder Anmeldung am Rechner, mit drei Minuten Verzögerung, zusätzlich
manuell. Der Auslöser ist bewusst an die Anmeldung gekoppelt und nicht an eine feste Uhrzeit:
Ein nächtlicher Lauf setzt voraus, dass der Rechner nachts läuft — bei einem Arbeitsgerät, das
abends ausgeschaltet wird, fällt er schlicht aus. So beginnt stattdessen jeder Arbeitstag mit
einem aktuellen Report.

Die Verzögerung und eine Warteschleife im Startskript fangen denselben Fall ab: Der
synchronisierte Ordner ist unmittelbar nach der Anmeldung noch nicht eingebunden. Das Skript
prüft in Zehn-Sekunden-Intervallen bis zu zehn Minuten lang, ob das Archiv erreichbar ist, und
bricht erst danach kontrolliert ab.

Die verbindliche Regel für alle schreibenden Skills lautet: **vor jedem Archiv-Write den Report
lesen**; Fehler im Zielbereich zuerst beheben. Damit wird verhindert, dass neue Dokumente auf
kaputter Struktur aufsetzen.

Bereiche ohne Deputat im laufenden Schuljahr sind eingefroren und werden nicht geprüft. Ebenso
Dokumente mit dem Frontmatter-Flag `endspurt: true` — Material, das unter Zeitdruck am
Schuljahresende entstanden ist und nicht rückwirkend auf Schemakonformität gebracht wird. Beides
ist eine bewusste Entscheidung gegen Vollständigkeit: Ein Linter, der dauerhaft hunderte
unbehebbarer Befunde meldet, wird ignoriert und ist damit wertlos.

---

## 9 Konfigurationsschicht

Neben dem Schema liegen in der Konfigurationsebene die Dateien, aus denen sich der Betrieb
speist:

| Datei | Funktion |
|---|---|
| Lerngruppen | kanonische Liste der Lerngruppenkürzel je Schuljahr, mit Schulzweig und Parallelgruppen |
| Stundenplan | Zuordnung von Zeitfenstern zu Lerngruppen |
| Stoffverteilung je Fach | Reihenfolge und Umfang der Reihen |
| Kalender | Ferien, Feiertage, schulische Termine |
| Notenschlüssel | Prozent-Noten-Zuordnung |
| THEMA-Kanon | abschließendes Vokabular für das dritte Tag-Element |
| Moodle-Capabilities | was die eingesetzte Moodle-Instanz kann und was nicht |
| Lint-Report | Ergebnis des letzten Lint-Laufs |

Die Lerngruppenliste ist dabei mehr als eine Liste: Sie ist die **Existenzprüfung**. Ein
Gruppenkürzel, das dort nicht steht, gilt als Tippfehler — auch dann, wenn es dem regulären
Ausdruck genügt. Ein reiner Formatprüfer würde eine Gruppe ohne Zweigbuchstaben durchwinken,
obwohl es sie nicht gibt.

Die Schreibweise desselben Kürzels unterscheidet sich je nach Kontext, und das ist festgelegt:
`FACH GRUPPE` mit Leerzeichen im Stundenplan, `FACH-GRUPPE` mit Bindestrich im Frontmatter,
`GRUPPE` allein im Tracker. Intern arbeiten alle Werkzeuge mit dem Paar (Fach, Gruppe) — eine
nackte Gruppenbezeichnung ohne ihr Fach existiert nie.

Wichtig für das Verständnis der folgenden Abschnitte: Auf dieser Ebene werden **Lerngruppen als
Einheit** geführt — Kürzel, Schulzweig, Zeitfenster. Der Lernstand wird also gruppenbezogen
erfasst und genutzt, und genau das macht die spätere Wiederverwendung wertvoll.

---

## 10 Die Skill-Schicht

Über dem Dateibestand liegt eine Schicht spezialisierter KI-Skills. Jeder ist auf eine Aufgabe
zugeschnitten und kennt die Archivregeln. Die folgende Übersicht nennt die Skills mit direktem
Archivbezug:

| Skill | Aufgabe |
|---|---|
| Reihenplanung PH/CH | Reihenplanung mit vorgeschaltetem Methodenwahl-Gate |
| Reihenplanung MA | Reihenplanung im Dreiphasenrhythmus |
| Block-Iteration | Überarbeitung bestehender Blöcke auf Basis der Durchläufe; Einschubplanung |
| Archiv-Alltag | Durchläufe erfassen, Tracker pflegen, Tages- und Wochenplan |
| Jahresplanung | Jahres- und Halbjahresplanung, Ausfallpuffer, Prüfungstermine |
| Lernkontrolle | Erstellung differenzierter Lernkontrollen, Erfassung der Auswertung |
| Moodle-Hub | Übersichtsseiten für Übungsphasen |
| Moodle-Quiz | Freischalt-Quizzes als importierbares XML |
| HTML-Simulation | Bau, Ablage und Deployment interaktiver Simulationen |
| Skill-Veröffentlichung | Ablage, Prüfung und Ausrollen neuer Skills |

Die Skills sind nicht unabhängig, sondern übergeben aneinander. Ein typischer Weg:

```
Reihenplanung
   └─ Methodenwahl-Gate
        ├─ HTML-Simulation   → Sim bauen, ablegen, deployen
        ├─ Moodle-Hub        → Übersichtsseite
        └─ Moodle-Quiz       → Freischalt-Quiz
   └─ Blueprint entsteht
        └─ Archiv-Alltag     → Durchlauf nach dem Unterricht erfassen
             └─ Block-Iteration → beim nächsten Durchgang überarbeiten
```

Der Kreis schließt sich bei der Block-Iteration: Dieser Skill liest die Durchläufe des letzten
Durchgangs und schlägt daraus Änderungen vor. Genau dafür wurden sie geschrieben.

**Lese-Ökonomie** ist ein durchgängiges Prinzip: Jeder Skill liest nur, was er für seine
Entscheidung braucht. Der Simulations-Skill liest den Methodenkatalog nicht, weil die
Methodenentscheidung beim Aufruf bereits gefallen ist. Ein Skill, der vorsorglich alles liest,
ist langsam und teuer.

**Zugriffsumfang:** Die Assistenz arbeitet nicht auf dem gesamten Rechner, sondern auf einem
ausdrücklich freigegebenen Verzeichnisbereich. Innerhalb dieses Bereichs darf sie Dateien lesen,
anlegen und ändern — außerhalb hat sie keinen Zugriff. Der Zuschnitt dieses Bereichs ist damit
die eigentliche Sicherheitsgrenze und wird bewusst eng gehalten.

---

## 11 Warum KI hier ansetzt

Der Grund für den Aufwand ist nicht Ordnungsliebe. KI-gestützte Materialerstellung scheitert
selten am Formulieren — Sprachmodelle formulieren gut. Sie scheitert am **fehlenden Kontext**.

Ein Modell weiß von sich aus nicht, was in einer Lerngruppe bereits gelaufen ist, welche Begriffe
eingeführt sind, welche Fehlvorstellung beim letzten Mal auftrat und wie viel Zeit tatsächlich
blieb. Ohne diese Information entsteht Material, das allgemein passt und konkret nicht.

Das Archiv macht genau diesen Kontext abrufbar — bezogen auf die jeweilige Lerngruppe, nicht als
Durchschnitt über alle. Ein Skill, der einen Blueprint überarbeitet, liest vorher die Durchläufe
derselben Stunde aus dem Vorjahr. Damit verschiebt sich die Frage von „Schreib mir eine Stunde zu
Thema X" zu „Verbessere diese Stunde auf Basis dessen, was beim letzten Mal passiert ist".

Deshalb sind die Durchläufe kein Nebenprodukt, sondern der eigentliche Wert des Archivs. Die
gesamte Struktur — Schema, Linter, Namenskonventionen — existiert, damit dieser eine
Rückkopplungsweg zuverlässig funktioniert.

---

## 12 Abhängigkeiten im Überblick

```
Kerncurricula, Leitfäden  (_Referenz)
        ↓
Stoffverteilung je Fach   (_Konfiguration)
        ↓
THEMA-Kanon ──────────────┐
        ↓                 │
   Reihe.md               │   Lerngruppenliste ─┐
        ↓                 │        ↓            │
   Blueprint  ←───── Tags ┘   Tracker  ←────────┘
        ↓                          ↑
   Durchläufe ─────────────────────┘
        ↓
   Block-Iteration  →  nächster Durchgang
```

Quer dazu liegen zwei Ebenen: **Archiv_Schema.yaml**, das für alle genannten Dokumente Struktur
und Pflichtfelder definiert, und **archiv_lint.py**, das die Einhaltung prüft.

Harte Abhängigkeiten, deren Verletzung das System bricht:

1. **THEMA-Kanon ← Stoffverteilung.** Neue Reihen müssen erst im Kanon ergänzt werden, sonst
   entstehen unzulässige Tags.
2. **Tracker ← Lerngruppenliste.** Ein Kürzel, das dort fehlt, ist im Tracker ungültig.
3. **Skill ← Schema-Version.** Weicht die Version ab, meldet der Skill und arbeitet nur nach
   ausdrücklicher Bestätigung weiter.
4. **Archiv-Write ← Lint-Report.** Vor jedem Schreibvorgang wird der Report gelesen.
5. **Lernkontrolle ↔ Erwartungshorizont.** Beide oder keiner.

---

## 13 Werkzeugkette

| Ebene | Werkzeug | Funktion |
|---|---|---|
| Bearbeitung | Zettlr | Markdown-Editor mit Wiki-Links und Tag-Suche |
| Ablage | Cloud-Ordner mit Desktop-Synchronisation | Abgleich zwischen Geräten |
| Prüfung | Python + PyYAML | Linter, bei jeder Anmeldung per Aufgabenplanung |
| Assistenz | KI-Skills mit Zugriff auf den freigegebenen Bereich | Lesen und Schreiben im Archiv |
| Bereitstellung | Moodle, GitHub Pages, digitales Klassenbuch | Auslieferung an Lernende |

Kein Build-Schritt, keine Datenbank, kein Server. Das Archiv funktioniert vollständig ohne alle
genannten Werkzeuge — es wäre dann nur ein Ordner mit Textdateien, und genau das ist die
Rückfallebene.

Beim Schreiben gilt eine Betriebsregel, die aus Erfahrung stammt: Schreibzugriffe laufen über den
lokalen Dateisystemzugriff, nie über den Cloud-Konnektor. Letzterer erzeugt bei gleichzeitigem
Zugriff Dateiduplikate — die Muster ` (1).pdf` stehen deshalb ausdrücklich auf der Verbotsliste
des Schemas.

---

## 14 Grenzen und bewusste Entscheidungen

**Das Archiv ist einpersonig.** Es gibt keine Rollen, keine Rechteverwaltung, keine
Mehrbenutzer-Konflikte. Für ein Fachschaftsarchiv wäre der Zuschnitt falsch.

**Der Linter prüft Struktur, nicht Inhalt.** Ein Blueprint kann schemakonform und didaktisch
schwach sein. Die Qualitätssicherung des Inhalts läuft über die Durchläufe, nicht über die
Automatik.

**Nicht alle Feldformate sind maschinell geprüft.** Das Schema definiert mehr, als der Linter
derzeit kontrolliert — die Tracker-Feldformate etwa sind skillseitig verbindlich, aber noch nicht
im Prüfskript umgesetzt. Diese Lücke ist im Schema markiert und nicht versteckt.

**Legacy wird nicht nachgezogen.** Bestandsdokumente aus Bereichen ohne aktuelles Deputat
bleiben, wie sie sind. Ein vollständiges Retrofit hätte Wochen gekostet und keinen
Unterrichtswert gehabt.

---

## 15 Was diese Dokumentation nicht enthält

Beschrieben ist das System, nicht sein Inhalt. Bewusst ausgenommen sind:

- **Konkrete Reihen, Lerngruppen und Stundenpläne** aus dem laufenden Betrieb. Alle Beispiele in
  diesem Dokument sind schematisch.
- **Aufgabenstellungen aus Lernkontrollen und Erwartungshorizonten.** Beschrieben ist, wie sie
  angelegt und ausgewertet werden — nicht, was in ihnen steht.
- **Der Wortlaut von Durchlaufreflexionen.** Beschrieben ist der Mechanismus und wofür die
  Einträge verwendet werden.
- **Interne Infrastruktur** der Schule: Server, Freigabewege, Zugänge.

Wer zu einem dieser Punkte etwas wissen möchte, spricht mich direkt an.
