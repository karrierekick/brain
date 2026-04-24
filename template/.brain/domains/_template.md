---
domain: _template
status: aktiv
modified: YYYY-MM-DD
review_after: YYYY-MM-DD
tags: []
watched_paths: []
---

<!--
status-Werte:
  aktiv       – gepflegt, Grundlage für Entscheidungen
  beta        – in Entwicklung, Details können sich ändern
  stub        – Domain bekannt, Inhalt unvollständig (kurzes review_after)
  stale       – bewusst nicht weiterentwickelt; vorhandenes Wissen noch nützlich, aber nicht Grundlage für neue Entscheidungen
  eingestellt – Feature abgelöst/entfernt, Datei nur für Historie

watched_paths:
  Glob-Liste der Code-Pfade, die diese Domain betreffen (relativ zum Projekt-Root).
  `brain audit` prüft per Git, ob seit `modified` an diesen Pfaden committed wurde,
  und markiert die Datei dann als review-fällig.
  Beispiel:
    watched_paths:
      - frontend/src/stores/userSettings.ts
      - frontend/src/css/themes/**
      - backend/app/Http/Controllers/UserSettingController.php
-->

## Ziel
Ein Satz: Was soll dieses Feature erreichen?

## Review-Auslöser
Freitext als Ergänzung zu `watched_paths`: fachliche Auslöser, die per Glob nicht erkennbar sind (z. B. „externe API-Vertragsänderung", „Produktentscheidung zu Rollenmodell"). Für dateibezogene Trigger ist `watched_paths` führend.

## Einstiegspunkte
- Frontend: `src/...`
- Backend: `app/...`

## Nicht verwechseln mit
Abgrenzung zu ähnlichen Konzepten, Dateien oder Domänen die leicht verwechselt werden.

## Zusammenhänge
Querverweise auf andere Domänen mit [[wiki-link]]. Einträge sind **bidirektional**: wenn diese Domain auf Domain B verweist, muss Domain B auch zurückverweisen. Stil: aufgabenorientiert — nicht „betrifft X", sondern „wenn Du an Y arbeitest, auch [[X]] laden".

## Design-Entscheidungen
Bewusste Entscheidungen mit Begründung, die nicht aus dem Code erkennbar sind.

## Harte Constraints
Was gilt explizit und darf nicht gebrochen werden. Was wird bewusst vermieden.

## Typische Fehlannahmen
Konzeptuelle Missverständnisse die immer wieder zu falschen Lösungen führen – nicht das gleiche wie Stolpersteine.

## Stolpersteine
Nicht-offensichtliche technische Fallstricke bei der Implementierung.
