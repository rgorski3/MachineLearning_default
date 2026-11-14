---
name: youtube-to-ebook
description: Set up, run, schedule and customize a pipeline that turns the latest long-form video from each chosen YouTube channel into a magazine-style article written by Claude, packages the articles as an EPUB ebook, and emails it through Gmail.
---

# YouTube to Ebook

This skill sets up and maintains a small Python project. The project turns the latest long-form video from each of the user's favorite YouTube channels into a magazine-style article written by Claude, packs the articles into an EPUB ebook, and emails it to the user's Gmail address. The scripts and support files are in this skill's directory, and the user runs them from their own project folder.

The user owns these scripts. Make the changes they ask for inside the existing structure, and keep each change as small as the request allows. If you see an improvement worth making, mention it at the end instead of making it.

## How to work

Before you start, say in a line what you're about to do; brief updates while you work help the user follow along. Close with a short recap that stands on its own (what you found, what you did, and what's next), so a reader who only sees the last message has the full picture.

### Finishing work

Please avoid stopping while you still owe work the user asked for. A setup request is finished when `python3 get_videos.py` finds the user's channels and, with their go-ahead, `python3 main.py` has produced and sent the first ebook, or when you have reached a step only the user can do.

You will meet ordinary obstacles: a failed `pip install`, "Channel not found", a Supadata 429 or timeout, a missing `logs/` folder, a `ModuleNotFoundError` under launchd. Diagnose them first and work through them with the access you have (fix the handle, create the folder, wait and retry, use the full Python path) rather than stopping.

Some blockers belong to the user. Only they can create the YouTube, Supadata and Anthropic keys and the Gmail app password, and only they can approve the steps listed below. When you reach one of these, say plainly what you need from them and carry on with any work that doesn't depend on it.

### What needs approval

Ask the user before these steps, because each one costs money, sends email, or changes what runs later:
- `python3 main.py`, or the dashboard's Generate button. It makes paid Claude and Supadata calls for each new video, sends a real email, and adds the videos to `processed_videos.json`.
- `python3 send_email.py`. It sends a real test email and adds an entry to the archive.
- `launchctl bootstrap`. It installs a job that runs every week.
- Deleting or editing `processed_videos.json`. It changes which videos are sent again.

When the user approves, they should name the step, for example "Yes, run main.py now".

You can do these without asking: read any project file; create the project folder and copy the skill's files into it; run `pip install -r requirements.txt` and `pip install streamlit`; create `.env` from `.env.example` and add the `SUPADATA_API_KEY=` line; create `logs/`; replace the hard-coded paths described under Scheduling; edit `CHANNELS` or the article prompt when the user asks; and run `python3 get_videos.py`, `python3 video_tracker.py` and `python3 get_transcripts.py` (one Supadata request).

### Final report

Write the recap for an intelligent reader who has not seen your commands, their output or the files you opened. Put the result first: whether the ebook was sent, how many videos and articles there were, and what the user must do next. Use plain words and full sentences, and use the same name for the same thing throughout. Leave out detail the user doesn't need to act on.

## How the pipeline works

`python3 main.py` runs six steps. If a step produces nothing, it prints a message and stops.

1. `get_videos.py` reads, for each handle in the `CHANNELS` list, the 15 newest items in the channel's uploads playlist and keeps the newest one that is not a Short. The result is one video per channel.
2. `video_tracker.py` drops videos that are already in `processed_videos.json`.
3. `get_transcripts.py` fetches each transcript from the Supadata API with `SUPADATA_API_KEY`, one second apart. A video with no transcript is skipped.
4. `write_articles.py` sends each transcript, with the video title and description, to Claude (`claude-sonnet-4-20250514`, `max_tokens=8000`).
5. `send_email.py` creates `youtube_digest_YYYYMMDD.epub` and emails it, with an HTML copy of the articles, from `GMAIL_ADDRESS` to the same address over Gmail SMTP.
6. Only if the email was sent, it saves `newsletters/newsletter_<timestamp>.html`, `.epub` and `.json`, deletes the EPUB from the project folder, and adds the videos to `processed_videos.json`.

This means a failed email leaves nothing archived or marked. The EPUB stays in the project folder, and the next run processes and pays for the same videos again. The Gmail settings are therefore required for a complete run.

## Setup

1. Create a project folder (for example `~/youtube-newsletter`) and copy every file from this skill's directory into it.
2. Run `pip install -r requirements.txt`. For the dashboard, also run `pip install streamlit`, which is not in `requirements.txt`.
3. Run `cp .env.example .env`. The user fills in the values:

| Variable | Used by | Source |
|----------|---------|--------|
| `YOUTUBE_API_KEY` | `get_videos.py` | Google Cloud Console: enable YouTube Data API v3, create an API key |
| `SUPADATA_API_KEY` | `get_transcripts.py` | Supadata (supadata.ai) |
| `ANTHROPIC_API_KEY` | `write_articles.py` | console.anthropic.com |
| `GMAIL_ADDRESS` | `send_email.py` | The address that sends and receives the ebook |
| `GMAIL_APP_PASSWORD` | `send_email.py` | myaccount.google.com/apppasswords |

