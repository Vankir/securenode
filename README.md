<div align="center">

# 🛡️ SecureNode for n8n
### The Missing Privacy Layer for AI Automation

<a href="https://securenode.app">
  <img src="https://img.shields.io/badge/Website-securenode.app-blue?style=for-the-badge" alt="Website">
</a>
<a href="https://hub.docker.com/r/vankir/securenode">
  <img src="https://img.shields.io/docker/pulls/vankir/securenode?style=for-the-badge" alt="Docker Pulls">
</a>
<a href="https://github.com/vankir/securenode/blob/main/LICENSE">
  <img src="https://img.shields.io/badge/License-Freemium-orange?style=for-the-badge" alt="License">
</a>

<br />

**GDPR-compliant middleware that sanitizes PII locally _before_ sending data to OpenAI, Anthropic, or DeepSeek.**

[ **Get Docker Image** ](https://securenode.app) | [ **Try Cloud Demo** ](#-try-it-instantly-cloud-demo) | [ **API Docs** ](#-api-usage-for-n8n-http-request)

</div>

---

## 🚨 The Problem
Sending raw client data (Names, Emails, IBANs) to public LLMs is a privacy risk and often violates **GDPR**, **CCPA**, and **SOC2**.

* ❌ **Risk:** Data leaks into public training sets.
* ❌ **Legal:** Contracts often forbid sending PII to 3rd party AI.
* ❌ **Trust:** Enterprise clients refuse automation due to privacy concerns.

## ✅ The Solution: "The Sandwich Method"
SecureNode acts as a local reversible proxy. No personal data leaves your infrastructure.

1.  **⬇️ Input:** `Client John Doe asks for a refund on order TIC-9999.`
2.  **🛡️ Anonymize (Local):** `Client <PERSON_1> asks for a refund on order <TICKET_1>.` (Sent to AI)
3.  **🤖 AI Processing:** `Approve refund for <TICKET_1>.`
4.  **✅ De-anonymize (Local):** `Approve refund for TIC-9999.`

---

## 🚀 Try it Instantly (Cloud Demo)
You don't need to install anything to test the logic. We hosted a public endpoint for demonstration.

> **⚠️ Warning:** This demo runs on our public server. Do not send real sensitive PII. For production, use the Docker version below.

1.  **[Click here to view the Workflow JSON](workflows/simple-cloud-demo.json)**
2.  Copy the code (`Ctrl+A` -> `Ctrl+C`).
3.  Paste it into your n8n editor (`Ctrl+V`).

---

## 🐳 Installation (Production)
For full privacy, run SecureNode locally alongside your n8n instance using Docker.

### Docker Compose
Add this service to your `docker-compose.yml`:

```yaml
services:
  securenode:
    image: vankir/securenode:latest
    container_name: securenode
    restart: unless-stopped
    ports:
      - "5005:5005"
    healthcheck:
      test:
        - CMD-SHELL
        - "python -c \"import urllib.request; urllib.request.urlopen('http://127.0.0.1:5005/health', timeout=2).read()\""
      interval: 30s
      timeout: 3s
      retries: 5
      start_period: 20s
    deploy:
      resources:
        limits:
          memory: 1G
```

### Higher accuracy (large NLP models)
For higher PII detection accuracy you can start SecureNode with large NLP models enabled.

Note: the first start takes longer because the container downloads the models on startup.

Example:

```yaml
services:
  securenode:
    environment:
      - MODELS=en:en_core_web_lg,de:de_core_news_lg
```

### Start
Run:

```bash
docker compose up -d
```

Open n8n:
- `http://localhost:5678`

Import the demo workflow (Docker):

1.  **[Open the Workflow JSON](workflows/simple-docker-demo.json)**
2.  Copy the code (`Ctrl+A` -> `Ctrl+C`).
3.  Paste it into your n8n editor (`Ctrl+V`).

### 🔑 License
SecureNode accepts the license in either of these ways:

- **Environment variable (recommended for production)**: set `LICENSE_KEY` on the container.
- **Request field (convenient for n8n)**: include `license` in the JSON body.

The license is optional. If you omit it, SecureNode will run with the free version limitations.

If both are provided, the request `license` can be used (e.g. to override).

### ❤️ Health
Use `GET /health` to verify the service is ready.

### 🔌 API Usage (for n8n HTTP Request)
To connect SecureNode to n8n, use the HTTP Request node.

Base URL:
- `http://securenode:5005` (inside Docker network)
- `http://localhost:5005` (from your host)

Headers:
- `Content-Type: application/json`

#### Step 1: Anonymize (`POST /anonymize`)
Detects PII and replaces it with tokens. Returns a state object needed for decryption.

Body JSON:
```json
{
  "text": "Client John asks about ticket TIC-9999",
  "license": "YOUR_LICENSE_KEY",
  "regex_entities": [
    { "name": "TICKET", "regex": "TIC-\\d{4}" }
  ]
}
```

`license` is optional. If you set `LICENSE_KEY` as an environment variable for the container, you can also omit `license` from the request body.

Note: `regex_entities` is optional. Use it to redact custom IDs like Order Numbers, SKUs, etc.

Response JSON:
```json
{
  "text": "Client <PERSON_1> asks about ticket <TICKET_1>",
  "state": { "...": "..." }
}
```

⚠️ Save `state`. You need it for deanonymization.

#### Step 2: De-anonymize (`POST /deanonymize`)
Restores the original data using the state from Step 1.

Body JSON:
```json
{
  "text": "Processed <TICKET_1> for <PERSON_1>",
  "license": "YOUR_LICENSE_KEY",
  "state": "{{ $('Previous Node Name').item.json.state }}"
}
```

## ✨ Supported Entities
SecureNode uses a hybrid engine (NLP Models + Strict Regex) to detect the following:

| Entity | Tag | Example | Supported? |
| --- | --- | --- | --- |
| Person Names | `<PERSON>` | John Smith, Elon Musk | ✅ |
| Emails | `<EMAIL_ADDRESS>` | john@example.com | ✅ |
| Phone Numbers | `<PHONE_NUMBER>` | +1-555-010-9999 | ✅ |
| IBAN / Finance | `<IBAN>` / `<CREDIT_CARD>` | DE89 3704... | ✅ |
| Locations/Cities | `<LOCATION>` | Berlin, New York | ✅ |
| Organizations | `<ORGANIZATION>` | Microsoft, OpenAI | ✅ |
| Dates/Time | `<DATE_TIME>` | 2025-01-01 | ✅ |
| IP Addresses | `<IP_ADDRESS>` | 192.168.1.1 | ✅ |
| Custom Regex | `<YOUR_TAG>` | Order IDs, SKUs, Internal Codes | ✅ |

## 📺 Video Tutorial
Watch the 30-second demo on [YouTube](https://youtu.be/3Wkc6_lPpCA?si=KQHABbFvZ5l8Tte_)

<div align="center">

Ready to make your automation secure?

[Get Your License Key](https://securenode.app)

Need help? [Contact support](mailto:support@securenode.app)

</div>