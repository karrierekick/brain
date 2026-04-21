# Brain Protocol

Dieses Projekt hat eine Wissensbasis unter `.brain/`. Sie enthält ausschließlich Wissen, das nicht sauber aus dem Code ableitbar ist und das Fehlannahmen, Rückfragen oder Fehländerungen reduziert: Design-Entscheidungen, Feature-Ziele, bewusste Constraints, Modul-Zusammenhänge, Begriffsdefinitionen.

Was direkt und sicher aus dem Code ablesbar ist, gehört nicht hier.

## Wann Brain-Lektüre übersprungen werden darf

Brain ist ein Werkzeug gegen falsche Annahmen, nicht ein Pflicht-Ritual. Für einfache Aufgaben erzeugt die Lektüre mehr Overhead als Nutzen. Überspringen ist explizit erlaubt bei:

- **Ein-Stellen-Änderungen ohne fachliche Entscheidung** – Tippfehler, Formatierung, Kommentare, Variablen-Rename in eng umrissenem Scope.
- **Vom User explizit benannte Dateien** – wenn der User konkrete Pfade oder Funktionen nennt, die verändert werden sollen, und die Aufgabe keine Domain-Logik berührt.
- **Explorative Aufgaben, bei denen relevante Dateien ohnehin individuell gesucht werden müssen** – z. B. „wo wird X eingesetzt?", Bug-Hunts, die Grep/Read als Primärwerkzeug haben.
- **Code-Zustands-Fragen** – „was macht Funktion Y?", „welche Felder hat Tabelle Z?" sind aus dem Code direkt beantwortbar.
- **Generische Coding-Tasks ohne Projekt-Bezug** – Snippet-Requests, die auf keine Domain des Projekts zielen.

**Nicht überspringen** bei: Domain-Logik-Änderungen, Design-Entscheidungen, Umbauten an `watched_paths`-Dateien, fachlichen Fragen zu Begriffen oder Flows, oder wenn aus der Aufgabe Begriffe aus dem Domain-Index vorkommen.

Im Zweifel lieber die eine relevante Domain-Datei öffnen als blind loslegen – aber nicht reflexhaft drei Dateien lesen, wenn die Aufgabe trivial ist.

## Lesestrategie (wenn Brain relevant ist)

1. `CLAUDE.md` immer lesen – Projektüberblick, Design-Prinzipien, Domain-Index
2. Domain-Index in `CLAUDE.md` prüfen – passende Domain-Datei aus `.brain/domains/` laden wenn Trigger zutrifft
3. `.brain/glossary.md` laden wenn Begriffe unklar sind oder Fachvokabular auftaucht
4. Maximal 2–3 Zusatzdateien laden, bevor in den Code gegangen wird
5. Erst bei Lücken Code lesen

Nicht alle Domain-Dateien laden. Nur die eindeutig relevante.

## Kanonische Wahrheit

Wenn Aussagen kollidieren, gilt diese Reihenfolge:

1. Explizite User-Aussage in der aktuellen Session
2. Verifiziertes Verhalten aus Code oder Laufzeit
3. Bestehender bestätigter Brain-Eintrag
4. README / AGENTS / andere Projektdoku
5. Inferenz aus Kontext

Konflikte nie stillschweigend auflösen.

## Knowledge-Operationen

Framework-Operationen – ausgelöst durch User-Befehl oder Auto-Trigger:

- `brain init` – Projekt initial erschließen (`.brain/prompts/init.md`)
- `brain capture` – Erkenntnisse aus dieser Session festhalten (`.brain/prompts/capture.md`)
- `brain audit` – Wissensbasis prüfen und bereinigen (`.brain/prompts/audit.md`)

**Projekt-spezifische Operationen:** Jede weitere Datei `.brain/prompts/<befehl>.md` definiert automatisch `brain <befehl>`. Damit der Trigger zuverlässig erkannt wird, den Befehl zusätzlich hier in der Liste und in der `description` von `.cursor/rules/brain-protocol.mdc` eintragen.

