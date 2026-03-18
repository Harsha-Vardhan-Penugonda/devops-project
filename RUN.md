# How to Run the Incident Management System (with AI)

## 1. Database migration (for AI and extra fields)

Run the migration so the app can store error/logs, impact, and AI analysis:

```bash
# Connect to your MySQL (incident_db) and run:
mysql -h <host> -P 4040 -u <user> -p incident_db < migrations/add_ai_analysis_columns.sql
```

Or run the `ALTER TABLE` statements from `migrations/add_ai_analysis_columns.sql` in your MySQL client.  
If you skip this, the app still works: new incidents use only service, severity, description, and no AI analysis is stored.

## 2. Gemini API key (for AI analysis)

**Why free tiers often “don’t work”:**
- **Groq:** Free but **blocks many regions** (403). If you’re not in US/EU etc., you get 403 even with a valid key.
- **Gemini/OpenAI:** Free tier has **quota limits**; once exceeded you see “quota” or “billing” until it resets or you add a new project/key.

**Use Hugging Face (free, works globally, no region block):**

- **No credit card.** ~300 requests/hour. Works in **all regions**.
- Get a token at **[https://huggingface.co/settings/tokens](https://huggingface.co/settings/tokens)** (sign up, then “Create new token” with **Read** access).

- **Windows (PowerShell):**
  ```powershell
  $env:GEMINI_API_KEY = "your-gemini-api-key"
  python app.py
  ```

- **Linux / macOS:**
  ```bash
  export GEMINI_API_KEY=your-gemini-api-key
  python app.py
  ```

If `GEMINI_API_KEY` is not set, incidents are still created but AI analysis will not be generated.

**EC2 or production server (key not in GitHub):**  
Set the key only on the server so the app can call Gemini:

1. **Option A – `.env` file on the server (recommended)**  
   In the app directory on EC2 (same folder as `app.py`), create a file named `.env` with one line:
   ```bash
   GEMINI_API_KEY=your-actual-gemini-api-key
   ```
   The app loads this file at startup (`.env` is in `.gitignore`, so it is never committed). Then restart the app (e.g. `sudo systemctl restart your-app-service` or restart gunicorn).

2. **Option B – Environment variable in systemd**  
   If you run the app with systemd, edit the service file and add:
   ```ini
   [Service]
   Environment="GEMINI_API_KEY=your-actual-gemini-api-key"
   ```
   Reload and restart: `sudo systemctl daemon-reload && sudo systemctl restart your-app-service`.

3. **Option C – Export before running**  
   If you start the app manually: `export GEMINI_API_KEY=your-actual-gemini-api-key` in the same shell before `python app.py` or `gunicorn ...`.

**Optional – Groq (free but region-restricted):** If you’re in a supported region, you can set `GEMINI_API_KEY` instead of or in addition to `HF_TOKEN`.

## 3. Install dependencies

```bash
cd incidents-management-main
pip install -r requirements.txt
```

## 4. Run the app

**Development (Flask):**
```bash
python app.py
```
Then open: **http://localhost:5000**

**Production (Gunicorn):**
```bash
gunicorn -b 0.0.0.0:5000 app:app
```

**With Docker:**
```bash
docker build -t incidents-app .
docker run -p 5000:5000 -e GEMINI_API_KEY=your-gemini-key incidents-app
```

**With Docker Compose:**  
Set `GEMINI_API_KEY` in the environment for the service (e.g. in `docker-compose.yml` or a `.env` file), then:
```bash
docker-compose up -d
```

## 5. What the AI integration does

- When you **create an incident** (service, severity, description, and optionally error/logs and impact), the app sends these details to **Google Gemini**.
- Gemini returns a **structured analysis**: summary, possible root causes, troubleshooting steps, suggested fixes, and preventive measures.
- That text is **saved** on the incident and **shown** on the incident’s **Manage** (edit) page under “AI Analysis & Recommendations”.

So: create incident → AI runs automatically → open the incident to see the analysis.
