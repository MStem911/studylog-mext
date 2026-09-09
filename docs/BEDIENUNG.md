# StudyLog-V2 — Bedienungsanleitung

> Anleitung für Studienleitende, die die App zur Durchführung nutzen. Keine technischen
> Details — dafür siehe [ARCHITECTURE.md](./ARCHITECTURE.md). Datenschutz-Hintergrund:
> siehe [DATENFLUSS.md](./DATENFLUSS.md).

## Installation & Start

1. Die von der Studienleitung genannte App-URL im Browser öffnen (auf dem iPhone: **Safari**
   verwenden, nicht Chrome — sonst funktioniert "Zum Home-Bildschirm" nicht zuverlässig).
2. **Auf dem Home-Bildschirm installieren** (empfohlen, damit die App offline funktioniert
   und wie eine normale App aussieht):
   - **iPhone/iPad:** Teilen-Symbol antippen → "Zum Home-Bildschirm" → Hinzufügen.
   - **Android:** Menü (⋮) öffnen → "Zum Startbildschirm hinzufügen" bzw. "App installieren".
3. App-Symbol auf dem Home-Bildschirm antippen — die App startet auch ohne Internetverbindung.

## Kernfunktionen im Überblick

Die App hat sieben Bereiche, erreichbar über die Navigation (unten auf dem Smartphone, links
auf Tablet/Desktop):

| Bereich | Zweck |
|---|---|
| 👤 **Teilnehmende** | Personen mit Pseudonym + Sensoriknummer anlegen und verwalten |
| 🩹 **Sensorik** | Checkliste der Sensorik-Hardware; pro Item wird beim Abhaken automatisch der Anlege-Zeitpunkt erfasst |
| ⏺ **Sitzung** | Timer starten/stoppen, Szenario wählen, Abweichungen/Notizen erfassen |
| ☰ **Protokoll** | Übersicht aller gespeicherten Sitzungen, filtern, bearbeiten, löschen |
| 📝 **Bewertung** | Trainerbewertungsbogen zu einer Sitzung ausfüllen |
| ↓ **Export** | Daten als CSV/JSON exportieren, Statistiken einsehen |
| ⚙ **Einstellungen** | App-Verhalten anpassen (z. B. Mehrfachauswahl von Teilnehmenden bei Sitzung **und** Bewertung), alle Daten löschen |

## Typischer Ablauf einer Nutzungssitzung

### 1. Teilnehmende Person anlegen (einmalig pro Person)

1. Im Bereich **Teilnehmende** auf **+** tippen.
2. **Pseudonym** eingeben — **kein Klarname**. Format ist vorgegeben: 1 Buchstabe, dann 4
   Zahlen, dann 3 Buchstaben (z. B. `P1234ABC`). Passt die Eingabe nicht zum Format, wird das
   Feld sofort rot markiert mit dem Hinweis "Format nicht korrekt" darunter; Anlegen ist erst
   möglich, wenn das Format stimmt.
3. **Sensoriknummer** (1–12) der zugewiesenen Sensor-Einheit eingeben.
4. **Händigkeit** wählen — "Rechts" oder "Links" (Pflichtfeld, relevant für die
   Sensorplatzierung). Ohne Auswahl ist Anlegen/Speichern nicht möglich.
5. Optional: Notiz eintragen (z. B. "Brille").
6. Mit **✓ Anlegen** speichern.

Die Uhrzeiten "Sensorik angelegt" / "Sensorik abgelegt" (am Teilnehmenden-Datensatz) werden
**nicht** beim Anlegen erfasst, sondern nachträglich über **Person bearbeiten** (Tippen auf
eine bereits angelegte Person). Davon unabhängig ist die geräteweite **Sensorik-Checkliste**
im gleichnamigen Tab (siehe nächster Abschnitt).

### 2. Sensorik anlegen und abhaken

1. In den Bereich **🩹 Sensorik** wechseln. Die Checkliste enthält feste Items:
   Shimmer ECG, Shimmer GSR+, Polar Brustgurt, Garmin.
2. Sobald eine Sensorik-Einheit angelegt ist, das entsprechende Item **antippen** — der
   aktuelle Zeitpunkt (Datum + Uhrzeit, Gerätezeit) wird automatisch erfasst, gespeichert
   und unter dem Item angezeigt.
3. Versehentlich abgehakt? Item erneut antippen und die Sicherheitsabfrage bestätigen — die
   Erfassung wird entfernt.
4. Vor einem neuen Durchlauf mit **↺ Zurücksetzen** (oben rechts) alle erfassten Zeitpunkte
   auf einmal löschen. Die Checkliste ist geräteweit (nicht pro Teilnehmende:r).

### 3. Sitzung durchführen

1. Im Bereich **Sitzung**: gewünschte Person im Dropdown auswählen. Ist unter
   **⚙ Einstellungen** die Option "Mehrere Teilnehmende gleichzeitig" aktiviert, erscheint
   stattdessen eine Liste zum Antippen mehrerer Personen (z. B. wenn ein Szenario von
   mehreren Teilnehmenden gemeinsam durchlaufen wird). Beim Speichern entsteht dann für
   jede ausgewählte Person eine eigene, unabhängige Sitzungsaufzeichnung mit identischer
   Start-/Endzeit.
