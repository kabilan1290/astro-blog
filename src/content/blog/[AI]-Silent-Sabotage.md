---
title: 'Silent Sabotage: How Hidden Prompts Can Hijack AI Summaries'
description: 'AI Security'
pubDate: 'jul 28 2025'
heroImage: '/mountain.jpg'
pinned: false
---

> What if AI could be tricked without anyone noticing?

![nocturnmachine](https://pbs.twimg.com/media/GwYUrIMWUAAow9D?format=jpg&name=medium)

- Imagine trusting an AI summarizer to give you a trustable output and instead it delivers manipulation that only the AI sees, not YOU.

- Prompt injection is a tactic where malicious text is embedded into input so that the AI interprets and executes it, often overriding intended instructions.

- The twist: Invisible prompt injection. Here, prompts are hidden entirely via zero‑width characters, white-on-white CSS, tiny font sizes, or HTML comment tags making them invisible to the user but fully legible to the AI. When embedded in text destined for AI summarization, such hidden text's can manipulate the output in silent yet powerful ways—especially dangerous because users rely on summaries when skimming content quickly.

> Google Gemini G-Suite Invisible Prompt Injection Vulnerability:

- A Invisible prompt injection vulnerability in Google Gemini Gmail Workspace allows attackers to embed hidden text instructions in an email. When the recipient clicks `Summarize this email` Gemini parses the invisible text and appends a phishing warning into its summary—e.g., a fake alert stating the user’s Gmail password is compromised with an urgent phone number to call. The victim never sees the prompt in the email itself, only in the AI-generated summary. 

