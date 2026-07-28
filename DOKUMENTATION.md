# 📘 Dokumentation · Fachwirt-Lern-App

Vollständige technische und inhaltliche Dokumentation der Lern-App **„Geprüfte/r Fachwirt/in im Gesundheits- und Sozialwesen"** (für Simone).

> Diese Datei ist die **Entwickler-/Wartungs-Doku**. Die kurze Nutzer-Anleitung für Simone steht in [`README.md`](README.md).

---

## 1. Überblick

| | |
|---|---|
| **Was** | Persönlicher Lerntrainer (Quiz-/Karteikarten-App) für die IHK-Prüfung |
| **Für wen** | Simone |
| **Technik** | Eine einzige, selbst-enthaltene `index.html` (kein Build, kein Framework, keine Abhängigkeiten) |
| **Läuft** | Offline im Browser, als PWA aufs iPhone installierbar |
| **Speicher** | Fortschritt lokal im Browser (`localStorage`), nichts verlässt das Gerät |
| **Hosting** | GitHub Pages |
| **Repo** | https://github.com/bastianstute88/fachwirt-lernen |
| **Live-URL** | https://bastianstute88.github.io/fachwirt-lernen/ |
| **Erstellt mit** | Skill `lern-app` (siehe Abschnitt 9) |

---

## 2. Dateien im Projekt

| Datei | Zweck |
|---|---|
| `index.html` | **Die komplette App.** HTML + CSS + JavaScript + alle 1.055 Fragen (als JS-Array `QUESTIONS` eingebettet). ~770 KB, ~744 Zeilen. |
| `fragen-index.json` | Schlanker Index aller Fragen (nur `nr, id, hb, typ, schwer, thema, frage`) — zum Nachschlagen/Prüfen, **wird von der App nicht geladen**. Muss bei Fragenänderungen mitgepflegt werden. |
| `app-icon.png` | App-Icon (🎓 auf IHK-blauem Verlauf), dient als `apple-touch-icon` und Favicon. |
| `README.md` | Nutzer-Anleitung für Simone (Installation, Funktionen). |
| `DOKUMENTATION.md` | Diese Datei. |

Die PDFs im übergeordneten Ordner (`../download*.pdf`, `../Skript Komplett.pdf` und `../_entsperrt/`) sind das **Quellmaterial** (Kursskript + IHK-Prüfungen), aus dem die Fragen erzeugt wurden. Sie gehören **nicht** ins Repo.

---

## 3. Fragenkatalog

**1.055 geprüfte Fragen** über die 6 offiziellen IHK-Handlungsbereiche.

### Nach Handlungsbereich (HB)

| HB | Thema | Fragen |
|---|---|---|
| 1 | Grundlagen: Wirtschaft, Sozial-/Gesundheitssystem, Recht | 250 |
| 2 | Qualitätsmanagement | 150 |
| 3 | Projekte & Schnittstellen | 110 |
| 4 | Betriebswirtschaft / Rechnungswesen / Controlling | 210 |
| 5 | Personal & Führung | 175 |
| 6 | Marketing & Öffentlichkeitsarbeit | 160 |
| | **Summe** | **1.055** |

### Nach Fragetyp

| Typ (`typ`) | Anzahl | Beschreibung |
|---|---|---|
| `mc` | 502 | Multiple Choice (Wissen) — eine richtige aus mehreren Optionen |
| `karte` | 367 | Karteikarte (Verständnis) — aufdecken + Selbstbewertung |
| `rechnen` | 186 | Zahleneingabe (Rechnen) — numerische Lösung mit Toleranz |

### Schwierigkeit / „IHK-Kracher"

`schwer` ist 1, 2 oder 3. **236 Fragen** mit `schwer: 3` sind die schweren **„IHK-Kracher"** 🔥 — sie werden gezielt eingestreut (siehe Abschnitt 6).

---

## 4. Datenmodell

Alle Fragen liegen als JS-Array `const QUESTIONS = [ … ]` in `index.html` (ab Zeile ~291). Die Themen als `const THEMES = [ … ]` direkt darüber.

### Thema-Objekt (`THEMES`)

```js
{ "id":1, "emoji":"🏛️", "name":"Grundlagen & System",
  "desc":"Wirtschaft · Sozial- & Gesundheitssystem · Recht" }
```

