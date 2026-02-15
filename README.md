# Qwen Scoring Service

This repo runs a local LLM service (Qwen2.5-3B Instruct, quantized) behind a simple HTTP server. The service is intended to be called by your extension backend to generate a resume-to-job match score. The backend decides whether to apply for a job based on that score.

Important: This repo only starts the model server. It does not include custom scoring code. Your backend supplies the scoring prompt and interprets the model response.

## What This Provides

- A containerized `llama.cpp` server exposing the model on port `8080`.
- A read-only bind mount for the model file in `./models`.
- CPU-only inference, with configurable context length and threads.

## Repo Structure

- `docker-compose.yml` - container configuration
- `models/` - place the model file here (not included)

## Quickstart

1. Put the model file at:
   - `qwen2.5-3B_Quantized/models/Qwen2.5-3B-Instruct-Q4_K_M.gguf`
2. Start the service:

```powershell
cd d:\qwen_llm_services\qwen2.5-3B_Quantized
docker compose up -d
```

The server listens on `http://localhost:8080`.

## Configuration

Edit `docker-compose.yml` if you need to change performance or memory tradeoffs.

- `-c 8192`: context length (lower is faster and uses less RAM)
- `--threads` / `--threads-batch`: CPU parallelism
- `--n-gpu-layers 0`: CPU-only inference
- `--mlock`: optional, pins model in RAM

## Scoring Contract (Suggested)

Your backend is responsible for prompting the model to return a deterministic JSON payload. A typical contract:

Input (from backend to model):
- Resume text
- Job description text
- Optional weights or rules

Output (from model to backend):
```json
{
  "match_score": 0,
  "summary": "short explanation",
  "strengths": ["..."],
  "gaps": ["..."],
  "confidence": 0.0
}
```

You can enforce this with a strict system prompt and JSON-only output instructions.

## Integration Notes

- The extension backend should call this service over HTTP.
- Use a stable prompt template and deterministic decoding settings for consistent scores.
- Avoid sending raw PII unless your privacy policy allows it.

## Troubleshooting

- If the container fails on startup, verify the model file path and name.
- If performance is slow, reduce context length or thread settings.
- If you see memory errors, remove `--mlock`.
