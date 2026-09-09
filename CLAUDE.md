# StudyLog-Mext — Projektkontext für Claude

## Was ist das
Progressive Web App zur Durchführung von Studien-Sessions mit Teilnehmenden in VR-Szenarien.
Abgeleitet von StudyLog-V2, angepasst für einen speziellen Verwendungszweck mit erweitertem
Funktionsumfang (siehe "Neue Anforderungen" unten). Mehrere Studienleitungen nutzen die App
gleichzeitig auf eigenen Smartphones, vollständig offline. Deployment: GitHub Pages
(Repo: `studylog-mext`).

## Tech-Stack
Vanilla HTML/CSS/JS, kein Build-Schritt, keine Frameworks/Dependencies. Datenhaltung
ausschließlich lokal via `localStorage`. Offline-Fähigkeit über Service Worker (`sw.js`).

## Neue Anforderungen (Abweichungen von StudyLog-V2)

Diese vier Punkte sind der Grund für die Ausgliederung in ein eigenes Repo. Details zur
Umsetzung (Datenmodell, UI) sind noch offen und werden iterativ mit Claude besprochen.

1. **Neuer Tab "Sensorik"**: Checkliste für Sensorik-Items, jedes Item abhakbar; beim Abhaken
   wird automatisch ein Timestamp erfasst (wann welche Sensorik angelegt wurde). Analog zum
   bestehenden Prinzip der Deviation-Tags, aber als eigener Tab mit eigenem Datenmodell.
2. **VR-Szenario-Ablauf mit Timestamps**: Der Ablauf innerhalb eines Szenarios soll in
   einzelne Schritte unterteilt und jeweils mit Timestamp abhakbar sein (Erweiterung des
   bestehenden Session-Timers, der bisher nur Start/Ende eines Szenarios erfasst).
3. **Trainerbewertungsbogen mehrfach pro Session**: Der bestehende 20-Item-Bewertungsbogen
   soll nicht mehr einmal pro Session, sondern einmal **pro VR-Szenario** ausgefüllt werden
   können — d.h. eine Session mit mehreren Szenarien erzeugt mehrere Bewertungsbögen,
   zugeordnet zu Teilnehmer*in + Szenario.
4. **Auswahl Links-/Rechtshänder**: Neues Attribut in der Teilnehmendenverwaltung (vermutlich
   relevant für Sensorplatzierung).

## Datenschutz (hart, nicht verhandelbar)
- Keine Daten verlassen das Gerät — kein externer Server, kein Tracking, keine Analytics.
- Pseudonymisierung nach Art. 4 Nr. 5 DSGVO.
- Gendergerechte Sprache im UI: "Teilnehmende", nicht "Probanden" (Variablennamen im Code
  dürfen weiterhin `Proband*` heißen — nur sichtbare UI-Texte müssen genderneutral sein).
- Handedness-Auswahl: keine besondere Sensitivität, aber wie alle Teilnehmendendaten
  ausschließlich lokal speichern.

## Nicht verhandelbare Arbeitsregeln
1. **Strict non-regression**: Änderungen sind rein additiv oder visuell. Bestehende
   Funktionalität darf nie brechen oder sich unangekündigt ändern.
2. **Minimale Diffs**: Kein Refactoring "nebenbei". Wenn ein Bug auftritt: auf den letzten
   bestätigt stabilen Stand zurück, dann nur die minimal nötige Änderung.
3. **Root Cause statt Symptom-Fix**: Ursache systematisch diagnostizieren, keine
   Vermutungs-Fixes.
4. Vor dem Debuggen von Deployment-Fehlern: https://githubstatus.com prüfen (in der
   Vergangenheit gab es dadurch False-Positive-Rabbit-Holes).

## Versionierung (WICHTIG — bei jeder inhaltlichen Änderung)
Single Source of Truth: `const APP_VERSION` ganz oben in `app.js`.

Bei **jedem Commit, der Funktionalität/Inhalt ändert** (nicht bei reinen Doku-Änderungen):
1. `APP_VERSION` in `app.js` hochzählen — Patch (`2.2.1` → `2.2.2`) für Bugfixes/kleine
   Änderungen, Minor (`2.2.x` → `2.3.0`) für neue Features, Major nur nach expliziter Absprache.
2. `CACHE`-Konstante in `sw.js` synchron auf denselben Wert setzen (z.B.
   `studylog-mext-v2.3.0`) — erzwingt Invalidierung des alten Service-Worker-Caches.
3. Die statischen `<span class="app-version">` Platzhalter in `index.html` (aktuell 2x:
   Sidebar-Footer + mobile Topbar) auf denselben Wert setzen — sie werden zusätzlich beim
   Laden per JS aus `APP_VERSION` überschrieben (Zeile mit
   `document.querySelectorAll('.app-version')...` im INIT-Block von `app.js`), das ist nur
   der No-Flash-Fallback für den ersten Paint.

## Bekannte, bereits gelöste Bugs (nicht wiederholen)
- iOS Safari PWA: `<div>` als Klick-Ziel funktioniert nicht zuverlässig → immer `<button>`.
- iOS Zoom bei Input-Fokus: `font-size: 16px` auf Inputs + Viewport-Meta-Fix.
- Samsung Android Touch-Bug: `maximum-scale=1.0, user-scalable=no` aus Viewport entfernen.
- Service-Worker-Deadlock: `sw.js` darf sich nicht selbst in `ASSETS` cachen.
- Confirm-Dialog z-index: `#confirm-overlay` braucht `z-index: 300`.
- `.hidden`-Klasse muss `display: none !important` sein — `opacity:0; pointer-events:none`
  lässt unsichtbare Elemente Klicks blockieren.
- Alles JS in `DOMContentLoaded` wrappen (sonst iOS-PWA-Button-Fails).
- Delete-Bug-Muster: IDs vor dem Nullen in `const` zwischenspeichern.

## Git-Workflow in dieser Umgebung
Der verbundene Ordner läuft über eine Sandbox-Bridge, die zwar Dateien schreiben, aber keine
Dateien löschen/umbenennen kann — `git commit`/`git push` funktionieren darüber **nicht
zuverlässig** (Lock-Dateien bleiben hängen). Deshalb: Claude bearbeitet Dateien direkt
(Edit/Write), Commit + Push erfolgt lokal in SourceTree oder Git Bash. Claude schlägt dazu
jeweils eine Commit-Message vor.

## Datenschutz für Projektdateien selbst
Keine personenbezogenen Daten (Namen, Arbeitgeber o.ä.) in Code-Kommentare, Doku-Dateien,
Commit-Messages oder sonstige Texte schreiben, die Claude erstellt oder bearbeitet.

## Technische Dokumentation (`/docs`)
Es existiert eine ausführliche technische Dokumentation unter `/docs`
(`OVERVIEW.md`, `DATENFLUSS.md`, `BEDIENUNG.md`, `ARCHITECTURE.md`) — u.a. Grundlage für eine
Datenschutz-Bewertung (DSGVO/DSFA) und für neue Entwickler:innen/KI-Modelle ohne Chatverlauf.
**Bei jeder Code-Änderung, die Datenerfassung, -speicherung, -übertragung, Architektur oder
Bedienung betrifft, die passende(n) Datei(en) in `/docs` automatisch mitaktualisieren** —
ohne dass extra danach gefragt werden muss. Unklare Datenschutz-Aspekte in `DATENFLUSS.md`
weiterhin mit "TODO: Datenschutz prüfen" markieren. **Bei den vier neuen Anforderungen oben
gilt dasselbe: sobald Umsetzungsdetails feststehen, entsprechende /docs-Dateien ergänzen.**
