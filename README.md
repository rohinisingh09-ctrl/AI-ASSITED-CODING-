# React + Vite

## Python backend

The backend uses Flask and runs locally on `http://127.0.0.1:5000`. Create and activate a Python virtual environment from the project root, then install the backend dependencies:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r server\requirements.txt
```

Configure `server/.env` (keep this file private and out of version control):

```dotenv
MONGO_URI=mongodb://127.0.0.1:27017/novara
GEMINI_API_KEY=your-google-ai-api-key
# Optional: GEMINI_MODEL=gemini-3.8-flash
```

MongoDB must be available for energy history, controls, predictions, and monitoring records. Gemini is only needed for the optional weather-based AI recommendation; deterministic energy recommendations continue to work without it. Start the frontend and backend together with `npm run dev:full`, or run `python server\app.py` separately. Vite forwards `/api` requests to Flask.

Run the backend tests with:

```powershell
python -m unittest discover -s server -p "test_*.py"
```

### Backend files

- `server/app.py` creates the Flask app, configures CORS and environment loading, exposes JSON API routes, connects to MongoDB, and stores prediction and monitoring records. MongoDB uses the existing `energies` and `controls` collections; generated records use `prediction_records` and `monitoring_records`.
- `server/prediction_service.py` validates weather and climate input, trains and evaluates Linear Regression, Random Forest, and Gradient Boosting models, generates the seven-day energy estimates, and runs Isolation Forest anomaly detection. Model artifacts are fingerprinted and cached in the ignored `server/.models/` folder. An anomaly reports an unusual modeled value and direction, not a presumed cause.
- `server/recommendations.py` implements deterministic recommendations from saved energy records and forecast estimates; it does not depend on Gemini.
- `server/requirements.txt` lists Flask, MongoDB, Gemini, and machine-learning dependencies.
- `server/test_app.py` checks Flask API validation and record persistence; `server/test_prediction_service.py` covers forecast generation, model caching, and anomaly detection; `server/test_recommendations.py` exercises deterministic recommendation rules.

### API overview

- `GET`/`POST /api/energy` read and store energy history.
- `GET`/`POST /api/controls` read and update simulation controls.
- `POST /api/predictions` computes a forecast, anomalies, and model evaluation, and persists the prediction and per-day monitoring estimates. `GET /api/predictions` and `GET /api/monitoring` retrieve stored results.
- `POST /api/recommendations` returns deterministic suggestions from saved readings and forecasts.
- `POST /api/ai/recommendation` provides an optional weather-based Gemini recommendation.

Energy generation and consumption values are climate/weather-derived estimates, not readings from connected equipment. The backend does not assign an exact cause to an anomaly.

## AI-Based Recommendations

The searchable **AI-Based Recommendations** page uses the standard-library Python rules in `server/recommendations.py` to analyze saved energy records and the weather-based forecast. It does not require Gemini; the optional AI-powered recommendation endpoint is separate.

## Frontend

The responsive renewable-energy UI is built with React, CSS, and Recharts. The sidebar provides Dashboard, Energy Monitoring, Analytics, AI Prediction, Anomaly Detection, AI Recommendations, Alerts & Notifications, Reports, and Settings. On small screens the navigation becomes a horizontally scrollable bottom bar.

Dashboard energy totals and charts are derived from the selected location's seven-day modeled forecast. Values are estimates and simulated consumption, not meter telemetry. The recommendation module is deterministic Python analysis of saved readings and forecasts; it does not use an LLM.

This template provides a minimal setup to get React working in Vite with HMR and some Oxlint rules.

Currently, two official plugins are available:

- [@vitejs/plugin-react](https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react) uses [Oxc](https://oxc.rs)
- [@vitejs/plugin-react-swc](https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react-swc) uses [SWC](https://swc.rs/)

## React Compiler

The React Compiler is not enabled on this template because of its impact on dev & build performances. To add it, see [this documentation](https://react.dev/learn/react-compiler/installation).

## Expanding the Oxlint configuration

If you are developing a production application, we recommend using TypeScript with type-aware lint rules enabled. Check out the [TS template](https://github.com/vitejs/vite/tree/main/packages/create-vite/template-react-ts) for information on how to integrate TypeScript and Oxlint's TypeScript related rules in your project.
