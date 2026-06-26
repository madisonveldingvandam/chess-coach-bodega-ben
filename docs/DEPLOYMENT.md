# Deployment

## GitHub Pages

The public Bodega Ben site is designed to work like the original
`chess-tracker` site: GitHub Actions refreshes public Chess.com data, builds a
static dashboard, and deploys the finished frontend to GitHub Pages.

Expected URL:

https://madisonveldingvandam.github.io/chess-coach-bodega-ben/

Workflow:

- `.github/workflows/pages.yml`
- Runs on push to `main`, every six hours, and manual dispatch.
- Generates `frontend/public/data/default-dashboard.json` during the workflow.
- Builds Vite with `VITE_BASE_PATH=/chess-coach-bodega-ben/`.
- Uploads `frontend/dist` to GitHub Pages.

The static Pages site is Bodega Ben-specific. It can show the live-handle form,
but arbitrary handle analysis still requires the FastAPI backend deployment
below.

## Render

This repo is also configured for a Docker-based Render web service when live
arbitrary-handle analysis is needed.

Use this Blueprint link:

https://render.com/deploy?repo=https://github.com/madisonveldingvandam/chess-coach-bodega-ben

Expected service settings:

- Name: `chess-coach-bodega-ben`
- Runtime: Docker
- Plan: Free
- Branch: `main`
- Health check path: `/api/health`
- Runtime data directory: `/tmp/chess-coach-data`

The free Render plan uses an ephemeral filesystem, so cached Chess.com archives
and generated results can disappear when the instance restarts or redeploys.
That is acceptable for the first public URL because the app can refetch public
Chess.com data. For durable caching later, upgrade the service and mount a
persistent disk, then set `CHESS_COACH_DATA_DIR` to that disk path.

Render provides the runtime `PORT` environment variable for web services. The
Docker command binds Uvicorn to `0.0.0.0` and `${PORT:-10000}`.
