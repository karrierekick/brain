# Brain Skill

Installiert und aktualisiert das Brain-System in einem Projekt.

## Verwende diesen Skill wenn der User schreibt:
- `brain install`
- `brain upgrade`

## Andere brain-Befehle (nicht dieser Skill)

Knowledge-Operationen wie `brain init`, `brain capture`, `brain audit` und projekt-spezifische Befehle (z. B. `brain check`) werden **nicht** von diesem Skill behandelt. Sie werden über die projekteigene Cursor-Rule `.cursor/rules/brain-protocol.mdc` und die Prompt-Dateien unter `.brain/prompts/*.md` ausgelöst.

Wenn der User einen solchen Befehl nutzt und nichts passiert: Prüfe ob `.brain/` existiert. Wenn ja → die zugehörige Prompt-Datei lesen und ausführen. Wenn nein → auf `brain install` hinweisen.

## brain install

Installiert das Brain-System in das aktuelle Projekt.

### Ablauf

1. Prüfe ob `.brain/` bereits existiert.
   - Wenn ja: Abbrechen und hinweisen: "Brain ist bereits installiert. Nutze `brain upgrade` um das Framework zu aktualisieren."
   - Wenn nein: weitermachen.

2. Kopiere alle Dateien aus `C:\Users\danie\.cursor\skills\brain\template\` ins Projekt-Root.
   - `.brain/PROTOCOL.md`
   - `.brain/prompts/init.md`
   - `.brain/prompts/capture.md`
   - `.brain/prompts/audit.md`
   - `.brain/domains/_template.md`
   - `.brain/config.json`
   - `.brain/state.json`
   - `.brain/.gitignore`
   - `.cursor/rules/brain-protocol.mdc`

3. Setze in `.brain/config.json` das Feld `"project"` auf den Namen des aktuellen Ordners (nicht den vollen Pfad).

4. Melde: "Brain installiert. Führe `brain init` aus um die Wissensbasis aufzubauen."

---

## brain upgrade

Aktualisiert nur die Framework-Dateien – Projekt-Daten bleiben unberührt.

### Framework-Dateien (werden überschrieben)
- `.brain/PROTOCOL.md`
- `.brain/prompts/init.md`
- `.brain/prompts/capture.md`
- `.brain/prompts/audit.md`
- `.brain/domains/_template.md`
- `.cursor/rules/brain-protocol.mdc`

### Nicht anfassen (Projekt-Daten)
- `.brain/config.json`
- `.brain/state.json`
- `.brain/glossary.md`
- `.brain/domains/*.md` (außer `_template.md`)
- `.brain/prompts/*.md` außer den vier Framework-Prompts oben (projekt-spezifische Befehle wie `check.md` bleiben erhalten)
- `CLAUDE.md`
- `PRIVATE.md`

### Ablauf

1. Prüfe ob `.brain/` existiert. Wenn nicht: Hinweis "Brain nicht installiert. Nutze `brain install`."
2. Lese die aktuelle `version` aus `.brain/config.json`.
3. Kopiere die Framework-Dateien (siehe Liste oben) aus dem Template-Ordner ins Projekt. Projekt-spezifische Prompts in `.brain/prompts/` nicht überschreiben.
4. Melde welche Dateien aktualisiert wurden.

---

## Projekt-spezifische brain-Befehle

Jedes Projekt kann eigene Knowledge-Operationen definieren, indem es eine Prompt-Datei unter `.brain/prompts/<befehl>.md` anlegt. Beispiel: `brain check` in KATS läuft über `.brain/prompts/check.md`.

Damit der Trigger von Cursor sicher erkannt wird, muss der neue Befehl an zwei Stellen auftauchen:

1. In der Liste der Knowledge-Operationen in `.brain/PROTOCOL.md`
2. In der `description` von `.cursor/rules/brain-protocol.mdc` (damit Cursor die Rule bei diesem Schlagwort lädt)

Diese beiden Einträge sind das, was den Befehl für die KI sichtbar macht – ohne sie wird er ignoriert.
