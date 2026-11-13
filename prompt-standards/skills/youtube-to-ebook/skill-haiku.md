---
name: youtube-to-ebook
description: Set up, run, schedule and customize a pipeline that turns the latest long-form video from each chosen YouTube channel into a magazine-style article written by Claude, packages the articles as an EPUB ebook, and emails it through Gmail.
---

# YouTube to Ebook

Turn the latest videos from the user's favorite YouTube channels into well-written magazine-style articles, delivered as an EPUB ebook by email.

The scripts and support files are in this skill's directory. The user runs them from their own project folder.

## What This Skill Does

`python3 main.py` runs these steps in order. If a step produces nothing, it prints a message and stops.

1. **Fetch videos** (`get_videos.py`). For each handle in the `CHANNELS` list, read the 15 newest items in the channel's uploads playlist and keep the newest one that is not a Short. One video per channel.
2. **Skip repeats** (`video_tracker.py`). Drop videos whose IDs are already in `processed_videos.json`.
3. **Get transcripts** (`get_transcripts.py`). Call the Supadata API with `SUPADATA_API_KEY`, one second apart. Videos with no transcript are skipped.
4. **Write articles** (`write_articles.py`). Send each transcript, with the video title and description, to Claude (`claude-sonnet-4-20250514`, `max_tokens=8000`).
5. **Build and send** (`send_email.py`). Create `youtube_digest_YYYYMMDD.epub` and email it, with an HTML copy of the articles, from `GMAIL_ADDRESS` to the same address through Gmail SMTP.
6. **Archive and mark** (only if the email was sent). Save `newsletters/newsletter_<timestamp>.html`, `.epub` and `.json`, delete the EPUB from the project folder, and add the videos to `processed_videos.json`.

If the email fails, nothing is archived or marked. The EPUB stays in the project folder, and the next run processes the same videos again, with new Claude and Supadata calls.

## Requirements

- Python 3 and `pip install -r requirements.txt`.
- For the dashboard only: `pip install streamlit` (it is not in `requirements.txt`).
- A `.env` file in the project folder:

| Variable | Used by | Where to get it |
|----------|---------|-----------------|
| `YOUTUBE_API_KEY` | `get_videos.py` | Google Cloud Console: enable YouTube Data API v3, then create an API key |
| `SUPADATA_API_KEY` | `get_transcripts.py` | Supadata (supadata.ai). Not in `.env.example`, so add the line |
| `ANTHROPIC_API_KEY` | `write_articles.py` | console.anthropic.com |
| `GMAIL_ADDRESS` | `send_email.py` | The Gmail address that sends and receives the ebook |
| `GMAIL_APP_PASSWORD` | `send_email.py` | An app password from myaccount.google.com/apppasswords |

The Gmail settings are needed for a complete run (see step 6 above).

## Quick Start

When the user asks "Set up YouTube to ebook for me":

1. Create a project folder (for example `~/youtube-newsletter`) and copy every file from this skill's directory into it.
2. Run `pip install -r requirements.txt` in that folder.
3. Run `cp .env.example .env`, add a `SUPADATA_API_KEY=` line, and have the user fill in all five values.
4. Set the channels: edit the `CHANNELS` list in `get_videos.py`. It ships with 8 example channels. Use `@handles` from the channel URL (`youtube.com/@name` → `"@name"`).
5. Check the channels: run `python3 get_videos.py`. It prints each channel and the video it found, or "Channel not found".
6. Generate the first ebook: run `python3 main.py`, after the user agrees (see Working Rules).
7. Optional: the dashboard and the weekly schedule (sections below).

## Commands

| Command | Description |
|---------|-------------|
| `python3 main.py` | Full pipeline: fetch, transcribe, write, build EPUB, email, archive |
| `python3 -m streamlit run dashboard.py` | Web dashboard (needs streamlit) |
| `python3 get_videos.py` | Only list the latest long-form video per channel |
| `python3 get_transcripts.py` | Test the Supadata key on one sample video |
| `python3 video_tracker.py` | List the videos already processed |
| `python3 send_email.py` | Send a test newsletter with one mock article. This sends a real email and adds an archive entry |

