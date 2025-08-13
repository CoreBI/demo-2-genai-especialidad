# Vertex AI Agent Builder – Generative Search Demo (CoreBI)

This project demonstrates a **Generative AI Search Agent** built with **Google Cloud Vertex AI Agent Builder** and **Gemini**, designed to query and summarize **unstructured corporate data** (Word, PPT, PDF) stored in Google Cloud.  
It includes a lightweight **Go backend** serving a static HTML frontend with the Agent Builder widget, plus Python examples for **safety filtering** and **API integration**.

---

## 1. Overview

The solution enables secure, scalable, and context-aware search over a knowledge base, combining:

- **Frontend** – HTML/CSS interface with Agent Builder search widget.
- **Backend** – Go server for serving templates and static files.
- **Vertex AI Search + Gemini** – Retrieval-augmented generation (RAG) with summarization and safety filters.
- **Safety Filtering** – Python example for controlling model outputs.
- **API Invocation** – Example `curl` call to invoke the deployed search engine directly.

---

## 2. Architecture

**High-level components:**
1. **Client UI** – Renders the search widget and integrates with Agent Builder.
2. **Go HTTP server** – Serves HTML templates (`index.html`) from `/templates`.
3. **Vertex AI Search Engine** – Configured to index unstructured documents stored in Cloud Storage.
4. **Gemini Model** – Provides contextual summarization and natural-language synthesis.
5. **Safety Filters** – Applied at generation time to block harmful content.
6. **Deployment** – Hosted on Google Cloud, scalable via App Engine or Cloud Run.

---

## 3. Frontend

The HTML template (`index.html`) implements:
- **Responsive layout** with gradient background.
- **CoreBI branding** and descriptive content.
- **`gen-search-widget`** integration configured with `configId` for the search engine.
- Accessibility tags (`aria-label`) for input elements.
- Data source list and minimal styling for clarity.

Example widget configuration:
```html
<gen-search-widget
  configId="423bc9d4-1374-4691-ab8f-d9782f7693cc"
  triggerId="searchWidgetTrigger"
></gen-search-widget>
<script src="https://cloud.google.com/ai/gen-app-builder/client?hl=es_419"></script>
```

---

## 4. Backend (Go HTTP Server)

The backend uses a minimal Go application to:
- Parse templates from `templates/*`
- Serve the `index.html` file on the root path
- Handle dynamic port assignment via `$PORT` environment variable (App Engine/Cloud Run friendly)

**Main features:**
- **Simple deployment** – no complex routing or dependencies.
- **Error handling** – logs fatal errors when template execution fails.

Run locally:
```bash
go run main.go
```
Then visit `http://localhost:8080`.

---

## 5. Safety Filtering (Python Example)

We use **Google Vertex AI Generative AI SDK** to configure safety thresholds and system instructions for Gemini.

```python
from google import genai
from google.genai.types import GenerateContentConfig, SafetySetting, HarmBlockThreshold

client = genai.Client(vertexai=True, location="us-central1")
config = GenerateContentConfig(
    safety_settings=[
        SafetySetting(category="HARM_CATEGORY_DANGEROUS_CONTENT",
                      threshold=HarmBlockThreshold.BLOCK_LOW_AND_ABOVE),
        SafetySetting(category="HARM_CATEGORY_HARASSMENT",
                      threshold=HarmBlockThreshold.BLOCK_LOW_AND_ABOVE),
    ],
    system_instructions="Always cite the source document and avoid sensitive data exposure.",
)

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[{"role": "user", "parts":[{"text":"Explain safety compliance in Aluar docs"}]}],
    config=config,
)

if response.candidates[0].safety_feedback.blocked:
    print("Response blocked due to safety filters.")
else:
    print("Response:", response.candidates[0].content.parts[0].text)
```

---

## 6. API Invocation Example

The deployed Vertex AI Search engine can be invoked directly via REST API.

```bash
curl -X POST \
-H "Authorization: Bearer $(gcloud auth print-access-token)" \
-H "Content-Type: application/json" \
"https://discoveryengine.googleapis.com/v1alpha/projects/288743223172/locations/global/collections/default_collection/engines/demo-2_1752256599359/servingConfigs/default_search:search" \
-d '{
  "query": "<QUERY>",
  "pageSize": 10,
  "queryExpansionSpec": {"condition": "AUTO"},
  "spellCorrectionSpec": {"mode": "AUTO"},
  "languageCode": "es-419",
  "contentSearchSpec": {
    "extractiveContentSpec": {"maxExtractiveAnswerCount": 1}
  },
  "userInfo": {"timeZone": "America/Bogota"},
  "session": "projects/288743223172/locations/global/collections/default_collection/engines/demo-2_1752256599359/sessions/-"
}'
```

Replace `<QUERY>` with your search term.

---

## 7. Deployment

**Google Cloud recommended deployment:**
- **Backend & Frontend** → Deploy with **App Engine Standard** or **Cloud Run**.
- **Search Engine** → Managed in Vertex AI Search console.
- **Document Sources** → Stored in Cloud Storage, indexed via ingestion pipelines.
- **Continuous Delivery** → Use Cloud Build for automated deployment on commit.

---

## 8. Security & Privacy

- **Data storage** in Cloud Storage with IAM roles restricting access.
- **Safety filtering** at generation time to block harmful content.
- **No PII exposure** – sensitive fields masked via DLP API before ingestion.
- **HTTPS enforced** for all communications.
- **System instructions** explicitly limit model output to factual, source-grounded content.

---

## 9. Requirements

- Go 1.20+
- Python 3.9+ (for safety filter scripts)
- Google Cloud SDK with appropriate permissions
- Vertex AI API enabled
- Access to a configured Vertex AI Search Engine & Agent Builder project

---

## 10. License

© 2025 CoreBI. All rights reserved.  
For internal demo and evaluation purposes only.
