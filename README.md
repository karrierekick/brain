# Brain System

Ein portables KI-Wissensbasis-System für Cursor-Projekte.

## Was es macht

Strukturierte Wissensbasis pro Projekt – speichert nur was nicht direkt aus dem Code ablesbar ist: Design-Entscheidungen, Constraints, Modul-Zusammenhänge, Begriffe.

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
  SKILL.md              ← Install/Upgrade-Logik
  template/
    .brain/
      PROTOCOL.md       ← Lese-Strategie und Operationen
      prompts/
        init.md         ← brain init
        capture.md      ← brain capture
        audit.md        ← brain audit
      domains/
        _template.md    ← Vorlage für Domain-Dateien
      config.json       ← Blank-Template
      state.json        ← Blank-Template
    .cursor/
      rules/
        brain-protocol.mdc
```
