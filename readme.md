# CodeCure

COVID prediction dashboard with a FastAPI forecasting backend and a Next.js + Tailwind frontend.

## Overview

CodeCure visualizes global COVID-19 spread on an interactive heatmap and shows region-specific forecast trends in a clean analytics dashboard. The backend serves region metadata, health status, and walk-forward daily case forecasts. The frontend consumes that API, renders the world map, and plots predicted daily new cases in a line chart.

The project is designed to stay simple but scalable:

- Backend handles model loading, feature engineering, and forecasting.
- Frontend handles presentation, interaction, and responsive layout.
- API communication is centralized so the UI can adapt to local backend ports.

## Tech Stack & Tools

### Backend

- Python
- FastAPI
- Uvicorn
- Pandas
- NumPy
- LightGBM
- Pydantic

### Frontend

- Next.js 16 App Router
- React 19
- TypeScript
- Tailwind CSS v4
- Leaflet
- Recharts

### Supporting Tools

- ESLint
- npm
- Python virtual environment

## Features

- Global COVID heatmap with interactive region markers
- Region-level forecast panel with last known cases and forecast horizon
- Forecast visualization as a time-series chart
- Responsive dashboard layout for desktop and mobile
- Backend health check and region discovery endpoints
- Simple API service layer on the frontend
- Auto-detection of common local backend ports during development

## Technical Workflow

1. The backend loads the trained LightGBM model and the historical COVID dataset at startup.
2. The backend prepares lag, rolling, mobility, and calendar features for each region.
3. The frontend calls the backend `/health` and `/regions` endpoints to confirm availability and load selectable regions.
4. The frontend builds a global heatmap using available country coordinates and last-known case values.
5. When a region is selected, the frontend sends a forecast request to `/forecast` with the chosen horizon.
6. The backend performs walk-forward prediction, returning the region name, last known cases, and a daily forecast list.
7. The frontend renders the response in the sidebar and line chart, while also updating the heatmap intensity for the selected region.

## Installation & Setup

### 1. Clone or open the project

Open the repository root

### 2. Start the backend

Create and activate a Python virtual environment if needed, then install the backend dependencies:

```bash
cd backend
python3.11 -m venv venv
source venv/bin/activate
pip install fastapi uvicorn pandas numpy lightgbm pydantic
```

Start the API server:

```bash
python3.11 -m uvicorn main:app --reload --host 127.0.0.1 --port 8000
```

Notes:

- The backend reads the model from `backend/models/model.txt`.
- The backend reads the dataset from `backend/data/covid_dataset.csv`.
- If you prefer another port, update the frontend API URL or use the auto-detection built into the frontend client.

### 3. Start the frontend

In a new terminal:

```bash
cd frontend
npm install
npm run dev
```

Open the app at:

```text
http://localhost:3000
```

### 4. Optional API URL override

If your backend runs on a custom address, set it in `frontend/.env.local`:

```bash
NEXT_PUBLIC_API_URL=http://localhost:8000
```

The frontend also checks common local backend ports during development.

## API Endpoints

### `GET /health`

Returns service status and model availability.

### `GET /regions`

Returns the list of available region names.

### `POST /forecast`

Request body:

```json
{
  "region": "india",
  "horizon": 14
}
```

Response shape:

```python
class DayForecast:
		date: date
		predicted_new_cases: int

class ForecastResponse:
		region: str
		horizon_days: int
		last_known_date: date
		last_known_cases: int
		forecast: list[DayForecast]
```

## Project Structure

```text
codecure/
	backend/
		main.py
		models/
		data/
	frontend/
		app/
		components/
		lib/
		public/
	notebooks/
```

### Demo

- ![alt text](image.png)

### Frontend Files

- `frontend/app/page.tsx`: dashboard orchestration and state
- `frontend/components/CovidMap.tsx`: interactive Leaflet map
- `frontend/components/ForecastChart.tsx`: forecast chart and summary KPIs
- `frontend/components/MapSidebar.tsx`: controls and prediction details
- `frontend/components/Header.tsx`: top navigation and status display
- `frontend/components/StatCard.tsx`: reusable metrics cards
- `frontend/lib/api.ts`: backend API client
- `frontend/lib/countryCoords.ts`: country coordinate lookup table

### Backend Files

- `backend/main.py`: API, feature engineering, and walk-forward forecast logic
- `backend/models/model.txt`: trained LightGBM model

## Development Notes

- The frontend uses Leaflet and Recharts, so the map and chart should be rendered client-side only.
- The dashboard expects lowercase region names to match the backend model data.
- The heatmap uses approximate country centroids, so it is intended for high-level geographic visualization rather than precise epidemiological mapping.
- Predictions are model-generated estimates and should be treated as analytical guidance only.

## Troubleshooting

- If the frontend cannot reach the backend, confirm the API server is running and that the port matches the frontend configuration.
- If you see a startup error in the backend, check that `backend/data/covid_dataset.csv` and `backend/models/model.txt` exist.
- If the chart or map does not render correctly, ensure browser JavaScript is enabled and the frontend dependencies are installed.
