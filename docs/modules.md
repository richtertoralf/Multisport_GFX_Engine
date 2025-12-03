# Snowgames Live – Modul-Entwicklungsleitfaden

Dieser Leitfaden beschreibt den vollständigen Ablauf zur Erstellung eines neuen Grafik-Moduls im Snowgames-gfx-System.
Alle Module – z. B. Startliste, Start, Finish, Splits, Biathlon-Range, Skisprung-Gate – folgen der gleichen Architektur.

## Projektstruktur (Kurzüberblick)

Das Projekt folgt einer klaren Trennung in Backend, Renderer, Dashboard und Input-Parser.

```
gfx-engine/
│
├── api/                → FastAPI-Endpunkte, dünne Backend-Schicht (Redis rein/raus)
├── dashboard/          → Bedienoberfläche (Control Panel)
├── input/              → Parser (KES, HoRa, Startlisten, RaceResult …)
├── renderer/           → Overlay-Ausgabe für OBS / ATEM / Webbrowser
│     ├── index.html    → Haupt-Overlay mit allen Platzhaltern
│     ├── update.js     → holt State-Daten & steuert Module
│     └── modules/      → einzelne Grafiken als JS-Module (start.js, finish.js, ...)
└── static/             → CSS, Bilder, Assets
```

## Wichtig zu wissen

> Redis ist das zentrale Model. Alle Module lesen ausschließlich Daten aus der Redis-Struktur bzw. aus der API.

> Dashboard schreibt nur State, rendert aber nichts.

> Renderer liest State + Daten und blendet Grafiken ein/aus.

> Jedes Modul ist vollständig gekapselt: eigenes HTML-Fragment, eigenes CSS, eigene JS-Logik.

## Ablauf zur Erstellung eines neuen Moduls

Dieser Workflow gilt für jedes Overlay-Modul, unabhängig von Sportart oder Funktion.

1️⃣ Dashboard-Buttons erstellen

Im Dashboard (dashboard/control.html) werden Buttons oder UI-Elemente hinzugefügt, um das Modul zu steuern:
```
<button id="btn_myModule_show">Modul anzeigen</button>
<button id="btn_myModule_hide">Modul ausblenden</button>
```

2️⃣ Dashboard-Events → Redis-State schreiben

In `dashboard/control.js` wird definiert, was die Buttons tun.
Standard ist das Setzen eines Eintrags in `gfx:state`:

```
document.getElementById("btn_myModule_show").onclick = () => {
    sendStateUpdate("mymodule", { show: true });
};

document.getElementById("btn_myModule_hide").onclick = () => {
    sendStateUpdate("mymodule", { show: false });
};
```

➡️ Das Dashboard steuert immer nur den State.
➡️ Keine Renderer-Logik im Dashboard.

3️⃣ Neues Modul unter renderer/modules/XYZ.js anlegen

Jedes Modul exportiert ein Objekt mit mindestens show() und hide():
```
export const myModule = {
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

4️⃣ HTML-Fragment in renderer/index.html hinzufügen

Jedes Modul bekommt einen eigenen Container:
```
<div id="mymodule_box" class="mymodule hidden">
    <div class="title"></div>
    <div class="content"></div>
</div>
```

Dieser Container wird vom JS-Modul sichtbar/unsichtbar gemacht.

5️⃣ CSS in static/css/ anlegen

Neue Datei z. B. static/css/mymodule.css:
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

Stilregeln bleiben vollständig getrennt von der Logik.

6️⃣ Modul in update.js anbinden

update.js reagiert auf Änderungen im Redis-State und ruft das Modul:
```
import { myModule } from "./modules/mymodule.js";

function applyState(state) {
    if (state.mymodule?.show) {
        myModule.show(state.mymodule);
    } else {
        myModule.hide();
    }
}
```

➡️ Damit ist das Modul mit dem globalen System verbunden.

7️⃣ Optional: API-Logik ergänzen

Wenn ein Modul zusätzliche Daten benötigt (z. B. Schießstand, Zwischenzeiten, Windmessung):

neuer Router in api/<sport>/router.py

oder Erweiterung bestehender Redis-Keys

Module greifen immer über die API auf Daten zu, nicht auf Parser-Code

Viele Module benötigen keine Backend-Erweiterung — nur State.

## Zusammenfassung

Ein Modul besteht aus **5 Komponenten**:

| Teil              | Ort                    | Funktion         |
|-------------------|------------------------|------------------|
| Dashboard-Buttons | `/dashboard`           | Bedienung        |
| Redis-State       | API / `sendStateUpdate`| Steuert Renderer |
| JS-Modul          | `/renderer/modules`    | Grafiklogik      |
| HTML-Fragment     | `/renderer/index.html` | Container        |
| CSS               | `/static/css`          | Darstellung      |


➡️ Trennung von Logik, Layout und Daten ist Grundprinzip des Systems.
➡️ Jedes Modul ist eigenständig, erweiterbar und austauschbar.