There is no `--channels` flag or other command-line option. `python3 write_articles.py` (its self-test) stops with `KeyError: 'description'` because its mock video has no description. Test article writing through `main.py` instead.

## Key Files

```
youtube-newsletter/
├── main.py                 # Run the full pipeline
├── get_videos.py           # Fetch latest videos; CHANNELS list lives here
├── get_transcripts.py      # Transcripts via Supadata API
├── write_articles.py       # Articles via Claude; article prompt lives here
├── send_email.py           # Create EPUB, send email, archive
├── video_tracker.py        # Track processed videos
├── dashboard.py            # Streamlit web dashboard
├── requirements.txt
├── .env.example            # Copy to .env
├── run_newsletter.sh       # Called by launchd on schedule
├── com.youtube.newsletter.plist            # Weekly schedule (launchd)
├── com.youtube.newsletter.dashboard.plist  # Dashboard at login (launchd)
├── launch_dashboard.command                # Double-click to open dashboard
├── processed_videos.json   # Created on first successful send
├── newsletters/            # Archive of sent newsletters
└── logs/                   # Create this before scheduling
```

## Dashboard

`python3 -m streamlit run dashboard.py` opens these pages:

- **Generate**: runs `python3 main.py` (10-minute timeout) and shows the log.
- **Channels**: add or remove channels. It rewrites the `CHANNELS` list in `get_videos.py`.
- **Writing Style**: edit the article prompt in `write_articles.py`.
- **Archive**: download past newsletters (HTML, EPUB) and see processed videos.
- **Schedule**: change the day and hour in `~/Library/LaunchAgents/com.youtube.newsletter.plist` and reload it with `launchctl`.

Limits of the dashboard code:
- **Clear All History** only shows a warning. It deletes nothing. To process videos again, delete `processed_videos.json` or remove entries from it.
- It reads handles with the pattern `@[\w]+`. A handle with `.` or `-` is not shown, and is dropped from `get_videos.py` the next time the dashboard saves the list. Edit such handles in `get_videos.py` directly.
- A `youtube.com/channel/UC...` URL becomes `@UC...`, which the YouTube API does not find. Use the channel's `@handle`.

## Scheduling (macOS launchd)

The included files run `run_newsletter.sh` every Wednesday at 07:00.

1. Replace `/Users/bytedance/youtube-newsletter` with the project folder path in `run_newsletter.sh` and `com.youtube.newsletter.plist`. Do the same in `com.youtube.newsletter.dashboard.plist` and `launch_dashboard.command` if the user wants them.
2. Run `mkdir -p logs` in the project folder. The script and plist write to `logs/newsletter.log`, `logs/launchd.log` and `logs/launchd_error.log`.
3. Run `cp com.youtube.newsletter.plist ~/Library/LaunchAgents/`. Keep this file name: the dashboard's Schedule page reads that exact path.
4. Run `launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.youtube.newsletter.plist`.

To change the time, use the dashboard's Schedule page, or edit `Weekday` (0 = Sunday, 3 = Wednesday) and `Hour` in the plist and load it again. The Mac must be awake at that time.

The dashboard plist starts the dashboard at login and keeps it running. Its Python path is hard-coded to `/Library/Frameworks/Python.framework/Versions/3.11/bin/python3`. Replace it with the output of `which python3`.

## Known Pitfalls & Solutions

These are built into the scripts. Keep them when you edit the code.

### 1. YouTube Shorts Detection
**Problem**: Filtering by duration doesn't work. Some Shorts are longer than 60 seconds.

**Solution**: `get_videos.py` checks whether the `/shorts/` URL stays on `/shorts/`:
```python
def is_youtube_short(video_id):
    shorts_url = f"https://www.youtube.com/shorts/{video_id}"
    response = requests.head(shorts_url, allow_redirects=True, timeout=5)
    return "/shorts/" in response.url
```
If the request fails, the video is treated as not a Short.

