# ABC_V2 - Buscador de Novedades

## Project Overview
Single-page application for incident reporting and management in a commercial center. Users navigate through zones (MARCA, ZONA COMUN, ESTACIONAMIENTO, PERIMETRO) to report and view step-by-step protocols for security, maintenance, and operational incidents.

## Architecture
- **Single HTML file** (`Index.html`) containing all logic - no build system or external dependencies beyond CDN-loaded Tailwind CSS
- **Pure vanilla JavaScript** - no frameworks, state managed in-memory
- **Data-driven UI** - All incident types, zones, and protocols stored in embedded JSON array (`data` variable)
- **Modal-based navigation** - Progressive disclosure from zones → incident types → sub-incidents → detailed steps

## Key Components

### Data Structure
The `data` array (line ~191+) contains ~100+ incident records with this schema:
```javascript
{
  "ZONA ": "MARCA" | "ZONA COMUN" | "ESTACIONAMIENTO" | "PERIMETRO",
  "DIRECCION": "SEGURIDAD" | "MANTENIMIENTO" | "IIT",
  "SUB AREA ": "SEGURIDAD" | "ELECTRICO" | "HIDRAULICO" | "CIVIL" | "ELECTROMECANICO",
  "TIPO DE NOVEDAD buscador": "HURTO" | "INCENDIO" | "FALLA EN LA RED ELECTRICA EN EL LOCAL" | ...,
  "SUB NOVEDAD ": "MECHERO" | "ENGAÑO" | "AUSENCIA TOTAL DE ENERGIA" | ...
}
```
**Note:** All keys have trailing spaces - this is intentional and critical for data access.

### Step Generation Logic
`generateSteps(zona, tipoNovedad, subNovedad)` function (line ~437+) provides hardcoded action protocols:
- Returns 5 generic steps as fallback when no specific match exists
- Specific protocols defined for key scenarios (e.g., ESTACIONAMIENTO → HURTO → ROBO DE AUTOPARTES)
- Yellow-highlighted steps indicate generic fallback protocols

### UI Flow
1. **Zone Selection** → `renderZoneButtons()` - Displays unique zones (excluding "IIT")
2. **Search View** → `showSearchResults(zone)` - Live filtering by incident type
3. **Sub-incident Selection** → `showSubNovedas(novedad)` - Lists all sub-categories
4. **Protocol Modal** → `showDetails(item)` - Animated step-by-step display

## Development Patterns

### Styling Approach
- **Tailwind CSS via CDN** for utility classes
- **Custom CSS in `<style>` block** for animations, gradients, and hover effects
- Background image (`logo-flor.png`) with fixed attachment for depth
- Card-based design with hover transforms (`translateY(-6px)`)

### String Handling
**Critical:** Data keys have trailing spaces. Always use:
```javascript
item["ZONA "]  // NOT item["ZONA"]
item["TIPO DE NOVEDAD buscador"]
```

### Filtering & Search
- Case-insensitive search: `.toLowerCase().includes(query)`
- Deduplication: `[...new Set(array.map(...))]`
- Zone-specific filtering applied before incident type search

## Adding New Incidents

### 1. Add Data Entry
```javascript
{
  "ZONA ": "MARCA",
  "DIRECCION": "SEGURIDAD",
  "SUB AREA ": "SEGURIDAD",
  "TIPO DE NOVEDAD buscador": "NEW_INCIDENT_TYPE",
  "SUB NOVEDAD ": "SPECIFIC_SUBTYPE"
}
```

### 2. Add Protocol Steps (Optional)
Edit `generateSteps()` function to add specific procedures:
```javascript
"ZONA_NAME": {
  "INCIDENT_TYPE": {
    "SUB_INCIDENT": [
      "Step 1 description",
      "Step 2 description",
      // ...
    ]
  }
}
```

### 3. Test Workflow
Open `Index.html` in browser → Select zone → Search for new incident → Verify steps display correctly

## Local Development
- **Environment:** XAMPP server at `c:\xampp\htdocs\ABC_V2`
- **No build step required** - edit `Index.html` and refresh browser
- **Testing:** Open `http://localhost/ABC_V2/Index.html`

## Common Tasks

**Update incident steps:** Modify `steps` object in `generateSteps()` function

**Change zone names:** Update `data` array entries and adjust zone filtering logic in `renderZoneButtons()`

**Modify UI animations:** Edit `@keyframes` rules in `<style>` block (fadeIn, scaleUp)

**Add new zone:** Insert entries in `data` array with new `"ZONA "` value - UI auto-generates buttons

## Known Patterns
- Modal close on overlay click not implemented - only via X button
- Search requires exact match on incident type, not fuzzy search
- IIT zone entries exist in data but hidden from UI (`toUpperCase() !== "IIT"` filter)
- Animation delays staggered at 0.07s intervals for step reveals
