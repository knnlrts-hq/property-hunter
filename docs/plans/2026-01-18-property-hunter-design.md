# Property Hunter - Design Document

**Datum:** 18 januari 2026
**Status:** Goedgekeurd
**Taal:** Nederlands (Vlaams)

## Overzicht

Property Hunter is een eenvoudige tool voor het bijhouden van woningen tijdens een huizenjacht in België. Gebruikers kunnen woningen bookmarken van immoweb.be, zimmo.be en biddit.be, bezoeken plannen en bijhouden, plus- en minpunten noteren, en scores toekennen om woningen te vergelijken.

## Doelstellingen

- Woningen bookmarken door links te plakken van Belgische vastgoedsites
- Bezoekstatus en -geschiedenis bijhouden
- Plus- en minpunten noteren per woning
- Scores toekennen op vaste categorieën voor eenvoudige vergelijking
- Kaartweergave met alle gemarkeerde woningen
- Woningen filteren en doorzoeken
- Data exporteren/importeren voor synchronisatie tussen apparaten
- Mobile-first: primair gebruik op smartphones

## Technische Stack

- **Single HTML file** met inline CSS en JavaScript
- **Leaflet.js** + OpenStreetMap voor kaartfunctionaliteit (CDN)
- **LocalStorage** voor data-opslag
- **Geen build stap** - direct openen in browser of hosten op GitHub Pages
- **Geen framework** (geen React, Vue, etc.)

### CDN Dependencies

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

## Data Model

```javascript
// LocalStorage key: "propertyHunter"
{
  version: 1,
  customCategories: [],  // user-added scoring categories
  properties: [
    {
      id: "uuid-string",
      url: "https://www.immoweb.be/...",
      address: "Kerkstraat 42, Leuven",
      price: 325000,
      coordinates: { lat: 50.9859, lng: 4.8310 },

      // Bezoekstatus
      status: "nieuw" | "bezoek_gepland" | "bezocht" | "ja" | "nee" | "misschien",

      // Woningstatus
      propertyStatus: "beschikbaar" | "onder_optie" | "verkocht",

      // Scores (1-5, 0 = niet beoordeeld)
      scores: {
        locatie: 0,
        energie: 0,
        staat: 0,
        prijs: 0,
        grootte: 0,
        buitenruimte: 0
        // ...custom categories added here
      },

      // Notities
      pros: "",  // vrije tekst
      cons: "",  // vrije tekst

      // Bezoekgeschiedenis
      visits: [
        { date: "2026-01-15", notes: "Eerste bezichtiging met makelaar" }
      ],

      dateAdded: "2026-01-10T14:30:00Z"
    }
  ]
}
```

### Status Waarden

**Bezoekstatus (`status`):**
| Waarde | Label | Beschrijving |
|--------|-------|--------------|
| `nieuw` | Nieuw | Pas toegevoegd, nog geen actie |
| `bezoek_gepland` | Bezoek gepland | Afspraak is gemaakt |
| `bezocht` | Bezocht | Minstens één bezoek gedaan |
| `ja` | Ja | Finalist - serieuze kandidaat |
| `nee` | Nee | Afgewezen |
| `misschien` | Misschien | Twijfelgeval |

**Woningstatus (`propertyStatus`):**
| Waarde | Label | Beschrijving |
|--------|-------|--------------|
| `beschikbaar` | Beschikbaar | Te koop |
| `onder_optie` | Onder optie | Iemand heeft optie genomen |
| `verkocht` | Verkocht | Niet meer beschikbaar |

### Score Categorieën

**Standaard categorieën:**
1. **Locatie** - buurt, woon-werk, voorzieningen
2. **Energie** - EPC-score, isolatie
3. **Staat** - renovatie nodig vs instapklaar
4. **Prijs** - prijs-kwaliteit verhouding
5. **Grootte** - kamers, indeling, ruimte
6. **Buitenruimte** - tuin, terras, parking

**Custom categorieën:** Gebruikers kunnen extra categorieën toevoegen (bijv. "Garage", "Kelder").

**Totaalscore:** Gemiddelde van alle beoordeelde categorieën (negeert 0-scores).

## UI Layout

### Mobile Layout (primair)

```
┌─────────────────────────┐
│ Property Hunter    [≡]  │  ← hamburger menu
├─────────────────────────┤
│                         │
│      KAART (40vh)       │
│   (Leaflet/OSM)         │
│                         │
├─────────────────────────┤
│ 🔍 Zoeken...            │
│ [Filters ▼]             │
├─────────────────────────┤
│ ┌─────────────────────┐ │
│ │ Woning Card 1       │ │
│ └─────────────────────┘ │
│ ┌─────────────────────┐ │
│ │ Woning Card 2       │ │
│ └─────────────────────┘ │
│ ... scrollbaar ...      │
├─────────────────────────┤
│    [+ Toevoegen]        │  ← sticky bottom
└─────────────────────────┘
```

