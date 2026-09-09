# StudyLog-V2 — Architektur (für Entwickler:innen & KI-Modelle)

> Diese Datei ist so geschrieben, dass ein neues Modell **ohne vorherigen Chatverlauf**
> allein daraus produktiv am Projekt weiterarbeiten kann. Für Kontext zu Zweck/Zielgruppe
> siehe [OVERVIEW.md](./OVERVIEW.md), für Datenschutz-Details [DATENFLUSS.md](./DATENFLUSS.md).
> **Verbindliche Arbeitsregeln für Änderungen an diesem Projekt stehen in `/CLAUDE.md`
> im Repo-Root — unbedingt vorher lesen** (u. a. Strict-Non-Regression, minimale Diffs,
> Versionierungspflicht bei jeder inhaltlichen Änderung).

## Ordnerstruktur

```
StudyLog-V2/
├── index.html          # App-Shell: alle Screens + Overlays als statisches Markup
├── style.css            # Gesamtes Styling (Dark-Mode, responsives Layout)
├── app.js                # Gesamte Anwendungslogik (einzige JS-Datei der ausgelieferten App)
├── sw.js                 # Service Worker für Offline-Fähigkeit
├── manifest.json          # PWA-Manifest
├── icons/
│   ├── icon-192.png
│   └── icon-512.png
├── README.md              # Kurzüberblick, Deployment-Anleitung
├── CLAUDE.md               # Verbindliche Arbeitsregeln für Änderungen an diesem Projekt
└── docs/                    # Diese Dokumentation
    ├── OVERVIEW.md
    ├── DATENFLUSS.md
    ├── BEDIENUNG.md
    └── ARCHITECTURE.md
```

