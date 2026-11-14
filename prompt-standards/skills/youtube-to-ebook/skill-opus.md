---
name: youtube-to-ebook
description: Set up, run, schedule and customize a pipeline that turns the latest long-form video from each chosen YouTube channel into a magazine-style article written by Claude, packages the articles as an EPUB ebook, and emails it through Gmail.
---

# YouTube to Ebook

This skill sets up and maintains a small Python project. The project turns the latest long-form video from each of the user's favorite YouTube channels into a magazine-style article written by Claude, packs the articles into an EPUB, and emails it to the user's Gmail address. The scripts and support files are in this skill's directory. The user runs them from their own project folder.

## Before you act

Read the project's current state before you change anything. List and open the files that could bear on the request, including ones it doesn't mention: `get_videos.py` (the `CHANNELS` list), `write_articles.py` (the article prompt, model and `max_tokens`), which variables `.env` defines, `processed_videos.json`, `newsletters/`, `logs/`, and `~/Library/LaunchAgents/com.youtube.newsletter.plist`. The answer to "why didn't my video show up" or "why didn't it run" is usually in one of these.

Before your first command, say in one line what you are about to do. At the end, give a short recap: what you ran, what it printed (videos found, transcripts, articles, whether the email was sent), what you changed, and what the user still needs to do.

Ask before any step that costs money, sends email, or changes what runs later:
- `python3 main.py` or the dashboard's Generate button: paid Claude and Supadata calls for each new video, a real email, and changes to `processed_videos.json`.
- `python3 send_email.py`: sends a real test email and adds an archive entry.
- `launchctl bootstrap`: installs a job that runs every week.
- Deleting or editing `processed_videos.json`: changes which videos get sent again.

The skill's interface is fixed by the code. There is no `channels.txt`, no `--channels` flag, and no other command-line option. Don't add them unless the user asks for that change.

## How the pipeline works

`python3 main.py` runs these steps. If a step produces nothing, it prints a message and stops.

1. `get_videos.py`: for each handle in `CHANNELS`, read the 15 newest items in the channel's uploads playlist and keep the newest one that is not a Short. One video per channel.
2. `video_tracker.py`: drop videos already listed in `processed_videos.json`.
3. `get_transcripts.py`: fetch transcripts from the Supadata API (`SUPADATA_API_KEY`), one second apart. A video with no transcript is skipped.
4. `write_articles.py`: send each transcript, with the video title and description, to Claude (`claude-sonnet-4-20250514`, `max_tokens=8000`).
5. `send_email.py`: create `youtube_digest_YYYYMMDD.epub` and email it, with an HTML copy of the articles, from `GMAIL_ADDRESS` to the same address over Gmail SMTP.
6. Only if the email was sent: save `newsletters/newsletter_<timestamp>.html`, `.epub` and `.json`, delete the EPUB from the project folder, and add the videos to `processed_videos.json`.

So a failed email means nothing is archived or marked. The EPUB stays in the project folder, and the next run processes and pays for the same videos again. Gmail settings are required for a complete run.

## Setup

When the user asks to set this up:

1. Create a project folder (for example `~/youtube-newsletter`) and copy every file from this skill's directory into it.
2. `pip install -r requirements.txt`. Add `pip install streamlit` if they want the dashboard; it is not in `requirements.txt`.
3. `cp .env.example .env`. The user fills in the values:

| Variable | Used by | Source |
|----------|---------|--------|
| `YOUTUBE_API_KEY` | `get_videos.py` | Google Cloud Console: enable YouTube Data API v3, create an API key |
| `SUPADATA_API_KEY` | `get_transcripts.py` | Supadata (supadata.ai) |
| `ANTHROPIC_API_KEY` | `write_articles.py` | console.anthropic.com |
| `GMAIL_ADDRESS` | `send_email.py` | Sender and recipient of the ebook |
| `GMAIL_APP_PASSWORD` | `send_email.py` | myaccount.google.com/apppasswords |

4. Edit `CHANNELS` in `get_videos.py`. It ships with 8 example channels. Use `@handles` (`youtube.com/@name` → `"@name"`).
5. `python3 get_videos.py` to confirm each channel resolves. It prints "Channel not found" for a bad handle and makes no Claude or email calls.
6. With the user's go-ahead, `python3 main.py` for the first ebook.

## Commands

| Command | What it does |
|---------|--------------|
| `python3 main.py` | Full pipeline |
| `python3 -m streamlit run dashboard.py` | Web dashboard |
| `python3 get_videos.py` | List the latest long-form video per channel only |
| `python3 get_transcripts.py` | Test the Supadata key on one sample video |
| `python3 video_tracker.py` | List processed videos |
| `python3 send_email.py` | Send a test newsletter with one mock article (real email) |

