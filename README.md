# Teebay Deployment Repository

This repository orchestrates the full Teebay stack via Docker Compose, grouping frontend and backend as Git submodules:

- **db**: PostgreSQL 15
- **api**: Spring Boot backend (Maven → JAR)
- **frontend**: React + Vite UI (TypeScript → Nginx)

Each service lives in its own Git repo; this repo holds:
- `docker-compose.yml` for building & running all services  
- `.gitmodules` configured to track `master` on each submodule  
- This `README.md` with usage instructions

---

## Prerequisites

- Git ≥ 2.22  
- Docker & Docker Compose  

---

## Cloning with Submodules

Clone the parent repo along with frontend and backend:

```bash
git clone --recursive https://github.com/ihashib/deployment-repo.git
cd deployment-repo
```

If already cloned without submodules:
```bash
git submodule update --init --recursive
```

---

## Running the Stack

From the repo root, build and start all services:

```bash
docker-compose up --build
```

- **Postgres** data persists in a Docker volume (`db-data`).  
- **API** is available at `http://localhost:8080`.  
- **UI** is served at `http://localhost:5173` (Nginx on container port 80).  

To stop the stack:
```bash
docker-compose down
```

