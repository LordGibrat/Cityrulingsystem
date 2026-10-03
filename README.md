# Mayor for 5 Hours · Astana Lab

A Python and Streamlit web application for the HackAlem AI case: distribute **100 conditional units**, assess the consequences for five conditional districts, and receive an expert OpenAI report.

## Game Situation Center

Interactive Astana map on the main screen, HUD with treasury and QoL, five district cards, five decision slots, and a store with 14 decrees. One game coin equals one conditional unit of the case; internal amounts and JSON retain the previous format in tenge.

The map and district cards select the same district for new decrees. The three weakest indicators and warnings are calculated from the displayed “Before / After” state; all ten indicators are hidden under “📊 District Details”. Card colors use the same function and selected layer as the map. Map labels show the number of planned decrees, including city-wide ones in all districts.

The store shows the actual cost, type, full effect, and lag; details contain the unchanged ID and the contribution adjusted for lag. The district field is visible before applying a decree. Unavailable actions are accompanied by the exact reason from `validate_decisions`. The “Run Simulation” button re-checks the plan and uses `simulate_decisions`. A live forecast is available with five valid decisions; the toast and confetti appear only on the first explicit launch of this scenario during the game session, not during automatic updates. Synergies are displayed only when present in the calculator result.

**The calculation model has not been changed.** In this project, the final Score function is called `score_indicators` (the external name `calculate_score` is not used). The dataset, prices, lags, weights, conflicts, synergies, penalty, and rules remain in the original `src/data.py`, `src/model.py`, `src/planner.py`.

## Quick Start

The simulator is isolated in the `astana_simulator/` folder. Its dependencies, Streamlit settings, data, and tests do not require changes to files from another project in the root of the command repository.

First, move from the repository root into the simulator folder:

```powershell
cd astana_simulator
```

All following commands are executed inside this folder. This also ensures that its own theme from `.streamlit/config.toml` and its separate `.env` file are loaded.

Tested on **Python 3.12.14**. Dependencies are pinned in `requirements.txt`; the full environment snapshot is in `requirements-lock.txt`.

### In the prepared Windows working directory

The local `.venv` environment has already been created. Run from the project root:

```powershell
.\.venv\Scripts\python.exe -m streamlit run streamlit_app.py
```

Open <http://127.0.0.1:8501>. No keys are required for calculations and chart viewing.

You can also run `python -m streamlit run app.py` from the `astana_simulator/` folder: this entry point launches the same interface. The repository root `app.py` belongs to another project and is not modified.

### Fresh installation

Use standard CPython 3.12 from python.org, not Python from MSYS2.

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
Copy-Item .env.example .env  # only if .env does not already exist
.\.venv\Scripts\python.exe -m streamlit run streamlit_app.py
```

Linux / macOS:

```bash
python3.12 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
cp -n .env.example .env
.venv/bin/python -m streamlit run streamlit_app.py
```

For a complete reproduction of the tested environment, you can install `requirements-lock.txt` instead of `requirements.txt`. This snapshot includes test dependencies and was created on Windows / Python 3.12.

## OpenAI Integration

In the local `.env` file, set:

```dotenv
OPENAI_API_KEY=your_OpenAI_key
```

OpenAI key: <https://platform.openai.com/api-keys>. Model access, quotas, and billing are determined by your account. Do not send the key in chat or add `.env` to Git — the file is already excluded through `.gitignore`.

The only AI service is OpenAI, model **`gpt-4o-mini`**, endpoint `https://api.openai.com/v1`. The file for this project is loaded using `load_dotenv(ROOT / ".env", override=False, interpolate=False)`, then the key is read through **`os.getenv("OPENAI_API_KEY")`**. The key remains only on the server and is not output to the browser, exports, or error messages.

After changing `.env`, restart the Streamlit process, then click “Get AI Analysis” below the results. The environment variable takes priority over `.env`. The “key set” status means that a value is present; it does not verify access.

- One `AsyncOpenAI` client and one report are created when the button is pressed.
- OpenAI writes strengths, hidden risks, adjustments, and a verdict.
- The model receives only synthetic source data, the budget, results, deltas, and measure contributions. The key is not included in the request content or export.
- An HTTP attempt has a 25-second timeout, with one SDK retry allowed. The total limit for each report is 55 seconds.
- Missing keys, 401/403, 404, 429, 5xx, connection problems, timeouts, empty responses, and token-limit cutoffs are handled. Errors do not expose raw exception text or secrets.
- Reports are stored only in the current Streamlit session. A successful report is not requested again when sliders are moved or the page is refreshed within the session. Changing the scenario hides the outdated report. The retry button retries only failed requests.
- Without keys, real calculations, charts, and exports work; AI text is not replaced with a prepared report.