**Nicht Teil der ausgelieferten/deployten App** (von `index.html`/`sw.js` nicht referenziert):
`debug.html` und `download` im Repo-Root sind ältere, textuelle Kopien einer früheren
`app.js`-Version (ohne die Bewertungsbogen-Funktion) — vermutlich Debug-/Backup-Artefakte.
Sie werden von `sw.js` nicht gecacht und von `index.html` nicht eingebunden. Vor dem
Löschen/Ändern dieser Dateien Rücksprache halten, da unklar ist, ob sie noch als Referenz
gebraucht werden (siehe Arbeitsregel "Root Cause statt Symptom-Fix" / "nicht ungefragt
löschen" in `CLAUDE.md`). Ebenso `icon-512 (1).png` im Root ist vermutlich ein Duplikat von
`icons/icon-512.png` und nicht Teil des aktiven Manifests.

## Laufzeitmodell

Es gibt **keinen Build-Schritt**. `index.html` lädt `style.css` und `app.js` direkt per
`<link>`/`<script>`-Tag; alle Dateien werden unverändert per GitHub Pages ausgeliefert.
`app.js` ist komplett in einen `DOMContentLoaded`-Listener gewrappt (Pflicht laut
`CLAUDE.md` — verhindert bekannte iOS-PWA-Button-Bugs). Alle Funktionen, State-Variablen und
Event-Listener leben in diesem einen Closure-Scope; es gibt keine Module/Imports.

## Kernkonzept: State + Persistenz

`app.js` hält den gesamten Anwendungszustand in modul-lokalen `let`-Variablen (z. B.
`probanden`, `sessions`, `scenarios`, `tags`, `bewertungen`, `settings` sowie
UI-/Timer-State wie `sessionRunning`, `selectedScenId`, `detailSessionId`). Zwei zentrale
Funktionen synchronisieren diesen State mit `localStorage`:

- **`load()`** — beim Start einmal aufgerufen, liest alle sechs
  `localStorage`-Keys, parsed JSON, füllt fehlende/leere Konfigurationslisten
  (`scenarios`, `tags`) mit Defaults auf und stellt sicher, dass jeder `probanden`-Eintrag
  ein `sensorik`-Objekt hat. Der frühere globale Key `sl_sensorik` (v2.9.0) wird verworfen
  (`localStorage.removeItem`).
- **`save()`** — nach **jeder** datenverändernden Aktion aufgerufen, schreibt alle sechs
  State-Variablen zurück in `localStorage`. Kein Debouncing/Batching — jede einzelne
  Aktion (Person anlegen, Sitzung speichern, Tag umbenennen …) löst einen vollständigen
  `save()`-Durchlauf aus.

**Wichtig für Änderungen:** Da `save()` immer alle sechs Keys neu schreibt, reicht es bei
neuen Feldern, die betroffene State-Variable (z. B. ein Objekt in `probanden`) zu ergänzen —
es muss keine Migration/Schema-Version gepflegt werden. Es gibt **keine
Schema-Versionierung** von `localStorage`-Daten; neue Felder müssen daher stets mit
`undefined`/`null`-Fallbacks für Alt-Daten umgehen können (siehe z. B. `p.sensorAngelegtISO
|| p.createdAt` in `app.js:357`).

### `localStorage`-Keys und Datenmodelle

| Key | State-Variable | Datensatz-Form (wichtigste Felder) |
|---|---|---|
| `sl_probanden` | `probanden` | `{ id, pseudo, sensor, note, handedness, sensorik, sensorAngelegtISO, sensorAbgelegtISO, createdAt }` — `sensor`: Sensoriknummer 1–12 **oder `''`**; seit v2.10.1 **nicht mehr im Anlege-Formular**, im Bearbeiten-Dialog optional (nur bei Eingabe auf 1–12 + Eindeutigkeit geprüft). `handedness`: `'Rechts'` \| `'Links'` — Pflichtfeld beim Anlegen/Bearbeiten (Formular erzwingt eine Auswahl). `sensorik`: Objekt `{ [itemId]: isoString }` — Zeitpunkt „angelegt" je Sensorik-Item (Tab „Sensorik"), fehlender Schlüssel = noch nicht angelegt. Item-IDs/-Labels fest in `SENSORIK_ITEMS` (Shimmer ECG / Shimmer GSR+ / Polar Brustgurt / Garmin). Alt-Daten ohne `handedness`/`sensorik` werden beim Laden als leer normalisiert |
| `sl_sessions` | `sessions` | `{ id, probandId, pseudo, sensor, scenarioId, scenarioName, scenarioAbbr, date, startISO, endISO, duration_s, pauses[], pauseCount, pauseDuration_s, deviations[], notes, deviceLabel, createdAt, editedAt? }` |
| `sl_bewertungen` | `bewertungen` | `{ id, sessionId, pseudo, sensor, scenarioId, scenarioName, scenarioAbbr, date, scores: { a1..z20 }, notes, savedAt }` |
| `sl_scenarios` | `scenarios` | `{ id, name, abbr, icon }` — Default (nur bei leerem `sl_scenarios`): Tutorial / Hologate / Rollercoaster. Über den Szenario-Manager im Szenario-Screen editierbar |
| `sl_tags` | `tags` | `string[]` — freie Liste von Abweichungs-Bezeichnungen |
| `sl_settings` | `settings` | `{ deviceLabel, lastExport, multiProband }` |

**Pausen-Timer (`pauses[]`):** Jeder Eintrag hat die Form `{ startISO, endISO, duration_s }`.
`duration_s` auf Sitzungsebene ist die **aktive Dauer ohne Pausen** — der Timer wird beim
Pausieren eingefroren und beim Fortsetzen exakt an der eingefrorenen Stelle fortgesetzt
(implementiert, indem `timerStart` beim Fortsetzen um die Pausendauer nach vorne verschoben
wird, siehe `resumeTimer()` in `app.js`). `pauseCount`/`pauseDuration_s` sind reine
Bequemlichkeitsfelder (Anzahl bzw. Summe von `pauses[].duration_s`) für Log-Liste und
CSV-Export. Wird eine Sitzung während einer laufenden Pause gestoppt, wird die offene Pause
beim Stoppen automatisch geschlossen (kein hängender/undokumentierter Zeitraum). Die manuelle
Sitzungs-Bearbeitung (`btn-save-edit`) rechnet `duration_s` bei geänderter Start-/Endzeit
weiterhin als rohe Differenz `endISO - startISO` neu — `pauses[]` wird dabei nicht
nachjustiert; das ist ein bekannter Randfall, kein Bug.

