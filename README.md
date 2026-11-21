# Local AI Router Stack
 
Run a terminal coding agent through your own local AI gateway, all in Docker.
 
Three containers, one compose file:
 
| Tool | What it does |
|---|---|
| [9Router](https://github.com/decolua/9router) | Local gateway. One OpenAI-compatible endpoint (`localhost:20128/v1`) in front of many AI providers, with fallback when one runs out. |
| [Headroom](https://github.com/chopratejas/headroom) | Compresses tool output, logs and files before they reach the model, to save tokens. |
| [OpenClaude](https://github.com/Twigpine/openclaude) | Open-source coding agent for your terminal. Here it talks only to 9Router. |
 
Everything lives in containers and volumes. Nothing is installed on your machine, and one command removes it all.
 
> **Unofficial.** This repo only wires the three tools together. It is not affiliated with Anthropic, 9Router, Headroom or OpenClaude.

## Quick start
 
**1. Clone the repo**
 
```bash
git clone https://github.com/fahrihanif/local-ai-router-stack
cd local-ai-router-stack
```
 
**2. Create the `.env` file**
 
```bash
nano .env
```
 
Open `.env` and fill it in:
 
- `API_KEY_SECRET`: a random value. One way to make it: `openssl rand -hex 32`.
- `INITIAL_PASSWORD`: a password you choose for the dashboard. Optional, but without it 9Router uses `123456`.
- `OPENCLAUDE_MODEL`: leave as is for now. You set it in step 4.
**3. Start the gateway**
 
```bash
docker compose up -d 9router
```
 
This also starts Headroom, because 9Router depends on it.
 
**4. Set up 9Router**
 
1. Open http://localhost:20128 and log in with `INITIAL_PASSWORD`.
2. Connect a provider.
3. Create or pick a model or combo, then put its name in `.env` as `OPENCLAUDE_MODEL`.
4. Optional: check that Headroom is on (Endpoint > Token Saver > Headroom). Menu names can change between 9Router versions.
**5. Start the agent**
 
```bash
docker compose run --rm openclaude
```
 
The first run builds the OpenClaude image. Put your project in the `workspace/` folder. Inside the container it appears at `/workspace`.
 
## Configuration
 
| Variable | Purpose |
|---|---|
| `API_KEY_SECRET` | Secret 9Router uses to generate API keys. `docker-compose.yml` also passes it to OpenClaude as its API key. |
| `INITIAL_PASSWORD` | Dashboard password. Optional. Default: `123456`. |
| `OPENCLAUDE_MODEL` | Model or combo name from the 9Router dashboard. Default: `openrouter/free-tiers-combo`. |

# Remove everything
 
```bash
docker compose --profile cli down -v --rmi all
```
 
- `--profile cli` includes the OpenClaude container and image.
- `-v` deletes the volumes, including the providers and API keys saved in 9Router.
- `--rmi all` deletes the images and frees the disk space.
To stop it but keep your settings, run `docker compose down`.
 
## Security
 
A router sees every prompt and every key. Treat it like a password manager.
 
- Bind the port to `127.0.0.1` (see "Before you start").
- Keep it updated. 9Router has had several critical security advisories, including bugs that exposed API keys.
- Use separate API keys with spending limits, so you can revoke them.
- Do not connect Claude Pro or Max subscription logins. Anthropic's terms do not allow them in third-party tools. This setup is for API keys and other providers.
- Never commit `.env`. It is in `.gitignore`.
## Troubleshooting
 
| Problem | Fix |
|---|---|
| Port 20128 or 8787 is already in use | Change the left side of the port mapping in `docker-compose.yml`, for example `"20129:20128"`. |
| The agent starts but gets authentication errors from 9Router | The compose file sends `API_KEY_SECRET` as the API key. If 9Router rejects it, create an API key in the dashboard and set it as `OPENAI_API_KEY` in `docker-compose.yml`. |
| The agent says the model is not found | `OPENCLAUDE_MODEL` must match a model or combo name in the 9Router dashboard. |
| The agent cannot write to `/workspace` (Linux) | The container user has id 1000. Run `sudo chown -R 1000:1000 workspace`. |
 
## Files
 
| File | Purpose |
|---|---|
| `docker-compose.yml` | The three services |
| `openclaude/Dockerfile` | OpenClaude image, non-root user |
| `.env.` | your `.env` |
| `workspace/` | Your code, mounted into the agent container |
 