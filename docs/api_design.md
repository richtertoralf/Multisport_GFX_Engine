# API-DESIGN.md – Snowgames.Live Unified Sports API
Version 1.0 – 2025  
Sportarten: Skilanglauf, Biathlon, MTB, Lauf, Schießen, Mehrkampf → modular erweiterbar

 ## 0. API-Grundprinzipien

1. Redis ist Single Source of Truth.
2. Alle Endpunkte sind minimal, klar, eindeutig.
3. Keine Doppelstrukturen: Athletendaten stehen nur unter athlete:<bib>.
4. Frontends können wählen zwischen:
* „leichtgewichtigen“ Antworten (IDs/BIBs)
* „expandierten“ Antworten (vollständige Athletenobjekte)
5. Alle Antworten JSON, alle Zeiten ISO oder Sekunden.
6. Universelles Schema für alle Sportarten → keine Spezial-Endpunkte nötig.

## 1. EVENT API
GET /api/event

Liefert aktuelle Event-Metadaten aus event:current.

Response:
```json
{
  "title": "Silvesterlauf",
  "location": "Sayda",
  "date": "31.12.2025",
  "sport": "xc",
  "discipline": "interval"
}
```

PATCH /api/event

Schreibt Eventdaten nach event:current.

## 2. ATHLETE API
GET /api/athlete/<bib>

Lädt Athletenstammdaten.

Response:
```json
{
  "bib": 1494,
  "firstname": "Noah",
  "lastname": "Brandt",
  "club": "SK Ramsau",
  "association": "WSV",
  "nation": "NOR",
  "class_name": "Herren 66",
  "birthyear": "1957",
  "gender": "m"
}
```

This is always fast, da Redis GET athlete:<bib> direkt liefert.

## 3. STARTLIST API
GET /api/startlist/

Liste aller Klassen + Startzeiten.
→ Ersatz für startlist:* Iteration.

Response:
```json
{
  "classes": [
    {"class": "Schüler U11 w", "start_time": "10:32:00"},
    {"class": "Herren 66", "start_time": "22:08:30"}
  ]
}
```

GET /api/startlist/<class>

Lightweight Startliste (nur BIBs).

Response:
```json
{
  "class_name": "Schüler U11 w",
  "start_time": "10:32:00",
  "athletes": [65, 66, 67, 68, 69, 70] 
}
```

GET /api/startlist/<class>?expanded=1

Vollständige Startliste (inkl. Athletenobjekte).
Wird vom Overlay benutzt.

Response:
```json
{
  "class_name": "Schüler U11 w",
  "start_time": "10:32:00",
  "athletes": [
    {"bib": 65, "firstname": "...", "lastname": "...", ...},
    {"bib": 66, "firstname": "...", "lastname": "...", ...}
  ]
}
```

POST /api/startlist/import

Importiert komplette WinLaufen-Datei.
→ Speichert startlist:<class> Keys (BIB-Liste)
→ Speichert athlete:<bib> Keys (Athletendetails)

Request:
```json
{
  "raw_csv": "....",
  "race_id": "31.12.2025_Sayda_Silvesterlauf"
}
```

## 4. CLASS API (optional, aber nützlich)
GET /api/class/<class>/athletes

Direkt mit Bib-Expands.

Response:
```
{
  "class_name": "Jugend U18 w",
  "athletes": [
    {...}, {...}, ...
  ]
}
```

GET /api/class/<class>/meta

Gibt Streckeninfos zurück.

Response:
```json
{
  "distance_km": 5.0,
  "laps": 2,
  "interval_start_sec": 30,
  "race_type": "interval"
}
```

## 5. RESULTS API (Sortierte Leaderboards)
GET /api/result/<class>

Sortiert nach Zeit (ZSET)

Response:
```json
{
  "class_name": "Herren 21",
  "results": [
    {"bib": 42, "time": 1534.2, "rank": 1},
    {"bib": 58, "time": 1540.7, "rank": 2}
  ]
}
```

## 6. SPLIT API (Zwischenzeiten)
GET /api/split/<class>/<split_id>
Response:
```json
{
  "split": 1,
  "times": {
    "65": 734.5,
    "66": 742.8,
    "73": 752.4
  }
}
```

## 7. BIATHLON APIs
Bahnorientiertes Modell
GET /api/lane/<lane>
lane:<lane>:current

Response:
```json
{
  "lane": 12,
  "bib": 65,
  "series": 1,
  "position": "prone",
  "shots": ["H","M","H","H","M"],
  "shot_times": [2.3,2.1,2.5,2.2,2.8],
  "penalties": 2
}
```

Athletenorientiertes Modell
GET /api/shooting/<bib>/<series>
shooting:<bib>:<series>

Response:
```
{
  "bib": 65,
  "series": 1,
  "position": "prone",
  "hits": 3,
  "misses": 2,
  "penalties": 2,
  "shot_times": [...],
  "timestamp": 1733552100
}
```

## 8. Overlay / Renderer State API
GET /api/state
PATCH /api/state

Zum Steuern von:

Einblendungen

Startliste

Eventanzeige

Start-Overlay

Branding

Caption

Schedule

Outro

## 9. SYSTEM & DIAGNOSTICS
GET /api/debug/keys

Gibt alle sportrelevanten Keys:
```json
{
  "startlist": [...],
  "athletes": [...],
  "results": [...],
  "splits": [...],
  "shooting": [...],
  "lanes": [...]
}
```

GET /api/debug/redis-size
```json
{ "keys": 513 }
```

🎯 API-DESIGN ZUSAMMENFASSUNG
|Bereich |	Endpunkte|
|---|---|
|Event	|/api/event|
|Athleten	|/api/athlete/<bib>|
|Startlisten	|/api/startlist/…|
|Klassen	|/api/class/<class>/…|
|Ergebnisse	|/api/result/<class>|
|Splits	|/api/split/<class>/<id>|
|Biathlon	|/api/lane/<lane>, /api/shooting/<bib>/<series>|
|Renderer	|/api/state|
|Debug	|/api/debug/*|
