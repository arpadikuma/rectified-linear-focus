# Model Router

FastAPI service running on your notebook, exposed via Cloudflare Tunnel.
Handles STT, TTS, LLM with automatic fallback to cloud APIs.

---

## Architecture

```
VPS (OpenClaw)
    ↓ HTTPS (Cloudflare Tunnel)
Notebook :8000 (FastAPI router)
    ├── /health     → status of all services
    ├── /stt        → Parakeet → Whisper API fallback
    ├── /tts        → Kokoro → OpenAI TTS fallback
    └── /llm        → Ollama → Anthropic/OpenAI fallback
```

---

## Full Router Code

```python
# main.py
import os
import io
import asyncio
import httpx
from fastapi import FastAPI, UploadFile, HTTPException, Header
from fastapi.responses import StreamingResponse
from pydantic import BaseModel
from typing import Optional
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

app = FastAPI(title="Ikuma Model Router")

SECRET = os.getenv("MODEL_ROUTER_SECRET", "changeme")
OLLAMA_HOST = os.getenv("OLLAMA_HOST", "http://localhost:11434")
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY", "")

# ─── Auth ───────────────────────────────────────────────────────────────────

def verify_secret(x_secret: str = Header(None)):
    if x_secret != SECRET:
        raise HTTPException(status_code=401, detail="Unauthorized")

# ─── Health ─────────────────────────────────────────────────────────────────

@app.get("/health")
async def health():
    services = {}

    # Check Ollama
    try:
        async with httpx.AsyncClient(timeout=2.0) as client:
            r = await client.get(f"{OLLAMA_HOST}/api/tags")
            services["ollama"] = "ok" if r.status_code == 200 else "error"
    except Exception:
        services["ollama"] = "unavailable"

    # Check Parakeet (just check if module loads)
    try:
        import nemo.collections.asr as nemo_asr
        services["parakeet"] = "ok"
    except Exception:
        try:
            from faster_whisper import WhisperModel
            services["parakeet"] = "faster-whisper"
        except Exception:
            services["parakeet"] = "unavailable"

    # Check Kokoro
    try:
        from kokoro_onnx import Kokoro
        services["kokoro"] = "ok"
    except Exception:
        services["kokoro"] = "unavailable"

    all_ok = all(v in ["ok", "faster-whisper"] for v in services.values())
    return {
        "status": "ok" if all_ok else "degraded",
        "services": services
    }

# ─── STT ────────────────────────────────────────────────────────────────────

@app.post("/stt")
async def transcribe(
    audio: UploadFile,
    x_secret: str = Header(None),
    fallback: Optional[str] = None  # "whisper" to force cloud
):
    verify_secret(x_secret)
    audio_bytes = await audio.read()

    # Force fallback if requested
    if fallback == "whisper":
        return await _whisper_api(audio_bytes, audio.filename)

    # Try Parakeet first
    try:
        return await _parakeet_transcribe(audio_bytes)
    except Exception as e:
        logger.warning(f"Parakeet failed: {e}, falling back to Whisper API")
        try:
            return await _whisper_api(audio_bytes, audio.filename)
        except Exception as e2:
            raise HTTPException(500, f"All STT failed: {e2}")


async def _parakeet_transcribe(audio_bytes: bytes) -> dict:
    """Transcribe using local Parakeet v2"""
    import tempfile
    import soundfile as sf
    import numpy as np

    # Try NeMo Parakeet first
    try:
        import nemo.collections.asr as nemo_asr
        
        # Save to temp file (NeMo needs file path)
        with tempfile.NamedTemporaryFile(suffix=".wav", delete=False) as f:
            f.write(audio_bytes)
            tmp_path = f.name
        
        # Load model (cached after first load)
        if not hasattr(_parakeet_transcribe, "_model"):
            _parakeet_transcribe._model = nemo_asr.models.ASRModel.from_pretrained(
                "nvidia/parakeet-tdt-0.6b-v2"
            )
        
        result = _parakeet_transcribe._model.transcribe([tmp_path])
        os.unlink(tmp_path)
        return {"text": result[0], "model": "parakeet-v2"}
    
    except ImportError:
        pass  # Fall through to faster-whisper

    # Fallback to faster-whisper (local)
    from faster_whisper import WhisperModel
    if not hasattr(_parakeet_transcribe, "_fw_model"):
        _parakeet_transcribe._fw_model = WhisperModel("medium", device="cpu")
    
    with tempfile.NamedTemporaryFile(suffix=".wav", delete=False) as f:
        f.write(audio_bytes)
        tmp_path = f.name
    
    segments, _ = _parakeet_transcribe._fw_model.transcribe(tmp_path)
    os.unlink(tmp_path)
    text = " ".join([s.text for s in segments])
    return {"text": text.strip(), "model": "faster-whisper"}


async def _whisper_api(audio_bytes: bytes, filename: str = "audio.wav") -> dict:
    """Transcribe using OpenAI Whisper API"""
    if not OPENAI_API_KEY:
        raise Exception("No OpenAI API key configured for fallback")
    
    async with httpx.AsyncClient(timeout=30.0) as client:
        r = await client.post(
            "https://api.openai.com/v1/audio/transcriptions",
            headers={"Authorization": f"Bearer {OPENAI_API_KEY}"},
            files={"file": (filename, audio_bytes, "audio/wav")},
            data={"model": "whisper-1"}
        )
        r.raise_for_status()
        return {"text": r.json()["text"], "model": "whisper-api"}

# ─── TTS ────────────────────────────────────────────────────────────────────

class TTSRequest(BaseModel):
    text: str
    voice: str = "mirai"  # "mirai" or "fujiko"
    fallback: Optional[str] = None  # "openai" to force cloud

# Voice presets
VOICE_PRESETS = {
    "mirai": {"kokoro_voice": "af_bella", "openai_voice": "nova"},
    "fujiko": {"kokoro_voice": "af_sarah", "openai_voice": "shimmer"},
}

@app.post("/tts")
async def synthesize(req: TTSRequest, x_secret: str = Header(None)):
    verify_secret(x_secret)
    preset = VOICE_PRESETS.get(req.voice, VOICE_PRESETS["mirai"])

    if req.fallback == "openai":
        return await _openai_tts(req.text, preset["openai_voice"])

    try:
        return await _kokoro_tts(req.text, preset["kokoro_voice"])
    except Exception as e:
        logger.warning(f"Kokoro failed: {e}, falling back to OpenAI TTS")
        try:
            return await _openai_tts(req.text, preset["openai_voice"])
        except Exception as e2:
            raise HTTPException(500, f"All TTS failed: {e2}")


async def _kokoro_tts(text: str, voice: str) -> StreamingResponse:
    """Synthesize using local Kokoro"""
    from kokoro_onnx import Kokoro
    import soundfile as sf
    import numpy as np

    if not hasattr(_kokoro_tts, "_model"):
        _kokoro_tts._model = Kokoro("kokoro-v0_19.onnx", "voices.bin")

    samples, sample_rate = _kokoro_tts._model.create(
        text, voice=voice, speed=1.0, lang="en-us"
    )

    buf = io.BytesIO()
    sf.write(buf, samples, sample_rate, format="WAV")
    buf.seek(0)

    return StreamingResponse(
        buf,
        media_type="audio/wav",
        headers={"X-TTS-Model": "kokoro"}
    )


async def _openai_tts(text: str, voice: str) -> StreamingResponse:
    """Synthesize using OpenAI TTS API"""
    if not OPENAI_API_KEY:
        raise Exception("No OpenAI API key for TTS fallback")

    async with httpx.AsyncClient(timeout=30.0) as client:
        r = await client.post(
            "https://api.openai.com/v1/audio/speech",
            headers={"Authorization": f"Bearer {OPENAI_API_KEY}"},
            json={"model": "tts-1", "input": text, "voice": voice}
        )
        r.raise_for_status()

    return StreamingResponse(
        io.BytesIO(r.content),
        media_type="audio/mpeg",
        headers={"X-TTS-Model": "openai-tts"}
    )

# ─── LLM ────────────────────────────────────────────────────────────────────

class LLMRequest(BaseModel):
    messages: list
    model: str = "mistral"  # Ollama model name
    temperature: float = 0.7
    max_tokens: int = 1000

@app.post("/llm")
async def complete(req: LLMRequest, x_secret: str = Header(None)):
    verify_secret(x_secret)

    try:
        return await _ollama_complete(req)
    except Exception as e:
        logger.warning(f"Ollama failed: {e}, falling back to cloud")
        return await _cloud_llm_complete(req)


async def _ollama_complete(req: LLMRequest) -> dict:
    async with httpx.AsyncClient(timeout=60.0) as client:
        r = await client.post(
            f"{OLLAMA_HOST}/api/chat",
            json={
                "model": req.model,
                "messages": req.messages,
                "options": {
                    "temperature": req.temperature,
                    "num_predict": req.max_tokens
                },
                "stream": False
            }
        )
        r.raise_for_status()
        data = r.json()
        return {
            "content": data["message"]["content"],
            "model": f"ollama/{req.model}"
        }


async def _cloud_llm_complete(req: LLMRequest) -> dict:
    """Fallback to Anthropic Claude"""
    import anthropic
    client = anthropic.Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))
    
    message = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=req.max_tokens,
        messages=req.messages
    )
    return {
        "content": message.content[0].text,
        "model": "claude-fallback"
    }


# ─── Run ────────────────────────────────────────────────────────────────────

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

---

## Service Setup (runs on notebook startup)

### Option A: systemd (WSL2 with systemd enabled)
```bash
# Create service file
sudo nano /etc/systemd/system/model-router.service
```

```ini
[Unit]
Description=Ikuma Model Router
After=network.target

