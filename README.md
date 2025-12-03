# Multisport_GFX_Engine
Multisport GFX Engine

by SnowgamesLive

>Die Multisport GFX Engine ist ein universelles Grafiksystem zur Live-Darstellung von Sportwettkämpfen.
>Sie verbindet Daten aus unterschiedlichen Quellen mit modernen TV-Überblendungen und ermöglicht eine professionelle Präsentation – von Vereinsrennen bis hin zu großen Events.

Die Engine unterstützt mehrere Sportarten wie Skilanglauf, Biathlon, Skispringen, Nordic Combined, MTB und weitere Formate.
Alle Grafiken folgen einem einheitlichen, klaren Design und können automatisch oder manuell gesteuert werden.

Ziel ist ein einfach bedienbares, zuverlässiges und erweiterbares System, das hochwertige Echtzeit-Grafiken produziert und dabei flexibel genug bleibt, um in verschiedensten Szenarien eingesetzt zu werden – von Livestreams bis zur Videowall im Stadion.

SnowgamesLive entwickelt diese Engine speziell für kleine und mittlere Veranstalter, die professionelle TV-Grafiken ohne große Produktionsstrukturen einsetzen möchten.

---

## Systemarchitektur

Die Engine besteht aus drei klar abgegrenzten Ebenen:

* Input Layer – Parser & Datenerfassung
* Core Data Layer – Datenmodell & Redis
* Output Layer – Dashboard, API & Renderer

Diese Struktur ermöglicht hohe Flexibilität, einfache Erweiterbarkeit und Stabilität im Live-Betrieb.

### 1. Input Layer – Parser-Module

Alle externen Datenquellen werden über spezialisierte Parser in das interne Datenmodell übertragen.
Jeder Parser ist ein eigenständiges Modul.

Unterstützte Parser

* Startlist Parser
* HoRa Shooting Parser
* KES Shooting Parser
* Uwe Brechenmacher Distance Parser (Skispringen)
* WinLaufen / WinSpringen Parser
* RaceResult Parser

Aufgabe der Parser

* Einlesen und Validieren externer Daten
* Transformation in das interne Unified Sports Data Model
* Schreiben der Daten in Redis

Die Parser erzeugen ausschließlich Daten – keine Darstellung.

## 2. Core Data Layer – Unified Model & Redis
Unified Sports Data Model

Alle Parser schreiben in ein gemeinsames, sportartübergreifendes Datenmodell, bestehend aus u. a.:

+ event:*
+ athlete:*
+ startlist:*
+ timing:*
+ shooting:*
+ distance:*
+ results:*
+ sportspecific:* (Erweiterungen nach Disziplin)

Dieses Modell bildet die Grundlage für sämtliche Ausgabekanäle.

### Redis – Single Source of Truth

Redis dient als zentraler Datenspeicher für:

* Event- und Athletendaten
* Startlisten
* Zwischenzeiten, Treffer, Weiten
* Laufende Ranglisten
* Systemmetriken
* Grafikstatus (gfx:state)

Alle Systeme im Output Layer greifen ausschließlich über API-Endpunkte auf Redis zu.

## 3. Output Layer – Dashboard, API & Renderer

### Control Dashboard

Das Dashboard ist die zentrale Bedienoberfläche der Engine.

Funktionen:

* Setzen und Aktualisieren des Grafikstatus via gfx:state
* Aktivieren/Deaktivieren einzelner Eingangskanäle
* Importieren von Startlisten (über die Parser)
* Live-Monitoring aller Datenquellen
* Auswählen und Triggern von Grafiken

Das Dashboard verarbeitet keine sportliche Logik und rechnet nichts; es steuert lediglich Zustände.

### API Layer

Die Engine stellt eine einheitliche API bereit, über die alle Renderer Daten abrufen.

Typische API-Endpunkte liefern:

* Eventinformationen
* Athletendaten
* Startlisten
* Timingdaten
* Schießdaten
* Ergebnisse
* den aktuellen Grafikstatus (gfx:state)

### Renderer

Die Ausgabe der Daten erfolgt über spezialisierte Renderer:

* TV / Streaming Overlay Renderer
(HTML/CSS/JS-basierte Grafiken für OBS, vMix, Webbrowser)

* Videowall Renderer
(Großformatige, animierte Darstellungen)

* Commentator Renderer
(Zusatzinformationen und Detaildaten für Moderation & Analyse)

* Mobile Renderer
(Vereinfachte dynamische grafische Ansicht auf Smartphones/Tablets)

Alle Renderer beziehen ihre Daten aus der API und sind vollständig entkoppelt vom Input Layer.

## Ziel des Systems

Die Multisport GFX Engine soll eine einfach bedienbare, universelle und stabile Plattform für Live-Sportgrafiken bieten – unabhängig von Sportart, Veranstalter oder Datenquelle.