### 2. Videos Not in Chronological Order
**Problem**: The YouTube Search API doesn't return truly chronological results.

**Solution**: `get_videos.py` reads the channel's uploads playlist (`channels().list(part="snippet,contentDetails", forHandle=...)`, then `playlistItems().list(..., maxResults=15)`).

### 3. Transcript Errors and Limits
Supadata errors are printed and the video is skipped: 404 (no transcript), 401 (bad key), 429 (rate limit), or a 60-second timeout. "SUPADATA_API_KEY not set" means the `.env` line is missing or still the placeholder.

### 4. Transcript Accuracy (Names, Terms)
**Problem**: Auto-transcripts misspell names and technical terms.

**Solution**: The prompt includes the video title and description and tells Claude to use them for correct spellings.

### 5. Articles Cut Off
**Solution**: Raise `max_tokens` (8000) in `write_articles.py`.

### 6. `ModuleNotFoundError` Under launchd
**Problem**: launchd's `python3` can be a different Python from the one that has the packages.

**Solution**: In `run_newsletter.sh`, replace `python3` with the full path from `which python3`.

## Customization

### Writing Style
Edit the `prompt = f"""..."""` text in `write_articles.py`, or use the dashboard's Writing Style page. Default: a New Yorker or Atlantic-style long read for a smart non-expert. The prompt is a Python f-string:
- Keep the `{video['title']}`, `{video['channel']}`, `{video['url']}`, `{video['description']}` and `{video['transcript']}` placeholders.
- Write any literal `{` or `}` as `{{` or `}}`.
- Keep the `prompt = f"""` form, or the dashboard can't find the prompt.

### Model and Length
`model` and `max_tokens` are set in `write_articles.py`.

### Email
The email always goes to `GMAIL_ADDRESS` (`main.py` passes no other recipient).

## Workflow

```
┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌───────────────┐   ┌──────────────┐
│ Fetch videos │──▶│ Skip already │──▶│ Transcripts  │──▶│ Write articles│──▶│ EPUB + email │
│ (YouTube API)│   │  processed   │   │  (Supadata)  │   │   (Claude)    │   │ (ebooklib,   │
└──────────────┘   └──────────────┘   └──────────────┘   └───────────────┘   │  Gmail SMTP) │
                                                                             └──────────────┘
```

## Example Output

The EPUB is titled "YouTube Digest - <Month DD, YYYY>" and contains:
- A table of contents, one chapter per article (chapter title = first 50 characters of the video title)
- A note naming the source video and channel, then the article converted from Markdown
- The original video URL at the end of each chapter, as plain text
- Simple serif styling for ebook readers

## Working Rules

Keep working until everything the user asked for is done, and only stop to ask when you can't go on without the user or before a risky step.
When the work the user asked for is done and checked, stop and report. Don't add new features, docs, or refactors that weren't asked for. If you think one would help, mention it at the end instead of doing it.

When you change code that can be run, run a real check that exercises the change before reporting it done. After editing `CHANNELS`, run `python3 get_videos.py`. After editing any other script, run the changed command. A syntax-only check, or a check command that failed to start, does not count. If all that is missing is the project's declared dependencies, install them with `pip install -r requirements.txt` (and `pip install streamlit` for the dashboard), not through sudo or the system package manager, unless told not to. For changes to the article prompt or the email code, the real check is `python3 main.py` or `python3 send_email.py`, which need the user's go-ahead. If you can't run the real check, say which one you did not run and why instead of reporting the change as done.

Ask the user before these risky steps:
- `python3 main.py` or the dashboard's Generate button: paid Claude and Supadata calls for each new video, a real email, and changes to `processed_videos.json`.
- `python3 send_email.py`: sends a real email.
- `launchctl bootstrap`: installs a job that runs every week.
- Deleting or editing `processed_videos.json`: changes which videos get sent again.