### Frage-Objekt (`QUESTIONS`) — gemeinsame Felder

| Feld | Typ | Bedeutung |
|---|---|---|
| `id` | string | Eindeutige ID, z. B. `hb1a_001` |
| `nr` | number | Fortlaufende Anzeigenummer (1…1055) |
| `hb` | number | Handlungsbereich 1–6 (→ `THEMES.id`) |
| `typ` | string | `"mc"` \| `"karte"` \| `"rechnen"` |
| `schwer` | number | 1 (leicht) … 3 (IHK-Kracher) |
| `thema` | string | Kurzes Themenlabel |
| `frage` | string | Der Fragetext |
| `erklaerung` | string | Erläuterung / Merkhilfe (nach dem Antworten sichtbar) |
| `quelle` | string | Herkunft, z. B. `"HB1 · Bedürfnisse"` |

### Typspezifische Felder

**`mc` (Multiple Choice)**
```js
"opts": ["Option A","Option B","Option C","Option D"],
"correct": 0   // Index der richtigen Option
```

**`karte` (Karteikarte)**
```js
"antwort": "Der aufzudeckende Antworttext."
```

**`rechnen` (Zahleneingabe)**
```js
"loesung": 1234.5,          // korrekter Zahlenwert
"toleranz": 0.5,            // erlaubte Abweichung (±)
"einheit": "€",            // Anzeige-Einheit
"loesungsweg": "..."       // wird nach dem Antworten gezeigt
```

---

## 5. Zustand & Speicherung (`localStorage`)

- **Key:** `fachwirt_v1` (Konstante `KEY`)
- Geladen beim Start via `load()`, geschrieben via `save()` nach jeder Antwort.

### State-Objekt

```js
{
  name: "",          // Anzeigename (optional)
  boxes: {},         // { [frageId]: leitnerBox 0–5 }  ← Kern des Lernfortschritts
  answered: 0,       // Gesamtzahl beantworteter Fragen
  bestStreak: 0      // längste Serie richtiger Antworten
}
```

`boxes[id]` = Leitner-Fach der Frage:
`0` = noch nie beantwortet · `1–4` = im Lernen · `5` = **gemeistert** (`MAX_BOX = 5`, `isMastered` ab Box 5).

Die **Session** (aktueller Lauf: Pool, Streaks, Verlauf, `sinceKracher`) lebt nur im Arbeitsspeicher und wird **nicht** persistiert.

---

## 6. Lernlogik

### Leitner-Prinzip (`grade`, Zeile ~487)
- **Richtig** → Box +1 (max. 5).
- **Falsch** → Box zurück auf **1** (nicht 0), Frage kommt also bald wieder.

### Auswahl der nächsten Frage (`pickNext`, Zeile ~371)
Gewichtete Zufallsauswahl — je „schwächer" eine Frage sitzt, desto höher die Chance:

| Box | Gewicht |
|---|---|
| 0 (neu) | 7 |
| 1 | 5 |
| 2 | 4 |
| 3 | 3 |
| 4 | 2 |
| 5 (gemeistert) | 1 |

Die zuletzt gezeigte Frage wird ausgeschlossen (außer der Pool hat nur 1 Frage).

### IHK-Kracher-Einstreuung (`nextQuestion`, Zeile ~429)
Ein Zähler `sinceKracher` erhöht sich pro Frage. Bei **≥ 15** wird erzwungen eine noch nicht gemeisterte `schwer:3`-Frage gezogen (`forceHard`), danach Zähler zurück auf 0.

### Modi
- **Thema** (`startTheme`): nur Fragen eines HB; endet, wenn **alle** Fragen des Themas gemeistert sind (`renderThemeDone`).
- **Shuffle** (`startShuffle`): alle Themen gemischt, endlos; Fortschrittsbalken zählt in 20er-Blöcken.

---

## 7. UI / Screens

Umschaltung über CSS-Klasse `.active` per `show(id)`:

