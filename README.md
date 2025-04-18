# PlexSubSync-Docker

A Dockerized subtitle synchronization service that combines the power of [sc0ty/subsync](https://github.com/sc0ty/subsync) with a lightweight Python API wrapper for automated subtitle syncing in Plex libraries.

## Overview

**PlexSubSync-Docker** is a containerized solution that:

- Accepts webhook requests containing a Plex media ID
- Uses the Plex API to resolve the file path of the video and its subtitle
- Runs [`subsync`](https://github.com/sc0ty/subsync) to sync the subtitles to the audio track
- Sends webhook notifications to Home Assistant (or any service) with the sync status

This is ideal for Home Assistant automations or manual sync triggers from Plex metadata.

## Components

- [`PlexSubSync`](https://github.com/ChrisHansenTech/PlexSubSync): A lightweight Python app that handles media lookup and sync command execution.
- [`sc0ty/subsync`](https://github.com/sc0ty/subsync): The actual synchronization engine, included in the container.
- **Docker**: Bundled into a single image for portability and ease of deployment.

## Docker Image

### Build & Publish

```bash
# Pull the published image from GitHub Container Registry (GHCR)
docker pull ghcr.io/<your-github-username>/plexsubsync-docker:latest

# (Optional) To build locally:
# docker build -t ghcr.io/<your-github-username>/plexsubsync-docker:latest .
```

### Run

```bash
docker run -d \
  --name plexsubsync \
  -e PLEX_TOKEN=your_plex_token \
  -e PLEX_BASE_URL=http://your.plex.server:32400 \
  -e HA_WEBHOOK_URL=https://your-home-assistant/api/webhook/your-hook-id \
  -v /path/to/plex/media:/media \
  -p 8000:8000 \
  ghcr.io/ChrisHansenTech/plexsubsync-docker:latest
```

> Make sure the `/media` path matches your Plex library structure inside the container.

## Environment Variables

| Variable           | Description                                                                 |
|--------------------|-----------------------------------------------------------------------------|
| `PLEX_TOKEN`        | Your Plex access token                                                      |
| `PLEX_BASE_URL`     | Base URL of your Plex server (e.g., `http://192.168.1.10:32400`)           |
| `HA_WEBHOOK_URL`    | Optional. Home Assistant webhook URL to notify status                      |

## Webhook API

Send a POST request with a `media_id`:

```http
POST /sync
Content-Type: application/json
```

```json
{
  "media_id": "123456",      # Plex metadata ID
  "audio_lang": "en",        # ISO-639-1 audio language code
  "sub_lang": "es"           # ISO-639-1 subtitle language code
}
```

The `media_id` corresponds to the Plex metadata ID of the media you want to sync. Provide both audio and subtitle
language codes so that PlexSubSync can match the correct audio track and subtitle file.

### Response

- `200 OK` on successful start

## Example Home Assistant Automation

```yaml
alias: Sync Subtitles After Playback
trigger:
  - platform: state
    entity_id: media_player.living_room_plex
    to: 'idle'
action:
  - service: rest_command.sync_subtitles
```

Define the `rest_command.sync_subtitles` in `configuration.yaml` to call your Docker API.

## Folder Structure

The Docker container expects both video and subtitle files to be in the same directory. It supports common subtitle extensions like `.srt`, `.eng.srt`, `.en.srt`, `.sdh.srt`.

## License

This project is licensed under the MIT License. The embedded [`subsync`](https://github.com/sc0ty/subsync) tool is licensed under GPL-3.0.

---

### Contact

Created by [ChrisHansenTech](https://chrishansen.tech) — feel free to submit issues or pull requests!
