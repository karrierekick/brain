# Brain Skill

Installiert und aktualisiert das Brain-System in einem Projekt.

## Verwende diesen Skill wenn der User schreibt:
- `brain install`
- `brain upgrade`

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
- `CLAUDE.md`
- `PRIVATE.md`

### Ablauf

1. Prüfe ob `.brain/` existiert. Wenn nicht: Hinweis "Brain nicht installiert. Nutze `brain install`."
2. Lese die aktuelle `version` aus `.brain/config.json`.
3. Kopiere alle Framework-Dateien aus dem Template-Ordner ins Projekt.
4. Melde welche Dateien aktualisiert wurden.
