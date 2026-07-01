# Design — Frontend Streamlit pour Housing API

Date : 2026-07-01

## Contexte

`housing-api` est une API FastAPI (repo git, remote `github.com/Dorsumsellae/housing-api`)
qui sert un modèle de prédiction de prix immobilier (California Housing). Elle expose :

- `GET  /health`        → `{"status": "ok", "model_loaded": true}`
- `POST /predict`        → 8 features → `{"predicted_house_value": float}`
- `POST /predict_batch`  → liste de logements → `{"predicted_house_values": [float]}`

Features (ordre du modèle) : `MedInc, HouseAge, AveRooms, AveBedrms, Population, AveOccup, Latitude, Longitude`.

## Objectif

Créer un frontend **Streamlit** (`housing-front`) qui consomme cette API, dans un
**repo git séparé**, avec une **image Docker**. Déplacer le `docker-compose.yml` vers
le dossier racine (`tp 2/`) pour qu'il lance **les deux services**. Le dossier racine
devient un **repo parent** qui référence les deux projets comme **submodules git**.

## Architecture cible

```
tp 2/                          repo PARENT (git + submodules)
├── .gitmodules                housing-api & housing-front → GitHub (Dorsumsellae)
├── docker-compose.yml         lance les 2 services
├── README.md
├── docs/superpowers/specs/    ce document
├── housing-api/               submodule (existant)
└── housing-front/             submodule (nouveau)
    ├── app.py                 UI Streamlit (3 onglets)
    ├── api_client.py          client HTTP de l'API (isolé, testable)
    ├── requirements.txt
    ├── Dockerfile
    ├── .dockerignore
    ├── .gitignore
    └── README.md
```

## Composants

### `api_client.py`
Client HTTP isolé de l'UI. Une classe `HousingAPIClient(base_url, timeout)` avec
`health()`, `predict(features)`, `predict_batch(items)`. Les erreurs réseau/HTTP sont
converties en `HousingAPIError` avec un message lisible (détail FastAPI extrait quand présent).
Dépend uniquement de `requests`. Testable sans Streamlit.

### `app.py` (Streamlit)
- **Sidebar** : champ URL API (pré-rempli via env var `API_URL`, défaut `http://housing-api:8000`),
  bouton/indicateur de santé `/health` (🟢/🔴).
- **Onglet 1 — Prédiction unique** : 8 `number_input` (bornes + valeurs par défaut =
  exemple de l'API) → `predict()` → `st.metric` + carte du point (Lat/Long).
- **Onglet 2 — Prédiction batch** : upload CSV (colonnes = 8 features), validation des
  colonnes, `predict_batch()` → tableau + téléchargement CSV. Bouton « CSV d'exemple ».
  Résultats stockés en `st.session_state`.
- **Onglet 3 — Carte** : prédictions du batch sur `st.map`, couleur selon la valeur prédite.

## Flux de données

`UI (app.py)` → `HousingAPIClient` → `requests` → API FastAPI → réponse JSON → affichage.
En docker-compose, `API_URL=http://housing-api:8000` (réseau Docker). L'URL Codespaces
(`https://…app.github.dev`) reste utilisable en surchargeant le champ de la sidebar.

## Gestion d'erreurs

- Chaque appel API dans un try/except ; `HousingAPIError` → `st.error` avec message clair
  (API injoignable, timeout, 400 avec détail, statut inattendu).
- Batch : validation des colonnes du CSV avant envoi (message explicite si colonnes manquantes).

## Docker

- **Front** : `python:3.12-slim`, install `requirements.txt`, `EXPOSE 8501`,
  `HEALTHCHECK` sur `/_stcore/health`, `streamlit run app.py --server.address=0.0.0.0 --server.port=8501 --server.headless=true`.
- **Compose (parent)** : `housing-api` (`build: ./housing-api`, `8000:8000`) +
  `housing-front` (`build: ./housing-front`, `8501:8501`, `API_URL=http://housing-api:8000`,
  `depends_on: housing-api service_healthy`).

## Repo parent + submodules

`tp 2/` = repo git. `housing-api` et `housing-front` = submodules pointant vers leurs
remotes GitHub (`github.com/Dorsumsellae/housing-api`, `.../housing-front`). Le front doit
être poussé sur GitHub pour que le submodule soit clonable ailleurs ; le README parent
documente les commandes (`git submodule update --init`, création/push du repo front).

## Hors périmètre (YAGNI)

- Pas d'authentification, pas de base de données, pas de tests E2E navigateur.
- Pas de CI/CD (peut être ajouté ensuite).
