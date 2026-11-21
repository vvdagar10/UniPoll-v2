# UniPoll Score Smart — deployment notes

## Deploy free on Render (Mac or Windows)

1. Extract this ZIP.
2. Open the extracted `unipoll-score-smart` folder in VS Code.
3. Create a GitHub repository and upload the **contents** of this folder (not the ZIP).
   Keep `app.py`, `requirements.txt`, `render.yaml`, and `templates/` at the repository root.
4. In Render, choose **New + → Blueprint**, connect the GitHub repository, and apply the
   `render.yaml` configuration. Alternatively, choose **New + → Web Service** and use:
   - Runtime: Python
   - Build command: `pip install -r requirements.txt`
   - Start command: `gunicorn app:app`
   - Instance type: Free
5. When Render asks for **ADMIN_PASSWORD**, type a strong password. This is your admin login (email
   `admin@unipoll.com` unless you also set `ADMIN_EMAIL`). Optionally also enter `AI_API_KEY` (your Groq or Gemini key)
   so AI marking keeps working after Render restarts. Render generates `SECRET_KEY` itself. Do not publish your
   `.env` file or API keys.

## Important limitations

- This is a Flask app backed by SQLite. Render's free web-service filesystem is ephemeral:
  the database may be reset when the service restarts, redeploys, or is moved. This free setup
  is suitable for a college demonstration, not reliable long-term storage. For persistent
  accounts, polls, and submissions, use a managed persistent database such as PostgreSQL and
  update the application's database layer.
- The free Render service may spin down when idle, so the first visit after inactivity can take
  longer.
- The app creates an Admin account (and demo Teacher/Student accounts) whenever the database is empty, which on
  Render's free tier happens after every restart. **No password is stored in the code or shown on the login page.**
  The Admin password is the `ADMIN_PASSWORD` value you type into Render (see below). Anyone can sign up as a
  respondent or creator, so do not put real personal data or secrets in the site.
- An AI provider key is optional. If you configure one, add it in Render's Environment settings
  as `AI_API_KEY` (and optionally `AI_PROVIDER` / `AI_MODEL`) or use the app's AI Key Setup page.
  Never commit `.env` or API keys to GitHub.

## Run locally on macOS

In VS Code Terminal, from this folder:

```bash
python3 -m venv .venv
source .venv/bin/activate
python3 -m pip install -r requirements.txt
python3 app.py
```

Then visit http://127.0.0.1:5000.
