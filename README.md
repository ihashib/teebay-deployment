# Teebay Deployment Repository

This repository orchestrates the full Teebay stack via Docker Compose, grouping your frontend and backend as Git submodules:

- **db**: PostgreSQL 15  
- **api**: Spring Boot backend (Maven)  
- **frontend**: React + Vite UI (TypeScript)  

Each service lives in its own Git repo; this repo holds:
- `docker-compose.yml` for building & running all services  
- `bootstrap.sh` helper for new clones  
- `.gitmodules` configured to track `master` on each submodule  

## Cloning with Submodules

```bash
# 1. Clone the parent repo and all submodules
git clone --recursive https://github.com/ihashib/deployment-repo.git
cd deployment-repo

# 2. If you already cloned without --recursive:
git submodule update --init --recursive