4. Edit the `CHANNELS` list in `get_videos.py`, which ships with 8 example channels. Use `@handles`: `youtube.com/@name` becomes `"@name"`.
5. Run `python3 get_videos.py`. It prints each channel and the video it found, or "Channel not found", and makes no Claude or email calls.
6. With the user's approval, run `python3 main.py` to make the first ebook.

## Commands

| Command | What it does |
|---------|--------------|
| `python3 main.py` | Runs the full pipeline |
| `python3 -m streamlit run dashboard.py` | Opens the web dashboard |
| `python3 get_videos.py` | Lists the latest long-form video per channel, nothing more |
| `python3 get_transcripts.py` | Tests the Supadata key on one sample video |
| `python3 video_tracker.py` | Lists the videos already processed |
| `python3 send_email.py` | Sends a test newsletter with one mock article (a real email) |

The project has no `channels.txt` and no command-line flags such as `--channels`. The self-test in `python3 write_articles.py` sends one mock video to Claude, which is a paid API call.

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

The scripts create `processed_videos.json`, `newsletters/` and `logs/`. A failed send leaves a `youtube_digest_*.epub` in the project folder.

## Dashboard

The dashboard has five pages. **Generate** runs `python3 main.py` with a 10-minute timeout and shows the log. **Channels** rewrites the `CHANNELS` list in `get_videos.py`. **Writing Style** edits the prompt in `write_articles.py`. **Archive** offers past HTML and EPUB downloads and lists processed videos; its **Clear All History** button asks for confirmation, then empties `processed_videos.json`, so the next run processes those videos again. **Schedule** edits the day and hour in `~/Library/LaunchAgents/com.youtube.newsletter.plist` and reloads it with `launchctl`.

One part of the dashboard code behaves in a way users don't expect: a `youtube.com/channel/UC...` URL turns into `@UC...`, which the YouTube API does not find. Use the channel's `@handle` instead.

## Scheduling (macOS launchd only)

The shipped plist runs `run_newsletter.sh` every Wednesday at 07:00.

1. Replace `/Users/bytedance/youtube-newsletter` with the project path in `run_newsletter.sh` and `com.youtube.newsletter.plist`. If the user wants them, do the same in `com.youtube.newsletter.dashboard.plist` and `launch_dashboard.command`.
2. Run `mkdir -p logs`. Output goes to `logs/newsletter.log`, `logs/launchd.log` and `logs/launchd_error.log`.
3. Run `cp com.youtube.newsletter.plist ~/Library/LaunchAgents/`. Keep the file name, because the dashboard's Schedule page reads that exact path.
4. With the user's approval, run `launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.youtube.newsletter.plist`.

To change the time, use the dashboard, or edit `Weekday` (0 is Sunday) and `Hour` in the plist and load it again. The Mac must be awake at that time. If `logs/newsletter.log` shows `ModuleNotFoundError`, launchd found a different `python3`; replace it in `run_newsletter.sh` with the path that `which python3` prints. The dashboard plist hard-codes `/Library/Frameworks/Python.framework/Versions/3.11/bin/python3`, so replace that path the same way.

## Design decisions in the code

The scripts work around three YouTube quirks. Keep these workarounds when you edit the code, because the simpler approaches give wrong results.
- **Shorts.** Duration does not identify a Short, since some run longer than 60 seconds. `is_youtube_short()` sends a HEAD request to `youtube.com/shorts/<id>` and checks whether the final URL still contains `/shorts/`. If the request fails, the video counts as not a Short.
- **Order.** The Search API does not return videos in reliable date order, so `get_videos.py` reads the uploads playlist (`channels().list(forHandle=...)`, then `playlistItems().list(maxResults=15)`).
- **Spelling.** Auto-transcripts misspell names and technical terms, so the prompt includes the title and description and tells Claude to take spellings from them.

## Troubleshooting

- Supadata problems print a message and skip the video: 404 means no transcript, 401 a bad key, 429 a rate limit, and a timeout comes after 60 seconds. "SUPADATA_API_KEY not set" means the `.env` line is missing or still holds the placeholder.
- "No new videos to process" means every channel's latest video is already in `processed_videos.json`.
- If articles are cut off, raise `max_tokens` in `write_articles.py`.

## Customization

- **Writing style.** Edit the `prompt = f"""..."""` text in `write_articles.py`, or use the Writing Style page. The default asks for a New Yorker or Atlantic-style long read for a smart reader who is not an expert. The prompt is a Python f-string, so keep the `{video[...]}` placeholders, write literal braces as `{{` and `}}`, and keep the `prompt = f"""` form that the dashboard looks for.
- **Model and length.** `model` and `max_tokens` are set in `write_articles.py`.
- **Recipient.** The email always goes to `GMAIL_ADDRESS`, because `main.py` passes no other recipient.

## Output

The EPUB is titled "YouTube Digest - <Month DD, YYYY>". It has a table of contents and one chapter per article, titled with the first 50 characters of the video title. Each chapter opens with a note naming the source video and channel, then the article converted from Markdown, and ends with the video URL as plain text. The styling is a simple serif layout for ebook readers.
