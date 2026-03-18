# Enable AI Analysis on EC2 – Step-by-Step

Use this guide in two parts: **you** do the steps in "Part 1 (You)", then **your friend** (who has EC2 access) does "Part 2 (EC2)".

---

## Part 1 – What you do (on your laptop)

1. **Commit and push the latest code**  
   So the EC2 server can pull the version that reads the API key from a `.env` file.
   - In your project folder (where `app.py` is), run:
     ```bash
     git add .
     git commit -m "Add .env support for GEMINI_API_KEY on EC2"
     git push origin main
     ```
     (Use your branch name if it’s not `main`, e.g. `master`.)

2. **Get your Gemini API key**  
   - Go to https://aistudio.google.com/apikey  
   - Create or copy your API key.

3. **Share the key with your friend securely**  
   - Send it in a **private** way (e.g. WhatsApp, Signal, or a one-time link), not in public chat or email subject.  
   - Tell her: “This is the Gemini API key. Use it only in the `.env` file on EC2 and don’t put it in any file that gets committed to GitHub.”

4. **Share this file**  
   - Send her **Part 2** below (or this whole file). She only needs to follow Part 2.

---

## Part 2 – Steps for your friend (on EC2)

Do these on the EC2 instance (SSH into it first). Replace `YOUR_ACTUAL_GEMINI_KEY` with the key you received.

### Step 1 – Go to the app folder

```bash
cd /home/ubuntu/incidents-management-main
```

If the app lives somewhere else (e.g. `/var/www/incidents` or inside another folder), change the path. The goal is to be in the **same directory as `app.py`**.

To check you’re in the right place:

```bash
ls -la app.py
```

You should see `app.py` listed.

---

### Step 2 – Pull the latest code (so .env support is there)

```bash
git pull origin main
```

(If the branch is `master`, use `git pull origin master`.)

---

### Step 3 – Install the new dependency

```bash
pip install python-dotenv
```

If the app runs in a virtual environment (e.g. `venv` or `.venv`), activate it first, then run the same command:

```bash
source venv/bin/activate
# or:  source .venv/bin/activate
pip install python-dotenv
```

---

### Step 4 – Create the `.env` file with the API key

**Option A – Using echo (one line, replace the key):**

```bash
echo 'GEMINI_API_KEY=YOUR_ACTUAL_GEMINI_KEY' > .env
```

**Option B – Using an editor (safer if the key has special characters):**

```bash
nano .env
```

In the editor, type exactly (with the real key, no quotes):

```
GEMINI_API_KEY=YOUR_ACTUAL_GEMINI_KEY
```

Save: `Ctrl+O`, Enter, then exit: `Ctrl+X`.

**Check the file (don’t share this output – it shows the key):**

```bash
cat .env
```

You should see one line: `GEMINI_API_KEY=...`

---

### Step 5 – Restart the app

**If the app runs as a systemd service (e.g. “incidents” or “flask”):**

```bash
sudo systemctl restart incidents
```

(Replace `incidents` with the real service name. To list services: `ls /etc/systemd/system/*.service` or ask your teammate.)

**If you start the app manually with gunicorn:**

- Stop the current process (e.g. `Ctrl+C` in the terminal where it’s running, or find the process and kill it).
- Start it again from the app directory:
  ```bash
  gunicorn -b 0.0.0.0:5000 app:app
  ```

**If you use a different command or script to run the app:**  
Stop it, then start it again from the same directory where `app.py` and `.env` are.

---

### Step 6 – Test

1. Open the incident management website in a browser.
2. **Create a new incident** (fill service, severity, description, and optionally error/logs and impact).
3. Open that incident’s **Manage / Edit** page.
4. You should see an **“AI Analysis & Recommendations”** section with generated text.

If it’s still empty, check the app logs (e.g. `sudo journalctl -u incidents -n 50` for systemd, or the terminal where gunicorn is running) for messages like `[AI] No API key` or `[AI] Gemini error`.

---

## Quick checklist

| Who   | Step |
|-------|------|
| You   | 1. Push latest code (`git push`). |
| You   | 2. Get Gemini key from https://aistudio.google.com/apikey |
| You   | 3. Send key to friend privately and send her Part 2 (EC2 steps). |
| Friend| 4. SSH to EC2, `cd` to app folder, `git pull`. |
| Friend| 5. `pip install python-dotenv`. |
| Friend| 6. Create `.env` with `GEMINI_API_KEY=...`. |
| Friend| 7. Restart the app. |
| Friend| 8. Create a new incident and check AI analysis on its edit page. |
