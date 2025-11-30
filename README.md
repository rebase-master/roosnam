# Roosnam — Portfolio Platform

> A full-stack, single-user portfolio platform built with Rails 8.1 and Next.js

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub release](https://img.shields.io/github/v/release/rebase-master/roosnam?include_prereleases)](https://github.com/rebase-master/roosnam/releases)
[![Docker Ready](https://img.shields.io/badge/Docker-Ready-blue.svg?logo=docker)](https://docs.docker.com/compose/)
[![Rails](https://img.shields.io/badge/Rails-8.1-red.svg?logo=rubyonrails)](https://rubyonrails.org/)
[![Next.js](https://img.shields.io/badge/Next.js-14-black.svg?logo=next.js)](https://nextjs.org/)

---

Roosnam is a full-stack, single-user portfolio platform built with Rails 8.1 (API backend) and Next.js (frontend). This README is for the parent repository and includes local + Docker usage, plus explicit steps to create the single admin/portfolio user via the Rails console running inside Docker.

## Architecture overview

- **Backend**: Ruby on Rails 8.1 (API), Devise authentication, RailsAdmin for admin UI
- **Frontend**: Next.js with ISR, consumes the Rails JSON API
- **Database**: SQLite (development by default) 
- **Auth model**: single-user (singleton) mode enforced at the model level — only one User allowed and that user is automatically admin

--- 
  
## Repository structure

```
roosnam/
├── roosnam-backend/ # Rails API backend (contains Dockerfile and docker-compose.yml)
└── roosnam-frontend/ # Next.js frontend (contains Dockerfile and docker-compose.yml)
```
Each sub-folder has its own `Dockerfile` and `docker-compose.yml`. A root-level `docker-compose.yml` (in this parent repo) can be used to run both services together.

--- 


## Prerequisites

Docker and Docker Compose (v1.27+ or Compose v2)

(Optional for local dev) Ruby, Bundler, Node.js, npm/yarn

--- 

## How to run

Place the root docker-compose.yml in the repo root (if not already present) and then from the parent repo root run:

`docker-compose up --build`

**What this does**

- Builds and starts both services: backend (Rails API) and frontend (Next.js)
- Backend will be available at http://localhost:3000
(mapped from container port 80)
- Frontend will be available at http://localhost:3001

**Notes**

- The root compose file defines the backend service name as backend. Use that service name when interacting with the running backend container via docker-compose exec commands.
- `NEXT_PUBLIC_API_URL` defaults to the internal Docker network URL (http://backend:80/api/v1
). You can override via environment variables or a `.env` file.

--- 

### Useful Docker commands

| Action                                | Command                                                                                 |
| ------------------------------------- | --------------------------------------------------------------------------------------- |
| Start both services (build if needed) | `docker-compose up --build`                                                             |
| Start in detached mode                | `docker-compose up -d --build`                                                          |
| Tail logs (both services)             | `docker-compose logs -f`                                                                |
| Tail logs (backend only)              | `docker-compose logs -f backend`                                                        |
| Run migrations/seeds                  | `docker-compose exec backend bash -lc "bundle exec rails db:create db:migrate db:seed"` |
| Open shell in backend                 | `docker-compose exec backend bash`                                                      |
| Run one-off backend command           | `docker-compose exec backend <command>`                                                 |
| Stop and remove containers            | `docker-compose down`                                                                   |

---

## Running Rails console via Docker

To open the Rails console from the parent repo (no need to cd into roosnam-backend), run the one-liner:

`docker-compose exec backend bundle exec rails console`

Alternative: open a shell and then run console
docker-compose exec backend bash
bundle exec rails console

**Why this works**

- The root docker-compose.yml defines the service as backend. docker-compose exec backend runs the command inside the running backend container. The backend container is set up with the app environment and volumes so the console will operate on the same codebase and database used by the running app.

---

## Steps to create the admin / portfolio user via Rails console (Docker)

Because Roosnam is a single-user app, you should create exactly one User and that user will be the admin.

1. Start the Rails console in the backend container:

    `docker-compose exec backend bundle exec rails console`
2. In the Rails console, create the user (example):
   
    ```
    user = User.create!(email: "you@example.com", password: "your_secure_password", password_confirmation: "your_secure_password")
   ```
   
3. Verify the user exists and has admin privileges:
    ```
    User.count
    user.persisted?
    ```

4. Exit the console:
   
   `exit`

--- 

### Admin interface

The admin UI is typically at http://localhost:3000/admin
when running the backend container. Your single user will have admin access automatically (singleton enforced at model level).

--- 

## Local (non-docker) quickstart (if you prefer to run services without Docker)

**Backend**
```
cd roosnam-backend
bundle install
rails db:create db:migrate
rails server -b 0.0.0.0 -p 3000
```

**Frontend**
```
cd roosnam-frontend
npm install
npm run dev # or yarn dev
```

Set `NEXT_PUBLIC_API_URL` in roosnam-frontend/.env.local to http://localhost:3000
(or your API URL)

**Environment variables (examples)**

Backend environment examples
```
RAILS_ENV=development
RAILS_MASTER_KEY=...
FRONTEND_URL=http://localhost:3001

RAILS_HOST=localhost
RAILS_PORT=3000
CORS_ORIGINS=*
```

Frontend environment examples (.env.local)
```
NEXT_PUBLIC_API_URL=http://localhost:3000/api/v1
NEXT_PUBLIC_SITE_URL=http://localhost:3001
NEXT_PUBLIC_SHOW_MOCK_DATA=true
```

Place actual secrets (RAILS_MASTER_KEY, SECRET_KEY_BASE, ADMIN_PASSWORD if used by seeds) in a `.env` file or your deployment secrets store and do not commit them.

---

## Mock Data Mode

The frontend includes a **mock data mode** that allows you to preview the portfolio with sample data without needing the backend API running. This is useful for:

- **Demo purposes**: Showcase the portfolio design without setting up the full backend
- **Frontend development**: Work on UI components without backend dependencies
- **Quick previews**: Test the site layout before populating real data

### How it works

The `NEXT_PUBLIC_SHOW_MOCK_DATA` environment variable controls data sourcing:

| Value | Behavior |
|-------|----------|
| `true` (default) | Uses mock data from `lib/mockData.js`. No API calls are made. |
| `false` | Fetches all data from the Rails API. Requires backend to be running. |

### Setting the flag

**Via `.env.local` (recommended for local development):**
```bash
# In roosnam-frontend/.env.local
NEXT_PUBLIC_SHOW_MOCK_DATA=true   # Use mock data
NEXT_PUBLIC_SHOW_MOCK_DATA=false  # Use real API data
```

**Via Docker Compose:**

The `roosnam-frontend/docker-compose.yml` includes this setting:
```yaml
environment:
  NEXT_PUBLIC_SHOW_MOCK_DATA: ${NEXT_PUBLIC_SHOW_MOCK_DATA:-false}
```

Override via command line:
```bash
NEXT_PUBLIC_SHOW_MOCK_DATA=true docker-compose up
```

### What's included in mock data

When mock data mode is enabled, the frontend displays sample data for:
- Profile information (name, bio, headline, location, social links)
- Work experiences (7 positions at real company names)
- Skills (categorized: Languages, Frameworks, Databases, Cloud & DevOps, ML/AI, etc.)
- Client projects (12 sample projects with descriptions and tech stacks)
- Client testimonials/reviews
- Education and certifications
- Blog posts (sample articles)

### Important notes

- When `NEXT_PUBLIC_SHOW_MOCK_DATA=false`, **all data comes from the API**. If the API returns empty results, the frontend shows appropriate empty states.
- The mock data is defined in `roosnam-frontend/lib/mockData.js` and can be customized.
- Profile photos only display if the API returns a `profile_photo_url` or if mock data mode is enabled (uses a default image).

--- 

### Tips and caveats

- The root-level `docker-compose up --build` relies on the subrepos' Dockerfiles and the root compose mapping to ./roosnam-backend and ./roosnam-frontend. Ensure those folders exist and contain the Dockerfiles.
- The backend container maps host port `3000` to container port `80` (the Rails command in the backend container runs on port `80`). The frontend maps host port `3001` to container port `3001`.
- The service name backend in the root-level compose is the identifier used by docker-compose exec backend ... commands. If you run docker-compose from inside roosnam-backend (using the subrepo compose) the service name is web, so the equivalent commands there use web (for example docker-compose exec web bundle exec rails console). Use the one matching the compose file you run.
- The app enforces single-user mode at the model level. Do not create multiple user records unless you intentionally modify the model to allow multi-user support.
- If you prefer a server-grade DB for production, replace SQLite with PostgreSQL and update the backend configuration plus docker-compose accordingly.

--- 

### Troubleshooting

- If docker-compose exec backend bundle exec rails console fails with "No such service", confirm you are in the parent repo where the root docker-compose.yml defines the backend service name as backend. If you invoked docker-compose inside roosnam-backend subfolder, use the service name web instead.
- If migrations are not applied, run:

   `docker-compose exec backend bash -lc "bundle exec rails db:migrate"`
- If the app cannot connect to DB or files are not persisted, check the named Docker volume (roosnam-backend-storage) and permissions.

--- 

## LICENSE

### MIT License

Copyright (c) 2025 Mansoor Khan

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights  
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell  
copies of the Software, and to permit persons to whom the Software is  
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in  
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR  
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,  
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE  
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER  
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,  
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN  
THE SOFTWARE.


------------------------------------------------------------
Preferred Citation
------------------------------------------------------------

If you use this software in your work, research, or derivative projects,  
please cite it as follows:

Mansoor Khan (2025). *Roosnam — Portfolio Platform*.  
GitHub repository: [https://github.com/rebase-master/roosnam](https://github.com/rebase-master/roosnam)