2. Passendes **Szenario** antippen (z. B. VR Welt, Verkehrsunfall, Krankenhaus).
3. **▶ Start** drücken, sobald das Szenario beginnt — der Timer läuft.
4. Falls die Sitzung unterbrochen werden muss (z. B. technische Störung, Rückfrage):
   **⏸ Pause** drücken — der Timer friert ein. Mit **▶ Fortsetzen** läuft er weiter, ab
   dem eingefrorenen Stand. Jede Pause wird mit genauer Start-/Endzeit und Dauer
   protokolliert; eine Sitzung kann beliebig oft pausiert werden.
5. Nach Abschluss des Szenarios **⏹ Stopp** drücken (auch aus einer laufenden Pause heraus
   möglich).
6. Falls während der Sitzung etwas vom geplanten Ablauf abgewichen ist: passende
   **Abweichungs-Tags** antippen (z. B. "Techn. Fehler") und/oder eine Freitextnotiz
   eintragen.
7. Mit **💾 Sitzung speichern** abschließen.
8. Die App fragt danach, ob direkt der **Trainerbewertungsbogen** ausgefüllt werden soll
   (siehe Schritt 4) — kann auch später über den Bereich **Bewertung** nachgeholt werden.
   War nur eine Person ausgewählt, bezieht sich der Bogen auf diese eine Sitzung. Wurden
   mehrere Personen gleichzeitig ausgewählt, schlägt die App vor, alle gemeinsam in **einem**
   Bewertungsbogen zu bewerten (die eingetragenen Noten/Anmerkungen werden dann identisch in
   die jeweils eigenständige Bewertung jeder Person übernommen) — eine getrennte
   Einzelbewertung je Person bleibt über den Bereich **Bewertung** weiterhin möglich.

**Wichtig:** Wird nach dem Stoppen erneut **▶ Start** gedrückt, ohne vorher zu speichern,
fragt die App zur Sicherheit nach ("Aufzeichnung verwerfen?"), bevor die noch nicht
gespeicherte Aufzeichnung durch die neue Sitzung überschrieben wird.

### 4. Trainerbewertungsbogen ausfüllen (optional, pro Sitzung)

1. Im Bereich **Bewertung** die gewünschte Sitzung im Dropdown auswählen (bereits bewertete
   Sitzungen sind mit ✓ markiert). Ist unter **⚙ Einstellungen** die Option "Mehrere
   Teilnehmende gleichzeitig" aktiviert, erscheint stattdessen eine Liste zum Antippen
   mehrerer Sitzungen — so lassen sich mehrere Teilnehmende in **einem** gemeinsamen Bogen
   bewerten (z. B. wenn sie dasselbe Szenario gemeinsam durchlaufen haben). Beim
   Vorbefüllen mit einer bereits vorhandenen Bewertung ist das nur bei Auswahl einer
   einzelnen Sitzung möglich; bei Mehrfachauswahl startet der Bogen leer.
2. Für jede der 19 Bewertungsfragen eine Note von **1 (sehr gut)** bis **6 (ungenügend)**
   vergeben.
3. Optional Anmerkungen eintragen.
4. Mit **💾 Bewertung speichern** abschließen. Bei Mehrfachauswahl werden dieselben Noten und
   Anmerkungen als **eigenständige** Bewertung für jede ausgewählte Sitzung gespeichert.
   Erneutes Speichern für eine bereits bewertete Sitzung überschreibt deren vorherige
   Bewertung. Sind noch nicht alle 19 Fragen beantwortet, fragt die App vor dem Speichern
   nach ("Trotzdem speichern?" / "Abbrechen") — so bleibt die unvollständige Bewertung nicht
   versehentlich als vermeintlich fertig stehen.

### 5. Sitzung im Protokoll prüfen oder korrigieren

1. Im Bereich **Protokoll** die Liste aller Sitzungen einsehen, bei Bedarf nach Szenario
   oder Person filtern.
2. Eintrag antippen, um Details zu sehen.
3. Über **✏ Bearbeiten** lassen sich Start-/Endzeit, Szenario, Person, Abweichungen und
   Notizen nachträglich korrigieren (das Datum selbst ist nicht änderbar).
4. Über **Sitzung löschen** kann ein fehlerhafter Eintrag entfernt werden.

### 6. Am Ende der Erhebung: Export

1. Im Bereich **Export** optional ein **Geräte-/Betreuungslabel** eintragen (hilfreich, wenn
   mehrere Geräte parallel genutzt wurden).
2. Optional nach Szenario filtern.
3. **⬇ CSV exportieren** oder **⬇ JSON exportieren** antippen — die Datei wird auf dem Gerät
   gespeichert (z. B. im Download-Ordner).
4. Die exportierte Datei anschließend gemäß den Vorgaben der Studienleitung sicher
   weitergeben bzw. ablegen (dies erfolgt außerhalb der App).
5. **Erst nach erfolgreichem Export und Sicherung der Daten:** Falls gewünscht, im Bereich
   **⚙ Einstellungen** über **⚠ Alle Daten löschen** sämtliche Teilnehmenden-, Sitzungs- und
   Bewertungsdaten auf diesem Gerät unwiderruflich entfernen. Die App warnt vor dieser
   Aktion — sie kann nicht rückgängig gemacht werden.

## Mehrere Geräte

Jedes Gerät führt seine eigenen, unabhängigen Daten. Es findet **keine automatische
Synchronisation** zwischen Geräten statt. Wurden mehrere Geräte parallel genutzt, muss auf
jedem Gerät einzeln exportiert werden; das Zusammenführen der Export-Dateien erfolgt
anschließend außerhalb der App (z. B. in Excel).