### Desktop Layout (> 768px)

```
┌────────────────────────────────────────────────────────┐
│ Property Hunter                    [Exporteren] [≡]    │
├──────────────────────────┬─────────────────────────────┤
│                          │ 🔍 Zoeken...                │
│                          │ [Filters ▼]                 │
│        KAART             ├─────────────────────────────┤
│     (50% breedte)        │ Woning Card 1               │
│                          │ Woning Card 2               │
│                          │ Woning Card 3               │
│                          │ ... scrollbaar ...          │
├──────────────────────────┴─────────────────────────────┤
│                    [+ Toevoegen]                       │
└────────────────────────────────────────────────────────┘
```

## Componenten

### 1. Header

- Titel "Property Hunter"
- Hamburger menu (≡) met:
  - Exporteren
  - Importeren
- Responsive: op desktop zijn export/import knoppen direct zichtbaar

### 2. Kaart

- Leaflet.js met OpenStreetMap tiles
- Initiële weergave: gecentreerd op Aarschot (50.9859° N, 4.8310° E)
- Zoom level: Vlaams-Brabant en Antwerpen zichtbaar
- Auto-fit naar alle markers als er woningen zijn

**Markers:**
| Status | Kleur | Icoon |
|--------|-------|-------|
| Nieuw | Blauw | - |
| Bezoek gepland | Lila/Paars | - |
| Bezocht | Groen | - |
| Ja | Groen | ★ |
| Nee | Rood | ✕ |
| Misschien | Oranje | ? |

**Woningstatus overlay:**
- "Onder optie" → diagonale strepen op marker
- "Verkocht" → marker is vervaagd/semi-transparant

**Interacties:**
- Tik op marker → popup met adres + score
- Tik op popup → opent detailweergave
- Bij actief detail → marker pulseert
- Filters verbergen markers van uitgefilterde woningen

### 3. Woning Card

```
┌─────────────────────────────┐
│ Kerkstraat 42, Leuven       │
│ €325.000                    │
│ [Bezocht] [Beschikbaar]     │  ← status badges
│ ★★★★☆ 3.5                  │  ← totaalscore
└─────────────────────────────┘
```

- Tap anywhere → opent detailweergave
- Grote touch targets (min 44px)

### 4. Filter Bar

```
┌─────────────────────────────┐
│ 🔍 Zoeken...                │
├─────────────────────────────┤
│ [Filters ▼]                 │  ← collapsed by default
├─────────────────────────────┤
│ Status:     [Alle ▼]        │
│ Woning:     [Alle ▼]        │
│ Sorteren:   [Score ▼]       │
│ [Filters wissen]            │
└─────────────────────────────┘
```

**Zoeken:** Case-insensitive, doorzoekt adres, notities, plus/minpunten.

**Filter opties:**
- Status: Alle / Nieuw / Bezoek gepland / Bezocht / Ja / Nee / Misschien
- Woning: Alle / Beschikbaar / Onder optie / Verkocht
- Sorteren: Score (hoog→laag) / Prijs (laag→hoog) / Datum toegevoegd (nieuw→oud)

**Gedrag:**
- Filters passen direct toe (geen "toepassen" knop)
- Actieve filters tonen count: "Filters (2)"
- Kaart en lijst filteren synchroon

### 5. Detail Weergave (Modal)

Full-screen modal bij tap op woning card of marker.

```
┌─────────────────────────────┐
│ ← Terug              [...]  │  ← overflow: verwijderen
├─────────────────────────────┤
│ Kerkstraat 42, Leuven       │
│ €325.000                    │
│ 🔗 Bekijk op Immoweb        │  ← tappable link
├─────────────────────────────┤
│ Status:  [Bezocht ▼]        │
│ Woning:  [Beschikbaar ▼]    │
├─────────────────────────────┤
│ SCORES                      │
│ Locatie:      ★★★★☆    (4) │
│ Energie:      ★★★☆☆    (3) │
│ Staat:        ★★★★★    (5) │
│ Prijs:        ★★★☆☆    (3) │
│ Grootte:      ★★★★☆    (4) │
│ Buitenruimte: ★★☆☆☆    (2) │
│ [+ Categorie toevoegen]     │
│ ─────────────────────────── │
│ Gemiddeld:         3.5 / 5  │
├─────────────────────────────┤
│ PLUSPUNTEN                  │
│ ┌─────────────────────────┐ │
│ │ Mooie tuin, rustige     │ │
│ │ buurt, dichtbij school  │ │
│ └─────────────────────────┘ │
├─────────────────────────────┤
│ MINPUNTEN                   │
│ ┌─────────────────────────┐ │
│ │ Verouderde keuken,      │ │
│ │ EPC label D             │ │
│ └─────────────────────────┘ │
├─────────────────────────────┤
│ BEZOEKEN                    │
│ 15 jan 2026 - Eerste bez... │
│ 22 jan 2026 - Tweede bez... │
│ [+ Bezoek toevoegen]        │
└─────────────────────────────┘
```

