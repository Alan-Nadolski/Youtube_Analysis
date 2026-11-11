# YouTube Data Pipeline

An automated pipeline that collects data on new data-science videos on YouTube, stores it in a cloud database and reads the text on video thumbnails. The goal is to find out what makes videos in a niche perform well, and to give content creators data-backed advice.

**Status: work in progress.** The data collection is built and ran on its own every week from July to November 2025. It is paused for now. The analysis and modelling part has not been built yet (see [Next steps](#next-steps)).

---

## How it works

```
GitHub Actions (weekly schedule)
        │
        ▼
main.py ──► YouTube Data API ──► MySQL database on AWS RDS
        │                         (videos, channels, comments)
        ▼
  thumbnails/ ──► EasyOCR ──► thumbnail text saved to the database
```

1. **Search:** for each of seven search terms (*data science, data analysis, data engineering, data scientist, data analyst, data engineer, machine learning*), the script fetches the 50 most-viewed English videos published in the last 7 days.
2. **Collect:** for each video it gets the title, description, publish time, duration, views, likes, comment count, top 5 comments, and the channel's subscriber count and country.
3. **Store:** everything goes into a MySQL database hosted on AWS RDS. Existing channels and videos are skipped, so each run only adds what is new.
4. **Thumbnails:** downloads each thumbnail (with retries and a check for broken images), then reads any text on it with EasyOCR and stores the text, its position and the confidence score.
5. **Schedule:** a GitHub Actions workflow ran the whole job every Saturday and committed the new thumbnails back to the repo. About 4,700 thumbnails were collected.

## Database tables

| Table | What it holds |
| --- | --- |
| `channels` | channel name, subscriber count, country |
| `videos` | title, description, publish day and hour, duration, views, likes, comments, URLs, search term |
| `comments` | top comments per video |
| `thumbnail_text` | text found on each thumbnail, its position and OCR confidence |

## Tech stack

Python · YouTube Data API v3 · MySQL (AWS RDS) · GitHub Actions · EasyOCR · pandas · Pillow

## Run it yourself

1. Install the dependencies: `pip install -r requirements.txt`
2. Create a `.env` file with your database login and API key:
   ```
   host=...
   user=...
   password=...
   database=...
   API_KEY=...
   ```
3. Create the four tables above in your MySQL database.
4. Run `python main.py`. To schedule it, add the same values as repository secrets (`DB_HOST`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`, `API_KEY`) and enable the workflow in `.github/workflows/weekly-run.yml`.

## Next steps

- [ ] Explore the collected data: what separates top-performing videos (length, publish time, title words, thumbnail text)?
- [ ] Topic modelling on titles and descriptions
- [ ] A simple model that predicts views from video features
- [ ] Dashboard with the findings

## Known issues

- Comments can be stored more than once, because the insert step re-runs for every search term.
- Each video uses several separate API calls, which uses up the daily YouTube API quota quickly. Batching the calls would fix this.
- Thumbnails are stored in the repository itself, which makes it large. They should move to cloud storage.
- `functions.py` uses `time.sleep` without importing `time`, so a failed download crashes instead of retrying.

---

Made by [Alan Nadolski](https://github.com/Alan-Nadolski), Data Science student at ITU.
