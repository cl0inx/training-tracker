# Training Tracker – Claude Code Kontext

## Projekt-Überblick
Single-file Progressive Web App (PWA) für persönliches Krafttraining-Logging.
Gehostet auf GitHub Pages, Backend via Supabase.

**Repository:** https://github.com/cl0inx/training-tracker
**Live-URL:** https://cl0inx.github.io/training-tracker/

---

## Workflow & Regeln

- **Einzige relevante Datei:** `index.html` – alles (HTML, CSS, JS) ist in einer Datei
- **Deployment:** Direkt in `main` pushen – kein Pull Request, kein Branch
- **Commits:** Immer mit aussagekräftiger Commit Message + Extended Description
- **Nach Änderungen:** Syntaxcheck des JS-Blocks durchführen bevor gepusht wird
- **Encoding:** Datei ist UTF-8, Sonderzeichen (ä/ö/ü/✓/✕) direkt verwenden, keine Escape-Sequenzen in Python-Strings

---

## Tech Stack

| Komponente | Details |
|---|---|
| Frontend | Vanilla HTML/CSS/JS, keine Frameworks |
| Backend | Supabase (PostgreSQL + Auth) |
| Hosting | GitHub Pages |
| Charts | Chart.js (via CDN) |
| Fonts | Bebas Neue, DM Sans (Google Fonts) |

---

## Supabase

- **URL & Key:** In `index.html` als Konstanten `SUPABASE_URL` und `SUPABASE_KEY`
- **Auth:** Supabase Auth (Email + Password), JWT-Token wird in `state.token` gespeichert
- **RLS:** Alle Tabellen haben Row Level Security aktiv

### Tabellen

#### `profile`
| Spalte | Typ | Beschreibung |
|---|---|---|
| id | uuid | Primary Key = auth.uid() |
| name | text | Anzeigename des Users |
| created_at | timestamp | Automatisch |

RLS-Policy: `auth.uid() = id`

#### `training`
| Spalte | Typ | Beschreibung |
|---|---|---|
| id | uuid | Primary Key |
| nutzer | text | auth.uid()::text des Users |
| datum | date | Trainingsdatum |
| uhrzeit | time | Uhrzeit des Satzes |
| trainingsart | text | Beine/Schultern, Brust/Trizeps, Rücken/Bizeps, Cardio |
| uebung | text | Name der Übung |
| satz | int | Satznummer |
| gewicht_kg | numeric | Gewicht in kg (Krafttraining) |
| wiederholungen | int | Wiederholungen (Krafttraining) |
| zeit_minuten | numeric | Zeit in Minuten (Cardio) |
| runden | int | Runden (Cardio) |

RLS-Policy: `auth.uid()::text = nutzer`

---

## App-Architektur

### Screens (in Reihenfolge)
1. `screen-login` – Email/Password Login
2. `screen-onboarding` – Namenseingabe bei Erstanmeldung
3. `screen-type` – Trainingsart wählen (3er Split + Cardio)
4. `screen-exercise` – Übung wählen + Sätze eintragen
5. `screen-summary` – Zusammenfassung nach Training
6. `screen-stats` – Auswertung mit Chart + CSV-Export

### State-Objekt
```js
let state = {
  nutzer:             null,  // UUID als Text
  token:              null,  // JWT Token
  userId:             null,  // UUID für Profil-INSERT
  type:               null,  // aktuelle Trainingsart
  currentExercise:    null,  // aktive Übung
  currentSets:        [],    // Sätze der aktuellen Übung
  completedExercises: [],    // abgeschlossene Übungen
  customExercises:    [],    // benutzerdefinierte Übungen
  allExercises:       {}     // alle Übungen pro Trainingsart
};
```

### Session Cache (sessionStorage)
- `trainingSession` – speichert type, currentExercise, currentSets, completedExercises
- Wird bei jeder State-Änderung aktualisiert
- Beim Login wiederhergestellt → User landet direkt in laufendem Training
- Bei `resetAll()` explizit geleert

### Headers
```js
const headers = {
  'apikey':        SUPABASE_KEY,
  'Authorization': `Bearer ${state.token}`,
  'Content-Type':  'application/json',
  'Prefer':        'return=minimal'
};
```
⚠️ Für INSERT mit Rückgabe: eigene Headers mit `'Prefer': 'return=representation'` verwenden

---

## Trainingsarten & Übungen

```
Beine/Schultern  → Kniebeugen, Beinpresse, Ausfallschritte, Beinstrecker,
                   Beincurl, Wadenheben, Schulterdrücken, Seitheben,
                   Frontheben, Schulterpresse, Face Pulls

Brust/Trizeps    → Bankdrücken, Schrägbankdrücken, Fliegende, Dips,
                   Kabelfliegende, Trizepsstrecken, Pushdown, Overhead Extension

Rücken/Bizeps    → Klimmzüge, Rudern, Kreuzheben, Latziehen,
                   Kabelrudern, Bizepscurl, Hammercurl, Konzentrationscurl

Cardio           → Laufen, Fahrrad, Seilspringen, Rudergerät
```

---

## Bekannte Eigenheiten & Fallstricke

- **Semikolon-CSV:** Export verwendet `;` als Trennzeichen + UTF-8 BOM für deutsches Excel
- **Literal Newlines in JS-Strings:** Niemals `\n` als echten Zeilenumbruch in einen String einbetten – immer `\\n` in Python-Ersetzungen oder Template Literals verwenden
- **`return=minimal` vs `return=representation`:** Bei `saveSatz()` wird `return=representation` benötigt um die `id` des neu erstellten Eintrags zu erhalten
- **UUID-Casting:** Supabase RLS vergleicht `auth.uid()` (uuid) mit `nutzer` (text) → `auth.uid()::text = nutzer`
- **`window.confirm()`** funktioniert auf mobilen Geräten unzuverlässig → eigener `showConfirm()`-Dialog ist im Code

---

## Equipment (relevant für Trainingsberatung)
- Heimstudio mit Rack + verstellbarer Langhantel
- Kabelzug mit zwei festen Befestigungspunkten (oben + unten)
- Keine anderen Geräte
