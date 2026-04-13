# InkScroller Backend

![FastAPI](https://img.shields.io/badge/FastAPI-0.128-009688?style=flat-square&logo=fastapi&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.12-3776AB?style=flat-square&logo=python&logoColor=white)
![Cloud Run](https://img.shields.io/badge/Deploy-Cloud%20Run-4285F4?style=flat-square&logo=googlecloud&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)

REST API backend for the **InkScroller** manga reader. Aggregates MangaDex and Jikan (MyAnimeList) data, and provides authenticated user library management and preferences via Firebase Auth.

---

## Features

| Area | Description |
|------|-------------|
| **Manga catalogue** | Browse, filter, and paginate the MangaDex catalogue |
| **Search** | Title-based manga search (up to 5 results) |
| **Detail enrichment** | MangaDex data augmented with Jikan/MAL metadata (score, rank, authors, genres) |
| **Chapter listing** | Per-manga chapter list with language filtering |
| **Page URLs** | MangaDex@Home image URLs for any chapter |
| **Auth** | Firebase ID token verification on protected endpoints |
| **User profile** | Auto-created user row on first authenticated request (`/users/me`) |
| **Preferences** | Reading preferences per user (`defaultReaderMode`, `defaultLanguage`) |
| **User library** | Save/remove titles, track reading status (added / reading / completed / paused) |
| **Latest chapters** | Homepage feed of recent chapter updates across the catalogue |
| **Caching** | In-memory 5-minute TTL cache on all service calls |
| **Health check** | Liveness probe at `/ping` |

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | FastAPI 0.128 |
| HTTP client | httpx (async) |
| Data validation | Pydantic v2 |
| ASGI server | Uvicorn |
| Auth | Firebase Admin SDK |
| Persistence | SQLite via aiosqlite |
| Runtime | Python 3.12 |
| Deploy | Google Cloud Run (3 environments) |

---

## API Reference

### Public

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/ping` | Health check → `{"ok": true}` |
| `GET` | `/manga` | Paginated manga list (`limit`, `offset`, `title`, `demographic`, `status`, `order`) |
| `GET` | `/manga/search?q=` | Title search — max 5 results |
| `GET` | `/manga/{id}` | Manga detail with Jikan enrichment |
| `GET` | `/chapters/latest` | Latest chapter updates (homepage feed) |
| `GET` | `/chapters/manga/{id}` | Chapter list for a manga (filtered by `lang`, default `en`) |
| `GET` | `/chapters/{id}/pages` | Page image URLs via MangaDex@Home |

### Authenticated (requires `Authorization: Bearer <firebase-id-token>`)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/users/me` | Get or create user profile |
| `GET` | `/users/me/preferences` | Get reading preferences |
| `PUT` | `/users/me/preferences` | Update `defaultReaderMode` and/or `defaultLanguage` |
| `GET` | `/users/me/library` | List user's saved manga with status metadata |
| `POST` | `/users/me/library/{manga_id}` | Add a manga to the user's library |
| `PATCH` | `/users/me/library/{manga_id}` | Update reading status (added / reading / completed / paused) |
| `DELETE` | `/users/me/library/{manga_id}` | Remove a manga from the user's library |

> Full API details: [`docs/PROJECT_STATUS.md`](docs/PROJECT_STATUS.md)

---

## Running Locally

```bash
# 1. Create and activate virtualenv
python -m venv venv
source venv/bin/activate        # Linux / macOS
# venv\Scripts\activate         # Windows

# 2. Install dependencies
python -m pip install -r requirements.txt

# 3. Configure environment
cp .env.example .env
# Edit .env — set FIREBASE_PROJECT_ID and GOOGLE_APPLICATION_CREDENTIALS

# 4. Start server
python -m uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

| URL | Description |
|-----|-------------|
| `http://localhost:8000/ping` | Health check |
| `http://localhost:8000/docs` | Swagger UI |
| `http://localhost:8000/redoc` | ReDoc |

> **Windows note:** Use `python -m pip` and `python -m uvicorn` — never bare `pip`/`uvicorn` — to avoid launcher path issues.

---

## Deployment

Deploy to **Google Cloud Run** with the included `Dockerfile`:

> Full deployment guide: [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md)

---

## Project Structure

```
inkscroller-api/
├── main.py                        # App entry — mounts all routers
├── Dockerfile                     # Multi-stage build for Cloud Run
├── requirements.txt
│
└── app/
    ├── api/                       # FastAPI route handlers
    │   ├── health.py              # GET /ping
    │   ├── manga.py               # GET /manga, /manga/search, /manga/{id}
    │   ├── chapters.py            # GET /chapters/latest, /chapters/manga/{id}, /chapters/{id}/pages
    │   └── users.py               # /users/me, /users/me/preferences, /users/me/library
    │
    ├── core/
    │   ├── cache.py               # SimpleCache — TTL-based in-memory cache
    │   ├── config.py              # Settings via env vars
    │   ├── auth.py                # Firebase ID token verification middleware
    │   ├── database.py            # DB connection (SQLite local / Cloud SQL in prod)
    │   └── dependencies.py        # FastAPI DI factories
    │
    ├── models/                    # Pydantic response models
    │   ├── manga.py
    │   ├── chapter.py
    │   └── user.py                # UserProfile, UserPreferences, LibraryMetadata
    │
    ├── services/                  # Business logic
    │   ├── manga_service.py
    │   ├── chapter_service.py
    │   ├── chapter_pages_service.py
    │   └── user_service.py        # User creation, preferences, library management
    │
    └── sources/                   # External API clients (async httpx)
        ├── mangadex_client.py
        └── jikan_client.py

tests/
├── test_app.py                    # Smoke tests with DI overrides
└── test_users_auth.py             # Auth flow tests
```

---

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feat/your-feature`
3. Commit using [Conventional Commits](https://www.conventionalcommits.org/)
4. Open a Pull Request

---

## Atribución y Disclaimer

InkScroller Backend agrega datos de las siguientes fuentes externas:

- **MangaDex** — fuente primaria de catálogo, capítulos e imágenes de manga. InkScroller no está afiliado a MangaDex. Todo el contenido pertenece a sus respectivos autores y grupos de scanlation. Se respetan los [Términos de Servicio de MangaDex](https://mangadex.org/about/terms-of-service).
- **Jikan / MyAnimeList** — capa de enriquecimiento de metadatos (score, rank, géneros). Jikan es un servicio no oficial de terceros. InkScroller no está afiliado a MyAnimeList ni a Jikan. Se respetan los [Términos de Uso de MyAnimeList](https://myanimelist.net/about/terms_of_use).

Este proyecto actúa como **proxy de lectura**. No almacena ni redistribuye imágenes de manga. Los derechos sobre el contenido pertenecen a sus titulares originales.

Para consultas legales o solicitudes de takedown, ver [`docs/legal/api-compliance.md`](docs/legal/api-compliance.md).

---

## License

MIT License — see [LICENSE](LICENSE) for details.
