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
    restart: always
    ports:
      - "5000:5000"
    deploy:
      resources:
        limits:
          memory: 1G
```

### 🔌 API Usage (for n8n HTTP Request)
To connect SecureNode to n8n, use the HTTP Request node.

Base URL:
- `http://securenode:5000` (inside Docker network)
- `http://localhost:5000` (from your host)

Headers:
- `Content-Type: application/json`

#### Step 1: Anonymize (`POST /anonymize`)
Detects PII and replaces it with tokens. Returns a state object needed for decryption.

Body JSON:
```json
{
  "text": "Client John asks about ticket TIC-9999",
  "license": "securenode2025beta",
  "regex_entities": [
    { "name": "TICKET", "regex": "TIC-\\d{4}" }
  ]
}
```

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
  "license": "securenode2025beta",
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
Watch the 30-second demo on YouTube

<div align="center">

Ready to make your automation secure?

Get Your License Key

</div>