| Screen-ID | Inhalt |
|---|---|
| `home` | Startseite: 6 Themenkacheln mit Fortschritt, Shuffle-Button, Streak-Pill |
| `quiz` | Frageanzeige (alle 3 Typen), Feedback, Taschenrechner (bei `rechnen`) |
| Overlays | `overlay` (Popups/Meilensteine), `reviewLayer` (Rückblick), Toasts |

Design-Notizen:
- iOS-nativer Look, CSS-Variablen in `:root` (IHK-Blau `--blue`, Gold für Streak/Kracher).
- Vollbild-PWA über `apple-mobile-web-app-*` Meta-Tags; `viewport-fit=cover` + `env(safe-area-inset-*)` für Dynamic Island / Notch.
- Integrierter **Taschenrechner** (`calc*`-Funktionen) für Rechenaufgaben, inkl. Vorzeichen-Umschaltung (±).
- **Motivation** (`checkMotivation`, `MILES`): Streak-Feiern, Aufmunterung nach Fehlern, Meilensteine bei 25/50/100/…/500 gemeisterten Fragen.
- **Rückblick** (`reviewOpen`): letzte bis zu 60 beantwortete Fragen inkl. Lösung erneut ansehen.

---

## 8. Daten sichern / zurücksetzen

Im UI verfügbar (Funktionen in `index.html`):
- `exportData()` / `importData()` — Fortschritt als Datei sichern / laden.
- `resetTheme(hb)` — Fortschritt eines Themas zurücksetzen.
- `resetAll()` — kompletten Fortschritt löschen.

Manuell: im Browser `localStorage.removeItem("fachwirt_v1")` löscht alles.

---

## 9. Fragen ändern / hinzufügen

1. **Fragen** im `QUESTIONS`-Array in `index.html` bearbeiten (Schema siehe Abschnitt 4). Auf eindeutige `id` und fortlaufende `nr` achten.
2. **`fragen-index.json` mitpflegen**, damit der Index synchron bleibt (die App nutzt ihn nicht, aber Prüf-/Kontrollskripte).
3. **Gegenprüfen:** Fakten und alle Rechenlösungen gegen das Quellmaterial verifizieren (das war der ursprüngliche Qualitätsanspruch — jede Frage ist geprüft).
4. **Konsistenz-Check** (Terminal):
   ```bash
   python3 -c "import json;d=json.load(open('fragen-index.json'));print(len(d),'Fragen')"
   ```
5. **Deployen** (Abschnitt 10).

Für einen kompletten Neuaufbau aus PDFs existiert der Skill **`lern-app`** (PDFs entsperren → Inhalt kartieren → Fragen via Autoren-Agenten → Faktencheck via Prüfer-Agenten → App bauen → deployen).

---

## 10. Deployment (GitHub Pages)

Die App wird direkt aus dem `main`-Branch via GitHub Pages ausgeliefert.

```bash
cd "fachwirt-lern-app"
git add -A
git commit -m "Beschreibung der Änderung"
git push
```

Nach dem Push ist die neue Version nach ~1 Minute live unter
https://bastianstute88.github.io/fachwirt-lernen/

**Cache-Hinweis:** Bei Icon-/Asset-Änderungen ggf. neuen Dateinamen wählen (Cache-Bust) — so wurde es beim App-Icon gehandhabt.

---

## 11. Installation auf dem iPhone

1. Live-URL in **Safari** öffnen.
2. **Teilen** → **„Zum Home-Bildschirm"**.
3. Startet danach als Vollbild-App. Fortschritt bleibt auf dem Gerät gespeichert.

---

## 12. Konventionen & Stolpersteine

- **Keine externen Abhängigkeiten** — alles inline in `index.html`. Bewusst so, damit die App offline und ohne Build läuft. Bitte so halten.
- `localStorage`-Key `fachwirt_v1`: bei inkompatiblen State-Änderungen Key erhöhen (`_v2`), sonst brechen alte gespeicherte Stände.
- Falsch beantwortete Frage geht auf **Box 1**, nicht 0 — 0 bedeutet „nie gesehen".
- `fragen-index.json` und `QUESTIONS` in `index.html` **getrennt** pflegen (nicht automatisch synchronisiert).
- Zahlen-Parsing akzeptiert Komma und Minus (`parseNum`, `toggleSign`); Rechenaufgaben mit `toleranz` bewerten, nicht auf exakte Gleichheit.
