# SnowgamesLive – API-Entwicklungsleitfaden

Dieser Leitfaden beschreibt den Standard-Workflow für neue API-Routen in der SnowgamesLive Multisport GFX Engine.

Alle API-Definitionen folgen dem Prinzip:

Parser erzeugen Daten in Redis → API strukturiert diese Daten → Renderer nutzt die API.

Die API enthält keine Präsentationslogik und keine Renderer-Logik.
Der API-Layer dient ausschließlich als Datenschicht.

## Projektstruktur (Backend/API)

Aktuell:
```
api/
│
├── athletes/
│     └── router.py
├── biathlon/
│     └── router.py
├── event/
│     └── router.py
├── start/
│     ├── logic.py
│     ├── scheduler.py
│     └── router.py
├── timing/
│     └── router.py
├── xc/
│     └── router.py
└── utils/
      └── redis.py
```

Neue APIs folgen demselben Muster:

→ eigener Ordner
→ eigenes router.py
→ sauber definierte Endpunkte
→ Zugriff auf Redis über eine Utility-Funktion

## Ablauf zur Erstellung einer neuen API

Dieser Workflow gilt für jede neue Funktion, z. B.:

Athleten anzeigen

Startlisten bereitstellen

Sportler nach Bib auswählen

Zwischenstände nach Schießen 1/2/3/4

Specials: z. B. Wind, Gate, Penalties, Shooting Statistics

### 1️⃣ Schritt: Datenmodell in Redis prüfen

Bevor die API definiert wird, prüfst du:

👉 Welche Daten liegen bereits in Redis?
👉 Welche Keys sind relevant?

Für unser Beispiel Zwischenstand nach Schießen 1–4 könnten Keys so aussehen:

shooting:{bib}:{series}
result:{bib}
lane:{nr}:current


Prüfe im Terminal:

redis-cli KEYS shooting:*
redis-cli KEYS result:*
redis-cli KEYS lane:*

### 2️⃣ Schritt: Neuen API-Ordner + Router anlegen

Beispiel: API für Zwischenstand nach Schießen
```
api/
├── biathlon/
│     ├── __init__.py
│     └── router.py
```

Falls Biathlon schon existiert → dort erweitern.
Falls nicht → neuen Ordner erstellen.

### 3️⃣ Schritt: FastAPI-Router erstellen

Beispiel: api/biathlon/router.py
```
from fastapi import APIRouter
from api.utils.redis import redis_get_json

router = APIRouter(prefix="/biathlon", tags=["Biathlon"])

@router.get("/shooting/intermediate/{series}")
async def get_shooting_intermediate(series: int):
    """
    Liefert den Zwischenstand nach Schießen-Serie 1, 2, 3 oder 4.
    Rückgabe erfolgt sortiert nach Gesamtzeit (inkl. Strafen).
    """

    # Alle Athleten (Bib-Nummern) laden
    keys = redis_get_json("athletes:list")  # Beispielstruktur

    results = []

    for bib in keys:
        shooting_key = f"shooting:{bib}:{series}"
        result_key = f"result:{bib}"

        shooting_data = redis_get_json(shooting_key)
        result_data = redis_get_json(result_key)

        if shooting_data and result_data:
            results.append({
                "bib": bib,
                "name": result_data["name"],
                "nation": result_data["nation"],
                "series": series,
                "hits": shooting_data["hits"],
                "penalties": shooting_data["penalties"],
                "time": result_data["time_after_series"].get(str(series), None)
            })

    # Sortieren nach Zeit
    results_sorted = sorted(results, key=lambda x: x["time"] or 99999)

    return {
        "series": series,
        "count": len(results_sorted),
        "ranking": results_sorted
    }
```

### 4️⃣ Schritt: Router in main.py registrieren

In api/main.py:
```
from api.biathlon.router import router as biathlon_router

app.include_router(biathlon_router)


Jetzt ist der Endpunkt aktiv:

GET /biathlon/shooting/intermediate/1
GET /biathlon/shooting/intermediate/2
```

### 5️⃣ Schritt: Renderer testen

Im Browser:

`http://<server>:8001/biathlon/shooting/intermediate/1`


Wenn JSON sauber kommt → ready für Integration.

### 6️⃣ Schritt: State für Renderer definieren (optional)

Falls das Dashboard dieses API-Modul als Overlay steuern soll → State definieren:

Im Dashboard:
```
sendStateUpdate("biathlon_intermediate", {
    show: true,
    series: 2
});
```
```
In renderer/update.js:

if (state.biathlon_intermediate?.show) {
    const data = await fetch(`/api/biathlon/shooting/intermediate/${state.biathlon_intermediate.series}`)
    biathlonIntermediate.show(await data.json());
} else {
    biathlonIntermediate.hide();
}
```

### 7️⃣ Schritt: Rendering-Modul bauen (analog Modul-Leitfaden)

→ JavaScript-Modul renderer/modules/biathlon_intermediate.js
→ HTML-Container in index.html
→ CSS in static/css/biathlon.css

Das Overlay könnte z. B. Platzierungen + Bib + Treffer/Fehler anzeigen.

### 8️⃣ Schritt: Dokumentieren

Kurze README im Ordner:

api/biathlon/README.md


Mit Infos:

Zweck

Endpunkte

Datenquellen in Redis

Beispiele für Output

Verwendung im Renderer

## Zusammenfassung API-Entwicklungsleitfaden

Ein API-Modul besteht aus 5 Schritten:

Schritt	Aufgabe	Ergebnis
1	Redis-Daten prüfen	Was existiert? Was wird benötigt?
2	Router-Struktur anlegen	api/<sport>/router.py
3	Endpunkte definieren	Liefert strukturiertes JSON
4	Router in main.py registrieren	API erreichbar
5	Renderer-Integration testen	Overlay erhält Daten
6	(optional) Dashboard-State	Steuerung möglich
7	(optional) Dokumentation	Entwickler können es erweitern
🧩 Beispiel-Endpunkte für zukünftige Erweiterungen

Hier sind Endpunkt-Ideen, die super zur Engine passen:

✔ /biathlon/shooting/current

Zeigt live alle Athleten, die gerade schießen.

✔ /biathlon/shooting/series/{series}

Zwischenstand nach Serie X.

✔ /xc/splits/{km}

Zwischenstand an einem Zeitmesspunkt im XC.

✔ /athletes/{bib}

Einzelner Athlet (Name, Nation, Startzeit, aktuelle Zeit).

✔ /start/current

Wer startet als Nächstes, wer ist aufgerufen.

✔ /startlist/{class}

Startliste einer Klasse.

All das folgt exakt dem oben beschriebenen Vorgehen.
