# claude-gateway

A lightweight Python HTTP gateway that accepts prompts from external clients and streams responses from Claude CLI via Server-Sent Events (SSE).

## How it works

```
Client → POST /chat → FastAPI → subprocess: claude -p → SSE stream → Client
```

Each request invokes `claude -p` as a local subprocess. Responses are streamed back as SSE events as Claude produces them. Concurrency is capped via a semaphore; requests beyond the cap are queued rather than rejected.

## Setup

**Requirements:** Python 3.10+, [Claude CLI](https://claude.ai/code) installed and authenticated.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

cp .env.example .env
# Edit .env and set a strong API_KEY
```

## Configuration

All config lives in `.env`:

| Variable        | Default | Description                                      |
|-----------------|---------|--------------------------------------------------|
| `API_KEY`       | —       | Required. Secret passed in `X-API-Key` header.  |
| `MAX_CONCURRENT`| `5`     | Max simultaneous Claude invocations.             |
| `TIMEOUT`       | `120`   | Seconds before a hung invocation is killed.      |
| `PORT`          | `8000`  | Port the server listens on.                      |

## Running

```bash
uvicorn main:app --host 0.0.0.0 --port 8000
```

## API

### `POST /chat`

**Headers:**
```
X-API-Key: your-api-key
Content-Type: application/json
```

**Body:**
```json
{ "prompt": "your question here" }
```

**Response** (SSE stream):
```
data: {"text": "Hello"}
data: {"text": " there!"}
data: [DONE]
```

On error:
```
data: {"error": "timeout"}
```

**Example:**
```bash
curl -N -X POST http://localhost:8000/chat \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your-api-key" \
  -d '{"prompt": "Explain async/await in one sentence"}'
```

### `GET /health`

Returns `{"status": "ok"}`. No auth required.

## Scalability notes

- **Concurrency:** The semaphore (`MAX_CONCURRENT`) prevents resource exhaustion under burst traffic. Tune it to your Claude API rate limit tier.
- **Timeout:** Each invocation is killed after `TIMEOUT` seconds, ensuring stuck processes release their semaphore slot.
- **Horizontal scaling:** Each instance manages its own semaphore. Put a load balancer in front to scale across multiple instances.

## Roadmap

- [ ] SSH-based remote invocation (swap `invoke_claude` in `main.py`)
- [ ] Multi-turn conversation sessions
- [ ] Per-client rate limiting