Reloading the tab and losing the session removes the AI response cache; the next explicit request creates a new report.

## Initial City Overview

The HUD and map immediately show a base Score of **52.56**, **100 coins**, and five districts. District cards are located below the map; “Manage” selects a district for the store, while the link next to the map takes you to the decrees. “New Scenario” clears plans, AI reports, and the history of game notifications.

The detailed initial overview with recommendations is saved under “📈 Results and Methodology” → “Initial State”. The data is educational and synthetic. The critical indicator threshold is strictly below 40; the diagnostic threshold of 50 in the initial overview does not change the Score.

## Two Modes

The five-slider request differs from the rules of the attached catalogue: the catalogue allows two measures in the same direction and no measures in the other direction. Therefore, the modes are presented separately and explicitly labeled.

### Budget Allocation — Additional Mode

Five sliders: transport, landscaping, social infrastructure, safety, and city services. Values range from 0 to 100 units in increments of 1 unit. All five decisions are present; zero funding is allowed. The starting scenario is 20 units per direction. There are three ready-made allocations.

If the total exceeds 100 units, Score, AI analysis, and export are unavailable. Validation exists both in the UI and in the calculation function. To preserve the calculation model, internal values and JSON use tenge: 1 unit = 10 million ₸, full budget = 1 billion ₸. The interface and AI report use conditional units.

For direction `s` and each of its indicators in each district:

```text
W_s = sum of the weights of the two indicators in the direction
f_s = 0.35 × (1 − exp(−b_s / (B × W_s)))
I'_dk = I_dk + (100 − I_dk) × f_s
```

`b_s` is the allocated budget, `B = 1 000 000 000`. The same program closes the same share of the deficit in all districts. With a lower initial value, the absolute increase is larger. As funding increases, marginal returns decrease; zero funding does not change indicators. Remaining funds provide no bonus.

**This is an original continuous educational model**, not the effect formula from the catalogue. The coefficient `0.35` is an assumption for a two-year horizon, not an empirical estimate. It uses the original dataset and the final Score formula from the case. Measure lags and synergies are not applied in this mode.

### Measure Catalogue — Main Mode

Fully reproduces the catalogue from the second document:

- Budget of 100 conditional units, or 1 billion ₸; **1 unit = 10 million ₸**.
- Exactly five unique measures; no more than two from one direction.
- A district is mandatory for a district-level measure; it is not specified for a city-wide measure.
- Horizon of 8 quarters; the effect is multiplied by `(8 − lag) / 8`.
- Synergies M1+M2, M10+M12, and M5+M6 are fixed and applied in the district of the first measure.
- M1 and M3 are incompatible in any district; M4/M7 and M5/M13 are incompatible within the same district.
- The negative effect of M11 on T1 is preserved. All effects are summed, then indicators are clipped to the 0–100 range. Selection order does not affect the result.
- Any violation blocks calculation and AI.

The application opens with an empty plan. The filter shows cards by direction; a card contains the price, lag, and effects. Invalid additions are blocked with an explanation. Changing the district of a selected measure re-checks the constraints; in case of a conflict, the original district is retained. A measure can be removed from its store card or from one of the five plan slots.

The “Load Example” button adds M7/Nura, M8/Nura, M10/Nura, M12/city, M5/Saryarka. This set costs **95 units** and gives **56.54307** points. The triggered synergy is M10+M12. While the plan contains fewer than five measures, the map shows the baseline, while the final Score, AI, and export are unavailable.

The results of the two modes should be compared **within the same mode**, because the impact models differ.

## Interactive Map of Astana

The main-screen map shows **Yesil, Almaty, Saryarka, Baikonur, and Nura** — the five districts in the dataset. The map uses real OpenStreetMap boundaries saved in `data/astana_districts.geojson` and synthetic case indicators. This is a selection of the case districts, not the complete current administrative map of the city.

- Clicking a boundary or label selects the district, highlights its border, and updates the panel with the score and two weakest indicators.
- The selected district is inserted into new district-level cards. Already-added measures retain their assignments.
- “Move Selected Measure” can assign an existing measure to the selected district; the budget and constraints are checked again and the result is recalculated.
- Layers show the district score, increase from the baseline, or presence of critical indicators. The “Before / After” switch compares states; with an incomplete plan, only the baseline is available.
- Zoom changes with the mouse wheel and buttons, and the map can be moved by dragging. “Entire City” returns to the initial view. A district can also be selected from the list using the keyboard.
- Boundaries, labels, and district selection work without a basemap. The CARTO street basemap requires Internet access and can be disabled with the switch.

