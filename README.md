# 📞 AI Calling Agent using Twilio

An AI-powered voice calling agent that handles real phone calls via **Twilio ConversationRelay**. It understands natural speech, answers medical FAQ questions using RAG, books appointments, and logs everything to **Google Sheets** — all in real time.

---

## ✨ Features

- 📲 **Live Phone Call Handling** — Connects to real phone calls via Twilio ConversationRelay
- 🤖 **AI Voice Assistant** — Powered by Google Gemini 2.5 Flash with a natural, conversational voice (ElevenLabs TTS)
- 🔍 **RAG-based FAQ Answering** — Retrieves answers from a medical FAQ knowledge base using Pinecone + Sentence Transformers
- 📅 **Appointment Booking** — Detects booking intent and schedules appointments from natural speech
- 📊 **Google Sheets Logging** — Automatically logs all Q&A interactions and booked appointments
- 🔄 **Daily FAQ Sync** — Scheduler re-ingests FAQ data into Pinecone every day at 1AM

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|------------|
| Backend | FastAPI + Uvicorn |
| LLM | Google Gemini 2.5 Flash |
| TTS | ElevenLabs (via Twilio) |
| Telephony | Twilio ConversationRelay |
| Vector DB | Pinecone |
| Embeddings | Sentence Transformers (`all-MiniLM-L6-v2`) |
| Logging | Google Sheets (gspread) |
| Tunneling | ngrok |
| Scheduler | APScheduler |

---

## 👥 Team

| Name | GitHub |
|------|--------|
| Shruti Khisa | [@ShrutiKhisa](https://github.com/ShrutiKhisa) |
| Farhan Tanvir | [@FarhanTanvir](https://github.com/FarhanTanvir) |
| Shaira Akhter Diba | [@ShairaDiba](https://github.com/ShairaDiba) |
| Fazli Rabbi Noor | [@FarhanNoor](https://github.com/FarhanNoor) |

---

## 📁 Project Structure

```
calling-agent-using-twillo/
├── main.py                   # FastAPI app — TwiML endpoint + WebSocket handler
├── rag_query.py              # RAG retrieval using Pinecone + Gemini
├── rag_ingest.py             # Ingests FAQ data into Pinecone
├── appointment_scheduler.py  # Appointment booking logic
├── google_sheets_logger.py   # Logs Q&A and appointments to Google Sheets
├── parse_info.py             # Extracts appointment details from speech using Gemini
├── intent.py                 # Basic intent detection
├── scheduler.py              # Daily FAQ re-ingestion scheduler
├── Data/
│   └── faq.txt               # Medical FAQ knowledge base
└── requirements.txt
```

---

## ⚙️ How It Works

1. **Twilio** receives an inbound call and hits the `/twiml` endpoint
2. The server returns TwiML that opens a **WebSocket** connection via ConversationRelay
3. The caller's speech arrives as text over the WebSocket
4. **Gemini** classifies the message as either:
   - `appointment` → extracts name, date, time → checks availability → books and logs to Google Sheets
   - `qa` → answers using the RAG pipeline (Pinecone + Gemini) → logs Q&A to Google Sheets
5. The response text is sent back over WebSocket and spoken aloud by **ElevenLabs TTS**

---

## 🚀 Getting Started

### 1. Install dependencies

```bash
pip install -r requirements.txt
```

### 2. Create a `.env` file

```env
GOOGLE_API_KEY=your_google_gemini_api_key
NGROK_URL=your_ngrok_domain_without_https
PINECONE_API_KEY=your_pinecone_api_key
PINECONE_INDEX=your_pinecone_index_name
GSPREAD_CREDS_JSON={"type":"service_account", ...}   # Full JSON as a single string
PORT=8080
```

### 3. Ingest FAQ data into Pinecone

```bash
python scheduler.py
```

### 4. Start the server

```bash
python main.py
```

### 5. Expose with ngrok

```bash
ngrok http 8080
```

Copy the ngrok HTTPS URL (without `https://`) and set it as `NGROK_URL` in your `.env`.

### 6. Configure Twilio

In your Twilio phone number settings, set the **"A call comes in"** webhook to:

```
https://your-ngrok-url/twiml
```

Now call your Twilio number — the AI agent will answer!

---

## 📊 Google Sheets Setup

Create two Google Sheets in your Google Drive:

| Sheet Name | Columns |
|------------|---------|
| `Calling_Agenet_Log` | Question, Answer |
| `Appointments` | Timestamp, Name, Date, Time |

Share both sheets with your **service account email** (from your `GSPREAD_CREDS_JSON`) as an Editor.

---

## 🔄 Daily FAQ Sync

The `scheduler.py` script re-ingests `Data/faq.txt` into Pinecone once immediately on start, then again every day at 1AM. Run it separately if you want automatic updates:

```bash
python scheduler.py
```
