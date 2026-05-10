# Whisper (speaches)

OpenAI-API-compatible speech-to-text (and text-to-speech) server based on
[speaches](https://github.com/speaches-ai/speaches) — a wrapper around
[faster-whisper](https://github.com/SYSTRAN/faster-whisper) with a Gradio web
playground.

## What you get

- **`/`** — Gradio web UI: drag-and-drop transcription, translation, and TTS
  testing in the browser.
- **`/v1/audio/transcriptions`** — drop-in replacement for the OpenAI Whisper
  API. Any client that speaks OpenAI Whisper works against this URL.
- **`/v1/audio/speech`** — OpenAI-compatible TTS (Kokoro, Piper). Optional.
- **`/docs`** — OpenAPI/Swagger docs.

## Choosing a model

| Model | Params | RAM (int8) | Quality | Speed on Pi 5 (CPU) |
|-------|--------|-----------|---------|---------------------|
| `Systran/faster-whisper-tiny` | 39M | ~300 MB | basic | ~realtime |
| `Systran/faster-whisper-base` | 74M | ~400 MB | good | ~0.5x realtime |
| `Systran/faster-whisper-small` | 244M | ~600 MB | very good | ~0.2x realtime |
| `Systran/faster-whisper-medium` | 769M | ~1.5 GB | excellent | ~0.05x realtime |
| `Systran/faster-whisper-large-v3` | 1550M | ~3 GB | best | unbenutzbar auf Pi |

Default is `Systran/faster-whisper-small` — best quality that's still usable on
a Raspberry Pi 5. Override at deploy time:

```bash
sudo aradeploy deploy whisper --set whisper_model=Systran/faster-whisper-base
```

The model is downloaded from HuggingFace on first request and cached in the
`cache` volume. Subsequent restarts skip the download.

## Hooking other arastack apps up

Any container on the `aradeploy-net` network can reach Whisper via the internal
DNS name `whisper`:

```
http://whisper:8000/v1/audio/transcriptions
```

For example, openclaw uses its `openai` provider for audio. Configure the
provider with the local base URL:

```bash
docker exec -it openclaw-cli node /app/openclaw.mjs \
  models auth login --provider openai
# Prompted:
#   API key:   anything (Whisper does not check)
#   Base URL:  http://whisper:8000/v1
```

Then transcribe:

```bash
openclaw capability audio transcribe \
  --file /tmp/voicenote.wav \
  --model openai/Systran/faster-whisper-small
```

## Caveats

- **CPU only** in this deployment. Speaches has CUDA-tagged images; if you ever
  attach a GPU, switch to `0.8.3-cuda` and add `runtime: nvidia` plus the
  matching `device_requests` block.
- **First request is slow.** The model is loaded lazily on demand. After the
  first transcription finishes, subsequent ones are fast.
- **Backup is disabled** (`arabackup.enable=false`). Model cache is just a
  re-downloadable HuggingFace mirror; no point shoving it into Borg.
- **No auth in front of this service.** The Web UI and API are reachable from
  anything that can talk to the routing domain. If you expose this beyond the
  LAN, put authelia in front of it.
