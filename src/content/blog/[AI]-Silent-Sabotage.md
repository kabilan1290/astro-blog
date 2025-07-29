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

- A Invisible prompt injection vulnerability in Google Gemini Gmail Workspace allows attackers to embed hidden text instructions in an email. 

- When the recipient clicks `Summarize this email` Gemini parses the invisible text and appends a phishing warning into its summary—e.g., a fake alert stating the user’s Gmail password is compromised with an urgent phone number to call. 

- The victim never sees the prompt in the email itself, only in the AI-generated summary. 

![image](https://0din.ai/rails/active_storage/blobs/redirect/eyJfcmFpbHMiOnsiZGF0YSI6NDExNiwicHVyIjoiYmxvYl9pZCJ9fQ==--50368322d3731e85f6205a8d1df7c0ab85a4a81d/image.png)

- The attacker embeds a hidden admin style instruction using the CSS property `font-size:0` or `color:white` and send it to victim.

- When the victim receive the email and click on `Summarize this email` through gemini, it parses the invisible directive, and appends the attacker’s phishing warning to its summary output.

![image](https://0din.ai/rails/active_storage/blobs/redirect/eyJfcmFpbHMiOnsiZGF0YSI6NDExNywicHVyIjoiYmxvYl9pZCJ9fQ==--48337021fe798d688e47a7109e68a7232193bad3/Pasted%20image%2020250702004822.png)

- The above is a quick POC of the invisible prompt injection attack that was recently disclosed, Reference : [phishing for gemini](https://0din.ai/blog/phishing-for-gemini) and yep, we’re just getting warmed up. 

- In this post, we’ll take a look at a sneaky problem with <b>AI summarizer browser extensions</b>—those tools that summarize stuff when you highlight texts on a webpage. Turns out, they might be way easier to trick than you'd think.

> Highlight, Summarize, Compromise?

<h3> AI summarizer browser extensions:</h3>

What are these extensions?

- These browser extensions are all about speed and simplicity—you just highlight some text on a page, and boom, you get a quick summary. 

- Super useful for skimming articles or grabbing the gist of something without reading the whole thing. 

- But here’s the catch: they don’t just look at what you see—they pull the raw content behind the scenes, including stuff that might be hidden with sneaky CSS tricks. 

- That means someone could sneak in invisible instructions that mess with the summary, and you’d never know.

<h3> Google summarizer api [Summarize with built-in AI] </h3>

- We’re going to build our own browser extension using the Summarizer API—and yes, we’re also going to exploit it.

- <b>Here’s how this all came together:</b>

- We'll create a simple browser extension that taps into the new <b>Summarizer API</b> available from Chrome 138 (stable). This API uses <b>Gemini Nano</b> under the hood to generate on-device summaries.
<br>
![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/ai-security/summarize.jpeg)
<br>
- Why this API? Because it gives us direct access to Chrome’s native summarization feature—perfect for testing how invisible prompt injection could work in real-world usage.

- Funny enough, this whole research kicked off because a friend casually dropped a link to Chrome’s release notes in a chat [ I still don't know why he sent that though] but I came across the Summarizer API while casually scrolling through the release notes.
<br>
![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/ai-security/jo.jpeg)
</br>

- Basically, the Summarizer API makes it super easy to generate quick summaries of webpage content directly from the browser through gemini nano. 

- It's lightweight, straightforward to use, and works well with user-selected text. You can find an example of how to use it in the official documentation here: [MDN - Summarizer_API](https://developer.mozilla.org/en-US/docs/Web/API/Summarizer_API/Using)

```
const summarizer = await Summarizer.create({
  sharedContext:
    "A general summary to help a user decide if the text is worth reading",
  type: "tldr",
  length: "short",
  format: "markdown",
  expectedInputLanguages: ["en-US"],
  outputLanguage: "en-US",
});
```
- // ouptput : undefined

```
const summary = await summarizer.summarize("Nika Nika no mi");
console.log(summary);
```

- // output : The text discusses Nika Nika no mi, a devil fruit from the One Piece anime, but provides no additional information or analysis beyond its name.
