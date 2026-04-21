# Brain – Design-Rationale

Dieses Dokument erklärt das **Warum** hinter den strukturellen Entscheidungen des Brain-Systems. Für Installation und Nutzung siehe [README.md](./README.md). Für Versionshistorie siehe [CHANGELOG.md](./CHANGELOG.md).

## Grundprinzip

Brain speichert ausschließlich Wissen, das **nicht aus dem Code ableitbar** ist: Design-Entscheidungen mit Begründung, bewusst abgelehnte Alternativen, fachliche Constraints, Modul-Zusammenhänge. Alles was `grep` oder eine Handvoll Datei-Reads beantworten können, gehört nicht rein.

Konsequenz: Brain ist klein. Eine Domain-Datei mit 40 Zeilen ist oft wertvoller als eine mit 400.

## Schicht-Aufteilung

| Schicht | Inhalt | Datei |
|---|---|---|
| User-Doku | Install, Kommandos, Quickstart | `README.md` |
| Architektur-Rationale | Warum ist das System so gebaut | `DESIGN.md` (diese Datei) |
| Version-Journal | Breaking Changes, Migration | `CHANGELOG.md` |
| KI-Operationsvertrag | Lesestrategie, Trigger, Priorisierung | `template/.brain/PROTOCOL.md` |
| Install/Upgrade-Logik | Dateien kopieren, Framework vs. Daten trennen | `SKILL.md` |

Bewusste Entscheidung: **keine eigene `.brain/`-Domain für Brain selbst**. Das Framework ist klein genug, dass zusätzliche Selbstreflexion Overhead ohne Nutzen wäre. Architektur-Wissen gehört in `DESIGN.md`, nicht in eine meta-Domain.

## Strukturelle Entscheidungen

### Passive → aktivitätsbasierte Reviews (v2.1)

**Problem bis v2.0:** `review_after` war rein zeitbasiert und `Review-Auslöser` war Freitext. Eine Domain wurde nur review-fällig, wenn ihr Datum abgelaufen war – egal wie viel am unterlegenden Code passiert war. Umgekehrt wurde sie nie review-fällig, wenn nichts passierte, obwohl der Code längst divergiert war.

**Konkretes Beispiel:** In einem produktiven Projekt war der Theme-Katalog in der Domain-Datei bei zwei Werten stehengeblieben, obwohl ein dritter längst im Code existierte. Das `review_after` war noch nicht abgelaufen – also hat niemand hingeschaut.

**Lösung:** Strukturiertes Frontmatter-Feld `watched_paths` als Glob-Liste. `brain audit` prüft per `git log --since=<modified> -- <paths>` auf Commits seit der letzten Pflege und markiert Domains aktivitätsbasiert. Freitext-`Review-Auslöser` bleibt für fachliche Auslöser (externe API-Änderung, Produktentscheidung), die per Glob nicht erkennbar sind.

**Alternative verworfen:** Filesystem-Hash statt Git. Abgelehnt, weil Git-basiert kostenlos ist, chronologische Evidenz liefert (Commit-Datum, Autor) und Merge/Rebase-stabil.

### Reverse-Index Code → Domain (v2.1)

**Problem:** Die KI muss aus einer Code-Aufgabe raten, welche Domain greift. Der Domain-Index in `CLAUDE.md` ist nach Begriffen sortiert – aber eine „CSS-Anpassung an Notifications" matcht auf kein offensichtliches Begriffsfeld, obwohl die Antwort in der `user-settings`-Domain liegt.

**Lösung:** `.brain/_index.json` wird bei jedem Audit aus allen `watched_paths` generiert. Vor der Code-Arbeit kann die KI den Pfad nachschlagen, bevor sie in den Begriffs-Index springt.

**Alternative verworfen:** Manuelle Pflege durch Autoren. Synchronisationsaufwand wäre zu hoch, Index würde zwangsläufig veralten.

### Status-Erweiterung: `stub` und `stale` (v2.1)

**Problem:** `aktiv | beta | eingestellt` unterscheidet nicht zwischen „Domain existiert, aber Inhalt ist lückenhaft" (stub) und „Feature existiert, wird aber bewusst nicht gepflegt" (stale). Beides war vorher unter `aktiv` subsumiert und führte zu falschem Vertrauen in die Vollständigkeit.

**Lösung:**
- `stub`: Domain bekannt, Inhalt unvollständig. Kurzes `review_after` (30 Tage Default), damit sie zur Vervollständigung wiederkommt.
- `stale`: Feature lebt, Domain bewusst nicht weitergepflegt. Langes `review_after` (180 Tage), KI darf die Datei als Kontext nutzen, aber **nicht als Grundlage für neue Design-Entscheidungen**.

### Capture als Pflicht nach Code-Arbeit (v2.1)

