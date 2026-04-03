# WatchNext

<p align="center">
  <img src="https://img.shields.io/badge/Frontend-Next.js-black?style=for-the-badge&logo=next.js" alt="Next.js" />
  <img src="https://img.shields.io/badge/Backend-FastAPI-009688?style=for-the-badge&logo=fastapi" alt="FastAPI" />
  <img src="https://img.shields.io/badge/Styling-Tailwind_CSS-38B2AC?style=for-the-badge&logo=tailwind-css" alt="Tailwind CSS" />
  <img src="https://img.shields.io/badge/Containerized-Docker-2496ED?style=for-the-badge&logo=docker" alt="Docker" />
  <img src="https://img.shields.io/badge/Data-TMDB_CSV-orange?style=for-the-badge" alt="Dataset" />
</p>

<p align="center">
  A full-stack movie recommendation and analytics app where users can track watched movies, discover personalized recommendations, and explore movie insights in a clean dashboard experience.
</p>

---


## What WatchNext Does

WatchNext gives users a simple way to build their own movie activity space. After signing up, users can keep track of what they have watched, explore recommendations, and view analytics around their movie habits.

The project is split into three parts:

- a **Next.js frontend**
- a **FastAPI backend**
- a **data pipeline** that prepares recommendation data from a movie dataset

---

## Features

- User registration and login
- Track watched movies
- Personalized movie recommendations
- Movie analytics dashboard
- Dockerized local development setup
- Separate frontend, backend, and data pipeline services

---

## Tech Stack

### Frontend
- Next.js
- TypeScript
- Tailwind CSS

### Backend
- FastAPI
- Python

### Data / Storage
- TMDB CSV dataset
- Feature generation pipeline
- SQLite / local app data

### Dev Tools
- Docker
- Docker Compose
- Git

---

## Project Structure

```bash
WatchNext/
├── backend/
├── frontend/
├── docker-compose.yml
├── TMDB_movie_dataset_v11.csv
├── README.md
```

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/WatchNext.git
cd WatchNext
```

### 2. Add the dataset

Place the CSV file in the **root of the project**, in the same folder as `docker-compose.yml`.

Expected file name:

```bash
TMDB_movie_dataset_v11.csv
```

Correct structure:

```bash
WatchNext/
├── docker-compose.yml
├── TMDB_movie_dataset_v11.csv
├── backend/
└── frontend/
```

---

## Run the App

From the project root, run:

```bash
docker compose down -v
docker compose build --no-cache
docker compose up
```

---

## Local URLs

After startup:

- **Frontend:** `http://localhost:3000`
- **Backend:** `http://localhost:8000`
- **Health Check:** `http://localhost:8000/api/health`

---

## How the Services Work

### `frontend`
The user-facing web app built with Next.js.

### `backend`
The FastAPI server that handles authentication, business logic, and API endpoints.

### `data-pipeline`
A one-time setup container that processes the movie dataset and prepares files needed by the backend.

> The pipeline is expected to finish its work and exit successfully. It does not stay running forever.

---

## Common Issues

### Backend and pipeline are not running

If the frontend works but signup fails, the backend is probably not up yet.

Run:

```bash
docker compose ps
```

The backend depends on the pipeline finishing successfully first.

---

### Shell syntax error in `init_data.sh`

If you see:

```bash
Syntax error: end of file unexpected (expecting "then")
```

Open this file:

```bash
backend/scripts/init_data.sh
```

Then in VS Code:
- look at the bottom-right corner
- if it says `CRLF`, change it to `LF`
- save the file

Then rebuild:

```bash
docker compose down -v
docker compose build --no-cache
docker compose up
```

---

### Registration failed on the website

That usually means the frontend could not reach the backend.

Check the backend directly:

```bash
http://localhost:8000/api/health
```

If it does not load:
- make sure the CSV file is in the project root
- make sure `init_data.sh` uses `LF`, not `CRLF`
- rebuild the containers cleanly

---

### CSV file not found

If your dataset is inside another folder like `app/`, Docker will not find it unless you change the compose mount path.

Simplest fix: move the file to the project root.

---

## Useful Commands

### Show container status

```bash
docker compose ps
```

### View logs

```bash
docker compose logs data-pipeline --tail=100
docker compose logs backend --tail=100
docker compose logs frontend --tail=100
```

### Stop containers

```bash
docker compose down
```

### Stop and remove volumes

```bash
docker compose down -v
```

---

## Branding Notes

If you renamed the project from **CineMatch** to **WatchNext**, these are the main places to update:

```bash
frontend/src/app/page.tsx
frontend/src/components/Navbar.tsx
frontend/src/app/layout.tsx
backend/main.py
README.md
```

---

## Roadmap

- Add watchlist support
- Add ratings and reviews
- Improve recommendation quality
- Add better filtering and search
- Expand analytics and visualizations
- Deploy to a cloud platform

---

## Contributing

Contributions are welcome. Fork the repo, make your changes, and open a pull request.

---

## License

This project is intended for educational and personal development use unless stated otherwise.
