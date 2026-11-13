# FraudLens 🛡️

**See the risk. Stop the fraud.**

FraudLens is a focused transaction-risk review application. It pairs the existing PaySim-trained MLP fraud classifier with a React interface and a FastAPI service, following the frontend/API deployment pattern used by VoltCast and Estatora.

## Brand and product principles

- Formal, professional, and analytical
- Security-focused, minimal, and enterprise-ready
- Deep Burgundy `#853E52` for primary actions
- Graphite `#211C20` for the main background
- Charcoal `#51464C` for secondary surfaces
- Muted Silver `#C5B8BC` for borders and secondary text
- Soft Ivory `#F4F1F2` for light surfaces
- Restrained amber/red for elevated fraud risk; neutral tones for low-risk results

## Stack

| Layer | Technology | Version |
| --- | --- | --- |
| Frontend | React / React DOM | 18.3.1 |
| Frontend tooling | Vite | 6.4.6 |
| Styling | Tailwind CSS / `@tailwindcss/vite` | 4.1.11 |
| Icons | Lucide React | 0.525.0 |
| API | FastAPI | 0.115.10 |
| API server | Uvicorn | 0.32.1 |
| Validation | Pydantic | 2.10.6 |
| Data and preprocessing | NumPy / pandas / scikit-learn / joblib | 1.26.4 / 2.3.3 / 1.7.2 / 1.4.2 |
| Inference | TensorFlow / Keras (CPU inference) | 2.21.0 / 3.15.0 |

The frontend follows the shared React and Tailwind versions and the Vite 6 range used by VoltCast; Vite is pinned to a patched 6.x release. Lucide React follows the version used by Estatora. TensorFlow is retained for the existing Keras model; unused Streamlit, PyTorch, TabNet, and plotting dependencies are not part of the serving stack.

## Project structure

```text
FraudLens/
├── backend/
│   ├── app/
│   │   ├── api/
│   │   ├── artifacts/
│   │   ├── models/
│   │   ├── services/
│   │   ├── config.py
│   │   ├── main.py
│   │   └── schemas.py
│   ├── requirements.txt
│   └── tests/
├── frontend/
│   ├── public/
│   ├── src/
│   ├── index.html
│   ├── package.json
│   └── vite.config.js
├── render.yaml
└── README.md
```

The deployment keeps only the inference model, its metadata, the preprocessing bundle, and the preprocessing manifest. Training datasets, notebooks, prediction dumps, plots, and demo documents are not required to run the application.

## Local development

Requirements: Python 3.11.11 and Node.js 18 or newer. Python is pinned to 3.11.11, and TensorFlow/Keras are pinned to versions compatible with the saved model.

### Start the API

```powershell
cd backend
conda create --prefix .venv python=3.11 pip
.\.venv\python.exe -m pip install -r requirements.txt
Copy-Item .env.example .env
.\.venv\python.exe -m uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

API docs: `http://localhost:8000/api/docs`

### Start the frontend

In another terminal:

```powershell
cd frontend
npm ci
npm run dev
```

Open the Vite URL, normally `http://localhost:5173`. The development server proxies `/api` requests to `http://localhost:8000`.

## API

- `GET /` — service metadata
- `GET /api/health` — model health and prediction readiness
- `GET /api/model-info` — model name, supported transaction types, and features
- `GET /api/metrics` — hold-out evaluation metrics for the selected model
- `POST /api/predict` — score a transaction

Example request:

```json
{
  "transaction_type": "TRANSFER",
  "hour": 14,
  "amount": 1250.0,
  "sender_balance_before": 5000.0,
  "sender_balance_after": 3750.0,
  "receiver_balance_before": 10000.0,
  "receiver_balance_after": 11250.0
}
```

Fraud scores are decision support, not a guarantee or a substitute for an institution's review and controls. The model was trained on synthetic PaySim data and may not generalize to live payment data.

## Tests and production build

```powershell
cd backend
.\.venv\python.exe -m pytest
```

```powershell
cd frontend
npm ci
npm run build
```

## Render deployment

The root `render.yaml` defines a FastAPI web service and a React static site. The API health check is `/api/health`; the frontend receives the API origin through `VITE_API_URL`. If the final Render service URLs differ from the blueprint defaults, update `VITE_API_URL` and the API's `ALLOWED_ORIGINS` before redeploying.