Beachte: `sessions`- und `bewertungen`-Einträge speichern `pseudo`/`sensor`/`scenarioName`
**redundant als Kopie** zum Zeitpunkt der Erstellung (statt nur eine `probandId`/`scenarioId`-
Referenz zu halten). Das ist bewusst so gebaut, damit Protokolle auch nach Löschen einer
Person oder eines Szenarios noch lesbar bleiben — hat aber zur Folge, dass ein nachträgliches
Umbenennen eines Szenarios bestehende Sitzungen **nicht** rückwirkend aktualisiert (bei
Personen-Umbenennung dagegen schon, siehe `btn-save-proband-edit`-Handler in `app.js:341`,
der `sessions` aktiv nachzieht). TODO: bei künftigen Änderungen an Szenario-Edit-Funktion
beachten, falls eine Bearbeitungsmöglichkeit für Szenarien ergänzt wird (aktuell gibt es nur
Anlegen/Löschen/Reihenfolge ändern, kein Umbenennen bestehender Szenarien).

**Mehrfachauswahl Teilnehmende (`settings.multiProband`):** Ist diese Einstellung aktiv,
erlaubt der Sitzungsscreen die Auswahl mehrerer `probandId`s gleichzeitig (State-Array
`selectedProbandIds`, UI in `#proband-multi-list` statt des `<select id="sel-proband">`).
Es gibt **kein** neues Datenmodell für "gemeinsame Sitzungen" — beim Speichern
(`btn-save-session`-Handler) wird für **jede** ausgewählte Person ein eigener, vollständig
unabhängiger `sessions`-Eintrag mit eigener `id` erzeugt, alle mit identischem
`startISO`/`endISO`/`duration_s`/`pauses[]`/`scenarioId`/`deviations`/`notes`. Es existiert
also keine Gruppierungs-ID zwischen diesen Einträgen; ein Zusammenhang ist nur implizit über
identische Zeitstempel/Szenario erkennbar. Der Bewertungsbogen-Prompt nach dem Speichern
erscheint immer und schlägt bei mehreren entstandenen Sitzungen eine **gemeinsame** Bewertung
vor (Text unterscheidet Singular/Plural je nach `newSessionIds.length`).

**Gemeinsame Bewertung mehrerer Teilnehmender (Bewertungsbogen):** Ist `settings.multiProband`
aktiv, erlaubt auch der Bewertungsscreen die Auswahl mehrerer Sitzungen gleichzeitig
(State-Array `selectedBewSessionIds`, UI in `#bew-session-multi-list` statt des
`<select id="bew-session-select">` — analog zur Teilnehmenden-Mehrfachauswahl bei der
Sitzungsaufzeichnung, siehe `getSelectedBewSessionIds()`). Auch hier gibt es **kein** neues
Datenmodell: `saveBewertung()` legt für **jede** ausgewählte Sitzung einen eigenen,
unabhängigen `bewertungen`-Eintrag mit identischen `scores`/`notes` an (Überschreiben einer
bereits vorhandenen Bewertung pro Sitzung einzeln). Eine Vorbefüllung mit bereits vorhandenen
Bewertungswerten (beim erneuten Öffnen einer Sitzung) findet nur statt, wenn genau **eine**
Sitzung ausgewählt ist — bei Mehrfachauswahl wäre eine Vorbefüllung aus mehreren
möglicherweise unterschiedlichen Alt-Bewertungen nicht eindeutig, daher startet das Formular
dann leer. Nach dem Speichern einer Sitzungsaufzeichnung mit mehreren Teilnehmenden wird
`pendingBewertungSessionIds` (Array, ersetzt das frühere `pendingBewertungSessionId`) mit allen
neu erzeugten Sitzungs-IDs vorbelegt; klickt man im Prompt auf "Ja", werden diese Sitzungen im
Bewertungsscreen automatisch vorausgewählt (Mehrfachauswahl-Modus) bzw. die einzelne Sitzung im
Dropdown vorausgewählt (Einzel-Modus).

## Modul-Verantwortlichkeiten in `app.js` (in Dateireihenfolge)

