# AI CareerPilot

AI CareerPilot is a career and interview preparation platform with a responsive product landing page, authentication, an interview simulator, resume analysis, personalized career roadmaps, an application tracker, and per-user career analytics backed by an Express API.

## Requirements

- Node.js 20.19+ or 22.12+
- npm 10+

## Getting started

```powershell
npm install
Copy-Item server/.env.example server/.env
Copy-Item client/.env.example client/.env
npm run dev
```

The web app runs at `http://localhost:5173`; the API runs at `http://localhost:4000`. Check `http://localhost:4000/api/health` for API status.

Useful commands: `npm run build`, `npm --workspace server test`, `npm run dev:web`, `npm run dev:api`, and `npm start`.

## Structure

```text
client/                 React, Vite, Tailwind CSS, and React Router
  src/components/       Shared navigation, footer, and feature UI
  src/pages/             Landing, auth, dashboard, interview, resume, roadmap, tracker, and analytics screens
  src/styles/            Global styles and design tokens
server/                 Express API
  src/config/            Environment and optional MongoDB connection
  src/controllers/       Request handlers
  src/middleware/        Not-found and centralized error handling
  src/routes/            Versioned API route modules
  src/services/          AI, document extraction, and data services
```

## Configuration and security

Copy `server/.env.example` to `server/.env`. Set `JWT_SECRET` to a random value with at least 32 bytes and set `CLIENT_URL` to the exact frontend origin. Keep `.env` out of version control. The server loads auth and database configuration from `server/.env`.

When `MONGODB_URI` is set, the API uses that MongoDB deployment. With an empty URI in development, it starts a temporary `mongodb-memory-server`; all accounts are lost when the API restarts. A MongoDB URI is required outside development.

## Authentication

- `POST /api/auth/signup` validates the name, email, and password, stores a bcrypt password hash, and starts a session.
- `POST /api/auth/login` verifies credentials and starts a session.
- `POST /api/auth/logout` clears the session cookie.
- `GET /api/auth/me` returns the current user when the session is valid.

Sessions use a signed JWT in an `httpOnly`, `sameSite=lax` cookie, with `secure` enabled in production. Login and signup are rate-limited. The React app sends requests with credentials and provides a protected dashboard.

## Interview simulator

The protected `/interview` page starts technical, behavioral, or HR practice sessions for entry, mid, or senior roles. Answers are limited to 5,000 characters and receive per-question feedback plus a final score. When `ANTHROPIC_API_KEY` and `ANTHROPIC_MODEL` are configured in `server/.env`, the server uses Anthropic. If the key is empty, it labels sessions as mock and uses sample questions and feedback. With the development memory database, interview history is cleared whenever the API restarts.

Run the authentication and interview API integration tests with `npm --workspace server test`.

## Resume analyzer

The protected `/resume` page accepts one PDF or DOCX file up to 5 MB, an optional target role, and an optional job description. Uploads use in-memory storage, are parsed for text, and are never written to disk. The extracted text is retained only in the user's analysis record and is excluded from API responses; users can delete their saved analysis.

- `POST /api/resumes/analyze` uploads and analyzes a resume.
- `GET /api/resumes` returns the current user's analysis history.
- `GET /api/resumes/:id` returns one owned analysis.
- `DELETE /api/resumes/:id` deletes one owned analysis.

The analyzer reuses `ANTHROPIC_API_KEY` and `ANTHROPIC_MODEL` from `server/.env`. With an empty API key, it labels results as mock and returns sample feedback. With an empty `MONGODB_URI` in development, saved analysis data is temporary and is lost when the API restarts.

## Roadmaps, applications, and analytics

- `POST /api/roadmaps/generate` creates a roadmap using Anthropic or a clearly labeled mock plan; roadmap history, milestone status updates, and deletion are scoped to the signed-in user.
- `/api/applications` supports authenticated create, list, detail, update, and delete operations. Application status is one of wishlist, applied, interview, offer, or rejected.
- `GET /api/analytics/summary` returns the current user's application counts and eight-week trend, completed interview average, resume score history, and average roadmap completion.

The protected `/roadmap`, `/tracker`, and `/analytics` pages use those APIs. Charts are rendered with inline SVG, so no chart package is required. `npm --workspace server test` covers authentication, interview, resume, roadmap, application, and analytics flows using the development memory database.