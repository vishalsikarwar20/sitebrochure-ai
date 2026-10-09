# SiteBrochure AI

Paste a company name and its website URL, and get a marketing brochure written by an LLM, streamed live into a Gradio app. Choose between **Gemini** (cloud, free tier) and **Ollama** (local, no API key, no rate limits).


## What it does

1. Fetches the text content of the website you give it.
2. Sends that text to the model you selected, with a brochure-writing system prompt.
3. Streams the brochure back into the page as it is generated.

## Features

- Switch between Gemini and a local Ollama model from a dropdown.
- Streaming output: text appears as it is generated, not after a long wait.
- One OpenAI-compatible client library for both providers. Only the `base_url`, key, and model name change.
- Runs fully offline with Ollama (after the model is downloaded).

## How it works

```
URL -> fetch page text -> build prompt -> LLM (Gemini or Ollama) -> stream chunks -> Gradio
```

- `stream_brochure(name, url, model)` is a **generator**. It picks the backend and forwards its output with `yield from`.
- Each backend function yields the **full text so far**, not just the newest chunk. Gradio replaces the output box on every yield, so cumulative text is what makes the brochure appear to type out.
- Both providers are called through the OpenAI Python SDK:

| Provider | `base_url` | Key |
|---|---|---|
| Gemini | `https://generativelanguage.googleapis.com/v1beta/openai/` | `GOOGLE_API_KEY` |
| Ollama | `http://127.0.0.1:11434/v1` | any string (ignored) |

## Requirements

- Python 3.9 or newer
- A free Google AI Studio API key: https://aistudio.google.com/apikey
- Optional, for local mode: [Ollama](https://ollama.com) installed, with a model pulled

## Setup

1. Clone the repo and create a virtual environment.

   ```bash
   git clone https://github.com/<your-username>/sitebrochure-ai.git
   cd sitebrochure-ai
   python -m venv .venv
   .venv\Scripts\activate        # Windows
   # source .venv/bin/activate   # macOS / Linux
   ```

2. Install dependencies.

   ```bash
   pip install -r requirements.txt
   ```

3. Create your `.env` from the example and add your key.

   ```bash
   copy .env.example .env        # Windows
   # cp .env.example .env        # macOS / Linux
   ```

   ```dotenv
   GOOGLE_API_KEY=your_key_here
   ```

   Never commit `.env`. It is listed in `.gitignore`.

4. (Optional) Set up Ollama for local mode.

   ```bash
   ollama pull llama3.2
   ollama serve
   ```

## Run

```bash
python app.py
```

Open `http://127.0.0.1:7860`, enter a company name and URL, choose a model, and submit.

## Configuration

- **Gemini model ID:** set at the top of the code. Model names change often, so list the IDs your key can use before editing:

  ```python
  for m in gemini.models.list():
      print(m.id)
  ```

- **Ollama model:** any model you have pulled (`ollama list`). Smaller models (for example `llama3.2:1b`) are faster on CPU.
- **Prompt:** the brochure style lives in `system_prompt`. Edit it to change tone or length.

## Troubleshooting

| Error | Likely cause | Fix |
|---|---|---|
| `400 Please pass a valid API key` | Client built with the wrong key, for example a quoted variable name like `api_key='google_api_key'` | Pass the variable, not a string: `api_key=google_api_key`. Re-run the client cell or restart the app |
| `404 model not found` | Wrong or retired Gemini model ID | Copy an ID from `gemini.models.list()` |
| `429 RESOURCE_EXHAUSTED` | Free-tier rate limit | Wait a minute, or switch to Ollama |
| `APIConnectionError` with Ollama | Ollama server not running | Start Ollama, then check `http://127.0.0.1:11434` |
| Slow or frozen Ollama replies | Not enough free RAM, CPU-only inference | Close heavy apps or use a smaller model |
| Gradio `share=True` fails | Antivirus, VPN, or firewall blocks the tunnel | Not needed locally. Use Hugging Face Spaces to host it |

## Limitations

- Only reads the page you give it. Pages that need JavaScript to render may return little text.
- Brochure quality depends on the page content and the model. Small local models are weaker.
- Free Gemini tiers have rate limits, and Google may use free-tier inputs to improve its models. Do not send sensitive data.
- Output is not fact-checked. Review it before using it anywhere.

## Next steps

- Follow links on the site (About, Products) and include them in the prompt.
- Add a language selector.
- Deploy on Hugging Face Spaces (Gemini only, key stored in Space secrets).
- Add a small test set to compare brochure quality across models.

## What I learned

- Streaming with Python generators and `yield from`.
- Reading provider errors: 4xx means fix the request, 5xx means retry.
- Using one OpenAI-compatible client for several providers.
- A client is a snapshot: rebuild it after its key or URL changes.
