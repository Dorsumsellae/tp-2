# Housing — Projet parent (API + Frontend)

Repo **parent** qui regroupe, via des **submodules git**, les deux projets et fournit
le `docker-compose.yml` qui les lance ensemble.

| Sous-projet | Rôle | Repo |
|-------------|------|------|
| [`housing-api`](https://github.com/Dorsumsellae/housing-api) | API FastAPI de prédiction (California Housing) | submodule |
| [`housing-front`](https://github.com/Dorsumsellae/housing-front) | Interface Streamlit qui consomme l'API | submodule |

```
tp 2/                      ← ce repo (parent)
├── docker-compose.yml     ← lance les 2 services
├── .gitmodules
├── housing-api/           ← submodule
└── housing-front/         ← submodule
```

## Lancer les deux services (Docker Compose)

Depuis ce dossier :

```bash
docker compose up --build
```

| Service | URL | Description |
|---------|-----|-------------|
| API (Swagger) | http://localhost:8000/docs | FastAPI |
| Frontend | http://localhost:8501 | Streamlit |

Le front est configuré (`API_URL=http://housing-api:8000`) pour joindre l'API par le
réseau interne de compose. Pour pointer vers une autre API (p. ex. Codespaces
`https://…app.github.dev`), modifiez le champ **URL de l'API** dans la sidebar Streamlit.

Arrêt : `docker compose down`.

## Cloner ce repo avec ses submodules

```bash
# Clone + récupération des submodules en une commande
git clone --recurse-submodules <url-du-parent>

# Ou, si déjà cloné sans les submodules :
git submodule update --init --recursive
```

## Mettre à jour les submodules vers leur dernier commit

```bash
git submodule update --remote --merge
git add housing-api housing-front
git commit -m "chore: bump submodules"
```

## Important — pousser les sous-repos

Les submodules référencent des **commits précis** sur GitHub. Pour qu'un clone frais
résolve les submodules, les commits pointés doivent exister sur les remotes. Après toute
modification d'un sous-projet :

```bash
# Dans housing-api/ (le commit qui retire docker-compose.yml doit être poussé)
cd housing-api && git push && cd ..

# Dans housing-front/ (créer le repo GitHub s'il n'existe pas, puis pousser)
cd housing-front && git push -u origin main && cd ..

# Puis, dans le parent, enregistrer les nouvelles références de submodules
git add housing-api housing-front && git commit -m "chore: bump submodules" && git push
```

> Le repo `housing-front` doit être créé sur GitHub (`Dorsumsellae/housing-front`) avant
> le premier `git push`.

## Développement local (sans Docker)

```bash
# Terminal 1 — API
cd housing-api && uvicorn app:app --reload

# Terminal 2 — Front
cd housing-front && API_URL=http://localhost:8000 streamlit run app.py
```