`python3 write_articles.py` runs a self-test that sends one mock video to Claude (one paid API call).

## Project files

```
main.py               pipeline entry point
get_videos.py         YouTube fetch; CHANNELS list
get_transcripts.py    Supadata transcripts
write_articles.py     Claude articles; prompt, model, max_tokens
send_email.py         EPUB, email, archive
video_tracker.py      processed_videos.json helpers
dashboard.py          Streamlit dashboard
run_newsletter.sh     called by launchd
com.youtube.newsletter.plist             weekly schedule
com.youtube.newsletter.dashboard.plist   dashboard at login
launch_dashboard.command                 double-click dashboard launcher
requirements.txt, .env.example
```

Generated: `processed_videos.json`, `newsletters/`, `logs/`, and a `youtube_digest_*.epub` left behind by a failed send.

## Dashboard

Pages: **Generate** runs `python3 main.py` with a 10-minute timeout. **Channels** rewrites `CHANNELS` in `get_videos.py`. **Writing Style** edits the prompt in `write_articles.py`. **Archive** offers past HTML and EPUB downloads and lists processed videos; its **Clear All History** button asks for confirmation, then empties `processed_videos.json`, so the next run processes those videos again. **Schedule** edits the day and hour in `~/Library/LaunchAgents/com.youtube.newsletter.plist` and reloads it with `launchctl`.

One behavior of the dashboard code matters when users rely on it: a `youtube.com/channel/UC...` URL becomes `@UC...`, which the API doesn't find. Use the `@handle`.

## Scheduling (macOS launchd only)

The shipped plist runs `run_newsletter.sh` every Wednesday at 07:00.

1. Replace `/Users/bytedance/youtube-newsletter` with the project path in `run_newsletter.sh` and `com.youtube.newsletter.plist`, and in `com.youtube.newsletter.dashboard.plist` and `launch_dashboard.command` if used.
2. `mkdir -p logs`. Output goes to `logs/newsletter.log`, `logs/launchd.log` and `logs/launchd_error.log`.
3. `cp com.youtube.newsletter.plist ~/Library/LaunchAgents/`. Keep the file name, because the dashboard's Schedule page reads that exact path.
4. `launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.youtube.newsletter.plist`.

Change the time from the dashboard, or edit `Weekday` (0 = Sunday) and `Hour` in the plist and load it again. The Mac must be awake then. If `logs/newsletter.log` shows `ModuleNotFoundError`, launchd found a different `python3`: replace it in `run_newsletter.sh` with the path from `which python3`. The dashboard plist hard-codes `/Library/Frameworks/Python.framework/Versions/3.11/bin/python3`; replace that the same way.

## Design decisions to keep

These fix real YouTube quirks. Keep them when you edit the scripts.
- **Shorts**: duration doesn't identify Shorts, since some run over 60 seconds. `is_youtube_short()` sends a HEAD request to `youtube.com/shorts/<id>` and checks whether it stays on `/shorts/`. A failed request counts as not a Short.
- **Order**: the Search API isn't reliably chronological, so `get_videos.py` reads the uploads playlist (`channels().list(forHandle=...)` → `playlistItems().list(maxResults=15)`).
- **Spelling**: auto-transcripts garble names and terms, so the prompt passes the title and description and tells Claude to take spellings from them.

## Troubleshooting

- Supadata messages: 404 no transcript, 401 bad key, 429 rate limit, or timeout after 60 seconds. Each skips that video. "SUPADATA_API_KEY not set" means the `.env` line is missing or still the placeholder.
- "No new videos to process": every latest video is already in `processed_videos.json`.
- Articles cut off: raise `max_tokens` in `write_articles.py`.

## Customization

- **Writing style**: edit `prompt = f"""..."""` in `write_articles.py`, or use the Writing Style page. The default asks for a New Yorker or Atlantic-style long read for a smart non-expert. It is an f-string, so keep the `{video[...]}` placeholders, escape literal braces as `{{ }}`, and keep the `prompt = f"""` form that the dashboard searches for.
- **Model and length**: `model` and `max_tokens` in `write_articles.py`.
- **Recipient**: always `GMAIL_ADDRESS`; `main.py` passes no other.

## Output

An EPUB titled "YouTube Digest - <Month DD, YYYY>": a table of contents, one chapter per article (titled with the first 50 characters of the video title), a note naming the source video and channel, the article converted from Markdown, and the video URL as plain text at the end. Simple serif styling.
