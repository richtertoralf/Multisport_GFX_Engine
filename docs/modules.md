# Snowgames Live – Modul-Entwicklungsleitfaden

Dieser Leitfaden beschreibt den vollständigen Ablauf zur Erstellung eines neuen Grafik-Moduls in der **SnowgamesLive Multisport GFX Engine**.  
Alle Module – z. B. Startliste, Start, Finish, Splits, Biathlon-Range, Skisprung-Gate – folgen derselben Architektur.

## Projektstruktur (Kurzüberblick)

Das Projekt ist in Input, Core und Output getrennt. Module gehören immer zum Output Layer.

```
gfx-engine/
│
├── api/                → REST-API (FastAPI): liefert Daten aus Redis an Renderer
├── dashboard/          → Control Panel (setzt gfx:state)
├── input/              → Parser (Startlisten, KES, HoRa, RaceResult …)
├── renderer/           → Overlay für OBS / ATEM / Browser
│     ├── index.html    → Basis-Overlay mit Platzhaltern
│     ├── update.js     → reagiert auf State + lädt API-Daten
│     └── modules/      → einzelne Grafikmodule (start.js, finish.js, …)
└── static/             → CSS, Fonts, Bilder, Assets

```

## Wichtig zu wissen

> Redis ist das zentrale Datenmodell.
Renderer lesen Daten ausschließlich aus der API (niemals direkt aus Parsern).

> Dashboard (`dashboard/control.js`) schreibt ausschließlich gfx:state.
Es rechnet nichts und erzeugt keine Grafiken.

> Renderer (`renderer/update.js`) entscheidet, was gezeigt wird.
anhand von gfx:state + API-Daten.

> **`api/start/logic.py`** als Beispiel und nur zur Info
```python
# Start-Logik: berechnet Anzeigezeitpunkte, Startmodi etc.
# Schreibt in Redis (z.B. gfx:state:start)

```
Die Logik bleibt im Backend! Gehe davon aus, das es mehrere Ansichten gibt, die die exakt die gleichen Daten benötigen.

> Jedes Modul ist vollständig gekapselt:
eigenes HTML, eigenes CSS, eigene JS-Logik.

## Ablauf zur Erstellung eines neuen Moduls
Dieser Workflow gilt für jedes Overlay-Modul, unabhängig von Sportart oder Funktion.

### 1️⃣ Dashboard-Buttons erstellen
In dashboard/control.html UI-Elemente hinzufügen:
```
<button id="btn_myModule_show">Modul anzeigen</button>
<button id="btn_myModule_hide">Modul ausblenden</button>
```

### 2️⃣ Dashboard-Events → Redis-State schreiben
In dashboard/control.js:

```javascript
document.getElementById("btn_myModule_show").onclick = () => {
    sendStateUpdate("mymodule", { show: true });
};

document.getElementById("btn_myModule_hide").onclick = () => {
    sendStateUpdate("mymodule", { show: false });
};

```

➡️ Dashboard setzt nur State
➡️ Logik liegt beim Renderer

### 3️⃣ Neues Modul unter renderer/modules/ anlegen
Datei: `renderer/modules/mymodule.js`  
Ein Modul ist aktuell eine normale JavaScript-Datei, die ihre Funktionen in einen globalen Namespace einträgt.
Dies entspricht der aktuellen Architektur (kein ES-Module-System im Renderer).

```javascript
// Globalen Namespace anlegen, falls noch nicht vorhanden
window.gfxModules = window.gfxModules || {};

window.gfxModules.mymodule = {
    active: false,

    show(data) {
        this.active = true;
        const box = document.getElementById("mymodule_box");
        box.style.opacity = 1;
        box.style.visibility = "visible";

        // Inhalte füllen …
    },

    hide() {
        this.active = false;
        const box = document.getElementById("mymodule_box");
        box.style.opacity = 0;
        box.style.visibility = "hidden";
    }
};

```

🛈 Hinweis:  
Die Module arbeiten aktuell über einen globalen Namespace.  
In gfx-engine v2 wird der Renderer auf ES-Module / import { … } migriert.  
Dann werden Module mit export/import geladen und nicht mehr über window.*.  

### 4️⃣ HTML-Fragment in renderer/index.html
Jedes Modul bekommt einen eigenen Container:
```
<div id="mymodule_box" class="mymodule hidden">
    <div class="title"></div>
    <div class="content"></div>
</div>

```

### 5️⃣ CSS unter static/css/
Datei: `static/css/mymodule.css`
```
#mymodule_box {
    position: absolute;
    left: 3vw;
    bottom: 10vh;
    background: rgba(0,0,0,0.7);
    padding: 1.5vh 2vw;
    border-radius: 8px;
    transition: opacity 0.3s ease;
    opacity: 0;
    visibility: hidden;
}

```
>Stile werden in thematischen CSS-Dateien gepflegt (core.css, layout-tv.css, start.css, startlist.css, …)
>Für neue Module kann entweder eine neue Datei erstellt oder eine bestehende (z. B. start.css) erweitert werden.

### 6️⃣ Modul in renderer/update.js anbinden
update.js greift auf das Modul über den globalen Namespace zu.  
Dies entspricht der aktuellen, nicht-modularen JS-Struktur.  

```javascript
function applyState(state) {

    const M = window.gfxModules;

    // Modul ein-/ausblenden
    if (state.mymodule?.show) {
        M.mymodule.show(state.mymodule);
    } else {
        M.mymodule.hide();
    }
}


```
⚠️ In applyState stehen immer zwei Informationsquellen zur Verfügung:
1. gfx:state (steuert Ein/Aus, Parameter)
2. API-Daten (Athleten, Splits, Schussdaten, usw.)

🛈 Hinweis:
In v2 wird update.js auf ES-Module umgestellt und erhält dann Imports wie:
```javascript
import { myModule } from "./modules/mymodule.js";

```
Bis dahin bleibt die aktuelle Lösung mit window.gfxModules bewusst einfach und kompatibel.

### 7️⃣ Optional: API erweitern (falls nötig)
Wenn ein Modul zusätzliche Daten benötigt:
* Neue API-Routen unter api/<sport>/
* oder Erweiterung des Unified Sports Data Models in Redis

**Wichtig:**
Sportliche Logik liegt immer im Backend/Parser – nie im Modul.

## Zusammenfassung

Ein Modul besteht aus **5 Komponenten**:

| Teil              | Ort                    | Funktion         |
|-------------------|------------------------|------------------|
| Dashboard-Buttons | `/dashboard`           | Bedienung        |
| Redis-State       | API / `sendStateUpdate`| Steuert Renderer |
| JS-Modul          | `/renderer/modules`    | Grafiklogik      |
| HTML-Fragment     | `/renderer/index.html` | Darstellung/Container|
| CSS               | `/static/css`          | Layout & Animation   |


✔ klare Trennung von Logik, Layout, Daten  
✔ jedes Modul ist isoliert, testbar und erweiterbar  
✔ Renderer bleibt sauber und wartbar  
