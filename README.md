# Property Hunter

Een eenvoudige tool om woningen bij te houden tijdens je huizenjacht in België.

## Functies

- 🏠 Woningen toevoegen met link naar immoweb/zimmo/biddit
- 🗺️ Kaartweergave met alle woningen
- ⭐ Scores toekennen per categorie
- ✅ Bezoekstatus en -geschiedenis bijhouden
- 📝 Plus- en minpunten noteren
- 🔍 Zoeken en filteren
- 📤 Data exporteren/importeren (JSON)

## Gebruik

1. Open `index.html` in een moderne browser
2. Of bezoek de gehoste versie op GitHub Pages

### Lokaal draaien

Gewoon `index.html` openen in je browser. Geen server nodig.

Of met een lokale server:
```bash
python -m http.server 8000
# Open http://localhost:8000
```

### GitHub Pages

De app is gehost op: `https://[username].github.io/property-hunter/`

## Data

Alle data wordt opgeslagen in je browser (LocalStorage). Gebruik de export/import functie om data te back-uppen of te delen.

## Technologie

- Vanilla HTML/CSS/JavaScript
- Leaflet.js + OpenStreetMap
- Geen build stap nodig