**Trigger-Hinweise (aktiv, nicht optional):**

- **Bearbeitete oder gelesene Datei matcht `watched_paths` einer Domain** → diese Domain ist **Pflichtlektüre** vor der Code-Arbeit. Nicht per Glob/Grep rekonstruieren, was die Domain bereits kuratiert beantwortet. Ausnahme: gilt die Skip-Regel oben (einfache Aufgabe), reicht ein Blick auf das Frontmatter.
- **Session hat Code geändert, der zu `watched_paths` einer Domain passt** → am Session-Ende **läuft `brain capture` automatisch** (`confirmDefault: false` ab v2.2). Kein Pflicht-Vorschlag, keine Rückfrage – direkt ausführen. Der Prompt filtert streng (siehe `capture.md`), leere Durchläufe enden mit `lastResult: "skipped"`.
- Session mit substantiellen Erkenntnissen ohne Code-Änderung → ebenfalls Auto-Run von `brain capture` am Session-Ende.
- `audit.lastRun` + `audit.intervalDays` überschritten → hinweisen: „Ein Audit steht an."
- Geänderter oder untersuchter Code passt zu **keiner** Domain-Datei → Coverage-Lücke benennen und Capture/Audit vorschlagen.
- `brain init` nie automatisch vorschlagen – nur auf expliziten Wunsch.

**Discovery-Hilfe:** Wenn unklar ist, welche Domain für eine Aufgabe greift, generiert `brain audit` einen Reverse-Index `.brain/_index.json` (Pfad → Domain). Wenn vorhanden, zuerst dort nachschlagen, dann Domain-Index in `CLAUDE.md`.

## Pflege-Loop

- `brain init` erzeugt oder aktualisiert `CLAUDE.md`, `.brain/glossary.md`, Domain-Dateien, `.brain/config.json` und `.brain/state.json`.
- `brain capture` aktualisiert betroffene Wissensdateien, setzt `modified` / `review_after` und schreibt den Capture-Status nach `.brain/state.json`. Im Auto-Mode ohne Bestätigung; nur ungeklärte Konflikte pausieren den Lauf.
- `brain audit` prüft Inhalte und Basis-System gemeinsam: Domains, Glossar, Template-/Prompt-Konsistenz, State-Pflege, **`watched_paths`-Git-Aktivität**, Coverage.
- Review-Fälligkeit entsteht aus **zwei** Quellen:
  1. `review_after` überschritten (zeitbasiert)
  2. Git-Commits seit `modified` an Pfaden in `watched_paths` (aktivitätsbasiert)

## State-Semantik

- `lastRun` ist der Zeitstempel des **erfolgreich abgeschlossenen** Durchlaufs einer Knowledge-Operation.
- `lastRun` wird **am Ende** geschrieben, nachdem alle Wissensdateien erfolgreich aktualisiert wurden.
- Abgebrochene oder nicht bestätigte Durchläufe schreiben keinen neuen `lastRun`.
- Ein leerer Auto-Capture-Durchlauf (keine `accept`-Kandidaten) setzt `lastRun` und `lastResult: "skipped"` – das ist erfolgreich.
- Wenn ein historischer `lastRun` unbekannt ist, darf er bei einer bestätigten Re-Konsolidierung auf den aktuellen Stand gesetzt werden.

## Grundregeln

- Kein Code schreiben während einer Knowledge-Operation.
- Unsicherheit nie stillschweigend auflösen – kennzeichnen oder (bei Konflikten) nachfragen.
- Das System ist im Auto-Mode **selbstpflegend unter striktem Qualitätsfilter**: die KI nimmt nur auf, was einen der fünf Mehrwert-Kriterien (siehe `capture.md`) eindeutig erfüllt. Im Zweifel verwerfen, nicht verwässern.
- Offensichtliche Strukturkorrekturen (Pfade, Links, Tippfehler) werden ohnehin direkt ausgeführt.
