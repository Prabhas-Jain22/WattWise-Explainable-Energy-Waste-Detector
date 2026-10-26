# WattWise

WattWise is a dark, responsive energy-intelligence dashboard for campuses and small facilities. The Vite/React frontend charts calculated readings and offers CSV import, anomaly review, an evidence-first advisor, savings actions, and report exports. FastAPI validates uploads and performs deterministic energy and cost calculations. MongoDB is optional for dataset persistence; without it, the API stores datasets in process memory. Groq is optional and only called by the backend when configured.

## Start locally (Windows PowerShell)

Prerequisites: Node.js 20+, Python 3.11+, and optionally MongoDB.

```powershell
# Terminal 1 — API
cd D:\AI-ML-Training\wattwise\backend
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
$env:MONGODB_URI = "mongodb://localhost:27017" # optional
$env:GROQ_API_KEY = "your-key"                 # optional
$env:GROQ_MODEL = "llama-3.3-70b-versatile"
uvicorn app.main:app --reload --port 8000

# Terminal 2 — frontend
cd D:\AI-ML-Training\wattwise
npm install
npm run dev
```

Open http://localhost:5173. API docs are at http://localhost:8000/docs. Copy `.env.example` to `.env` for reference; the sample API currently reads settings from environment variables. Never put a Groq key in frontend variables. For frontend/API development, configure `CORS_ORIGINS` as a comma-separated list of trusted origins.

## Sample data and CSV format

Try `data/sample_energy.csv` using **Upload data**. Required fields are `timestamp` and `consumption_kwh` (also accepts common aliases `datetime`/`date` and `kwh`/`consumption`). `building` and `zone` are optional. `temperature_c` and `occupancy` can be included as context. Timestamps must parse consistently and energy must be numeric and nonnegative. The API accepts CSV files up to 8 MB and 100,000 rows, rejects invalid measurements and reports duplicate timestamps before retaining the latest duplicate.

## Analytics and AI

- Total kWh is the sum of validated readings. Estimated energy charges are kWh multiplied by the configured ₹/kWh tariff; taxes, fixed charges, demand charges, and time-of-use tariffs are excluded.
- Current anomaly flags identify readings greater than 80% above their hour-of-day mean. This transparent first-pass heuristic flags investigation candidates and does not determine cause or prove waste. It needs broader comparable history for a robust median baseline.
- Savings potential is a clearly labeled illustrative 8% reduction estimate multiplied by the current tariff, not a guaranteed saving.
- `/api/ai/ask` sends the user question and calculated summary to Groq. The system prompt requires evidence-based answers that distinguish observations from hypotheses. Without a key, the endpoint returns metrics and a configuration notice. Groq calls have an 18-second timeout and one retry.

## API examples

```powershell
curl http://localhost:8000/api/health
curl -F "file=@..\data\sample_energy.csv" http://localhost:8000/api/datasets/upload
curl http://localhost:8000/api/dashboard/summary
curl -X POST http://localhost:8000/api/ai/ask -H "Content-Type: application/json" -d '{"question":"Which usage periods need review?"}'
```

Core endpoints include dataset upload/list/detail, dashboard summary, consumption and baseline analytics, anomalies and investigation, AI question answering, settings/tariff, actions, and report generation. See `/docs` for request schemas.

## Manual smoke checklist

1. Start the API and frontend.
2. Upload `data/sample_energy.csv` and inspect its calculated totals and anomaly flags.
3. Upload a second CSV with valid optional context columns.
4. Try a malformed timestamp and a negative reading; confirm a clear validation error.
5. Review observed-versus-expected values in Anomaly Investigation.
6. Configure Groq and ask a question, or verify the unconfigured response.
7. Add a savings action and export readings from Reports.

## Limitations

The browser starts in demo mode so the dashboard remains explorable before an upload. The backend has no authentication or tenant isolation and should remain local/trusted until those are added. MongoDB persistence stores validated readings in a dataset document; the in-memory fallback is lost on restart. The current frontend has a local CSV fallback if the API is unavailable. Some non-core report/action screens are initial MVP views. Use real tariff data and facility review before using estimates for operational decisions.

## Screenshots

Add screenshots here after running the app.
