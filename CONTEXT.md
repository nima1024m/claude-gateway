# Context

## Glossary

**Claude Gateway** — the Python HTTP server that accepts prompts from external clients and returns Claude's responses. Runs on the same machine as the Claude CLI.

**Invocation** — a single stateless call to `claude -p "<prompt>"` via subprocess. No conversation history is retained between invocations.

**Concurrency Cap** — the maximum number of simultaneous Invocations allowed. Defaults to 5, configurable via environment variable. Excess requests are queued, not rejected.

**API Key** — a shared secret passed by clients in the `X-API-Key` request header. Validated server-side against an environment variable. Required on all requests.

**Stream** — the HTTP response to a client: a Server-Sent Events stream of text tokens produced by Claude as they are generated, rather than a single response returned after completion.

**Timeout** — the maximum wall-clock seconds a single Invocation may run before the subprocess is killed and an error event is sent to the client. Defaults to 120 seconds, configurable via environment variable.
