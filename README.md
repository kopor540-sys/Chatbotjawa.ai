# Chatbotjawa.ai
Ai jawa

chatbot-ai-render/
├─ public/
│  └─ index.html
├─ server.js
├─ package.json
└─ .env

<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Chatbot AI</title>
  <style>
    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: linear-gradient(135deg, #eef4ff, #f8fbff);
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      padding: 16px;
    }
    .chatbox {
      width: 100%;
      max-width: 720px;
      background: #fff;
      border-radius: 18px;
      box-shadow: 0 12px 35px rgba(0,0,0,0.12);
      overflow: hidden;
      display: flex;
      flex-direction: column;
      height: 85vh;
    }
    .header {
      background: #2563eb;
      color: #fff;
      padding: 16px 20px;
      font-size: 18px;
      font-weight: 700;
    }
    .messages {
      flex: 1;
      padding: 18px;
      overflow-y: auto;
      border-bottom: 1px solid #e5e7eb;
      background: #fafafa;
    }
    .msg {
      margin-bottom: 12px;
      padding: 11px 14px;
      border-radius: 12px;
      max-width: 85%;
      line-height: 1.5;
      white-space: pre-wrap;
      word-wrap: break-word;
    }
    .user {
      background: #dbeafe;
      margin-left: auto;
      text-align: right;
    }
    .bot {
      background: #f3f4f6;
    }
    .input-area {
      display: flex;
      gap: 10px;
      padding: 14px;
      background: #fff;
    }
    input {
      flex: 1;
      padding: 14px;
      border: 1px solid #d1d5db;
      border-radius: 12px;
      outline: none;
      font-size: 15px;
    }
    button {
      padding: 14px 18px;
      border: none;
      border-radius: 12px;
      background: #2563eb;
      color: white;
      cursor: pointer;
      font-size: 15px;
      font-weight: 600;
    }
    button:disabled {
      opacity: 0.6;
      cursor: not-allowed;
    }
  </style>
</head>
<body>
  <div class="chatbox">
    <div class="header">Chatbot AI</div>
    <div class="messages" id="messages">
      <div class="msg bot">Halo! Saya chatbot AI. Tanyakan apa saja.</div>
    </div>
    <div class="input-area">
      <input id="input" type="text" placeholder="Tulis pesan..." />
      <button id="sendBtn">Kirim</button>
    </div>
  </div>

  <script>
    const messages = document.getElementById("messages");
    const input = document.getElementById("input");
    const sendBtn = document.getElementById("sendBtn");

    function addMessage(text, type) {
      const el = document.createElement("div");
      el.className = `msg ${type}`;
      el.textContent = text;
      messages.appendChild(el);
      messages.scrollTop = messages.scrollHeight;
      return el;
    }

    async function sendMessage() {
      const text = input.value.trim();
      if (!text) return;

      addMessage(text, "user");
      input.value = "";
      sendBtn.disabled = true;

      const typing = addMessage("Mengetik...", "bot");

      try {
        const res = await fetch("/chat", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ message: text })
        });

        const data = await res.json();
        typing.remove();
        addMessage(data.reply || "Maaf, ada error.", "bot");
      } catch (err) {
        typing.remove();
        addMessage("Gagal terhubung ke server.", "bot");
      } finally {
        sendBtn.disabled = false;
      }
    }

    sendBtn.addEventListener("click", sendMessage);
    input.addEventListener("keypress", (e) => {
      if (e.key === "Enter") sendMessage();
    });
  </script>
</body>
</html>

const express = require("express");
const path = require("path");
const dotenv = require("dotenv");
const OpenAI = require("openai");

dotenv.config();

const app = express();
const port = process.env.PORT || 3000;

const client = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

app.use(express.json());
app.use(express.static(path.join(__dirname, "public")));

app.post("/chat", async (req, res) => {
  try {
    const userMessage = req.body.message || "";

    const response = await client.responses.create({
      model: "gpt-4o-mini",
      input: [
        {
          role: "system",
          content: "Kamu adalah asisten chatbot yang ramah, singkat, jelas, dan membantu."
        },
        {
          role: "user",
          content: userMessage
        }
      ]
    });

    res.json({ reply: response.output_text });
  } catch (error) {
    res.status(500).json({ reply: "Terjadi error saat memanggil OpenAI API." });
  }
});

app.listen(port, () => {
  console.log(`Server berjalan di port ${port}`);
});
{
  "name": "chatbot-ai-render",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "dotenv": "^16.4.5",
    "express": "^4.21.2",
    "openai": "^4.0.0"
  }
}
OPENAI_API_KEY=isi_api_key_kamu
