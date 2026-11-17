# Real-Time ML Inference REST API & Capstone

A FastAPI microservice serving the trained Scikit-learn champion pipeline from the Adult Census Income classification project. The model predicts whether an individual's income is above `$50K` and returns the probability for the `>50K` class.

> **Model artifact required:** Place the exact fitted `champion_model.joblib` from the supervised-classification project in `models/champion_model.joblib`. The API intentionally reports `503 Service Unavailable` until the real model artifact is present.

## Architecture

```text
Client (JSON)
    |
    v
FastAPI /predict
    |
    v
Pydantic request validation
    |
    v
Pandas DataFrame (14 raw Adult Census features)
    |
    v
Fitted Scikit-learn Pipeline
  - numeric imputation + scaling
  - categorical imputation + one-hot encoding
  - selected champion classifier
    |
    v
Prediction + P(Income > $50K)
    |
    v
JSON response
```

The saved artifact is expected to be the complete fitted pipeline, including preprocessing. Send the original feature values; do not one-hot encode or scale them in the client.

## Project structure

```text
.
├── app/
│   ├── __init__.py
│   └── main.py
├── models/
│   └── champion_model.joblib   # add your trained model here
├── tests/
│   └── test_api.py
├── Dockerfile
├── requirements.txt
├── .dockerignore
├── .gitignore
└── README.md
```

## Input schema

The `/predict` endpoint expects all 14 raw Adult Census features:

| Field | Type | Example |
|---|---|---|
| `age` | integer | `39` |
| `workclass` | string | `"Private"` |
| `fnlwgt` | number | `77516` |
| `education` | string | `"Bachelors"` |
| `education-num` | integer | `13` |
| `marital-status` | string | `"Never-married"` |
| `occupation` | string | `"Adm-clerical"` |
| `relationship` | string | `"Not-in-family"` |
| `race` | string | `"White"` |
| `sex` | string | `"Male"` |
| `capital-gain` | number | `2174` |
| `capital-loss` | number | `0` |
| `hours-per-week` | number | `40` |
| `native-country` | string | `"United-States"` |

## Run locally

Use Python 3.11.

```bash
python -m venv .venv
```

Activate the environment:

```bash
# Windows PowerShell
.venv\Scripts\Activate.ps1

# macOS/Linux
source .venv/bin/activate
```

Install pinned dependencies:

```bash
python -m pip install -r requirements.txt
```

Copy the trained model to:

```text
models/champion_model.joblib
```

Start the API:

```bash
uvicorn app.main:app --reload
```

Open interactive API documentation at:

- Swagger UI: http://127.0.0.1:8000/docs
- ReDoc: http://127.0.0.1:8000/redoc
- Health: http://127.0.0.1:8000/health

## Example request

```bash
curl -X POST "http://127.0.0.1:8000/predict" \
  -H "Content-Type: application/json" \
  -d '{
    "age": 39,
    "workclass": "Private",
    "fnlwgt": 77516,
    "education": "Bachelors",
    "education-num": 13,
    "marital-status": "Never-married",
    "occupation": "Adm-clerical",
    "relationship": "Not-in-family",
    "race": "White",
    "sex": "Male",
    "capital-gain": 2174,
    "capital-loss": 0,
    "hours-per-week": 40,
    "native-country": "United-States"
  }'
```

## Example response

The following is an illustrative response shape, not a measured prediction:

```json
{
  "prediction": ">50K",
  "probability_over_50k": 0.8,
  "model": "champion_model"
}
```

## Error responses

| Status | Meaning |
|---|---|
| `200` | Prediction returned |
| `422` | Missing, invalid, or unexpected input field |
| `503` | Trained model file is not available |
| `500` | Model inference failed |

## Run unit tests

```bash
pytest -q
```

Tests validate the response schema, successful predictions using a lightweight fake model, missing/extra input validation, and the unavailable-model response. They do not replace a live integration test with the actual serialized champion model.

## Docker

Build the image after adding the model artifact:

```bash
docker build -t adult-income-ml-api .
```

Run the container:

```bash
docker run --rm -p 8000:8000 adult-income-ml-api
```

Then visit http://localhost:8000/docs.

## Model compatibility and security notes

- Scikit-learn does not guarantee that models serialized with `joblib` can be loaded across arbitrary Scikit-learn versions. The pinned versions here are a reproducible starting point; use the same Scikit-learn and relevant dependency versions used to train the model.
- Only load trusted `.joblib` files. Deserialization can execute code.
- The API is a demonstration service. Add authentication, rate limiting, HTTPS, logging controls, monitoring, and privacy review before public production deployment.
- The model output is a statistical estimate, not a guarantee and not a decision about an individual.

## Capstone checklist

- [ ] Add the real `models/champion_model.joblib` file.
- [ ] Confirm dependency versions match the training environment.
- [ ] Start the API and verify `/health`.
- [ ] Make a real `/predict` request using the saved model.
- [ ] Run `pytest -q`.
- [ ] Build and run the Docker image.
- [ ] Capture proof screenshots of Swagger `/docs`, a successful prediction, passing tests, and Docker startup.
