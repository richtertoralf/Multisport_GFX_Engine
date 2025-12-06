# REDIS_DATA_MODEL.md
Unified Redis Data Model for Snowgames.Live Graphics & Timing Engine  
Version 1.0 – 2025

## 1. Grundprinzipien

1. Jeder Athlet besitzt einen eigenen Key
2. Startlisten speichern nur BIB-Listen (keine Athletenobjekte!)
3. Eventdaten stehen zentral in event:current
4. Renderer-States stehen in gfx:* Keys
5. Zeitmessung & Biathlon-Daten werden sportartneutral gespeichert
6. Alle APIs bauen dynamische, kombinierte JSON-Antworten – Redis bleibt die Single Source of Truth

## 2. Athlete Data Model
Key
`athlete:<bib>`

JSON Schema
```
{
  "bib": 1234,
  "firstname": "Anna",
  "lastname": "Müller",
  "gender": "w",
  "birthyear": "2004",

  "nation": "GER",
  "club": "SSV Sayda",
  "association": "SAC",

  "class_name": "Jugend U18 w"
}
```

Erweiterbare Felder:

* "team" (für Staffeln, Teamsportarten)
* "rank" (optional, Overlay-relevant)
* "status": "dns" | "dnf" | "ok"

## 3. Event Data
Key:
`event:current`

JSON Schema:
```
{
  "title": "Silvesterlauf",
  "location": "Sayda",
  "date": "31.12.2025",
  "sport": "xc",
  "discipline": "individual",
  "organizer": "SSV Sayda"
}
```

## 4. Startlisten
Pro Klasse:
`startlist:<class_name>`

Beispiel:
`startlist:Schüler U11 w`

JSON Schema:
```
{
  "class_name": "Schüler U11 w",
  "start_time": "10:32:00",
  "athletes": [65, 66, 67, 68, 69, 70, 71, 72]
}
```

**WICHTIG:**
* keine eingebetteten Athletenobjekte
* reine BIB-Liste → extrem schnell, keine Redundanz

## 5. Klassen-Meta-Daten (optional, aber empfohlen)
Key:
`class_meta:<class_name>`

JSON Schema:
```
{
  "distance_km": 5.0,
  "interval_start_sec": 30,
  "laps": 2,
  "race_type": "interval"   // mass | interval | sprint
}
```

## 6. Results / Leaderboard
Athlete-Finalzeiten (pro Klasse):

Sorted Set:
`result:<class_name>`

Member → BIB, Score → time_in_seconds
```
ZADD result:Jugend U18 w 1534 65
ZADD result:Jugend U18 w 1540 66
```

Beispiel-Zeitformat für APIs:
```
{
  "bib": 65,
  "finish_time": 1534.2,
  "rank": 1
}
```

## 7. Zwischenzeiten (Splits)

Key:
`split:<class_name>:<split_id>`

Beispiel:
`split:Jugend U18 w:1`

JSON Schema:
```
{
  "split_id": 1,
  "times": {
    "65": 734.5,
    "66": 742.8,
    "67": 755.2
  }
}
```

## 8. Biathlon Shooting
Bahn-orientiertes Modell:
`lane:<lane_number>:current`
```
{
  "lane": 12,
  "bib": 65,
  "series": 1,
  "position": "prone",
  "shots": ["H", "M", "H", "H", "M"],
  "shot_times": [2.3, 2.1, 2.5, 2.2, 2.8],
  "penalties": 2
}
```

Athletenorientiertes Modell:
`shooting:<bib>:<series>`
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

## 9. Grafik / Overlay State
Renderer State
`gfx:state`

zentraler Speicher für Sichtbarkeiten:
```
{
  "show": {
    "event": true,
    "caption": false,
    "weather": false,
    "clock": true,
    "branding": true,
    "startlist": false,
    "schedule": false,
    "outro": false,
    "start": false
  },

  "event_view": "normal",
  "startlist_classes": ["Schüler U10 m", "Schüler U10 w"],
  "startlist_race_id": "31.12.2025_Sayda_Silvesterlauf",

  "settings": {
    "start": {
      "show_club": false,
      "show_association": false,
      "show_flag": true
    }
  }
}
```

## 10. System Keys
Alle vom Backend verwalteten Keys
```
system:keys:startlist
system:keys:athletes
system:keys:results
system:keys:splits
```
Optional zur Diagnose.

## 11. API-Konventionen (für alle Sportarten)
|Zweck|Endpoint|Beschreibung|
|-----|-----|-----|
|Event laden	|/api/event |liefert Eventdaten|
|Athlete	|/api/athlete/<bib>	|einzelne Athleten|
|Startlisten	|/api/startlist/<class>	|Athleten-BIBs|
|Startlisten (expandiert)	|/api/startlist/<class>?expanded=1	|Athletenobjekte<
|Klassenliste	|/api/startlist/	|alle Klassen|
|Leaderboard	|/api/result/<class>	|sortierte Liste|
|Splits	|/api/split/<class>/<id>	|Zwischenzeiten|
|Shooting (Bahn)	|/api/lane/<nr>	|aktueller Schütze|
|Shooting (Athlet)	|/api/shooting/<bib>/<series>	|Trefferbilder<
|Overlay-State	|/api/state	|Sichtbarkeiten & Einstellungen|

