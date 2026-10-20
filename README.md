# Pathan002
World changing9

import "dotenv/config";
import express from "express";
import OpenAI from "openai";
import rateLimit from "express-rate-limit";

const app = express();
const PORT = process.env.PORT || 3000;

app.use(express.json({ limit: "10kb" }));
app.use(express.static("public"));

app.use("/api/ask", rateLimit({
  windowMs: 60 * 1000,
  limit: 10,
  standardHeaders: true,
  legacyHeaders: false
}));

const client = process.env.OPENAI_API_KEY
  ? new OpenAI({ apiKey: process.env.OPENAI_API_KEY })
  : null;

app.get("/api/health", (req, res) => {
  res.json({
    ok: true,
    aiConfigured: Boolean(client)
  });
});

app.post("/api/ask", async (req, res) => {
  try {
    if (!client) {
      return res.status(503).json({
        error: "AI is not configured. Add OPENAI_API_KEY in Secrets."
      });
    }

    const question = req.body?.question;

    if (typeof question !== "string" ||
        question.trim().length === 0 ||
        question.length > 2000) {
      return res.status(400).json({
        error: "Please enter a question under 2000 characters."
      });
    }

    const response = await client.responses.create({
      model: "gpt-5-mini",
      instructions:
        "You are USE AI, a helpful personal assistant created for MODASSIR KHAN. " +
        "Public profile ID: MODASSIR-001. Answer clearly, accurately, and " +
        "in the language used by the user. For Hinglish questions, reply in Hinglish. " +
        "Use simple explanations and steps when useful. Do not claim to be a human.",
      input: question.trim(),
      max_output_tokens: 700
    });

    res.json({
      answer: response.output_text || "Sorry, no answer was generated."
    });
  } catch (error) {
    console.error("OpenAI request failed:", error.status || error.name);

    res.status(500).json({
      error: "AI could not answer right now. Check API billing, key and server logs."
    });
  }
});

app.listen(PORT, "0.0.0.0", () => {
  console.log(`USE AI server running on port ${PORT}`);
});
