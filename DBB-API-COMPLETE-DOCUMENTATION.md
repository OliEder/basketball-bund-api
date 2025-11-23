# DBB Basketball-Bund.net REST API - Vollständige Dokumentation

**Version:** 2.0 (Korrigiert mit echten Responses)  
**Basis-URL:** `https://www.basketball-bund.net/rest`  
**Authentifizierung:** Keine erforderlich (öffentliche API)  
**Datum:** 29. Oktober 2025

---

## 📚 Inhaltsverzeichnis

1. [Überblick](#überblick)
2. [Standard Response-Struktur](#standard-response-struktur)
3. [WAM Endpoints](#wam-endpoints)
4. [Competition Endpoints](#competition-endpoints)
5. [Match Endpoints](#match-endpoints)
6. [Team & Club Endpoints](#team--club-endpoints)
7. [Filter-Parameter](#filter-parameter)
8. [Beispiel-Workflows](#beispiel-workflows)
9. [Best Practices](#best-practices)

---

## Überblick

Die Basketball-Bund.net API bietet umfassenden Zugriff auf:
- ✅ Wettbewerbs-/Liga-Suche mit komplexen Filtern
- ✅ Tabellenstände und Kreuztabellen
- ✅ Spielpläne und Live-Ergebnisse
- ✅ Detaillierte Spiel-Statistiken (Boxscores)
- ✅ Team- und Vereinsinformationen

**Wichtig für PWA-Integration:**
- ⚠️ CORS-Proxy erforderlich für Browser-Requests
- ✅ Alle Daten öffentlich zugänglich
- ✅ Keine Rate-Limits dokumentiert (empfohlen: max. 1 req/sec)

---

## Standard Response-Struktur

**Alle Endpoints** liefern Responses im gleichen Wrapper-Format:

```typescript
interface ApiResponse<T> {
  timestamp: string;        // ISO 8601: "2025-10-17T17:12:38+0200"
  status: string;           // "0" = Success
  message: string;          // Fehlermeldung (leer bei Erfolg)
  data: T;                  // Eigentliche Nutzdaten
  version: string;          // API-Version: "11.42.2-b100829"
  dateFormat: string;       // "yyyy-MM-dd"
  timeFormat: string;       // "yyyy-MM-dd'T'HH:mm:ssZ"
  timeFormatShort: string;  // "HH:mm"
  serverInstance: string;   // "www"
  username: string | null;  // null (keine Auth)
  appContext: string;       // "https://www.basketball-bund.net"
}
```

**Status-Codes:**
- `"0"` = Erfolg
- `"1"` = Fehler (Details in `message`)

---

## WAM Endpoints

### 1. WAM Data - Filter-Optionen laden

**Endpoint:** `POST /rest/wam/data`

**Request:**
```json
{
  "token": 0,
  "verbandIds": [],      // Optional: [2] für Bayern
  "gebietIds": [],       // Optional: ["4_"] für Oberpfalz
  "ligatypIds": [],      // Optional
  "akgGeschlechtIds": [], // Optional
  "altersklasseIds": [], // Optional
  "spielklasseIds": []   // Optional
}
```

**Response:**
```json
{
  "timestamp": "2025-10-17T17:12:38+0200",
  "status": "0",
  "message": "",
  "data": {
    "wam": {
      "token": "0",
      "verbandIds": [100],
      "gebietIds": [],
      ...
    },
    "verbaende": [
      {
        "id": 2,
        "label": "Bayern",
        "hits": 374
      }
    ],
    "gebiete": [
      {
        "id": "4_",
        "bezirk": null,
        "kreis": null,
        "hits": 120
      }
    ],
    "altersklassen": [
      {
        "id": 10,
        "label": "U10",
        "hits": 45
      },
      {
        "id": 12,
        "label": "U12",
        "hits": 38
      }
    ],
    "spielklassen": [
      {
        "id": 320,
        "label": "Bezirksliga",
        "hits": 25
      }
    ],
    "ligaListe": {
      "startAtIndex": 0,
      "ligen": [...],  // Erste 10 Ligen
      "hasMoreData": true,
      "size": 10
    }
  },
  "version": "11.42.2-b100829",
  ...
}
```

**Wichtige Felder:**
- `verbaende`: Liste aller Landesverbände mit Hit-Count
- `gebiete`: Bezirke/Kreise (Bayern: `1_` bis `6_`)
- `altersklassen`: U8 bis U20 + Senior
- `spielklassen`: Bezirksliga, Bezirksklasse, etc.
- `ligaListe`: Vorschau der ersten 10 Ligen
- `hits`: Anzahl an Ligen in dieser Kategorie mit gewählter Filter-Option

---

### 2. WAM Liga List - Ligen suchen

**Endpoint:** `POST /rest/wam/liga/list?startAtIndex={offset}`

**Request:**
```json
{
  "token": 0,
  "verbandIds": [2],        // Bayern
  "gebietIds": ["4_"],      // Oberpfalz
  "altersklasseIds": [10],  // U10
  "spielklasseIds": [320]   // Bezirksliga
}
```

**Query Parameters:**
- `startAtIndex` (number): Pagination-Offset (default: 0)

**Response:**
```json
{
  "timestamp": "2025-10-17T18:00:00+0200",
  "status": "0",
  "message": "",
  "data": {
    "ligaListe": {
      "startAtIndex": 0,
      "ligen": [
        {
          "seasonId": 2025,
          "seasonName": "2025/2026",
          "actualMatchDay": null,
          "ligaId": 51961,
          "liganame": "U10m Bezirksliga Oberpfalz",
          "liganr": 1350,
          "skName": "Bezirksliga",
          "skNameSmall": null,
          "skEbeneId": 1,
          "skEbeneName": "Bezirk",
          "akName": "U10",
          "geschlechtId": 1,
          "geschlecht": "männlich",
          "verbandId": 2,
          "verbandName": "Bayern",
          "bezirknr": 4,
          "bezirkName": "Oberpfalz",
          "kreisnr": null,
          "kreisname": null,
          "statisticType": null,
          "vorabliga": false,
          "tableExists": true,
          "crossTableExists": true
        }
      ],
      "hasMoreData": false,
      "size": 10
    }
  },
  ...
}
```

**Wichtige Felder:**
- `ligaId`: Eindeutige Liga-ID (z.B. 51961)
- `liganame`: Vollständiger Name
- `geschlecht`: "männlich", "weiblich", "mix"
- `tableExists`: Hat diese Liga eine Tabelle?
- `crossTableExists`: Hat diese Liga eine Kreuztabelle?
- `hasMoreData`: Weitere Seiten verfügbar?

---

## Competition Endpoints

### 3. Competition List - Liga-Details

**Endpoint:** `POST /rest/competition/list`

**Request:**
```json
[51961, 51962]  // Array von Liga-IDs
```

**Response:** Ähnlich wie WAM Liga List, aber mit vollständigeren Details 
---

### 4. Competition Table - Tabelle

**Endpoint:** `GET /rest/competition/table/id/{ligaId}`

**Response:**
```json
{
  "timestamp": "2025-10-17T17:16:52+0200",
  "status": "0",
  "message": "",
  "data": {
    "prevSpieltag": null,
    "selSpieltag": null,
    "selSpielDatum": null,
    "nextSpieltag": null,
    "ligaData": {
      "seasonId": 2025,
      "seasonName": "2025/2026",
      "actualMatchDay": {
        "spieltag": 5,
        "bezeichnung": "5. Spieltag"
      },
      "ligaId": 51536,
      "liganame": "Herren ProB Süd",
      ...
    },
    "spieltage": null,
    "matches": [],
    "tabelle": {
      "ligaData": null,
      "entries": [
        {
          "rang": 1,
          "team": {
            "seasonTeamId": 429535,
            "teamCompetitionId": 429535,
            "teamPermanentId": 311275,
            "teamname": "BBC Coburg",
            "teamnameSmall": "CB",
            "clubId": 4575,
            "verzicht": false
          },
          "anzspiele": 4,
          "anzGewinnpunkte": 8,
          "anzVerlustpunkte": 0,
          "s": 4,        // Siege
          "n": 0,        // Niederlagen
          "koerbe": 362, // Erzielte Punkte
          "gegenKoerbe": 271, // Erhaltene Punkte
          "korbdiff": 91      // Differenz
        }
      ],
      "bbl": false
    },
    "kreuztabelle": null,
    "teamStatistik": null
  },
  ...
}
```

**Mapping für Datenbank:**
- `rang` → `platz`
- `teamname` → `name`
- `s` → `siege`
- `n` → `niederlagen`
- `koerbe` → `punkte_erzielt`
- `gegenKoerbe` → `punkte_erhalten`
- `korbdiff` → `differenz`
- `anzGewinnpunkte` → `tabellenpunkte`

**Wichtig:**
- `teamPermanentId`: Bleibt über Saisons gleich
- `seasonTeamId` / `teamCompetitionId`: Ändert sich pro Saison

---

### 5. Competition Spielplan - Spielplan

**Endpoint:** `GET /rest/competition/spielplan/id/{ligaId}`

**Response:**
```json
{
  "timestamp": "2025-10-17T18:09:02+0200",
  "status": "0",
  "message": "",
  "data": {
    "prevSpieltag": null,
    "selSpieltag": null,
    "selSpielDatum": null,
    "nextSpieltag": null,
    "ligaData": {
      "seasonId": 2025,
      "seasonName": "2025/2026",
      "actualMatchDay": {
        "spieltag": 8,
        "bezeichnung": "8. Spieltag"
      },
      "ligaId": 51933,
      "liganame": "U14 weiblich Bezirksoberliga",
      ...
    },
    "spieltage": null,
    "matches": [
      {
        "ligaData": null,
        "matchId": 2803682,
        "matchDay": 6,
        "matchNo": 1496,
        "kickoffDate": "2025-10-05",    // YYYY-MM-DD
        "kickoffTime": "18:00",         // HH:MM
        "homeTeam": {
          "seasonTeamId": 432429,
          "teamCompetitionId": 432429,
          "teamPermanentId": 186126,
          "teamname": "FC Tegernheim",
          "teamnameSmall": "TEG1",
          "clubId": 428,
          "verzicht": false
        },
        "guestTeam": {
          "seasonTeamId": 432428,
          "teamCompetitionId": 432428,
          "teamPermanentId": 194456,
          "teamname": "Regensburg Baskets",
          "teamnameSmall": null,
          "clubId": 546,
          "verzicht": false
        },
        "result": "36:62",              // String! "heim:gast"
        "ergebnisbestaetigt": false,
        "statisticType": null,
        "verzicht": false,
        "abgesagt": false,
        "matchResult": null,
        "matchInfo": null,
        "matchBoxscore": null,
        "playByPlay": null,
        "hasPlayByPlay": null
      }
    ]
  },
  ...
}
```

**Wichtige Felder:**
- `kickoffDate` + `kickoffTime`: String-Kombination (nicht ISO!)
- `result`: String im Format "heim:gast" (z.B. "36:62")
- `verzicht`: Team hat aufgegeben?
- `abgesagt`: Spiel wurde abgesagt?
- `ergebnisbestaetigt`: Ergebnis bestätigt?

**Status-Ableitung:**
```typescript
function getSpielStatus(match: Match): SpielStatus {
  if (match.abgesagt) return "abgesagt";
  if (match.result === null) return "geplant";
  if (!match.ergebnisbestaetigt) return "live";  // oder "vorläufig"
  return "beendet";
}
```

---

### 6. Competition Crosstable - Kreuztabelle

**Endpoint:** `GET /rest/competition/crosstable/id/{ligaId}`

**Verfügbarkeit:** Nur wenn `crossTableExists: true` in Liga-Daten

**Response:**
```json
{
  "timestamp": "2025-10-29T13:30:31+0100",
  "status": "0",
  "message": "",
  "data": {
    "ligaData": {...},
    "spieltage": null,
    "matches": [],
    "tabelle": null,
    "kreuztabelle": {
      "ligaData": null,
      "teams": [
        {
          "seasonTeamId": 432429,
          "teamCompetitionId": 432429,
          "teamPermanentId": 186126,
          "teamname": "FC Tegernheim",
          "teamnameSmall": "TEG1",
          "clubId": 428,
          "verzicht": false
        }
      ],
      "crossTable": [
        [
          {
            "home": true,
            "result": "36:62",
            "matchId": 2803682
          },
          {
            "home": false,
            "result": null,
            "matchId": 2803695
          }
        ]
      ]
    },
    "teamStatistik": null
  },
  "version": "11.42.2-d6f6aca",
  ...
}
```

**Struktur:**
- `teams`: Array aller Teams in der Liga (gleiche Reihenfolge wie Matrix)
- `crossTable`: 2D-Matrix (Array von Arrays)
  - Äußeres Array: Zeilen (Heimteams)
  - Inneres Array: Spalten (Gastteams)
  - Diagonale (i === j): Heimspiel-Markierung

**Kreuztabellen-Zelle:**
```typescript
interface CrossTableCell {
  home: boolean;      // true = Heimspiel, false = Auswärtsspiel
  result: string | null;  // "heim:gast" oder null (wenn noch nicht gespielt)
  matchId: number;    // Spiel-ID für Details
}
```

**Interpretation:**
```typescript
// Teams: [A, B, C, D]
// crossTable[i][j] = Spiel: Team i (Heim) vs Team j (Gast)

// Beispiel: crossTable[0][1]
// = Spiel zwischen Team 0 (FC Tegernheim) als Heim
//   und Team 1 als Gast
// → { home: true, result: "36:62", matchId: 2803682 }

// Beispiel: crossTable[1][0]  
// = Rückspiel (Team 1 Heim vs Team 0 Gast)
// → { home: false, result: null, matchId: 2803695 }
```

**Best Practice:**
```typescript
function getCrossTableResult(
  crossTable: CrossTableCell[][],
  teams: Team[],
  homeTeamId: number,
  guestTeamId: number
): string | null {
  const homeIdx = teams.findIndex(t => t.teamPermanentId === homeTeamId);
  const guestIdx = teams.findIndex(t => t.teamPermanentId === guestTeamId);
  
  if (homeIdx === -1 || guestIdx === -1) return null;
  
  const cell = crossTable[homeIdx][guestIdx];
  return cell?.result || null;
}
```

---

### 7. Competition Matchday - Spieltag-Ansicht

**Endpoint:** `GET /rest/competition/id/{ligaId}/matchday/{matchDay}`

**Parameter:**
- `ligaId` (required): Liga-ID
- `matchDay` (optional): Spieltag-Nummer (1-N)
  - Wenn nicht angegeben: Aktueller Spieltag

**Response:**
```json
{
  "timestamp": "2025-10-29T13:30:31+0100",
  "status": "0",
  "message": "",
  "data": {
    "prevSpieltag": {
      "spieltag": 5,
      "bezeichnung": "5. Spieltag"
    },
    "selSpieltag": {
      "spieltag": 6,
      "bezeichnung": "6. Spieltag"
    },
    "selSpielDatum": null,
    "nextSpieltag": {
      "spieltag": 7,
      "bezeichnung": "7. Spieltag"
    },
    "ligaData": {
      "seasonId": 2025,
      "seasonName": "2025/2026",
      "actualMatchDay": {
        "spieltag": 4,
        "bezeichnung": "4. Spieltag"
      },
      "ligaId": 51933,
      "liganame": "U14 weiblich Bezirksoberliga",
      ...
    },
    "spieltage": [
      {
        "spieltag": 1,
        "bezeichnung": "1. Spieltag"
      },
      {
        "spieltag": 2,
        "bezeichnung": "2. Spieltag"
      }
    ],
    "matches": [
      {
        "matchId": 2803682,
        "matchDay": 6,
        "kickoffDate": "2025-10-05",
        "kickoffTime": "18:00",
        "homeTeam": {...},
        "guestTeam": {...},
        "result": "36:62",
        "ergebnisbestaetigt": false,
        "verzicht": false,
        "abgesagt": false
      }
    ],
    "tabelle": null,
    "kreuztabelle": null,
    "teamStatistik": null
  },
  "version": "11.42.2-d6f6aca",
  ...
}
```

**Wichtige Felder:**
- `prevSpieltag`: Vorheriger Spieltag (Navigation)
- `selSpieltag`: Aktuell ausgewählter Spieltag
- `nextSpieltag`: Nächster Spieltag (Navigation)
- `actualMatchDay`: Echter aktueller Spieltag der Liga
- `spieltage`: Alle verfügbaren Spieltage als Array
- `matches`: Nur Spiele des ausgewählten Spieltags

**Use Case:**
```typescript
// Aktuellen Spieltag laden (ohne matchDay Parameter)
const current = await fetch(`/rest/competition/id/${ligaId}/matchday`);

// Bestimmten Spieltag laden
const spieltag6 = await fetch(`/rest/competition/id/${ligaId}/matchday/6`);

// Navigation: Vorheriger/Nächster Spieltag
const { prevSpieltag, nextSpieltag } = spieltag6.data;
if (prevSpieltag) {
  const prev = await fetch(`/rest/competition/id/${ligaId}/matchday/${prevSpieltag.spieltag}`);
}
```

### 8. Competition Actual - Live-Spiele

**Endpoint:** `GET /rest/competition/actual/id/{ligaId}`

**Response:** Ähnlich wie Spielplan, aber nur laufende Spiele mit Live-Zwischenständen.

---

### 9. Competition Team Statistics

**Endpoint:** `GET /rest/competition/teamstatistic/id/{ligaId}`

**Verfügbarkeit:** Nur verfügbar bei Ligen mit vollständiger Spielstatistik

**Response:**
```json
{
  "timestamp": "2025-10-18T01:34:21+0200",
  "status": "0",
  "message": "",
  "data": {
    "prevSpieltag": null,
    "selSpieltag": null,
    "selSpielDatum": null,
    "nextSpieltag": null,
    "ligaData": {
      "seasonId": 2025,
      "seasonName": "2025/2026",
      "actualMatchDay": {
        "spieltag": 1,
        "bezeichnung": "1. Spieltag"
      },
      "ligaId": 51938,
      "liganame": "U18 männlich Bezirksoberliga",
      "statisticType": 1,
      ...
    },
    "spieltage": null,
    "matches": [],
    "tabelle": null,
    "kreuztabelle": null,
    "teamStatistik": {
      "ligaData": null,
      "stand": "2025-10-11",  // Letztes Update
      "statisticEntries": [
        {
          "esz": 0.0,        // Einsatzzeit (meist nicht erfasst)
          "pts": 174.0,      // Punkte gesamt
          "twoPoints": {
            "made": 56.0,     // Erfolgreiche 2-Punkte-Würfe
            "attempted": 0.0, // Versuche (oft nicht erfasst)
            "quota": null     // Quote (null wenn attempted = 0)
          },
          "threePoints": {
            "made": 12.0,     // Erfolgreiche 3-Punkte-Würfe
            "attempted": 0.0,
            "quota": null
          },
          "wt": {            // Würfe Total (Field Goals)
            "made": 68.0,     // made = twoPoints.made + threePoints.made
            "attempted": 0.0,
            "quota": null
          },
          "onePoints": {     // Freiwürfe
            "made": 26.0,
            "attempted": 49.0, // Oft erfasst!
            "quota": 0.5306122448979592  // 26/49 = 53.06%
          },
          "ro": 0.0,         // Rebounds Offensiv (selten erfasst)
          "rd": 0.0,         // Rebounds Defensiv (selten erfasst)
          "rt": 0.0,         // Rebounds Total (selten erfasst)
          "as": 0.0,         // Assists (selten erfasst)
          "st": 0.0,         // Steals (selten erfasst)
          "to": 0.0,         // Turnovers (selten erfasst)
          "bs": 0.0,         // Blocks (selten erfasst)
          "fouls": 17.0,     // Fouls (meist erfasst)
          "eff": 219.0,      // Efficiency Rating
          "tableTeamEntry": {
            "rang": 1,
            "team": {
              "seasonTeamId": 432449,
              "teamCompetitionId": 432449,
              "teamPermanentId": 167036,
              "teamname": "Regensburg Baskets",
              "teamnameSmall": null,
              "clubId": 546,
              "verzicht": false
            },
            "anzspiele": 2,
            "anzGewinnpunkte": 4,
            "anzVerlustpunkte": 0,
            "s": 2,          // Siege
            "n": 0,          // Niederlagen
            "koerbe": 174,   // Punkte erzielt
            "gegenKoerbe": 126, // Punkte erhalten
            "korbdiff": 48   // Differenz
          }
        }
      ],
      "bbl": false  // Ist Bundesliga?
    }
  },
  "version": "11.42.2-b100829",
  ...
}
```

**TypeScript Interface:**
```typescript
interface TeamStatistik {
  ligaData: any | null;
  stand: string;  // YYYY-MM-DD (letztes Update)
  statisticEntries: StatisticEntry[];
  bbl: boolean;
}

interface StatisticEntry {
  esz: number;           // Einsatzzeit (meist 0.0)
  pts: number;           // Punkte gesamt
  twoPoints: WurfStats;  // 2-Punkte-Würfe
  threePoints: WurfStats; // 3-Punkte-Würfe
  wt: WurfStats;         // Würfe Total (Field Goals)
  onePoints: WurfStats;  // Freiwürfe
  ro: number;            // Rebounds Offensiv (selten erfasst)
  rd: number;            // Rebounds Defensiv (selten erfasst)
  rt: number;            // Rebounds Total (selten erfasst)
  as: number;            // Assists (selten erfasst)
  st: number;            // Steals (selten erfasst)
  to: number;            // Turnovers (selten erfasst)
  bs: number;            // Blocks (selten erfasst)
  fouls: number;         // Fouls
  eff: number;           // Efficiency Rating
  tableTeamEntry: TabellenEintrag; // Verknüpfung zur Tabelle
}

interface WurfStats {
  made: number;          // Erfolgreiche Würfe
  attempted: number;     // Versuche (oft 0.0 wenn nicht erfasst)
  quota: number | null;  // Quote (null wenn attempted = 0)
}
```

**Wichtige Hinweise:**

1. **Erfassung variiert stark:**
   - ✅ Immer erfasst: `pts`, `made` (Würfe), `fouls`, `eff`
   - ✅ Meist erfasst: `onePoints.attempted` (Freiwurfversuche)
   - ⚠️ Selten erfasst: `twoPoints.attempted`, `threePoints.attempted`
   - ❌ Sehr selten: `ro`, `rd`, `rt`, `as`, `st`, `to`, `bs`

2. **Quote-Berechnung:**
   ```typescript
   function calculateQuota(stats: WurfStats): number | null {
     if (stats.attempted === 0) return null;
     return stats.made / stats.attempted;
   }
   ```

3. **Field Goals (wt):**
   ```typescript
   wt.made === twoPoints.made + threePoints.made
   pts === (twoPoints.made * 2) + (threePoints.made * 3) + onePoints.made
   ```

4. **Efficiency Rating:**
   - Formel variiert je nach Liga
   - Meist: `(pts + rb + as + st + bs) - (missed_fg + missed_ft + to)`

5. **Stand-Datum:**
   - Zeigt letztes Update der Statistiken
   - Format: `YYYY-MM-DD`
   - Kann vom aktuellen Spieltag abweichen

**Use Cases:**

```typescript
// 1. Beste Wurfquote (Freiwürfe) finden
const bestFreeThrowTeam = teamStatistik.statisticEntries
  .filter(e => e.onePoints.attempted > 0)
  .sort((a, b) => (b.onePoints.quota || 0) - (a.onePoints.quota || 0))[0];

// 2. Top Scorer
const topScorer = teamStatistik.statisticEntries
  .sort((a, b) => b.pts - a.pts)[0];

// 3. Durchschnittliche Punkte pro Spiel
function calculateAveragePts(entry: StatisticEntry): number {
  if (entry.tableTeamEntry.anzspiele === 0) return 0;
  return entry.pts / entry.tableTeamEntry.anzspiele;
}

// 4. 3-Punkte-Quote (wenn erfasst)
function getThreePointPercentage(entry: StatisticEntry): number | null {
  const { threePoints } = entry;
  if (threePoints.attempted === 0) return null;
  return (threePoints.made / threePoints.attempted) * 100;
}

// 5. Sortiert nach Tabellenplatz
const sortedByRank = teamStatistik.statisticEntries
  .sort((a, b) => a.tableTeamEntry.rang - b.tableTeamEntry.rang);
```

**Hinweis für U8-U14:**
- Bei `statisticType: 1` (reduzierte Statistik)
- Noch weniger Felder erfasst
- Fokus auf Punkte und Fouls

---

## Match Endpoints

### 10. Match - Basis-Infos

**Endpoint:** `GET /rest/match/id/{matchId}`

**Response:** Einzelnes Spiel mit gleicher Struktur wie im Spielplan.

---

### 11. Match Info - Erweiterte Details

**Endpoint:** `GET /rest/match/id/{matchId}/matchInfo`

**Response:**
```json
{
  "timestamp": "2025-10-16T14:03:39+0200",
  "status": "0",
  "message": "",
  "data": {
    "ligaData": {...},
    "matchId": 2774284,
    "matchDay": 35,
    "matchNo": 406,
    "kickoffDate": "2025-08-31",
    "kickoffTime": "17:00",
    "homeTeam": {...},
    "guestTeam": {...},
    "result": null,
    "ergebnisbestaetigt": false,
    "statisticType": 0,
    "verzicht": false,
    "abgesagt": false,
    "matchResult": null,
    "matchInfo": {
      "topPerformances": [],
      "spielfeld": {
        "id": 108926,
        "bezeichnung": "S-Arena Bernau",
        "strasse": "Ladeburger Ch. 2",
        "plz": "16321",
        "ort": "Bernau"
      },
      "srList": [
        {
          "rolle": {
            "id": 1,
            "name": "1. Schiedsrichter",
            "kurzname": "SR1"
          },
          "personData": {
            "id": 23439,
            "vorname": "Susanne",
            "nachname": "Winking",
            "anonym": false
          }
        },
        {
          "rolle": {
            "id": 2,
            "name": "2. Schiedsrichter",
            "kurzname": "SR2"
          },
          "personData": {
            "id": 0,
            "vorname": "***",
            "nachname": "****",
            "anonym": true
          }
        }
      ]
    },
    "matchBoxscore": null,
    "playByPlay": null,
    "hasPlayByPlay": false
  },
  ...
}
```

**Wichtige Felder:**
- `spielfeld`: Halle mit Adresse
- `srList`: Schiedsrichter (kann anonymisiert sein)
- `anonym: true` → Platzhalter `"***"` / `"****"`

---

### 12. Match Boxscore - Spieler-Statistiken

**Endpoint:** `GET /rest/match/id/{matchId}/boxscore`

**Response:**
```json
{
  "timestamp": "2025-10-16T14:04:37+0200",
  "status": "0",
  "message": "",
  "data": {
    "ligaData": {...},
    "matchId": 2803589,
    "kickoffDate": "2025-10-04",
    "kickoffTime": "14:45",
    "homeTeam": {...},
    "guestTeam": {...},
    "result": "49:94",
    "ergebnisbestaetigt": true,
    "statisticType": 1,  // 0 = voll, 1 = reduziert (U8-U14)
    "matchResult": {
      "heimEndstand": 49,
      "gastEndstand": 94,
      "heimHalbzeitstand": 8,
      "gastHalbzeitstand": 25,
      "heimV1stand": 12,    // Viertel 1
      "gastV1stand": 32,
      "heimV3stand": 13,    // Viertel 3
      "gastV3stand": 18,
      "heimV4stand": 16,    // Viertel 4
      "gastV4stand": 19,
      "heimOt1stand": null, // Overtime
      "gastOt1stand": null,
      ...
    },
    "matchBoxscore": {
      "homePlayerStats": [
        {
          "esz": 0.0,         // Einsatzzeit (meist 0 bei Minis)
          "pts": 10.0,        // Punkte
          "twoPoints": {
            "made": 5.0,
            "attempted": 0.0,
            "quota": null
          },
          "threePoints": {
            "made": 0.0,
            "attempted": 0.0,
            "quota": null
          },
          "wt": {             // Würfe Total
            "made": 5.0,
            "attempted": 0.0,
            "quota": null
          },
          "onePoints": {      // Freiwürfe
            "made": 5.0,
            "attempted": 0.0,
            "quota": null
          },
          "ro": 0.0,          // Rebounds Offensiv
          "rd": 0.0,          // Rebounds Defensiv
          "rt": 0.0,          // Rebounds Total
          "as": 0.0,          // Assists
          "st": 0.0,          // Steals
          "to": 0.0,          // Turnovers
          "bs": 0.0,          // Blocks
          "fouls": 4.0,       // Fouls
          "eff": 15.0,        // Efficiency
          "player": {
            "playerId": 4153626,
            "no": "0",        // Rückennummer (oft "0" bei U8-U14)
            "person": {
              "id": 976797,
              "vorname": "Minh Khôi",
              "nachname": "Nguyen",
              "anonym": false
            },
            "anonym": false
          },
          "isf": false        // Is Foreigner?
        },
        {
          // ANONYMISIERTER SPIELER (U8-U14)
          "pts": 2.0,
          "player": {
            "playerId": 0,
            "no": "**",
            "person": {
              "id": 0,
              "vorname": "***",
              "nachname": "****",
              "anonym": true
            },
            "anonym": true
          },
          ...
        }
      ],
      "homeTeamStats": null,
      "homeTotalStats": {
        "pts": 49.0,
        "twoPoints": {
          "made": 22.0,
          "attempted": 0.0,
          "quota": null
        },
        ...
        "fouls": 23.0,
        "eff": 62.0
      },
      "guestPlayerStats": [...],
      "guestTeamStats": null,
      "guestTotalStats": {...}
    },
    "playByPlay": null,
    "hasPlayByPlay": false
  },
  ...
}
```

**Wichtig:**
- `statisticType: 1` → Reduzierte Stats (U8-U14)
- `anonym: true` → Anonymisierte Spieler
- `attempted: 0.0` → Oft nicht erfasst bei Minis
- `quota: null` → Quote nicht berechnet

**Anonymisierung (U8-U14):**
```typescript
if (player.person.anonym === true) {
  // vorname: "***", nachname: "****"
  // playerId: 0, person.id: 0
  // no: "**"
}
```

---

## Team & Club Endpoints

### 13. Team Matches - Team-Spiele

**Endpoint:** `GET /rest/team/id/{teamPermanentId}/matches`

**Wichtig:** `teamPermanentId` verwenden (nicht seasonTeamId)!

**Response:** Array aller Spiele des Teams (vergangene + zukünftige)

---

### 14. Club Actual Matches - Vereinsspiele

**Endpoint:** `GET /rest/club/id/{clubId}/actualmatches?justHome={bool}&rangeDays={days}`

**Query Parameters:**
- `justHome` (boolean): Nur Heimspiele? (default: false)
- `rangeDays` (number): Zeitraum in Tagen (default: 7)

**Response:** Alle aktuellen/kommenden Spiele eines Vereins

---

## Filter-Parameter

### Verbände (verbandIds)
| ID  | Name | Hits (Beispiel) |
|-----|-------------------------------|----:|
| 1	  | Baden-Württemberg	            | 224 |
| 2	  | Bayern                        | 374 |
| 3	  | Berlin	                      | 103 |
| 4	  | Bremen	                      | 40  |
| 5	  | Hamburg                       |	71  |
| 6   |	Hessen                        |	130 |
| 7   |	Niedersachsen                 | 205 |
| 8   |	Rheinland-Pfalz               |	83  |
| 9   |	Saarland                      |	 28 |
| 10  |	Schleswig-Holstein            |	 47 |
| 11  |	Nordrhein-Westfalen           |	347 |
| 12  |	Mecklenburg-Vorpommern        |	26  |
| 13  |	Sachsen-Anhalt                |	79  |
| 14  |	Brandenburg                   |	40  |
| 15  |	Sachsen                       |	106 |
| 16  |	Thüringen                     |	52  |
| 29  |	Deutsche Meisterschaften      |	8   |
| 30  |	Regionalliga Nord             |	8   |
| 31  |	Regionalliga Südwest	        | 21  |
| 32  |	Regionalliga Südost	          | 6   |
| 33  |	Regionalliga West	            | 10  |
| 40  |	Deutscher Rollstuhlbasketball	| 21  |
| 100	| Bundesligen	                  | 33  |


### Altersklassen (altersklasseIds)

| ID | Label.   | Hits (Beispiel) |
|----|----------|----------------:|
| 1	 | Senioren	| 573             |
| 8	 | U8	      | 18              |
| 9	 | U9	      | 11              |
| 10 | U10	    | 163             |
| 11 | U11	    | 17              |
| 12 | U12	    | 295             |
| 13 | U13	    | 13              |
| 14 | U14	    | 323             |
| 15 | U15	    | 18              |
| 16 | U16	    | 312             |
| 17 | U17	    | 6               |
| 18 | U18	    | 201             |
| 19 | U19	    | 10              |
| 20 | U20	    | 32              |
| 30 | Ü30	    | 2               |
| 32 | Ü35	    | 33              |
| 40 | Ü40	    | 30              |
| 45 | Ü45	    | 3               |
| 50 | Ü50	    | 1               |
| 55 | Ü55	    | 1               |

### Altersklassen (agkGeschlechtListIds)

| id   | akg      | geschlecht |
|------|----------|------------|
| 2_1	 | Jugend	  | m          |
| 2_2	 | Jugend	  | w          | 
| 3_1	 | Senioren | m          | 
| 3_2	 | Senioren | w          |

### Bayern Gebiete (gebietIds)

| ID | Name          |             
|----|---------------|   
| 1_ | Oberbayern    |   
| 2_ | Schwaben      |   
| 3_ | Mittelfranken |   
| 4_ | Oberpfalz     |   
| 5_ | Oberfranken   |   
| 6_ | Unterfranken  |   

### Spielklassen (spielklasseIds)

Eher nicht als Filter brauchbar, da jeder Verband seine Eigenen Spielklassen hat.
die Liste kann aber auf der Response von WAM Data ohne filter  mit folgenden Path ermittelt werden.

```jsonPath
$.data.spielklassen[].id // id
$.data.spielklassen[*].label // label
```

| ID    | Label                        |
|-------|------------------------------|
| 100	  | 1. Bundesliga                |
| 101	  | 2. Bundesliga                |
| 102	  | Jugend-Bundesliga            |
| 103	  | DBBL-Pokal                   |
| 104	  | BBL-Pokal                    |
| 105	  | Regionalliga                 |
| 110	  | Bezirksliga                  |
| 115	  | Oberliga                     |
| 120	  | Landesliga                   |
| 125	  | Kreisliga                    |
| 127	  | Kreisliga A                  |
| 128	  | Kreisliga B                  |
| 130	  | keine	                       |
| 205	  | Oberliga                     |
| 210	  | Landesliga                   |
| 215	  | Bezirksliga                  |
| 220	  | Kreisliga                    | 
| 225	  | Mini-F                       | 
| 230	  | Mini-A                       |  
| 232	  | Mini-N                       |  
| 315	  | Bayernliga                   |
| 317	  | Landesliga                   |
| 320	  | Bezirksliga                  |
| 325	  | Bezirksklasse                |
| 330	  | Kreisliga                    |
| 335	  | Kreisklasse                  |  
| 336	  | Bayernpokal                  |
| 337	  | Bay. Meisterschaft           |  
| 338	  | Bezirksoberliga              | 
| 339	  | Bezirkspokal                 |
| 340	  | Bezirksmeisterschaft         |        
| 402	  | Oberliga                     |
| 403	  | Landesliga                   |
| 404	  | Bezirksliga                  |
| 405	  | A-Klasse                     | 
| 406	  | B-Klasse                     | 
| 407	  | Bezirks-Leistungsklasse      |
| 408	  | Bezirks-Klasse               |  
| ...   | ...                          |
| 2812	| Pokal                        |
| 2813	| 1. Bundesliga	               |
| 2814	| 2. Bundesliga                |


### Geschlecht (geschlechtId)

| ID | Label |
|----|-------|
| 1 | männlich |
| 2 | weiblich |
| 3 | mix |

---

## Beispiel-Workflows

### Workflow 1: Liga suchen und Daten laden

```typescript
// 1. Filter-Optionen laden
const optionsResponse = await fetch('/rest/wam/data', {
  method: 'POST',
  body: JSON.stringify({ token: 0 })
});
const { data: { verbaende, gebiete, altersklassen } } = await optionsResponse.json();

// 2. Liga suchen
const searchResponse = await fetch('/rest/wam/liga/list?startAtIndex=0', {
  method: 'POST',
  body: JSON.stringify({
    token: 0,
    verbandIds: [2],      // Bayern
    gebietIds: ["4_"],    // Oberpfalz
    altersklasseIds: [10], // U10
    spielklasseIds: [320]  // Bezirksliga
  })
});
const { data: { ligaListe } } = await searchResponse.json();

// 3. Erste Liga auswählen
const ligaId = ligaListe.ligen[0].ligaId; // z.B. 51961

// 4. Tabelle laden
const tabelleResponse = await fetch(`/rest/competition/table/id/${ligaId}`);
const { data: { tabelle } } = await tabelleResponse.json();

// 5. Spielplan laden
const spielplanResponse = await fetch(`/rest/competition/spielplan/id/${ligaId}`);
const { data: { matches } } = await spielplanResponse.json();

// Ergebnis parsen (String "36:62" → {heim: 36, gast: 62})
const parseResult = (result: string) => {
  const [heim, gast] = result.split(':').map(Number);
  return { heim, gast };
};
```

### Workflow 2: Spiel-Details laden

```typescript
const matchId = 2803682;

// 1. Match-Info mit Halle & SR
const infoResponse = await fetch(`/rest/match/id/${matchId}/matchInfo`);
const { data: { matchInfo, spielfeld, srList } } = await infoResponse.json();

// 2. Boxscore (nur bei beendeten Spielen)
const boxscoreResponse = await fetch(`/rest/match/id/${matchId}/boxscore`);
const { data: { matchBoxscore, matchResult } } = await boxscoreResponse.json();

// Anonyme Spieler filtern
const realPlayers = matchBoxscore.homePlayerStats.filter(
  (p) => !p.player.person.anonym
);
```

---

## Best Practices

### 1. CORS-Proxy verwenden

```typescript
const PROXY = 'https://your-proxy.com/';
const BASE_URL = 'https://www.basketball-bund.net/rest';

async function apiCall(endpoint: string, options?: RequestInit) {
  const response = await fetch(`${PROXY}${BASE_URL}${endpoint}`, options);
  const json = await response.json();
  
  // Status prüfen
  if (json.status !== "0") {
    throw new Error(json.message || 'API Error');
  }
  
  return json.data;  // Nur die Nutzdaten zurückgeben
}
```

### 2. Typ-sichere Response-Handler

```typescript
interface ApiResponse<T> {
  timestamp: string;
  status: string;
  message: string;
  data: T;
  version: string;
  ...
}

async function fetchLigaData(ligaId: number): Promise<TabelleData> {
  const response = await fetch(`/rest/competition/table/id/${ligaId}`);
  const json: ApiResponse<TabelleData> = await response.json();
  
  if (json.status !== "0") {
    throw new Error(json.message);
  }
  
  return json.data;
}
```

### 3. Datum/Zeit parsen

```typescript
// API liefert: kickoffDate: "2025-10-05", kickoffTime: "18:00"
function parseKickoff(date: string, time: string): Date {
  return new Date(`${date}T${time}:00`);
}

// Beispiel
const spiel = matches[0];
const datum = parseKickoff(spiel.kickoffDate, spiel.kickoffTime);
// → Date: 2025-10-05T18:00:00
```

### 4. Ergebnis parsen

```typescript
interface Result {
  heim: number;
  gast: number;
}

function parseResult(result: string | null): Result | null {
  if (!result) return null;
  const [heim, gast] = result.split(':').map(Number);
  return { heim, gast };
}

// Beispiel
const spiel = matches[0];
const ergebnis = parseResult(spiel.result);
// → { heim: 36, gast: 62 }
```

### 5. Team-IDs verwenden

```typescript
// ❌ FALSCH: seasonTeamId ändert sich jede Saison
const teamId = team.seasonTeamId;

// ✅ RICHTIG: teamPermanentId bleibt konstant
const teamId = team.teamPermanentId;

// Für Team-Matches immer permanente ID verwenden
const matches = await fetch(`/rest/team/id/${team.teamPermanentId}/matches`);
```

### 6. Anonyme Spieler behandeln

```typescript
function isAnonymous(player: PlayerStats): boolean {
  return player.player.person.anonym === true;
}

function getPlayerName(player: PlayerStats): string {
  if (isAnonymous(player)) {
    return `Spieler ${player.player.no}`;  // "Spieler **"
  }
  return `${player.player.person.vorname} ${player.player.person.nachname}`;
}
```

### 7. Caching mit Timestamp

```typescript
interface CachedData<T> {
  data: T;
  timestamp: string;  // Von API
  cachedAt: number;   // Date.now()
}

const cache = new Map<string, CachedData<any>>();

async function fetchWithCache<T>(url: string, maxAge: number = 5 * 60 * 1000): Promise<T> {
  const cached = cache.get(url);
  
  if (cached && Date.now() - cached.cachedAt < maxAge) {
    return cached.data;
  }
  
  const response = await fetch(url);
  const json = await response.json();
  
  cache.set(url, {
    data: json.data,
    timestamp: json.timestamp,
    cachedAt: Date.now()
  });
  
  return json.data;
}
```

---

## ⚠️ Wichtige Hinweise & Fallstricke

### 🔴 KRITISCH: null vs false bei Booleans

**Problem:** Boolean-Felder können `null`, `true` oder `false` sein!

```typescript
// ❌ FALSCH - null wird als falsy interpretiert!
if (!match.abgesagt) {
  // Wird ausgeführt bei null UND false!
}

if (team.verzicht) {
  // Wird NICHT ausgeführt bei null!
}

// ✅ RICHTIG - Explizit auf false prüfen
if (match.abgesagt === false) {
  // Nur bei explizitem false
}

if (match.abgesagt !== true) {
  // Bei null ODER false
}

// ✅ RICHTIG - null-safe mit ??
const istAbgesagt = match.abgesagt ?? false;
const hatVerzichtet = team.verzicht ?? false;
```

**Betroffene Felder:**
- `verzicht` (in Team, Match)
- `abgesagt` (in Match)
- `ergebnisbestaetigt` (in Match)
- `tableExists` (in Liga)
- `crossTableExists` (in Liga)
- `hasPlayByPlay` (in Match)
- `anonym` (in Person, Player)

**Best Practice:**
```typescript
interface Match {
  abgesagt: boolean | null;
  verzicht: boolean | null;
  ergebnisbestaetigt: boolean | null;
}

// Type Guard
function isTruthy(value: boolean | null): value is true {
  return value === true;
}

function isFalsy(value: boolean | null): value is false {
  return value === false;
}

// Verwendung
if (isTruthy(match.abgesagt)) {
  // Spiel definitiv abgesagt
}

if (!isTruthy(match.abgesagt)) {
  // Spiel nicht abgesagt (null oder false)
}
```

### 1. String-Formate

- **Ergebnisse:** String `"36:62"` (nicht Objekt)
- **Datum/Zeit:** Separate Strings `kickoffDate` + `kickoffTime`
- **Keine ISO-Timestamps** bei Spielen

### 2. Anonymisierung (U8-U14)

- `person.anonym === true`
- `vorname: "***"`, `nachname: "****"`
- `playerId: 0`, `person.id: 0`
- `no: "**"`

### 3. Team-IDs

- `teamPermanentId`: Konstant ✅
- `seasonTeamId`: Ändert sich ❌
- `teamCompetitionId`: Ändert sich ❌

→ Immer `teamPermanentId` verwenden!

### 4. Statistiken oft leer

Bei U8-U14:
- `attempted: 0.0` (Versuche nicht erfasst)
- `quota: null` (Quote nicht berechnet)
- `ro`, `rd`, `as`, `st`, `to`, `bs`: Meist `0.0`

### 5. Live-Daten

- `/rest/competition/actual/id/{ligaId}` nur während Spielen
- Update-Frequenz: ~30-60 Sekunden (geschätzt)
- Nicht alle Ligen haben Live-Ticker

---

## Support & Ressourcen

- **Bruno Collection:** `/basketball-bund-net/` - Alle Requests zum Testen
- **OpenAPI Spec:** `/dbb-api-spec-v3.yaml`
- **Response-Beispiele:** `/Responses BBB-API/` - Reale API-Antworten
- **PWA Integration:** `/basketball-app/docs/development/DBB-API-EVALUATION.md`

---

**Letzte Aktualisierung:** 29. Oktober 2025  
**Version:** 2.0 (Korrigiert mit echten Responses)  
**Status:** ✅ Production Ready
