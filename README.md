# Bücher-Webseite – Technische Doku

## Aktuelle lokale Version
- **v0.2.1**

## Zweck
Diese Seite visualisiert die Google-Sheet-Tabelle **`Nero Normalisiert`** als browsbare Bücherübersicht mit Cover, Filtern und Detail-Modal.

Pfad der Webdateien:
- `site/index.html`
- `site/data.json`

Lokale Vorschau:
- URL: `http://192.168.178.120:8000/`
- Starten mit:
  ```bash
  python3 -m http.server 8000 --directory /home/thomas/.openclaw/workspace/site --bind 0.0.0.0
  ```

---

## Wichtige Grundregel
Die Website arbeitet im Frontend primär mit **Header-Namen aus `data.json`**, nicht mit hart codierten Spaltenbuchstaben.

Spaltenbuchstaben sind nur hilfreich, wenn sie wirklich mit dem aktuellen Live-Sheet übereinstimmen.
Für den kanonischen Tabellenstand ist `projects/buch-datenbank/SCHEMA.md` maßgeblich.

---

## Datenquelle
Google Sheet: **`Nero Normalisiert`**

Das Frontend nutzt u. a. diese Header-Namen:
- `Cover`
- `F_raw`
- `Status`
- `Vorschlag Mini-Tags`
- `Typ`
- `Overall`
- `Empfehlung`
- `Direktlink`
- `KU`
- `Amazon.de (Kindle • Band 1)`
- `Goodreads`
- `Erscheinungsdatum / Release-Status`
- `Leserfeedback (spoilerfrei)`
- `Kurzbeschreibung `
- `Reihenstatus`
- `Quellenbasis / Sicherheit`

---

## Cover-Architektur

### `Cover`
Diese Spalte enthält in vielen Fällen eine Google-Sheets-Formel wie:
```text
=IMAGE("https://...")
```

**Wichtig:**
- `Cover` nicht als normale Web-URL behandeln
- `Cover` dient primär der sichtbaren Darstellung im Sheet
- bei normalem Export kann `Cover` leer wirken, obwohl im Sheet ein Bild sichtbar ist

### `F_raw`
Diese Spalte ist die **Web-Rohdaten-Spalte**.

Sie enthält:
- die aus `Cover` extrahierte Bild-URL als reinen Text
- keine Formel
- keine lokale Bildverwaltung als Hauptweg

**Regel:**
- Webseite soll primär `F_raw` verwenden
- `Cover` bleibt die Sheet-/Anzeigequelle

---

## Wie die Cover-Extraktion funktioniert
Normale Sheet-Reads liefern `Cover` oft leer.
Die funktionierende Methode ist:

```bash
gog sheets get <sheetId> 'Nero Normalisiert!F1:F82' --json --render=FORMULA
```

Dann aus der Formel extrahieren:
```text
=IMAGE("https://...")
```

und die URL nach `F_raw` schreiben.

### Ergebnis
- `Cover` bleibt intakt
- `F_raw` enthält echte Bild-URLs
- die Webseite kann die Cover stabil laden

---

## Webseitenlogik

### Cover-Lade-Reihenfolge
In `site/index.html` gilt:
1. Nutze `F_raw`, wenn dort eine gültige URL steht
2. Falls `F_raw` leer ist, versuche URL direkt aus `Cover` zu extrahieren
3. Sonst zeige Fallback „Kein Cover“

### Qualitätsverbesserung der Cover
Die Seite schreibt bekannte Thumbnail-URLs beim Rendern auf größere Varianten um.

Beispiele:
- Goodreads: `..._SY75_.jpg` → `..._SY600_.jpg`
- Amazon: `..._AC_UY218_.jpg` → `..._SL1500_.jpg`

Das passiert **nur in der Anzeige**, nicht im Sheet.

---

## UI-Stand (aktuell)
Designrichtung: **Clean Dark Library**

### Vorhandene Funktionen
- Suchfeld
- Mehrfachfilter für:
  - Typen
  - Tags
  - konkrete Statuswerte
- Schnellfilter für Lesestatus:
  - Alle
  - Gelesen
  - Offen
- Cover-Filter:
  - Alle Cover
  - Nur mit Cover
  - Nur ohne Cover
- Sortierung nach:
  - Empfehlung
  - Overall
  - Titel
  - Autor
- Detail-Modal statt direktem Seitenwechsel
- Direktlink-Button im Modal
- KU-Badge auf Karte und im Modal
- Release-Timeline
- lokaler Search-/Refresh-Trigger

### Wichtige UX-Entscheidungen
- **Toolbar ist nicht sticky**
- Karten öffnen ein **Modal**, nicht direkt einen neuen Tab
- Externer Link geht erst aus dem Modal auf

---

## Modal-Logik
Beim Klick auf eine Karte öffnet sich ein Popup mit:
- Cover
- Titel
- Autor
- Status
- Typ
- Empfehlung
- Overall
- Tags
- Sprache / DE verfügbar
- Reihenstatus
- KU
- Intensität
- Quellenbasis / Sicherheit
- Kurzbeschreibung
- Leserfeedback
- Kommentar

Buttons im Modal:
- **Direktlink öffnen** → aus `Direktlink`
- Amazon → nur wenn in `Amazon.de (Kindle • Band 1)` eine echte URL steht
- Goodreads → nur wenn in `Goodreads` eine echte URL steht
- Schließen

### Link-Farblogik
- Direktlink **grün**, wenn echte URL vorhanden
- Direktlink **rot**, wenn kein Link vorhanden
- Amazon/Goodreads werden **nur angezeigt**, wenn echte URL vorhanden ist

---

## Status-Logik
Aktuell existieren zwei Ebenen:

### 1. Schnellfilter
Abgeleitet aus `Status`:
- **Gelesen** → wenn Status „gelesen“ enthält
- **Offen** → alles andere

### 2. Exakte Statuswerte
Zusätzliche Mehrfachauswahl direkt mit den echten Statuswerten aus der Tabelle.

---

## KU-Logik
KU wird aus `KU` gelesen.

Anzeige:
- Kartenbadge: `KU` oder `Kein KU`
- Modal: eigenes Infofeld + Badge
- Direktlink kann textlich als `Direktlink öffnen · KU` erscheinen

---

## Wichtige Dateien / Referenzen
- `site/index.html` – komplette UI / Logik
- `site/data.json` – exportierte Sheet-Daten für die Seite
- `site/backup-nero-20260315T032023Z.json` – historischer Backup-Stand
- `site/backup-nero-20260315T032023Z.csv` – historischer Backup-Stand

---

## Regeln für spätere Änderungen

### Niemals tun
- `Cover` blind überschreiben
- `F_raw` als freie manuelle Spielspalte behandeln
- tote Amazon/Goodreads-Buttons anzeigen
- alte Buchstaben-Mappings aus veralteten Zwischenständen als Wahrheit behandeln

### Immer tun
- erst prüfen, welche **Header-Namen** das Frontend wirklich nutzt
- bei Cover-Problemen zuerst `--render=FORMULA` prüfen
- `F_raw` als Web-Zwischenschicht beibehalten
- bei UI-Änderungen Mobile-Ansicht mitdenken
- bei Schemaänderungen `SCHEMA.md` und Export gemeinsam prüfen

---

## Nächste sinnvolle Ausbaustufen
- bessere Detailstruktur im Modal
- nächster/vorheriger Titel im Modal
- Favoriten / Merkliste
- öffentliche Deployment-Variante
- sauberer Build-/Refresh-Workflow für `data.json`