| Abschnitt (Kommentar-Marker im Code) | Zeilen (ca.) | Verantwortlichkeit |
|---|---|---|
| App Version / Storage Keys / Defaults | 1–~75 | Versions-Konstante, `localStorage`-Keys, Default-Szenarien/-Tags, `SENSORIK_ITEMS` (feste Sensorik-Item-Liste), State-Deklaration |
| Persistence | 58–92 | `save()`, `load()` |
| Utilities | 92–166 | `uid()`, Datum/Zeit-Formatierung (`formatTime`, `localTimeStr`, `isoToTimeInput`, `rebuildISO`), `esc()` (HTML-Escaping gegen XSS beim Rendern von Nutzereingaben), `showToast()`, `isValidPseudoFormat()`/`setPseudoFieldValidity()` (Pseudonym-Formatprüfung inkl. Live-Rotmarkierung im Formular) |
| Confirm Dialog | 166–183 | Generischer Bestätigungsdialog (`showConfirm`), von mehreren Lösch-Aktionen wiederverwendet |
| Navigation | 183–234 | `showScreen()` (Screen-Wechsel + Re-Render des Zielscreens), Nav-Button-Listener |
| TEILNEHMENDE | 234–~400 | Liste rendern/filtern, Anlegen, Bearbeiten, Löschen von Personen (inkl. Pseudonym-Formatprüfung — live per `input`-Listener und beim Speichern — beim Anlegen/Bearbeiten) |
| SENSORIK-CHECKLISTE (pro Teilnehmende:r) | vor TIMER | `buildSensorikProbandSelect()` (Personen-Dropdown; Default = zuletzt angelegte Person via `selectedSensorikProbandId`), `renderSensorik()` (Items für die gewählte Person; Empty-State ohne Personen), `toggleSensorik(id)` (setzt/löscht `p.sensorik[id]` = ISO-String, Löschen per `showConfirm`; sobald **alle** `SENSORIK_ITEMS` für die Person gesetzt sind → `showScreen('session')`), Reset-Button `btn-reset-sensorik` (`p.sensorik = {}` für die gewählte Person). Nach „Person anlegen" öffnet `sensorik-prompt-overlay` (Ja → `showScreen('sensorik')`) |
| SESSION (Nav-Label „Szenario", Screen-ID `session`) | 389–~830 | Szenario-Auswahl, Teilnehmenden-Auswahl (Einzel- **und** Mehrfachauswahl je nach `settings.multiProband`, `getSelectedProbandIds()`; `buildProbandSelect()` wählt im Einzelmodus die aktive Person vor: bestehende Auswahl → `selectedSensorikProbandId` → zuletzt angelegte Person), Start/Pause/Fortsetzen/Stopp-Timer (`startTimer`, `pauseTimer`, `resumeTimer`, `stopTimer`), Tag-Zeilen, Sitzung speichern (ggf. mehrere Einträge bei Mehrfachauswahl), Szenario-/Tag-Manager (CRUD für Konfiguration) |
| LOG | 830–1034 | Sitzungsliste mit Filtern, Detailansicht, Bearbeiten, Löschen |
| EXPORT | 1034–1145 | Statistiken, CSV-/JSON-Export, Geräte-Label |
| EINSTELLUNGEN | ~1195 | `renderSettingsScreen()`, Toggle "Mehrere Teilnehmende gleichzeitig" (`settings.multiProband`), "Alle Daten löschen" (`btn-clear-data`; leert `probanden` inkl. `p.sensorik`) |
| BEWERTUNGSBOGEN | 1157–1410 | Post-Session-Prompt (inkl. Vorschlag zur gemeinsamen Bewertung bei mehreren Teilnehmenden), Sitzungsauswahl (Einzel- **und** Mehrfachauswahl je nach `settings.multiProband`, `getSelectedBewSessionIds()`), 19 Bewertungsskalen (1–6), Speichern/Überschreiben (ggf. mehrere Einträge bei Mehrfachauswahl) |
| INIT | Dateiende | Startsequenz: `load()`, initiales Rendering aller Screens (inkl. `renderSensorik()`), Versionsanzeige |

## Konventionen im Code

- **XSS-Schutz:** Jeder aus State/Nutzereingabe gerenderte Text durchläuft `esc()` vor dem
  Einfügen in `innerHTML`. Bei neuem Rendering-Code diese Konvention beibehalten.
- **IDs:** `uid()` erzeugt Client-seitige IDs aus Zeitstempel + Zufallsstring (kein UUID-Format,
  aber kollisionsarm genug für den Einzelgeräte-Kontext dieser App).
- **Bestätigung vor destruktiven Aktionen:** Löschaktionen laufen immer über `showConfirm()`,
  nie über direktes Löschen beim Klick.
- **Delete-Bug-Muster (siehe `CLAUDE.md`):** IDs werden vor dem Zurücksetzen von
  `editingProbandId`/`detailSessionId` in eine lokale `const` zwischengespeichert, damit der
  `showConfirm`-Callback (asynchron ausgeführt) noch die richtige ID kennt.
- **`.hidden`-Klasse:** Muss laut `style.css` `display: none !important` sein (nicht
  `opacity:0`), sonst blockieren unsichtbare Elemente Klicks — siehe bekannte Bugs in
  `CLAUDE.md`.

## Service Worker (`sw.js`)

- Cache-Name `CACHE` ist an `APP_VERSION` aus `app.js` gekoppelt (manuell synchron zu halten,
  siehe Versionierungsregel in `CLAUDE.md`) — ändert sich die Versionsnummer, wird beim
  nächsten Laden der alte Cache automatisch gelöscht (`activate`-Handler).
- Strategie: **Cache-first mit Network-Fallback** — gecachte Antwort wird bevorzugt
  ausgeliefert; bei Cache-Miss wird das Netzwerk versucht und die Antwort zusätzlich in den
  Cache geschrieben; schlägt auch das fehl (offline + nicht gecacht), wird `index.html` als
  Fallback ausgeliefert (SPA-artiges Verhalten für Navigation).
- `sw.js` selbst ist **nicht** in der `ASSETS`-Liste enthalten (bewusst — verhindert laut
  `CLAUDE.md` einen bekannten Service-Worker-Deadlock).
- Nur `GET`-Requests werden behandelt; alles andere wird an den Browser durchgereicht.

## Rendering-Modell

Kein Virtual DOM, kein Reaktivitäts-Framework. Jede `render*()`-Funktion (z. B.
`renderProbanden()`, `renderLog()`, `renderBewertungScreen()`) baut den relevanten
DOM-Ausschnitt bei jedem Aufruf komplett neu aus dem aktuellen State via
Template-Strings + `innerHTML` auf und hängt anschließend Event-Listener an die neu
erzeugten Elemente. Es gibt keinen Diffing-Mechanismus — nach jeder State-Änderung muss die
betroffene `render*()`-Funktion explizit erneut aufgerufen werden (das übernehmen die
jeweiligen Event-Handler).

## Responsive Layout

`style.css` implementiert zwei Navigationsmuster über CSS media queries: eine Sidebar
(`.sidebar`, `.nav-item`) für Desktop/Tablet und eine Bottom-Navigation (`.bottom-nav`,
`.nav-btn`) für Mobile. Beide Navigationsleisten existieren gleichzeitig im DOM und werden
per CSS ein-/ausgeblendet; `app.js` hält deshalb für Navigation zwei parallele
Listener-Registrierungen (`.nav-btn` und `.nav-item`), die beide `showScreen()` aufrufen.

## Bewertungsbogen — Item-Struktur

19 Items, gruppiert in 6 Dimensionen, jeweils Schulnoten-Skala 1 (sehr gut) bis 6
(ungenügend). Item-Keys: `a1–a4` (Lageerkundung), `b5–b8` (Entscheidungsqualität), `c9–c10`
(Führung/Kommunikation), `d11–d12` (Struktur/Effizienz), `e13, e15, e16` (Umsetzung
Lehrgangsinhalte — **Nummer 14 ist im Quellbogen bewusst ausgelassen**, kein Bug), `z17–z20`
(Zusatzblock: subjektiver Leistungsvergleich). Definiert in `BEW_ITEMS` (`app.js:1161`) und
den zugehörigen Label-Texten direkt in `index.html`. Bei Änderungen an den Items müssen
**beide** Stellen synchron gehalten werden (Array in `app.js` + Markup in `index.html`) sowie
die CSV-Exportspalten (`app.js`, `btn-export-csv`-Handler).

## Bekannte Einschränkungen

- Kein automatisiertes Test-Setup (keine Unit-/E2E-Tests im Repo).
- Kein Lint/Format-Tooling konfiguriert.
- `localStorage`-Kapazität ist browserabhängig begrenzt (üblich 5–10 MB); die App fängt
  Schreibfehler nur pauschal per `try/catch` in `save()` ab und zeigt einen generischen
  Toast — kein differenziertes Verhalten bei Speicherplatzmangel.
- Keine Datenmigration/Schema-Versionierung für `localStorage`-Inhalte (siehe oben).
- Sensoriknummer wird nur noch im Bearbeiten-Dialog erfasst und ist dort **optional**; wird
  eine eingegeben, ist sie hart auf den Bereich 1–12 + Eindeutigkeit validiert
  (`btn-save-proband-edit`-Handler). Neue Personen bekommen `sensor: ''`.
- Pseudonym ist hart auf das Format "1 Buchstabe, 4 Zahlen, 3 Buchstaben" (z. B. `P1234ABC`)
  validiert — `PSEUDO_FORMAT_REGEX`/`isValidPseudoFormat()` in `app.js:99–100`. Live-Feedback
  übernimmt `setPseudoFieldValidity()` (`app.js:104–112`): sie markiert das jeweilige
  Eingabefeld (`#inp-pseudo`/`#edit-pseudo`) per CSS-Klasse `field-invalid` (roter Rahmen) und
  blendet den Hinweistext `#inp-pseudo-error`/`#edit-pseudo-error` ("Format nicht korrekt")
  ein, sobald ein nicht-leerer Wert nicht zum Format passt — ausgelöst per `input`-Listener bei
  jedem Tastendruck sowie beim Öffnen des Bearbeiten-Dialogs und beim Klick auf Speichern
  (`app.js:294`, `app.js:351`). Ein leeres Feld gilt bewusst nicht als ungültig (keine
  Fehlermarkierung vor der ersten Eingabe). Die Prüfung greift nur bei Neuanlage/Bearbeitung;
  bereits vorhandene Pseudonyme in älteren `localStorage`-Datenständen, die diesem Format nicht
  entsprechen, werden dadurch **nicht** automatisch verändert (keine Migration) — beim Öffnen
  des Bearbeiten-Dialogs für eine solche Person wird das Feld allerdings sofort als ungültig
  markiert.
- Löschen einzelner Teilnehmender/Sitzungen entfernt keine verknüpften Datensätze in anderen
  Tabellen (siehe [DATENFLUSS.md](./DATENFLUSS.md) für die Datenschutz-Implikation).
- Manuelles Bearbeiten von Start-/Endzeit einer Sitzung (`btn-save-edit`) rechnet
  `duration_s` neu als rohe Differenz, ohne `pauses[]` zu berücksichtigen — bei Sitzungen
  mit Pausen kann `duration_s` nach einer manuellen Zeitkorrektur von der Summe
  aktive Zeit + Pausenzeit abweichen.
- Zwei vermutliche Backup-/Debug-Dateien im Repo-Root (`debug.html`, `download`, siehe oben)
  sind nicht Teil der App und sollten bei größerer Aufräumarbeit hinterfragt, aber nicht
  ungefragt gelöscht werden.

## Offene TODOs

- TODO: Datenschutzrechtliche Prüfung der in [DATENFLUSS.md](./DATENFLUSS.md) markierten
  Punkte (Rechtsgrundlage, Aufbewahrungsfristen, Umgang mit Exportdateien).
- TODO: Klären, ob die verwaisten Dateien `debug.html`, `download` und
  `icon-512 (1).png` im Repo-Root entfernt werden können.
- TODO: Entscheiden, ob Löschen einer Sitzung künftig automatisch die zugehörige Bewertung
  mitlöschen soll (aktuell bewusst nicht der Fall, siehe oben).

## Pflichten bei Code-Änderungen (Kurzfassung von `CLAUDE.md`)

1. **Strict non-regression:** Änderungen additiv/visuell, nichts Bestehendes unangekündigt
   brechen.
2. **Minimale Diffs**, kein Refactoring nebenbei.
3. Bei jeder inhaltlichen Änderung: `APP_VERSION` in `app.js`, `CACHE` in `sw.js` und die
   beiden `.app-version`-Platzhalter in `index.html` synchron hochzählen.
4. **Diese `/docs`-Dateien bei jeder Änderung an Datenerfassung, -speicherung,
   -übertragung, Architektur oder Bedienung aktuell halten** — siehe Hinweis oben und
   ursprüngliche Anforderung an diese Dokumentation.

Vollständige Regeln: `/CLAUDE.md` im Repo-Root.
