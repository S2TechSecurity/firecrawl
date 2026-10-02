# S2 Firecrawl integration

This directory contains S2-only deployment customisation so upstream Firecrawl
updates can be pulled with minimal merge conflict.

## Repository policy

- `origin`: `https://github.com/S2TechSecurity/firecrawl.git`
- `upstream`: `https://github.com/firecrawl/firecrawl.git`
- Pull upstream changes into a review branch first. Never develop directly
  against an upstream checkout.

## Placement

Do not deploy the default Firecrawl stack onto the 8 GB dedicated production
web server. The upstream Compose file can reserve 8 GB for the API and 4 GB for
Playwright before Redis, RabbitMQ and PostgreSQL are counted.

Target: Proxmox/internal-services worker.

The first deployment stays private to the S2 network. Do not expose port 3002
to the public Internet until an explicit authentication and reverse-proxy policy
has been implemented and tested.

## AI provider

Firecrawl's OpenAI-compatible variables are pointed at the S2 AI Gateway:

- `OPENAI_BASE_URL` <- `S2_AI_GATEWAY_BASE_URL`
- `OPENAI_API_KEY` <- `S2_AI_GATEWAY_API_KEY`
- `MODEL_NAME` <- `S2_AI_GATEWAY_MODEL`

The value stored in `S2_AI_GATEWAY_API_KEY` is the S2 Gateway bearer token.
It is not a paid OpenAI API key.

## Conservative runtime

The S2 overlay lowers initial worker/browser concurrency and caps the API and
Playwright containers at 2 GB each. Tune upward only after observing real load.

Redis, RabbitMQ and NuQ PostgreSQL gain named persistent volumes in the S2
overlay. FoundationDB is moved behind the optional `fdb` profile because the
default S2 queue backend is PostgreSQL.

## Validate configuration

From the repository root, using a non-secret placeholder:

```powershell
$env:S2_AI_GATEWAY_API_KEY = "validation-only"
docker compose -f docker-compose.yaml -f s2/docker-compose.override.yaml config
```

Do not commit the generated environment or any real credential.
