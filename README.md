# DeployStack

DeployStack containerisiert die RiskScorer API – ein Machine-Learning-Modell zur Kreditrisikobewertung – mit Docker und docker-compose.
Das Projekt demonstriert, wie ein FastAPI-Dienst reproduzierbar verpackt, gestartet und betrieben wird.

## Architektur

```
Client
  │
  │  HTTP-Request
  ▼
Nginx (optional, Reverse Proxy)
  │
  │  Weiterleitung an Port 8000
  ▼
Docker Container
  │
  ├─► FastAPI (uvicorn)
  │     │
  │     └─► ML Model (scikit-learn, geladen aus models/)
  │
  └─► Volume: models/  ←──  Modell bleibt nach Neustart erhalten
```

## Quickstart

```bash
# 1. Repository klonen
git clone https://github.com/yourusername/deploystack.git
cd deploystack

# 2. Trainingsdatensatz ablegen (credit_risk_dataset.csv z. B. von Kaggle)
#    Zielort: data/raw/credit_risk_dataset.csv

# 3. Container bauen und starten
docker compose up --build
```

Die API ist danach unter `http://localhost:8000` erreichbar.

## Modell trainieren

Das Modell ist nicht im Repository enthalten und muss nach dem ersten Start trainiert werden.
Solange kein Modell geladen ist, antwortet `POST /predict` mit HTTP 503.

```bash
# Modell mit Gradient Boosting trainieren (Standard)
curl -X POST "http://localhost:8000/train?model_type=gradient_boosting"

# Alternativ: Random Forest oder Logistische Regression
curl -X POST "http://localhost:8000/train?model_type=random_forest"
curl -X POST "http://localhost:8000/train?model_type=logistic"
```

Das trainierte Modell wird in `models/` gespeichert und beim nächsten Container-Start automatisch geladen.

## Endpunkte

| Methode | Pfad       | Beschreibung                                          |
|---------|------------|-------------------------------------------------------|
| GET     | `/health`  | Betriebsstatus und ob ein Modell geladen ist          |
| POST    | `/train`   | Modell auf Kreditrisikodaten trainieren und speichern |
| POST    | `/predict` | Kreditrisiko für einen einzelnen Antrag vorhersagen   |
| GET     | `/docs`    | Interaktive API-Dokumentation (Swagger UI)            |

## Projektstruktur

```
deploystack/
├── app/
│   ├── __init__.py
│   ├── api.py           # FastAPI-Endpunkte und Lebenszyklus
│   ├── preprocessor.py  # Datenvorverarbeitung und Train/Test-Split
│   └── trainer.py       # Modell-Training, Evaluation und Persistenz
├── data/
│   └── raw/             # Trainingsdaten (nicht im Repo, via .gitignore)
├── models/              # Trainierte Modelle (persistent via Docker Volume)
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── .gitignore
└── README.md
```

## Erstellte Dateien

| Datei                    | Beschreibung                                              |
|--------------------------|-----------------------------------------------------------|
| `app/__init__.py`        | Paket-Markierung                                          |
| `app/api.py`             | FastAPI-App mit `/health`, `/train`, `/predict`           |
| `app/preprocessor.py`   | Datenvorverarbeitung (Imputation, Encoding, Split)        |
| `app/trainer.py`         | Modell-Training, Evaluation und joblib-Persistenz         |
| `models/.gitkeep`        | Platzhalter, hält leeres Verzeichnis im Repository        |
| `data/raw/.gitkeep`      | Platzhalter, hält leeres Verzeichnis im Repository        |
| `Dockerfile`             | Image-Definition auf Basis python:3.12-slim               |
| `docker-compose.yml`     | Service-Konfiguration mit Volume, Port und Umgebungsvars  |
| `requirements.txt`       | Python-Abhängigkeiten                                     |
| `.gitignore`             | Ausschluss von Modellen, Datensätzen und Cache            |
| `README.md`              | Diese Dokumentation                                       |