[Service]
Type=simple
User=youruser
WorkingDirectory=/home/youruser/model-router
Environment="PATH=/home/youruser/.local/bin:/usr/bin:/bin"
EnvironmentFile=/home/youruser/model-router/.env
ExecStart=/home/youruser/.local/bin/uvicorn main:app --host 0.0.0.0 --port 8000
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable model-router
sudo systemctl start model-router
sudo systemctl status model-router
```

### Option B: PM2 (simpler, no systemd needed)
```bash
npm install -g pm2
cd ~/model-router
pm2 start "uvicorn main:app --host 0.0.0.0 --port 8000" --name model-router
pm2 startup  # auto-start on boot
pm2 save
```

---

## Testing

```bash
# Health check (from VPS)
curl https://models.yourdomain.com/health \
  -H "X-Secret: your_secret"

# STT test
curl -X POST https://models.yourdomain.com/stt \
  -H "X-Secret: your_secret" \
  -F "audio=@test.wav"

# TTS test
curl -X POST https://models.yourdomain.com/tts \
  -H "X-Secret: your_secret" \
  -H "Content-Type: application/json" \
  -d '{"text": "Good morning, Ikuma. Ready for standup?", "voice": "mirai"}' \
  --output test_mirai.wav

# LLM test
curl -X POST https://models.yourdomain.com/llm \
  -H "X-Secret: your_secret" \
  -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "Hello"}]}'
```

---

## Performance Notes

- **Parakeet v2:** First call loads model (~2-3s). Subsequent calls ~200-500ms.
- **Kokoro TTS:** First call loads model (~1-2s). Subsequent calls ~100-300ms.
- **Ollama:** Depends on model size. Mistral 7B: ~1-3s first token.
- **Models are cached in memory** after first load — don't restart the router unnecessarily.
- **If latency bothers you:** Switch `/tts` and `/stt` to cloud APIs for daily use, keep local as primary only when on desktop (more RAM/CPU).

---

## Links
- [[infrastructure]] (VPS + Cloudflare Tunnel setup)
- [[agents]] (who calls what)
- [[database]] (schema)