**Problem:** `brain capture` war „wenn substantielle Erkenntnisse". Das ist ein schwacher Trigger – die KI neigt dazu, eigene Erkenntnisse zu unterschätzen („war doch nur CSS").

**Lösung (v2.1):** Pflicht-Vorschlag am Session-Ende bei `watched_paths`-Treffer.
**Lösung (v2.2):** Pflicht-Vorschlag durch **automatischen Lauf** ersetzt – siehe nächster Abschnitt.

### Auto-Capture mit strengem Mehrwert-Filter (v2.2)

**Problem bis v2.1:** Selbst „Pflicht-Vorschlag" erzeugte jedes Mal einen Bestätigungs-Dialog: KI formuliert Kandidaten, User liest, bestätigt. Bei disziplinierter Filterung auf KI-Seite ist das ein Leerlauf-Dialog, der Zeit und Aufmerksamkeit kostet. Gleichzeitig führte der Dialog zu Laxheit beim Filter („der User entscheidet ja nochmal").

**Lösung:** `capture.confirmDefault` im Default auf `false`. Capture läuft autonom, **aber mit härterem Aufnahmefilter**: jeder Kandidat muss in einem Satz benennen können, welche der fünf Mehrwert-Kategorien er erfüllt – Recherche-Ersparnis, Token-Reduktion, besserer Kontext, höhere Codequalität, oder explizite Design-Entscheidung. Ohne klare Mehrwert-Kategorie: `reject`.

Die Härte des Filters ist die Bedingung für den Entfall der Bestätigung, nicht ihr Ersatz. `brain capture` bricht nur noch in einem Fall aus: **ungeklärter Konflikt**, bei dem die Prioritätshierarchie keinen eindeutigen Sieger ergibt.

**Alternative verworfen:** Weicher Filter + Bestätigung weiterhin. Abgelehnt, weil der Bestätigungs-Dialog die Disziplin beim Filter untergraben hat und die meisten Bestätigungen ohnehin ohne Änderung durchgewunken wurden.

**Nebenwirkung:** Knapper Session-End-Output („2 Einträge, 5 verworfen, state → ok" oder „skipped"). Keine Capture-Zeremonie im Chat.

### Skip-Regeln für einfache Aufgaben (v2.2)

**Problem:** Die Brain-Lektüre war in v2.1 reflexiv Pflicht, sobald irgendein `watched_paths`-Treffer vorlag – auch bei Tippfehler-Fixes oder trivialen Änderungen. Der Overhead (Domain-Datei laden, Glossar konsultieren) rechnete sich in diesen Fällen nicht.

**Lösung:** Expliziter Abschnitt „Wann Brain-Lektüre übersprungen werden darf" in `PROTOCOL.md`: Ein-Stellen-Änderungen, vom User explizit benannte Dateien, explorative Such-Aufgaben, reine Code-Zustands-Fragen, projekt-agnostische Snippet-Requests. Bei Domain-Logik, Design-Entscheidungen oder fachlichen Begriffen bleibt die Domain-Datei Pflichtlektüre.

**Leitbild:** Brain ist ein Werkzeug gegen falsche Annahmen, kein Ritual. Einfache Aufgaben ohne fachliche Entscheidung brauchen es nicht.

## Bewusst nicht gebaut

- **Automatischer Write ohne Qualitätsfilter:** Brain schreibt autonom (v2.2), aber nur nach strenger Prüfung gegen die Mehrwert-Kategorien. Ungefilterter Auto-Dump ist explizit nicht gewollt.
- **Zentrale Server-Komponente:** Alles dateibasiert, git-versioniert. Portabilität vor Features.
- **Team-Features (Kommentare, Zuweisung, Historie):** Dafür gibt es Git. Brain-Dateien sind normale Markdown-Dateien.
- **Schema-Validator / Lint-Tool:** Prüfung läuft über `brain audit`, nicht über CI-Pipeline. Absichtlich minimal.

## Wann Brain nicht passt (Projekt-Ebene)

- **Ein-Datei-Scripts / Einzelaufgaben:** Overhead der Session-Einstiegskosten rechnet sich nicht.
- **Projekte ohne stabile Domänen-Struktur** (Prototypen mit wechselnder Architektur): Brain erzwingt keine verfrühte Strukturierung, wird aber auch nicht aktiv helfen.
- **Einzelentwickler-Projekte mit gutem Kopf-Gedächtnis:** Kosten-Nutzen kritisch prüfen; ggf. reicht `CLAUDE.md`.

## Wann Brain-Lektüre übersprungen wird (Aufgaben-Ebene)

Seit v2.2 formalisiert in `PROTOCOL.md`: trivialer Change ohne fachliche Entscheidung, User nennt Dateien explizit, explorative Code-Suche, reine Zustands-Fragen. Siehe Abschnitt „Wann Brain-Lektüre übersprungen werden darf" in `PROTOCOL.md`.
