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

[ **Get Docker Image** ](https://securenode.app) | [ **Try Cloud Demo** ](#-try-it-instantly-cloud-demo) | [ **Documentation** ](#-installation)

</div>

---

## 🚨 The Problem
Sending raw client data (Names, Emails, IBANs) to public LLMs is a privacy risk and often violates **GDPR**, **CCPA**, and **SOC2**.

* ❌ **Risk:** Data leaks into public training sets.
* ❌ **Legal:** Contracts often forbid sending PII to 3rd party AI.
* ❌ **Trust:** Enterprise clients refuse automation due to privacy concerns.

## ✅ The Solution: "The Sandwich Method"
SecureNode acts as a local reversible proxy. No personal data leaves your infrastructure.

1.  **⬇️ Input:** `Client John Doe asks for a refund.`
2.  **🛡️ Anonymize (Local):** `Client <PERSON_1> asks for a refund.` (Sent to AI)
3.  **🤖 AI Processing:** `Approve refund for <PERSON_1>.`
4.  **✅ De-anonymize (Local):** `Approve refund for John Doe.`

---

## 🚀 Try it Instantly (Cloud Demo)
You don't need to install anything to test the logic. We hosted a public endpoint for demonstration.

> **⚠️ Warning:** This demo runs on our public server. Do not send real sensitive PII. For production, use the Docker version below.

1.  **[Click here to view the Workflow JSON](workflows/cloud-demo.json)**
2.  Copy the code (`Ctrl+A` -> `Ctrl+C`).
3.  Paste it into your n8n editor (`Ctrl+V`).

---

## 🐳 Installation (Production)
For full privacy, run SecureNode locally alongside your n8n instance.

### 1. Docker Compose
Add this service to your `docker-compose.yml`:

```yaml
services:
  securenode:
    image: vankir/securenode:latest
    container_name: securenode
    restart: always
    environment:
      - LICENSE_KEY=BETA-2025  # Free Beta Key
    ports:
      - "5000:5000"