Boundaries © OpenStreetMap contributors, license [ODbL 1.0](https://www.openstreetmap.org/copyright). The snapshot date and links to the original OSM relations are stored in GeoJSON. Geometry can be updated with `\.\.venv\Scripts\python.exe scripts/fetch_districts.py`; it accesses the Overpass API and requires Internet access. Source information is in `data/README.md`.

## Astana Quality of Life Score Formula

All indicators are oriented in the same direction: higher is better. The weights for T1, T2, E1, E2, S1, S2, B1, B2, C1, C2 respectively are:

```text
0.10, 0.10, 0.09, 0.11, 0.11, 0.11, 0.09, 0.09, 0.10, 0.10

D_d   = Σ w_k × I'_dk
D_avg = Σ pop_d × D_d
Ncrit = number of pairs (district, indicator) where I'_dk < 40
Score = clip(0.7 × D_avg + 0.3 × min(D_d) − Ncrit, 0, 100)
```

Base result without actions: **52.55768**, average district score: **56.8624**, minimum: **49.18** in Nura, critical values: **2**. Example from the document: **56.54307**. A balance of 20 units per direction in the continuous model: **64.58** after rounding.

Intermediate calculations are not rounded. AI does not set or change the numerical Score. This is a simulation of synthetic data, not a forecast of the real budget or condition of Astana.

## Structure

```text
streamlit_app.py                    Streamlit UI, charts, session state, export
src/data.py                         original indicators, weights, 14 measures, and synergies
src/model.py                        pure calculation functions and validators
src/planner.py                      adding, removing, and moving measures with checks
src/catalogue.py                    cards, filters, and selected-district synchronization
src/game_ui.py                      game HUD, district cards, slots, and launch feedback
src/landing.py                      initial overview cards
src/geography.py                    GeoJSON, colors, labels, and map-selection parsing
src/map_view.py                     Pydeck map and selected-district panel
src/ai.py                           safe key loading and OpenAI report
app.py                              alternative entry point to the same Streamlit UI
assets/catalogue.css                catalogue, plan, and map styling
assets/game.css                     game theme, responsive grid, and card states
assets/landing.css                  preserved styling of the previous first screen
assets/city-hero.svg                preserved panorama of the previous first screen
data/                               local district boundaries and source information
scripts/fetch_districts.py          downloading and assembling OSM geometry
.streamlit/config.toml              theme, localhost, disabled telemetry
.env.example                        configuration example without secrets
docs/case.md                         copy of the case conditions
docs/dataset-source.md               copy of the dataset
tests/                               calculation model, API contracts, and Streamlit UI
```

The application is bound to localhost by default. For deployment in a container, you can run `python -m streamlit run streamlit_app.py --server.address 0.0.0.0 --server.port 8501` and pass keys through platform secrets as environment variables. Public hosting and access management are handled separately.

## Verification

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements-dev.txt
.\.venv\Scripts\python.exe -m pytest -q
```

Tests check dataset reference values, budget limits, invalid types, diminishing returns, the `<40` threshold, all incompatibilities, negative effects, synergies, order independence, and absence of mutation of source data. API tests use the **real OpenAI SDK with a local MockTransport**, checking one OpenAI request, body format, timeout, and safe error handling. `AppTest` checks the catalogue, sliders, moving and rolling back a conflicting measure, preserving plans between modes, blocking an invalid set, caching, stale reports, and retrying a report after an error. Geographic tests check closed boundaries, label placement inside districts, correspondence of colors and numbers to the calculation model, selection events, and parameters for Cyrillic labels.

Tests do not send requests to external services and do not consume API quotas. Testing real AI responses requires a working key and server access to OpenAI. Game UI tests additionally check the budget, slots, decree markers, district selection, actual critical warnings, synergies, re-validation on launch, and the absence of repeated notifications.

## Sources

- [HackAlem AI Case Conditions](https://docs.google.com/document/d/1oDZtYnBgbcn_Ii7vleP87hkARJ2HmbXl7Cw_rsCxqpo/edit)
- [District Dataset, Catalogue, and Rules](https://docs.google.com/document/d/1Uc-GdGoKhDY-spu8V50-ZMm33t2CjYLP/edit)
- [OpenAI GPT-4o mini](https://developers.openai.com/api/docs/models/gpt-4o-mini)
- [OpenAI Chat Completions](https://developers.openai.com/api/reference/resources/chat/subresources/completions/methods/create)
- [Streamlit AppTest](https://docs.streamlit.io/develop/api-reference/app-testing/st.testing.v1.apptest)

Materials were read on September 23, 2026. Local copies of the case conditions preserve the original synthetic setup; the application does not load Google Docs at startup.
