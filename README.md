# Multisport GFX Engine
by SnowgamesLive

>Die Multisport GFX Engine ist ein universelles, modular aufgebautes Grafiksystem zur Live-Darstellung von Sportwettkämpfen.
>Es verbindet Daten aus verschiedensten Zeitmess- und Schießstandsystemen mit modernen TV-Overlays und ermöglicht professionelle Grafiken – für Livestreams, Stadien und mobile Ansichten.

Das System unterstützt mehrere Disziplinen wie Skilanglauf, Biathlon, Skispringen, Nordic Combined, MTB und weitere Formate.
Alle Grafiken basieren auf einer einheitlichen Designsprache, werden aus einer zentralen Datenbank gespeist und lassen sich automatisch oder manuell steuern.

## Ziel:
Eine einfach bedienbare, zuverlässige und erweiterbare Plattform, die hochwertige Echtzeit-Grafiken erzeugt – speziell für kleine und mittlere Veranstalter ohne große TV-Infrastruktur.

---

## Systemarchitektur

Die Engine besteht aus drei klar getrennten Schichten:

1. **Input Layer** – Parser & Datenerfassung
2. **Core Data Layer** – Unified Model & Redis
3. **Output Layer** – Dashboard, API & Renderer

Diese Struktur sorgt für Stabilität im Live-Betrieb, minimale Kopplung der Module und maximale Erweiterbarkeit.

### 1. Input Layer – Parser & Datenerfassung

Alle externen Datenquellen werden über unabhängige Parser-Module eingelesen.
Jedes Modul ist vollständig isoliert und schreibt ausschließlich Daten in Redis.

Unterstützte Parser

* Startlist Parser (Winlaufen, RaceResult)
* HoRa Shooting Parser (Schießstand)
* KES Shooting Parser (Kurvinen-System)
* Uwe Brechenmacher Distance Parser (Skispringen/Weitenmessung)
* WinLaufen / WinSpringen Parser
* RaceResult Parser

Aufgabe der Parser

* Einlesen externer Formate
* Validierung und Fehlerbehandlung
* Transformation in das Unified Sports Data Model
* Schreiben strukturierter Redis-Keys

Die Parser erzeugen keine Grafiken und enthalten keine sportliche Logik.

## 2. Core Data Layer – Unified Model & Redis
Unified Sports Data Model

Alle Parser schreiben in ein sportartenübergreifendes Datenmodell.  
Typische Redis-Strukturen:

+ event:* – Event-Metadaten
+ athlete:* – Stammdaten zu Athleten
+ startlist:* – Startlisten pro Klasse/Disziplin
+ timing:* – Zwischenzeiten, Runden, Passagezeiten
+ shooting:* – Trefferbilder, Schusszeiten, Serien
+ distance:* – Weiten und Landedaten
+ result:* – Endergebnisse, Ranglisten
+ sportspecific:* – Disziplinspezifische Erweiterungen (z. B. Strafzeiten, Nachlader, Ramp-Info)

### Redis – Single Source of Truth

Redis speichert alle Echtzeitdaten:

* Event- und Athletendaten
* Startlisten
* Zwischenzeiten, Lap-Daten
* Schießdaten (Einzel und Bahnorientiert)
* Weitenmessdaten
* Laufende Ranglisten und Aggregationen
* Systemmetriken / Monitoring
* Grafikstatus (gfx:state)

Die gesamte Engine – Dashboard wie Renderer – arbeitet ausschließlich auf Basis dieser Redis-Daten.

## 3. Output Layer – Dashboard, API & Renderer

### Control Dashboard
Die zentrale Bedienoberfläche der Engine.

Hauptfunktionen:

* Setzen und Aktualisieren des Grafikstatus (gfx:state)
* Triggern von Overlays und automatischen Abläufen
* Monitoring der Datenströme und Verfügbarkeit
* Import von Startlisten (über Parser-Trigger)
* Umschalten von Ansichten und Modulen

**Wichtig:**
Das Dashboard berechnet keine sportliche Logik – es steuert ausschließlich Zustände.

### API Layer

Die Engine stellt eine REST-API bereit, über die alle Renderer Daten abrufen.

Typische Endpunkte liefern:

* Eventinformationen
* Athletendaten
* vollständige Startlisten
* Timing-Daten
* Shooting-Daten (Bahn- und Athletenmodell)
* Ergebnisse und Rankings
* aktuellen Grafikzustand (gfx:state)

Die API abstrahiert Redis und stabilisiert damit den Zugriff für alle Frontends.

### Renderer

Die Visualisierung erfolgt über spezialisierte Render-Clients:

#### TV / Streaming Overlay Renderer
* HTML/CSS/JS-basierte Grafiken
* Einbindung in OBS, vMix oder Browser (Osee/Blackmagic)

#### Videowall Renderer
* großformatige, stadionoptimierte Darstellungen
* reduzierte Komplexität, hohe Lesbarkeit

#### Commentator Renderer
* Live-Datenansicht für Moderation und Analyse
* Detaildaten zu Athleten, Shooting, Splits

#### Mobile Renderer
* adaptive Ansicht für Smartphones/Tablets
* Zuschauer- oder VIP-Ansicht möglich

Alle Renderer sind vollständig entkoppelt und greifen nur über die API zu – niemals direkt auf Parser oder Logik.

## Ziel des Systems
Die **Multisport GFX Engine** ist eine universelle, modular erweiterbare Plattform für Live-Sportgrafiken.

Sie bietet:
* klare Trennung aller Schichten
* zuverlässige Echtzeitfähigkeit
* hohe Bedienbarkeit
* volle Sportarten-Unabhängigkeit
* professionelle Darstellung ohne große Produktionsstrukturen

Damit eignet sie sich ideal für:
* Livestreaming (Social Media Kanäle)
* TV-Produktion
* Stadien / Videowalls
* Veranstaltungen mit begrenztem Personal
