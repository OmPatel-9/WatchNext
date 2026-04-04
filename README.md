# WatchNext

A full-stack movie tracking app with personalized recommendations, watchlist management, and viewing analytics. Built with FastAPI and Next.js.

![Python](https://img.shields.io/badge/Python-3.11-3776AB?logo=python&amp;logoColor=white)
![Next.js](https://img.shields.io/badge/Next.js-14-000000?logo=next.js&amp;logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&amp;logoColor=white)
![TMDB](https://img.shields.io/badge/TMDB-API-01D277?logo=themoviedatabase&amp;logoColor=white)

## Features

- Search &amp; Discover   Browse 200k+ movies with poster art, ratings, and metadata sourced from TMDB
- Watched List   Rate movies on a 1 10 scale, add notes, and track your viewing history
- Watchlist   Save movies you want to watch later and move them to watched when you're done
- Personalized Recommendations   Content-based engine using TF-IDF cosine similarity across genres, keywords, language, and release era, with human-readable explanations for each pick
- Analytics Dashboard   Interactive charts for genre distribution, release timeline, revenue breakdown, and your rating patterns (powered by Recharts)
- Auto-Sync with TMDB   Automatically fetches new and updated movies from the TMDB API daily so your database stays current with new releases
- Auth   JWT-based registration and login with bcrypt password hashing

## Tech Stack

Backend: FastAPI, SQLite (aiosqlite), scikit-learn, NumPy, SciPy, Pandas

Frontend: Next.js 14, React 18, TypeScript, Tailwind CSS, Recharts, Axios, Lucide Icons

Infrastructure: Docker Compose, multi-stage data pipeline

## Getting Started

### Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and Docker Compose
- A free [TMDB API key](https://www.themoviedb.org/settings/api) (for auto-sync)

### Setup

1. Clone the repo

   bash    git clone https://github.com/your-username/WatchNext.git    cd WatchNext    

2. Create your .env file

   bash    cp .env.example .env    

   Edit .env and fill in your values:

       SECRET_KEY=your-secure-random-string    TMDB_API_KEY=your-tmdb-api-key    

3. Download the TMDB dataset

   Download TMDB_movie_dataset_v11.csv from [Kaggle](https://www.kaggle.com/datasets/asaniczka/tmdb-movies-dataset-2023-930k-movies) and place it in the project root. This is used for the initial database seed only.

4. Start everything

   bash    docker compose up --build -d    

   This will:
   - Run the data pipeline (clean CSV, load into SQLite, build feature matrix)
   - Sync latest movies from the TMDB API
   - Start the backend on http://localhost:8000
   - Start the frontend on http://localhost:3000

5. Open the app at [http://localhost:3000](http://localhost:3000)

### Resetting the Database

To wipe the database and rebuild from scratch:

bash docker compose down -v docker compose up --build -d 

## Project Structure

 WatchNext/ %%% backend/ %   %%% main.py                  # FastAPI app, lifespan, auto-sync scheduler %   %%% config.py                # Environment config (DB, JWT, TMDB key, paths) %   %%% database.py              # SQLite schema and connection management %   %%% auth.py                  # Password hashing and JWT token utilities %   %%% models.py                # Pydantic request/response models %   %%% state.py                 # Shared app state (recommendation engine) %   %%% dependencies.py          # FastAPI dependency injection (auth) %   %%% routers/ %   %   %%% auth_router.py       # POST /api/auth/register, /login %   %   %%% movies_router.py     # GET /api/movies/search, /popular, /{id} %   %   %%% watched_router.py    # CRUD for user's watched movies %   %   %%% watchlist_router.py  # CRUD for user's watchlist %   %   %%% recommendations_router.py  # GET /api/recommendations %   %   %%% analytics_router.py  # GET /api/analytics/genres, /timeline, etc. %   %   %%% sync_router.py       # POST /api/sync, GET /api/sync/status %   %%% services/ %   %   %%% recommendation_service.py  # TF-IDF cosine similarity engine %   %%% scripts/ %   %   %%% 01_clean_csv.py      # Clean raw TMDB CSV �! Parquet %   %   %%% 02_load_db.py        # Load Parquet �! SQLite %   %   %%% 03_build_features.py # Build TF-IDF feature matrix %   %   %%% tmdb_sync.py         # TMDB API sync (incremental &amp; full) %   %   %%% init_data.sh         # Orchestrates pipeline + sync on startup %   %%% requirements.txt %   %%% Dockerfile %%% frontend/ %   %%% src/ %   %   %%% app/ %   %   %   %%% page.tsx              # Landing page / dashboard %   %   %   %%% search/page.tsx       # Movie search with debounce %   %   %   %%% watched/page.tsx      # Watched list with sort/edit/delete %   %   %   %%% watchlist/page.tsx     # Watchlist management %   %   %   %%% recommendations/page.tsx  # Personalized picks %   %   %   %%% analytics/page.tsx    # Charts and stats %   %   %   %%% movie/[id]/page.tsx   # Movie detail page %   %   %   %%% login/page.tsx        # Login form %   %   %   %%% register/page.tsx     # Registration form %   %   %%% components/ %   %   %   %%% MovieCard.tsx         # Reusable movie card with poster %   %   %   %%% Navbar.tsx            # Navigation bar %   %   %   %%% RatingModal.tsx       # Star rating modal %   %   %%% lib/ %   %       %%% api.ts                # Axios instance with auth interceptor %   %       %%% auth.tsx              # Auth context provider %   %       %%% types.ts             # TypeScript interfaces %   %%% package.json %   %%% tailwind.config.ts %   %%% Dockerfile %%% docker-compose.yml %%% .env.example %%% TMDB_movie_dataset_v11.csv       # Initial dataset (not committed) %%% README.md 

## How the Recommendation Engine Works

1. Feature extraction   Each movie is represented as a sparse vector built from TF-IDF features across four dimensions: genres (weight 3.0), keywords (weight 2.0), language (weight 2.0), and release decade (weight 1.5).

2. User profile   When you rate movies, your ratings are normalized to [0, 1] and used as weights to build a composite user profile vector from all your watched movies.

3. Scoring   Cosine similarity is computed between your profile and every movie in the database. Already-watched and watchlisted movies are excluded.

4. Explanations   Each recommendation comes with human-readable reasons like "Similar genres: Sci-Fi, Thriller" or "Shared themes: Time Travel, Dystopia" based on overlap with your taste profile.

## TMDB Auto-Sync

The app stays up to date with new releases through two sync mechanisms:

- On startup   The data pipeline runs a TMDB sync every time the containers start, pulling in now_playing, upcoming, and recently changed movies
- Background scheduler   The backend runs an incremental sync every 24 hours while running, fetching changes from the last 3 days plus current now_playing and upcoming lists
- Manual trigger   Hit POST /api/sync for an incremental sync or POST /api/sync?full=true for a broader sync. Check status with GET /api/sync/status

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | /api/auth/register | Create a new account |
| POST | /api/auth/login | Login and receive JWT token |
| GET | /api/movies/search?q= | Search movies by title |
| GET | /api/movies/popular | Get popular movies (paginated) |
| GET | /api/movies/{id} | Get movie details |
| GET | /api/watched | List user's watched movies |
| POST | /api/watched | Add a movie to watched |
| PUT | /api/watched/{movie_id} | Update rating/notes |
| DELETE | /api/watched/{movie_id} | Remove from watched |
| GET | /api/watchlist | List user's watchlist |
| POST | /api/watchlist | Add to watchlist |
| DELETE | /api/watchlist/{movie_id} | Remove from watchlist |
| GET | /api/recommendations | Get personalized recommendations |
| GET | /api/analytics/genres | Genre distribution data |
| GET | /api/analytics/timeline | Release decade timeline |
| GET | /api/analytics/revenue | Revenue bucket breakdown |
| GET | /api/analytics/ratings | User rating distribution |
| POST | /api/sync | Trigger TMDB sync |
| GET | /api/sync/status | Check last sync status |
| GET | /api/health | Health check |

## License

MIT
