# 🎮 Unified Game Backend

A multi-provider AI backend for text-based narrative games with dynamic image generation. Mix and match LLM narrators and image models from Google, OpenAI, and Anthropic.

---

## Features

- **Multi-provider narration** — Swap between Gemini, GPT, and Claude as your game narrator with a single environment variable
- **Automatic archiving** — Summarizes long conversation logs using a fast, cost-efficient model matched to your chosen provider
- **AI image generation** — Converts narrative text into visual scene prompts, then generates images via Google Imagen or GPT Image
- **Provider-aware routing** — All subsystems (narrator, archivist, image refiner) automatically select the right client based on your config

---

## Stack

| Role | Options |
|---|---|
| Narrator | `gemini-3.1-pro-preview`, `claude-sonnet-4-6`, `gpt-5.4` |
| Archivist / Refiner | `gemini-flash-lite-latest`, `claude-haiku-4-5`, `gpt-5.4-mini` |
| Image Generation | `imagen-4.0-fast-generate-001`, `gpt-image-1.5` |

---

## Setup

### 1. Install dependencies

```bash
pip install flask flask-cors google-genai anthropic openai
```

### 2. Set environment variables

```bash
# Required — set at least one narrator key
export GEMINI_API_KEY=your_gemini_key
export OPENAI_API_KEY=your_openai_key
export ANTHROPIC_API_KEY=your_anthropic_key

# Optional — override defaults
export NARRATOR_MODEL=gemini-3.1-pro-preview     # or claude-sonnet-4-6 / gpt-5.4
export IMAGE_MODEL=imagen-4.0-fast-generate-001  # or gpt-image-1.5 / None
```

Set `IMAGE_MODEL=None` to disable image generation entirely.

### 3. Run

```bash
python app.py
```

Server starts on `http://localhost:5000`.

---

## API Reference

### `GET /api/config`
Returns the active model configuration.

**Response**
```json
{
  "narrator_model": "gemini-3.1-pro-preview",
  "narrator_provider": "gemini",
  "image_model": "imagen-4.0-fast-generate-001",
  "image_provider": "gemini",
  "archivist_model": "gemini-flash-lite-latest"
}
```

---

### `POST /api/chat`
Generate a narrator response for a player action.

**Request**
```json
{
  "system_prompt": "You are a cyberpunk dungeon master...",
  "context": "The player is standing at the entrance to Neon Bazaar...",
  "player_action": "I ask the vendor about the strange device in the corner."
}
```

**Response**
```json
{
  "text": "The vendor's eyes flicker with neon light as she leans forward..."
}
```

---

### `POST /api/archive`
Summarize a long log segment to reduce context length.

**Request**
```json
{
  "context": "<long conversation log>",
  "system_instruction": "Summarize the key events, decisions, and world state."
}
```

**Response**
```json
{
  "text": "The player arrived at Neon Bazaar, negotiated with a vendor..."
}
```

---

### `POST /api/painter`
Generate a scene image from narrative text.

**Request**
```json
{
  "prompt": "The player steps into a rain-soaked alley lit by flickering holograms...",
  "refinement_instruction": "Convert this into a cinematic image prompt...",
  "aspect_ratio": "16:9",
  "narrator_provider": "gemini"
}
```

**Response**
```json
{
  "success": true,
  "image_base64": "<base64-encoded PNG>",
  "refined_prompt": "Wide-angle shot of a rain-drenched alley, holographic neon signs..."
}
```

`aspect_ratio` defaults to `"16:9"`. `refinement_instruction` is optional — a sensible default is used if omitted.

---

## How Archiving Works

When conversation logs grow long, call `/api/archive` to compress them. The archivist model is automatically selected to match your narrator's provider (e.g. if you use Claude as narrator, `claude-haiku-4-5` handles archiving). This keeps costs low while preserving narrative continuity.

---

## Project Structure

```
.
├── app.py              # Main Flask application
├── static/             # Frontend static assets
└── templates/
    └── index.html      # Game UI entry point
```

---

## Notes

- At least one API key must be set, matching the chosen `NARRATOR_MODEL`
- The image pipeline requires either `GEMINI_API_KEY` (for Imagen) or `OPENAI_API_KEY` (for GPT Image)
- Claude prompt caching is enabled for the system prompt via `cache_control` — this reduces costs on repeated calls with large system prompts
- A commented-out Gemini caching implementation is included in `app.py` for future use
