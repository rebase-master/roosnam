# Roosnam - Portfolio Platform

A full-stack portfolio platform consisting of two separate repositories:

## Structure

- **roosnam-backend/** - Rails 8.1 API backend with SQLite
- **roosnam-frontend/** - Next.js frontend with ISR

## Getting Started

### Backend Setup

```bash
cd roosnam-backend
bundle install
rails db:create db:migrate
rails server
```

See [roosnam-backend/README.md](roosnam-backend/README.md) for detailed setup instructions.

### Frontend Setup

```bash
cd roosnam-frontend
npm install
npm run dev
```

See [roosnam-frontend/README.md](roosnam-frontend/README.md) for detailed setup instructions.

## Architecture

- **Two separate repositories** - Backend and frontend are independently deployable
- **Backend**: Rails 8.1 API with SQLite database, RailsAdmin interface, Devise authentication
- **Frontend**: Next.js with ISR for performance
- **Communication**: RESTful API, CORS-enabled

## Development Workflow

1. Start backend: `cd roosnam-backend && rails server`
2. Start frontend: `cd roosnam-frontend && npm run dev`
3. Access admin: `http://localhost:3000/admin`
4. Access frontend: `http://localhost:3001` (or configured port)

## License

Private project - All rights reserved

