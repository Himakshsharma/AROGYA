# Deploying Arogya

The whole product (React app + API + WebSockets) runs as **one Docker container**, so it deploys to any host that runs Docker.

## Recommended: Hugging Face Spaces (free, 16 GB RAM, auto-deploy from GitHub)

1. Create a free account at https://huggingface.co.
2. **New Space** → name `arogya` → SDK **Docker** → template **Blank** → hardware **CPU basic (free)** → Create.
3. **Settings → Access Tokens** → create a token with **Write** permission.
4. In the GitHub repository → **Settings → Secrets and variables → Actions**:
   - New **secret** `HF_TOKEN` = the token from step 3
   - New **variable** `HF_SPACE` = `your-hf-username/arogya`
5. In the Space → **Settings → Variables and secrets** → add secret `AROGYA_SECRET_KEY` = any long random string.
6. Push to `main`. CI runs the tests; when they pass, **Deploy to Hugging Face** pushes the code and the Space builds the Docker image (about 8–10 minutes the first time).
7. Open `https://your-hf-username-arogya.hf.space`.

The full `Dockerfile` installs Whisper, gTTS and MarianMT; models download on first use.

**Data:** the free Space disk resets when the Space restarts or sleeps (after 48 h without visitors). That is fine for a demo. For lasting data, enable persistent storage in the Space settings and set `AROGYA_DB_PATH=/data/arogya.db`.

## Alternative: Render (free, 512 MB RAM)

Uses `Dockerfile.lite` (no server AI; the browser handles speech).

1. https://render.com → **New → Blueprint** → connect the GitHub repository.
2. Render reads `render.yaml`, creates the service and generates `AROGYA_SECRET_KEY`.
3. Free services sleep after 15 minutes idle; the first request then takes about a minute.

## Alternative: frontend on Vercel, API elsewhere

1. Deploy the API with either option above.
2. Vercel → **Add New Project** → import the repository → **Root Directory** `frontend`.
3. Environment variable `VITE_API_URL` = the API URL (for example `https://your-hf-username-arogya.hf.space`).
4. On the API, set `AROGYA_CORS_ORIGINS` to the Vercel URL.
5. `frontend/vercel.json` (already in the repo) sends every route to `index.html`, so links like `/rx/abc` work.

## Environment variables

| Variable | Default | Meaning |
|---|---|---|
| `AROGYA_SECRET_KEY` | dev value | **Set this in production.** Signs login tokens |
| `AROGYA_DB_PATH` | `backend/arogya.db` | SQLite file |
| `AROGYA_CORS_ORIGINS` | `*` | Comma-separated allowed origins |
| `AROGYA_ENABLE_WHISPER` | off | Server speech recognition |
| `AROGYA_WHISPER_MODEL` | `base` | `tiny`, `base`, `small` … |
| `AROGYA_ENABLE_SERVER_TTS` | off | Server voice (gTTS) |
| `AROGYA_TRANSLATION_ENGINE` | `auto` | `phrasebook`, `marian`, `indictrans2`, `auto` |
| `AROGYA_SEED_DEMO` | on | Create the demo doctor account |
| `PORT` | 7860 / 8000 | Port to listen on |

## Camera and microphone need HTTPS

Browsers only allow the camera and microphone on `https://` or `localhost`. All the hosts above give HTTPS automatically.
