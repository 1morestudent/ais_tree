# AI Safety Resources Directory

## Wrap-up Protocol

When the user says "wrap up": check if any changes need to be documented (update this file or README if schema/behavior changed), then commit all changes and push to `main` (which auto-deploys).

## Project Overview

A filterable directory website helping newcomers discover AI safety career pathways - courses, fellowships, advising programs, and other opportunities. Target audience: people in the EA community exploring AI safety careers.

**Live site:** https://fellowship-navigator.nielsdoehring.workers.dev

**Live data source:** Google Sheets CSV exported at:
```
https://docs.google.com/spreadsheets/d/1wrgVNPTMlwagr33sTPv-Ukadx14MBKXauUY1BEBnbBo/export?format=csv&gid=0
```

**Editable sheet:** [Google Sheets link - add if needed]

## Architecture

Single-file React application with no build step:
- React 18 via CDN (unpkg)
- Babel standalone for JSX transformation
- Tailwind CSS via CDN
- Data fetched from Google Sheets on page load

This architecture was chosen for:
- Simple deployment (single file upload or git push)
- No npm/node dependencies
- Community can update data via Google Sheets without code changes

## File Structure

```
ais-tree/
├── public/
│   ├── index.html      # Everything - HTML, CSS, React app
│   └── favicon-64.png
├── wrangler.jsonc      # Cloudflare Workers config
└── CLAUDE.md           # This file
```

## Running Locally

```bash
cd ais-tree/public
python3 -m http.server 8000
# Open http://localhost:8000
```

The local server is required because the Google Sheets fetch won't work with `file://` protocol (CORS).

## Data Schema

The CSV has these columns:

| Column | Type | Notes |
|--------|------|-------|
| ID | string | Unique row key, e.g. "aisdb_001". Variants of the same program share a numeric base and differ only by a trailing letter: "aisdb_001a" / "aisdb_001b". The app auto-groups these and shows each variant's sibling(s) with a diff note in the expanded card view. |
| name | string | Program name |
| organization | string | Org running it |
| url | string | Link, or "[unclear]" if unknown |
| description | string | Program summary |
| for_student | boolean | 1 if suitable for students |
| for_early_career | boolean | 1 if suitable for early career (0-3 yrs) |
| for_mid_career | boolean | 1 if suitable for mid-career (3-10 yrs) |
| for_senior | boolean | 1 if suitable for senior (10+ yrs) |
| for_career_switch | boolean | 1 if suitable for career switchers |
| track | string | "Technical", "Governance", "Neutral" (covers both/neither), or "General" |
| program_type | string | "Course", "Fellowship", "Advising", "Mentorship", "Bootcamp" |
| format | string | "Online", "In-person", "Hybrid" |
| pacing | string | "Cohort-based", "Rolling", "Self-paced" |
| geographic_focus | string | "Global", "US-centric", "UK/EU-centric" |
| duration | number | Duration in weeks (integer). App normalizes to: ≤4w→"1-4 weeks", 5-13w→"1-3 months", 14-26w→"3-6 months", >26w→"6-12 months". "[unclear]" passed through. |
| time_commitment | number | Hours per week (integer). App normalizes to: <5→"<5 hrs/week", 5-10→"5-10 hrs/week", 11-20→"10-20 hrs/week", >20→"Full-time". "[unclear]" passed through. |
| cost | number | 0 for free |
| recompensation | string | Monthly stipend in USD, "0", or "[unclear]" |
| prerequisites | string | Requirements or "None" |
| application_status | string | "open", "closed", "rolling", "[unclear]" (lowercase) |
| notes_for_claude_during_update | string | Internal field, not displayed |
| next_deadline | string | Date or "[unclear]" |
| next_cohort_start | string | Date or "[unclear]" |
| tags | string | Comma-separated: "Mentorship", "Stipend/funding", "Research output", etc. |
| last_verified | date | When entry was last checked |
| notes | string | Additional context |

## Filter Logic

### Multi-select filters (OR within, AND across)
- **track**: Selecting "Technical" or "Governance" also shows items with track="Both" or track="Neutral"
- **career_stage**: Backed by boolean columns `for_student`, `for_early_career`, `for_mid_career`, `for_senior`. Item shows if ANY selected stage maps to a `1` in the corresponding column.
- **format**: Strict match
- **geographic_focus**: "Global" items always show regardless of selection

### Upper-bound filters
- **duration**: Ordered list, selecting "≤ 1-3 months" shows "1-4 weeks" AND "1-3 months"
- **time_commitment**: Same logic, ordered "<5 hrs/week" → "Full-time"

### Toggle filter
- **career_switch**: When on, shows items containing "switchers" in career_switch field

### Ordering constants
```javascript
const DURATION_ORDER = ['1-4 weeks', '1-3 months', '3-6 months', '6-12 months', 'Ongoing/variable'];
const TIME_ORDER = ['<5 hrs/week', '5-10 hrs/week', '10-20 hrs/week', 'Full-time'];
```

## Component Structure

```
App
├── FilterSection (reusable) - multi-select pill buttons
├── UpperBoundFilter (reusable) - single-select "≤ X" buttons  
└── ResourceCard - expandable card showing program details
```

## Styling Conventions

- Tailwind utility classes throughout
- Color coding for tracks:
  - Technical: purple (bg-purple-100, text-purple-700)
  - Governance: amber (bg-amber-100, text-amber-700)
  - Both: blue (bg-blue-100, text-blue-700)
  - General: gray
- Selected filters: blue for multi-select, green for upper-bound/toggles
- Cards: white with subtle shadow and border

## Common Tasks

### Add a new filter
1. Add state: `const [newFilter, setNewFilter] = useState([]);`
2. Extract options from data: `const options = [...new Set(data.map(d => d.field_name))];`
3. Add FilterSection or UpperBoundFilter component in the filters area
4. Add filtering logic in the `filtered` computation
5. Add to "Clear all filters" reset

### Change what shows on cards
Edit `ResourceCard` component. Currently shows:
- Collapsed: name, track badge, organization, format/duration/time
- Expanded: description, career_stage, geographic_focus, prerequisites, application_status, compensation, notes

### Add a new data field
1. Update Google Sheet with new column
2. Reference it in the app as `item.new_field_name`
3. CSV parser handles it automatically

### Change filter behavior
Look for the `filtered = data.filter(item => {...})` block. Each filter has a clearly labeled section.

## Deployment

Deployed via Cloudflare Workers Static Assets. Push to `main` triggers a deploy automatically. No build step required.

## Known Limitations

- Babel transforms JSX in-browser (slight performance cost on load)
- No client-side caching of CSV data (fetches fresh each page load)
- Google Sheets has ~5 minute cache on published CSV

## Future Improvements (not yet implemented)

- Filter by application_status (hide closed programs)
- Sort results (by deadline, alphabetically, etc.)
- Search/text filter
- URL params for shareable filtered views
- Embedded fallback data if fetch fails
