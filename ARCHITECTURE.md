# Niblink (Trigger Detective) — System Architecture

## What It Does

Niblink is a food & symptom tracking app for people with Hashimoto's thyroiditis.
Users log meals (via photo or text), symptoms, and medications. The app detects
delayed correlations between foods and symptoms — the kind that happen 24-72 hours
later and are impossible to spot manually.

## Architecture: Hybrid AI + Deterministic

The system deliberately limits AI to what it's good at (perception and language)
while keeping the critical analysis in deterministic code that can't hallucinate.

### Layer 1: AI Perception

- **Photo analysis**: Multimodal LLM identifies ingredients from meal photos
- **Text parsing**: LLM extracts structured ingredient lists from meal descriptions
- Output: Structured JSON with ingredient names, quantities, confidence scores,
  and Hashimoto-specific flags (soy, goitrogens, iodine, gluten, nightshades)

### Layer 2: Deterministic Correlation Engine

- **Not AI** — a statistical algorithm written in C#
- Analyzes 90-day rolling windows across 4 time delays (12h, 24h, 48h, 72h)
- Compares co-occurrence rates against personal baseline symptom frequency
- Applies severity weighting (symptoms worse than usual → higher score)
- Requires 5+ co-occurrences to prevent false positives
- Outputs correlation scores (-1.0 to +1.0) with confidence ratings

### Layer 3: AI Translation

- LLM converts raw correlation data into natural language insights
- Example: "78% of your bloating episodes occur 18-24h after eating dairy"

### Layer 4: Grounded Nutrition Data

- **BLS (Bundeslebensmittelschluessel)**: ~15,000 German food items
- **DGE reference values**: B12, Iron, Zinc, Selenium, Iodine, Protein, Vitamin D, Omega-3
- 5-stage fuzzy matching: exact → synonym (228 mappings) → substring → German
  compound word splitting → Levenshtein distance
- Dual-language search (German + English simultaneously)

### Safety Systems

- Soy-medication timing warnings (soy blocks thyroid med absorption within 4h)
- Iodine traffic light (red: algae/seaweed, yellow: processed foods, green: safe)
- Goitrogen exposure alerts (raw cruciferous vegetables)
- Medical disclaimer enforcement in all AI outputs

## Tech Stack

- **Backend**: .NET 8, ASP.NET Core, EF Core, SQLite, Clean Architecture
- **Frontend**: Vue 3, TypeScript, Pinia, Tailwind CSS, Vite, PWA
- **AI**: LLM API behind an `IAIProvider` interface (provider-swappable)
- **i18n**: Full German/English support (UI + AI prompts)
- **Testing**: xUnit (unit + integration)

## Design Principles

- AI for perception and language; deterministic code for analysis
- Privacy-first: all processing server-side, no third-party data sharing
- Provider-agnostic AI: clean interface allows swapping LLM providers
- European focus: German food databases, EU nutrition standards
