# UniPoll Score Smart

A full-stack web app to create, share and **automatically score** polls, quizzes and surveys,
with a live analytics dashboard, drop-off tracking and CSV / PDF export.

**Stack:** Python (Flask) + SQLite + Tailwind CSS + Chart.js. No database setup needed.

## Run it in VS Code (5 steps)

1. Install Python 3.9+ from python.org (tick **"Add Python to PATH"**).
2. In VS Code choose **File > Open Folder** and open the `unipoll-score-smart` folder.
3. Open the terminal (**Terminal > New Terminal**) and install the libraries:
   ```
   pip install -r requirements.txt
   ```
   (If `pip` is not found, use `python -m pip install -r requirements.txt`.)
4. Start the app:
   ```
   python app.py
   ```
5. Open **http://127.0.0.1:5000** in your browser.

You need an internet connection when using the app, because Tailwind CSS and Chart.js load from a CDN.

## Accounts

On the first run the app creates three accounts (Admin, a demo Teacher, a demo Student). **No password is built into
the code.** Set `ADMIN_PASSWORD` (see `.env.example`) before the first run, or copy the random passwords printed in
the terminal. Set a new password for any account later with:

```bash
python app.py set-password admin@unipoll.com
```

Only the Admin can open **AI Key Setup**.

Two demo polls are included: **Python Basics Quiz** (scored) and **Campus Facilities Survey** (unscored).
To see the analytics fill up, sign up a few extra respondent accounts and take the quiz, and leave one half-finished.

To reset everything, stop the app and delete the file `unipoll.db`.

## Features

- **Roles:** Admin, Creator, Respondent. Sign up / log in, and the admin can change roles or delete users.
- **Poll builder:** 3 steps (details, questions, review). Question types: multiple choice, open text, rating.
  Set the correct answer or a keyword rubric, points, explanations and a passing score. Edit polls later.
- **Respondent view:** one question at a time, live progress bar, answer validation, progress saved
  automatically so a person can resume later.
- **Automated scoring:**
  - Multiple choice and rating are matched against the correct answer.
  - Open text is scored by rubric keywords (partial marks), e.g. `store, value, name`.
  - Leave the correct answer empty to make a question unscored (surveys).
  - Instant feedback with the correct answer and your explanation.
- **Analytics (auto-refreshes every 5 seconds):** total / completed / partial submissions, average score,
  pass vs fail, score distribution, drop-off by question, per-question accuracy and option counts.
- **Export:** CSV (opens in Excel) and PDF report per poll.

## Answer Sheet Evaluator (AI)

Open **Answer Sheet Evaluator** in the sidebar (all roles). Paste a marking scheme and a written answer
(or upload up to 4 photos of handwritten pages), set the total marks, and press **Mark this answer**.
You get the score, a mark-by-mark breakdown, the points you missed, what worked, and what to fix next time.
Recent scripts are kept in the browser on that device only. It needs an AI key (see below).

## Connecting an AI key (any provider)

1. Log in as Admin or Creator and open **AI Key Setup** in the sidebar.
2. Paste a key and press **Save key**. The provider is detected from the key, and the connection is tested.
3. The key is saved in the database and stays until you replace or remove it.

Free options: **Google Gemini** (key from https://aistudio.google.com/apikey, starts with `AIza`) or
**Groq** (https://console.groq.com/keys, starts with `gsk_`). Also supported: OpenRouter, OpenAI, Claude, and
any OpenAI-compatible URL. Free tiers have limits and their terms change; check the provider's current rules.
If marking says the model was not found, type a current model name under *Advanced* on that page.

Open-text quiz questions use the same key for AI grading and fall back to keyword matching if the AI call fails.
You can also put `AI_API_KEY` in a `.env` file instead (see `.env.example`).

## Project structure

```
unipoll-score-smart/
  app.py               backend: routes, database, scoring engine, analytics, exports
  requirements.txt
  .env.example         optional API key
  templates/
    base.html          layout, sidebar / navbar, Tailwind setup
    login.html  signup.html  message.html  flash.html
    dashboard.html     overview (different for creators and respondents)
    polls.html         active polls / quizzes list, search, manage
    builder.html       3-step poll builder
    take.html          respondent view
    result.html        score + feedback
    analytics.html     charts, drop-off, export buttons
    users.html         admin user management
    evaluator.html     Answer Sheet Evaluator page
    api_setup.html     AI Key Setup page (save / test / remove the AI key)
```

## Database tables

- **settings:** key, value (stores the saved API key)
- **users:** id, name, email, password_hash, role (admin / creator / respondent), created_at
- **polls:** id, title, description, category, passing_score, created_by, is_active, created_at
- **questions:** id, poll_id, position, question_text, question_type, options (JSON), correct_answer (answer or rubric), explanation, points
- **submissions:** id, poll_id, respondent_id, answers (JSON), feedback (JSON), calculated_score, status (completed / partial), progress, started_at, submitted_at

## How drop-off tracking works (good for your viva)

When a respondent opens a poll, a `partial` submission is created. Every time they move to another
question the app saves their answers and the furthest question reached. When they submit, the same
record becomes `completed` and gets its score. Partial records that never finish show up in the
analytics as drop-offs at the question where the person stopped.

## Security notes

Passwords are hashed, correct answers are never sent to the respondent's browser, creators can only
see their own polls, and user-entered text is escaped in the pages and CSV export.
For real deployment, set a strong `SECRET_KEY` and run behind a production server (e.g. gunicorn).
