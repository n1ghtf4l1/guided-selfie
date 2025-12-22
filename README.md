# A Web-Based Guided Selfie Assistant for People with Visual Impairments

<img width="1192" height="1055" alt="High-level architecture diagram of the selfie web app, showing camera and local image processing, speech services, simple guidance mode, voice control mode, backend LLM/VLM proxies with Redis key validation, and external providers (OpenRouter/Gemini and a self-hosted VLM API)." src="https://github.com/user-attachments/assets/d33c0627-a2af-4500-bc63-bf265a7bab63" />

This project aims to enable users to take selfies by talking with the system using natural language. Inspired by [Understanding How People with Visual Impairments Take Selfies](https://dl.acm.org/doi/10.1145/3517428.3550372).

Two modes are available:

- **Voice Control Mode**: Talk with the system to express needs.
- **Simple Guidance Mode**: Quick guidance to take a centered photo, similar to existing tools.

## Development

```bash
npm install
npm run dev
```

## Deployment (Vercel)

### 1. Connect Storage

In Vercel project dashboard:

- Add **Upstash Redis** (for user keys and rate limiting)
- Add **Edge Config** (for service configuration)

### 2. Environment Variables

| Variable                  | Required | Description                                            |
| ------------------------- | -------- | ------------------------------------------------------ |
| `OPENROUTER_API_KEY`      | Yes      | Your OpenRouter API key                                |
| `VLLM_API_KEY`            | No       | API key for self-hosted VLM server (if applicable)     |
| `CF_ACCESS_CLIENT_ID`     | No       | Cloudflare Access client ID (if VLLM behind CF Access) |
| `CF_ACCESS_CLIENT_SECRET` | No       | Cloudflare Access client secret                        |

<details>
<summary><b>About VLLM_API_KEY and CF_ACCESS_* variables</b></summary>

This project uses a self-hosted VLM to keep selfie images private instead of sending them to external APIs. You can consider different options based on your needs.

Example setup:

- Run [llama.cpp](https://github.com/ggml-org/llama.cpp) server with a vision model on a local machine:
  ```bash
  llama-server -hf Qwen/Qwen3-VL-2B-Instruct-GGUF --api-key "xxx"
  ```
- Expose it via [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)
- Protect the endpoint with [Cloudflare Access Service Auth](https://developers.cloudflare.com/cloudflare-one/identity/service-tokens/) (hence the `CF_ACCESS_*` variables)
- Set `VLLM_BASE_URL` in Edge Config to point to your tunnel URL

</details>

### 3. Edge Config Items

| Key                       | Type     | Description                    |
| ------------------------- | -------- | ------------------------------ |
| `SERVICE_ENABLED`         | boolean  | Enable/disable the service     |
| `SERVICE_DISABLED_REASON` | string   | Message shown when disabled    |
| `ALLOWED_MODELS`          | string[] | Allowlist of model IDs         |
| `MAX_TEMPERATURE`         | number   | Max temperature (default: 1.0) |
| `MAX_TOKENS`              | number   | Max tokens (default: 2048)     |
| `VLLM_BASE_URL`           | string   | Self-hosted VLLM endpoint      |
| `RATE_LIMITS`             | object   | `{ per10s, perHour, perDay }`  |

### 4. User Keys

Use the CLI tool to manage user keys:

```bash
# Requires KV_REST_API_URL and KV_REST_API_TOKEN
node scripts/userkeys.mjs create --userId user123
node scripts/userkeys.mjs list --userId user123
node scripts/userkeys.mjs disable --token <token>
```

See [scripts/README.md](scripts/README.md) for full CLI reference.

Users can set their key via `?key=<token>` URL parameter (saved automatically), or click the "Key" button in the app.
