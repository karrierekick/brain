# Brain System

Ein portables KI-Wissensbasis-System für Cursor-Projekte.

- **Architektur-Rationale:** [DESIGN.md](./DESIGN.md) – warum das System so gebaut ist
- **Versionshistorie:** [CHANGELOG.md](./CHANGELOG.md) – Änderungen und Migration

## Was es macht

Strukturierte Wissensbasis pro Projekt – speichert nur was nicht direkt aus dem Code ablesbar ist: Design-Entscheidungen, Constraints, Modul-Zusammenhänge, Begriffe.

Ab v2.1 **aktivitätsbasiert**: Domains deklarieren `watched_paths` (Glob-Liste). `brain audit` prüft per Git, welche Domains seit der letzten Pflege Code-Änderungen haben, und markiert sie als review-fällig – nicht erst wenn ein Kalenderdatum abläuft.

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

| Befehl | Was passiert |
|---|---|
| `brain install` | Framework-Dateien ins Projekt kopieren |
| `brain init` | Projekt erschließen – Scan + Interview + Wissensbasis aufbauen |
| `brain capture` | Erkenntnisse aus der Session festhalten |
| `brain audit` | Wissensbasis auf Aktualität und Konsistenz prüfen |
| `brain upgrade` | Framework-Dateien aktualisieren (Projekt-Daten bleiben) |

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
