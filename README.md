# Bücher-Webseite – Technische Doku

## Aktuelle lokale Version
- **v0.2.0**

## Zweck
Diese Seite visualisiert die Google-Sheet-Tabelle **"Nero Normalisiert"** als browsbare Bücherübersicht mit Cover, Filtern und Detail-Modal.

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

## Datenquelle
Google Sheet: **Nero Normalisiert**

Wichtige Spalten:
- **F = Cover**
- **G = F_raw**
- **M = Status**
- **N = Vorschlag Mini-Tags**
- **S = Amazon.de (Kindle • Band 1)**
- **T = Typ**
- **U = Overall**
- **W = Empfehlung**
- **AB = Direktlink**
- **H = KU**

---

## Kritische Regel: Cover-Spalte nicht anfassen

### Spalte F = `Cover`
Diese Spalte enthält in vielen Fällen eine Google-Sheets-Formel wie:
```text
=IMAGE("https://...")
```

**Wichtig:**
- Spalte **F niemals überschreiben**
- Spalte F ist **nicht zuverlässig direkt als Web-URL nutzbar**, wenn sie normal exportiert wird
- Beim normalen Export erscheinen die Werte oft leer, obwohl visuell ein Bild da ist

### Spalte G = `F_raw`
Diese Spalte wurde als **Rohdaten-Spalte für Web-Cover** eingeführt.

Sie enthält:
- die aus `F` extrahierte Bild-URL als reinen Text
- **keine Formel**
- **keine lokalen `/images/...`-Pfade** als Hauptweg

**Regel:**
- Webseite soll primär **G / `F_raw`** verwenden
- F bleibt visuell / Sheet-intern erhalten

---

## Wie die Cover-Extraktion funktioniert

Normale Sheet-Reads liefern `F` oft leer.
Die funktionierende Methode ist:

```bash
gog sheets get <sheetId> 'Nero Normalisiert!F1:F82' --json --render=FORMULA
```

Dann aus der Formel extrahieren:
```text
=IMAGE("https://...")
```

und die URL nach **G / `F_raw`** schreiben.

### Ergebnis
- `F` bleibt intakt
- `G` enthält echte Bild-URLs
- die Webseite kann die Cover stabil laden

---

## Webseitenlogik

## Cover-Lade-Reihenfolge
In `site/index.html` gilt:
1. Nutze `F_raw`, wenn dort eine gültige URL steht
2. Falls `F_raw` leer ist, versuche URL direkt aus `Cover` zu extrahieren
3. Sonst zeige Fallback „Kein Cover“

## Qualitätsverbesserung der Cover
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

### Wichtige UX-Entscheidungen
- **Toolbar ist nicht sticky** (Desktop und Mobile)
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
- **Direktlink öffnen** → aus **Spalte AB / Direktlink**
- Amazon → nur wenn echte URL vorhanden
- Goodreads → nur wenn echte URL vorhanden
- Schließen

### Link-Farblogik
- Direktlink **grün**, wenn echte URL vorhanden
- Direktlink **rot**, wenn kein Link vorhanden
- Amazon/Goodreads werden **nur angezeigt**, wenn echte URL vorhanden ist

---

## Status-Logik
Aktuell existieren zwei Ebenen:

### 1. Schnellfilter
Abgeleitet aus Status:
- **Gelesen** → wenn Status „gelesen“ enthält
- **Offen** → alles andere

### 2. Exakte Statuswerte
Zusätzliche Mehrfachauswahl direkt mit den echten Statuswerten aus der Tabelle, z. B.:
- `✅ Gelesen`
- `Neu`
- `⏳ Geplant`
- `📖 Lese ich gerade`
- `🧪 Vorschlag`
- usw.

---

## KU-Logik
KU wird aus der entsprechenden Sheet-Spalte gelesen.

Anzeige:
- Kartenbadge: `KU` oder `Kein KU`
- Modal: eigenes Infofeld + Badge
- Direktlink kann textlich als `Direktlink öffnen · KU` erscheinen

---

## Wichtige Dateien / Referenzen
- `site/index.html` – komplette UI / Logik
- `site/data.json` – exportierte Sheet-Daten für die Seite
- `site/backup-nero-20260315T032023Z.json`
- `site/backup-nero-20260315T032023Z.csv`

---

## Regeln für spätere Änderungen

### Niemals tun
- **Spalte F überschreiben**
- Cover wieder auf lokale `/images/...`-Pfade als Hauptsystem umstellen
- tote Amazon/Goodreads-Buttons anzeigen
- Sticky-Toolbar wieder aktivieren, ohne echten Grund

### Immer tun
- erst prüfen, welche Sheet-Spalte wirklich genutzt wird
- bei Cover-Problemen zuerst `--render=FORMULA` prüfen
- `F_raw` als Web-Zwischenschicht beibehalten
- bei UI-Änderungen Mobile-Ansicht mitdenken

---

## Nächste sinnvolle Ausbaustufen
- bessere Detailstruktur im Modal
- evtl. Navigieren zum nächsten/vorherigen Titel im Modal
- Favoriten / Merkliste
- öffentliche Deployment-Variante
- sauberer Build-/Refresh-Workflow für `data.json`
