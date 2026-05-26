# Brain System

Ein portables KI-Wissensbasis-System für Cursor-Projekte.

- **Architektur-Rationale:** [DESIGN.md](./DESIGN.md) – warum das System so gebaut ist
- **Versionshistorie:** [CHANGELOG.md](./CHANGELOG.md) – Änderungen und Migration

## Was Brain ist

**Brain** ist ein **dateibasiertes, git-versioniertes Wissens-Framework** für Cursor/Claude-Projekte. Es kuratiert **implizites Projektwissen**, das im Code nicht zuverlässig steht: fachliche Begriffe, bewusste Design-Entscheidungen, Abgrenzungen zwischen Modulen, harte Constraints und typische Fehlannahmen.

Brain ist **kein** Ersatz für Code-Lektüre, README oder API-Docs. Es ist die **schlanke Schicht darüber**, die eine KI **vor** breitem Glob/Grep laden soll, sobald eine Aufgabe Domänenlogik oder Projekt-Vokabular berührt.

| Baustein              | Rolle für die KI                                                                                              |
| --------------------- | ------------------------------------------------------------------------------------------------------------- |
| `CLAUDE.md`           | Einstieg: Überblick, Prinzipien, **Domain-Index** (Trigger → welche Datei laden)                              |
| `.brain/domains/*.md` | Kuratiertes Domänenwissen; Frontmatter: `watched_paths`, `status`, `modified`                                 |
| `.brain/glossary.md`  | Einheitliche Begriffe                                                                                         |
| `.brain/PROTOCOL.md`  | Vertrag: Lesestrategie, Skip-Regeln, Konflikt-Priorität, Auto-Capture                                         |
| `.brain/_index.json`  | Reverse-Index **Pfad → Domain** (von `brain audit` generiert)                                                 |
| `brain <befehl>`      | Strukturierte Pflege (`init`, `capture`, `audit`; weitere Befehle projekt-spezifisch unter `.brain/prompts/`) |

### Echte Mehrwerte (wann Brain zahlt)

Brain lohnt sich nur, wenn Einträge **mindestens einen** dieser Effekte klar liefern (Aufnahmefilter in `capture.md`):

1. **Recherche-Ersparnis** – erspart wiederholtes Grep/Read (z. B. Theme-Persistenz → Domain statt viele Dateien).
2. **Token-Reduktion** – kurze Domain ersetzt mehrere Tool-Runden.
3. **Besserer Kontext** – Geschäftslogik, Quellen und bewusste Abgrenzungen explizit (z. B. zwei ähnliche Entitäten, die nicht verwechselt werden dürfen).
4. **Höhere Codequalität** – Constraints verhindern falsche Änderungen (z. B. wo Geschäftslogik liegt, welche Modelle Soft Deletes nutzen).
5. **Explizite Design-Entscheidungen** – Warum und was bewusst **nicht** gebaut ist.

**Nicht** aufnehmen: Code-Zustände, Framework-Standardwissen, Session-Notizen, Dateiaufzählungen ohne Semantik. Brain ist ein Werkzeug gegen **falsche Annahmen**, kein Pflicht-Ritual bei trivialen Änderungen (Skip-Regeln in `PROTOCOL.md`).

### Lesereihenfolge bei relevanter Aufgabe

1. `CLAUDE.md` → Domain-Index oder `_index.json` (bei konkretem Pfad)
2. Passende `.brain/domains/<name>.md` (max. 2–3 Brain-Dateien insgesamt)
3. Optional `.brain/glossary.md`
4. Dann gezielt Code lesen – nicht Brain per Glob/Grep rekonstruieren
5. Nach Änderungen an `watched_paths`: **Auto-`brain capture`** am Session-Ende (strenger Mehrwert-Filter; nur bei Konflikten Rückfrage)

Ab v2.1 **aktivitätsbasierte Pflege**: `brain audit` vergleicht Git-Commits an `watched_paths` mit `modified` und markiert Domains als review-fällig – unabhängig vom Kalender-`review_after`.

## Installation (einmalig pro Rechner)

```bash
git clone git@github.com:karrierekick/brain.git ~/.cursor/skills/brain
```

## Nutzung in einem Projekt

In Cursor schreiben:

```
brain install
```

Dann:

```
brain init
```

## Befehle

| Befehl          | Was passiert                                                   |
| --------------- | -------------------------------------------------------------- |
| `brain install` | Framework-Dateien ins Projekt kopieren                         |
| `brain init`    | Projekt erschließen – Scan + Interview + Wissensbasis aufbauen |
| `brain capture` | Erkenntnisse aus der Session festhalten                        |
| `brain audit`   | Wissensbasis auf Aktualität und Konsistenz prüfen              |
| `brain upgrade` | Framework-Dateien aktualisieren (Projekt-Daten bleiben)        |

## Update (alle Rechner)

```bash
cd ~/.cursor/skills/brain
git pull
```

Dann in jedem Projekt das aktualisiert werden soll:

```
brain upgrade
```

## Struktur

```
~/.cursor/skills/brain/
  README.md             ← Install, Kommandos, Quickstart (diese Datei)
  DESIGN.md             ← Architektur-Rationale, Design-Entscheidungen
  CHANGELOG.md          ← Versionshistorie, Migration zwischen Versionen
  SKILL.md              ← Install/Upgrade-Logik
  template/
    .brain/
      PROTOCOL.md       ← Lese-Strategie und Operationen (KI-Vertrag)
      prompts/
        init.md         ← brain init
        capture.md      ← brain capture
        audit.md        ← brain audit (git-aware ab v2.1)
      domains/
        _template.md    ← Vorlage für Domain-Dateien (watched_paths ab v2.1)
      config.json       ← Blank-Template (version 2.1)
      state.json        ← Blank-Template
    .cursor/
      rules/
        brain-protocol.mdc
```

## Migration zu v2.1

Bestehende Installationen:

```
brain upgrade     # Framework-Dateien aktualisieren (Domain-Daten bleiben)
brain audit       # zeigt welche Domains watched_paths noch fehlen
```

Dann pro Domain `watched_paths` im Frontmatter ergänzen (meist direkt aus den „Einstiegspunkten" ableitbar). Details: [CHANGELOG.md](./CHANGELOG.md#migration-20--21).