**Gedrag:**
- Alle velden inline bewerken
- Auto-save naar LocalStorage bij elke wijziging
- Scores zijn tappable sterren
- Terug knop sluit modal

### 6. Woning Toevoegen (Modal)

**Stap 1: Basis info**

```
┌─────────────────────────────┐
│ Nieuwe woning toevoegen     │
├─────────────────────────────┤
│ Link (immoweb/zimmo/biddit) │
│ ┌─────────────────────────┐ │
│ │ https://...             │ │
│ └─────────────────────────┘ │
│                             │
│ Adres *                     │
│ ┌─────────────────────────┐ │
│ │                         │ │
│ └─────────────────────────┘ │
│                             │
│ Prijs (€)                   │
│ ┌─────────────────────────┐ │
│ │                         │ │
│ └─────────────────────────┘ │
│                             │
│         [Volgende →]        │
└─────────────────────────────┘
```

**Stap 2: Locatie op kaart**

```
┌─────────────────────────────┐
│ ← Terug                     │
├─────────────────────────────┤
│ Tik op de kaart om de       │
│ locatie aan te duiden       │
├─────────────────────────────┤
│                             │
│           KAART             │
│    (gecentreerd België)     │
│        📍 tik om te plaatsen│
│                             │
├─────────────────────────────┤
│         [Opslaan ✓]         │  ← actief na marker plaatsing
└─────────────────────────────┘
```

**Na opslaan:**
- Woning aangemaakt met status "Nieuw" en "Beschikbaar"
- Alle scores starten op 0
- Terug naar hoofdweergave, nieuwe woning gehighlight

### 7. Export/Import

**Exporteren:**

```
┌─────────────────────────────┐
│ Exporteren                  │
├─────────────────────────────┤
│ Je data wordt opgeslagen    │
│ als JSON bestand.           │
│                             │
│ [Delen...]                  │  ← Web Share API
│                             │
│ [Download bestand]          │  ← fallback
└─────────────────────────────┘
```

- Bestandsnaam: `property-hunter-YYYY-MM-DD.json`
- Web Share API op mobile (WhatsApp, Mail, etc.)
- Download fallback op desktop

**Importeren:**

```
┌─────────────────────────────┐
│ Importeren                  │
├─────────────────────────────┤
│ ⚠️ Let op: importeren       │
│ overschrijft al je huidige  │
│ data!                       │
│                             │
│ [Bestand kiezen...]         │
│                             │
│ Of plak JSON:               │
│ ┌─────────────────────────┐ │
│ │                         │ │
│ └─────────────────────────┘ │
│ [Importeren]                │
└─────────────────────────────┘
```

- Bestand kiezen werkt met iOS Files, Android, cloud drives
- Plak optie voor snelle sync via chat
- Waarschuwing voor overschrijven

## Responsive Design

**Breakpoints:**
- Mobile: < 768px (stack layout)
- Desktop: ≥ 768px (split view)

**Mobile-first principes:**
- Touch targets minimaal 44px
- Basis font-size 16px (voorkomt zoom op iOS)
- Geen hover-only interacties
- Scroll-vriendelijke layouts

## Bestandsstructuur

```
/property-hunter/
├── index.html          # De volledige applicatie
├── docs/
│   └── plans/
│       ├── 2026-01-18-property-hunter-design.md
│       └── 2026-01-18-property-hunter-implementation.md
└── README.md           # Optioneel: korte uitleg
```

## GitHub Pages Hosting

1. Repository settings → Pages
2. Source: Deploy from branch
3. Branch: main, folder: / (root)
4. Save → site beschikbaar op `https://username.github.io/property-hunter/`

Geen build stap nodig - `index.html` wordt direct geserveerd.

## Browser Support

- Chrome (laatste 2 versies)
- Safari (laatste 2 versies)
- Firefox (laatste 2 versies)
- iOS Safari (primair doelplatform)
- Android Chrome (primair doelplatform)

## Geschatte Omvang

- ~400-600 regels JavaScript
- ~200-300 regels CSS
- ~100-150 regels HTML
- Totaal: ~700-1050 regels in één bestand

## Toekomstige Uitbreidingen (Niet in Scope)

Deze features zijn expliciet **niet** in de huidige versie:
- Automatisch scrapen van woninggegevens
- Cloud sync / multi-device real-time sync
- Foto uploads
- Notificaties
- Meerdere gebruikers / accounts
- Vergelijkingsweergave naast elkaar
