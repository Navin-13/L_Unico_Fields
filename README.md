# 🌾 L UNICO Fields — Crop Advisory

A bilingual (English / தமிழ்), single-page crop advisory web app for small farmers in **Tamil Nadu, India**. Pick your district, crop, season, soil, growth stage and recent rainfall, and get an instant advisory on irrigation, fertilizer, pests and harvest timing. You can also check a leaf photo for pests and diseases, see market price context, and ask questions by text or voice.

**Live site:** https://l-unico-fields-6.vercel.app/

---

## Table of contents

1. [Who it is for](#who-it-is-for)
2. [Features](#features)
3. [How it works (the process)](#how-it-works-the-process)
4. [Using the website step by step](#using-the-website-step-by-step)
5. [Supported inputs](#supported-inputs)
6. [Architecture](#architecture)
7. [Online vs. offline behaviour](#online-vs-offline-behaviour)
8. [Running locally](#running-locally)
9. [Deployment](#deployment)
10. [Configuration and backend proxy](#configuration-and-backend-proxy)
11. [Project structure](#project-structure)
12. [Limitations and disclaimer](#limitations-and-disclaimer)
13. [Contributing](#contributing)

---

## Who it is for

- **Small and marginal farmers** who want quick, plot-specific guidance in their own language.
- **Extension workers and KVK staff** who need a shareable, printable advisory to hand to farmers.
- **Students and developers** looking for an example of a lightweight, dependency-free, bilingual, offline-tolerant web app.

## Features

| Feature | What it does |
|---|---|
| **Plot form** | Collects district, crop, season, soil type, growth stage and 7-day rainfall. |
| **Advisory ledger** | Generates six entries: zone suitability, irrigation, fertilizer, pest watch, sowing/harvest window and season note. Works fully offline. |
| **Market price trend** | Shows an illustrative 6-week price bar chart, plus live mandi (market) prices from AGMARKNET when available. |
| **Photo pest & disease check** | Upload or capture a leaf/stem photo and get an AI-assisted read. Falls back to a local pointer when offline. |
| **Ask the field** | Chat box for crop questions, answered by an AI model when online and by a built-in rule-based engine when not. |
| **Voice input** | Speak your question using the browser's Web Speech API (hidden if unsupported). |
| **Share & save** | Share the advisory over WhatsApp or save it as a PDF through the browser's print dialog. |
| **Bilingual UI** | One-tap switch between English and Tamil. Choice is remembered. |
| **Light / dark theme** | Follows system preference, with a manual toggle. |
| **Scroll-progress crop bar** | A field of crops grows as you scroll and turns golden at the end (respects reduced-motion settings). |

## How it works (the process)

```
┌──────────────┐    ┌────────────────────┐    ┌───────────────────┐
│ 1. Plot form │ →  │ 2. Advisory engine │ →  │ 3. Advisory ledger │
│ district,    │    │ local rules built  │    │ 6 entries + price  │
│ crop, soil,  │    │ from crop data     │    │ trend + live mandi │
│ stage, rain  │    │ (no network)       │    │ price (if online)  │
└──────────────┘    └────────────────────┘    └─────────┬─────────┘
                                                        │
              ┌─────────────────────────────────────────┼──────────────────────┐
              ▼                                         ▼                      ▼
     ┌─────────────────┐                     ┌───────────────────┐   ┌───────────────────┐
     │ 4a. Photo check │                     │ 4b. Ask the field │   │ 4c. Share / print │
     │ image → proxy → │                     │ question → proxy →│   │ WhatsApp or PDF   │
     │ AI (or fallback)│                     │ AI (or local rules)│  │                   │
     └─────────────────┘                     └───────────────────┘   └───────────────────┘
```

### 1. Input
The farmer selects a district, crop, season, soil type and growth stage, and enters rainfall for the last 7 days.

### 2. Rule-based advisory engine (runs locally)
- **Zone check:** each district is mapped to an agro-climatic zone (`delta`, `dry`, `western`, `hilly`, `coastal`). Each crop lists the zones it suits. If the district falls outside them, the ledger flags it and recommends checking with the nearest KVK (Krishi Vigyan Kendra).
- **Irrigation:** rainfall is banded as **low** (< 15 mm), **ok** (15–60 mm) or **high** (> 60 mm), and the crop's irrigation advice for that band is shown.
- **Fertilizer:** looked up by crop × soil type.
- **Pest watch:** looked up by crop × growth stage.
- **Sowing / harvest window:** crop-specific timing guidance.
- **Season note:** Kharif (Jun–Sep), Rabi (Oct–Jan) or Summer (Feb–May) context.

### 3. Market context
- An **illustrative** 6-week trend is generated locally from a base price per crop. It is not real data and is labelled as such.
- If online, the app also requests **live mandi prices** for the selected crop and district from the AGMARKNET / data.gov.in dataset through the backend proxy, and shows the top three markets.

### 4. Optional AI features (need a connection)
- **Ask the field:** the question plus the plot context is sent to a Gemini-powered proxy that replies in 2–4 short, practical sentences in the selected language. If the request fails, the local keyword-based answer engine responds instead (irrigation, fertilizer, pest, price, harvest, weather).
- **Photo check:** the image is resized on the device (max 1024 px), base64-encoded and sent to the proxy for a pest/disease read. If unavailable, a crop- and stage-specific local pointer is shown.

### 5. Share
The ledger (and any photo result) can be sent as plain text via WhatsApp, or printed / saved as a PDF using a print-optimised layout.

## Using the website step by step

1. Open the site and choose **English** or **தமிழ்** at the top.
2. Fill in **Your plot**: district, crop, season, soil type, growth stage and rainfall (mm, last 7 days).
3. Tap **Get advisory**. The **Advisory ledger** and **Market price trend** appear.
4. *(Optional)* Under **Photo-based pest & disease check**, upload a close-up photo of an affected leaf or stem (or tap **Take photo** on mobile), then tap **Analyze photo**. Use good light and avoid blur.
5. *(Optional)* Under **Ask the field**, type a question, or tap the 🎤 button and speak.
6. Tap **Share via WhatsApp** or **Save as PDF** to pass the advisory on.

## Supported inputs

**Crops (8):** Paddy, Sugarcane, Cotton, Groundnut, Maize, Ragi (finger millet), Banana, Turmeric.

**Districts (22):**

| Zone | Districts |
|---|---|
| Delta | Thanjavur, Tiruvarur, Nagapattinam, Cuddalore |
| Dry | Tiruchirappalli, Madurai, Virudhunagar, Dindigul, Vellore, Villupuram, Tiruvannamalai |
| Western | Coimbatore, Erode, Tiruppur, Salem, Namakkal |
| Hilly | The Nilgiris, Theni |
| Coastal | Kanyakumari, Ramanathapuram, Thoothukudi, Chennai |

**Seasons:** Kharif, Rabi, Summer  
**Soils:** Alluvial, Red loam, Black cotton, Sandy coastal, Laterite  
**Growth stages:** Sowing / land prep, Vegetative, Flowering, Maturity / near harvest

## Architecture

- **Frontend:** one self-contained `index.html` with inline CSS and vanilla JavaScript. There is no build step and no framework.
- **Fonts:** Source Serif 4, IBM Plex Sans and IBM Plex Mono from Google Fonts, with system fallbacks.
- **Hosting:** static hosting on Vercel.
- **Backend proxy:** a small serverless worker (Cloudflare Workers) that holds the Gemini API key and the data.gov.in key as secrets, so **no API keys are ever shipped in the page**. Routes used by the frontend:

| Route | Method | Purpose |
|---|---|---|
| `/api/advisory` | POST `{ prompt }` | Text Q&A → `{ reply }` |
| `/api/pest-check` | POST `{ prompt, image, mimeType }` | Photo analysis → `{ reply }` |
| `/api/mandi-prices` | GET `?state&district&commodity` | Live mandi price records |

- **Browser APIs used:** Web Speech API (voice), File / Canvas (photo resize), `window.print()` (PDF), `localStorage` (language preference, wrapped in try/catch).

## Online vs. offline behaviour

| Capability | Offline | Online |
|---|---|---|
| Plot form and advisory ledger | ✅ | ✅ |
| Illustrative price trend | ✅ | ✅ |
| Chat answers | Rule-based local answers | AI-generated answers |
| Photo check | Local pointer only | AI-assisted read |
| Live mandi prices | ❌ | ✅ |
| Share / PDF | ✅ | ✅ |

## Running locally

No install is needed.

```bash
git clone https://github.com/<your-username>/l-unico-fields.git
cd l-unico-fields

# Option 1: open directly
open index.html

# Option 2: serve locally (recommended, so fetch and speech features behave consistently)
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Deployment

**Vercel (current setup):**
1. Push the repo to GitHub.
2. Import the repo in Vercel and deploy. No framework or build command is needed.

It also works on GitHub Pages, Netlify or any static host.

## Configuration and backend proxy

Inside `index.html`, three constants point to the proxy:

```js
var ADVISORY_PROXY_URL = "https://<your-worker>.workers.dev/api/advisory";
var MANDI_PROXY_URL    = "https://<your-worker>.workers.dev/api/mandi-prices";
var PEST_PROXY_URL     = "https://<your-worker>.workers.dev/api/pest-check";
```

To run your own copy:
1. Deploy a Cloudflare Worker exposing the three routes above.
2. Store `GEMINI_API_KEY` and your data.gov.in API key as **worker secrets** (never in the HTML).
3. Enable CORS for your site's origin.
4. Replace the three URLs above with your worker's address.

If the proxy is unreachable, the app still works using its local advisory engine.

## Project structure

```
.
├── index.html   # the entire app (markup, styles, scripts, bilingual content)
├── README.md    # this file
├── LICENSE      # MIT
└── .gitignore
```

Inside `index.html`'s script, the main pieces are: the bilingual content dictionaries (crops, districts, soils, stages, UI strings), `buildAdvisory()` (advisory engine), `renderTrend()` / `fetchLiveMandiPrice()` (market data), `callGemini()` / `answer()` (chat), `handlePhotoFile()` / `callPestCheck()` (photo check), `buildShareText()` / `preparePrintView()` (sharing), and `applyLang()` (English/Tamil switching).

## Limitations and disclaimer

- Advisories are **general guidance** based on Tamil Nadu agro-climatic patterns, not a substitute for a local agronomist, soil test or your nearest **KVK**.
- The 6-week price chart is **illustrative only**. Live mandi data depends on availability and commodity naming in the AGMARKNET dataset.
- AI-generated chat and photo answers can be wrong. Verify before applying pesticides or fertilizer.
- Covers 22 districts and 8 crops so far.

## Contributing

Contributions are welcome, especially:
- More districts, crops and soil/stage advice
- Improved Tamil translations
- Additional languages
- Accessibility and low-bandwidth improvements

Fork the repo, create a branch, and open a pull request with a short description of the change.

## License

Released under the [MIT License](LICENSE).
