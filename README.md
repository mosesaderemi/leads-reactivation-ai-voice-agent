# Leads Reactivation AI Voice Agent

An automation service that phones back old sales leads with an AI voice agent. It pulls leads from a CRM, makes each call through a voice platform, analyses the transcript with a language model, and writes the outcome back to the CRM so a human can follow up on interested prospects.

## How It Works

1. A client sends `POST /execute` with a list of lead IDs.
2. The service loads those leads from the configured CRM (Airtable, Google Sheets or HubSpot).
3. Optional pre-call enrichment runs, then the lead data is packaged as call variables.
4. A call is placed with the voice agent (Vapi is wired up in `app.py`, with Retell AI and OpenAI Realtime provider classes also included).
5. When the call ends, the voice platform posts an event to `POST /webhook`.
6. The transcript is analysed by an LLM to produce a summary, an interest flag and a justification.
7. The lead record is updated with status `CONTACTED`, call ID, call status, duration, cost, end reason, transcript, summary, interest and comment.

The default agent persona is a sales representative re-engaging prospects who previously asked about computer hardware, with a defined conversation flow for greeting, interest check, objection handling and detail capture. The script lives in `src/prompts.py` and is easy to adapt to another industry.

## Features

- FastAPI service with interactive API docs at `/docs`
- Pluggable CRM layer with Airtable, Google Sheets and HubSpot loaders, and a base class for adding your own (see `docs/customization.md`)
- Pluggable voice providers: Vapi, Retell AI and OpenAI Realtime
- Automatic transcript analysis and CRM updates after every call
- Calendar tool for booking follow-up appointments
- Helper scripts to create or update the voice assistant and tools and to import a phone number

## Tech Stack

| Area | Technology |
| --- | --- |
| Language | Python 3 |
| API | FastAPI, Uvicorn, Gunicorn |
| Voice | Vapi, Retell AI, OpenAI Realtime |
| LLM | LiteLLM |
| CRM and data | Airtable (pyairtable), HubSpot, Google Sheets |
| Calendar | Google Calendar API |

## Project Structure

```text
.
├── app.py                          FastAPI app: /execute and /webhook
├── requirements.txt
├── .env.example
├── docs/customization.md           Adding a custom CRM
├── scripts                         Create or update assistant and tools, import phone number
└── src
    ├── vapi_automation.py          Call workflow and post-call CRM update
    ├── prompts.py                  Voice agent prompt
    ├── tools
    │   ├── call_analysis.py        Transcript analysis
    │   └── calendar_tool.py        Appointment booking
    └── base
        ├── leads_loader            Airtable, Google Sheets, HubSpot, base class
        └── voice_agent_providers   Vapi, Retell AI, OpenAI Realtime
```

## Getting Started

### Prerequisites

- Python 3.10 or later
- A Vapi account with an assistant and a phone number (or a Retell AI account)
- An OpenAI key for transcript analysis
- A CRM table with lead fields such as first name, last name, phone and a `Status` column
- A public URL for the webhook; use a tunnelling tool such as ngrok during development

### Installation

```bash
git clone https://github.com/mosesaderemi/leads-reactivation-ai-voice-agent.git
cd leads-reactivation-ai-voice-agent
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
copy .env.example .env
```

Fill in `.env`. Keys are grouped by service: Vapi, Retell, OpenAI, LangSmith, Airtable, HubSpot, Google Sheets, Twilio and `SERVER_URL` for your public address.

### Prepare the voice assistant

```bash
python scripts/create_or_update_tool.py
python scripts/create_or_update_assistant.py
python scripts/import_phone_number.py
```

### Run

```bash
python app.py
```

The API listens on port 8000. Open `http://localhost:8000/docs`.

### Start a campaign

```bash
curl -X POST http://localhost:8000/execute \
  -H "Content-Type: application/json" \
  -d '{"lead_ids": ["recXXXXXXXX", "recYYYYYYYY"]}'
```

Configure Vapi to send end-of-call events to `<SERVER_URL>/webhook`.

## Responsible Use

Automated outbound calls are regulated in many countries. Obtain the consent required by local law, honour opt-outs, and make sure the agent identifies itself appropriately.

## Author

Aderemi Moses Timileyin
GitHub: [@mosesaderemi](https://github.com/mosesaderemi)
LinkedIn: [linkedin.com/in/mosesaderemi-6a70a5405](https://www.linkedin.com/in/mosesaderemi-6a70a5405)

## License

Released under the MIT License. See the `LICENSE` file for details.
