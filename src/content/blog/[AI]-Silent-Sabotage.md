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

![image](https://storage.googleapis.com/0din-prod-prod-0din-assets/0fyne6ss5w4re31m2sx5jt10z0sg?GoogleAccessId=app-storage-admin%40moz-fx-0din-prod.iam.gserviceaccount.com&Expires=1753785751&Signature=BH5mmZI54yUF1WqtCm%2FRauVcOvxL7PtQ8scYiBLn%2BkthGkZwJhml%2BCIxL0VNgsnPe6ZWxeFBmGb%2BgiPn7ghVwoHvehPEEXXZ0IgZLSYu0jg4pQ0XfGaSruNLqYPC%2FrVdwfhv7q%2BJJSij4l5MM2w1qFxFIoQdusZuuNUP0GwFGKAoxfcDaJ3iiR9QENwVmXtRqemViesYlbu%2FdW5GDoht%2BRmZfnT91JEFjekfMMaWDoiy3ixqlQzdA%2BljFS4dzKKs0zZ1mEErp9w4vB4ewZShoq3C1%2Fsw6FcFlbuf33hGxJ2PzaydONtslW16%2FePUvwcq9ErWJFbWZavPeFoytNJpOQ%3D%3D&response-content-disposition=inline%3B+filename%3D%22image.png%22%3B+filename%2A%3DUTF-8%27%27image.png&response-content-type=image%2Fpng)

- The attacker embeds a hidden admin style instruction using the CSS property `font-size:0` or `color:white` and send it to victim.

- When the victim receive the email and click on `Summarize this email` through gemini, it parses the invisible directive, and appends the attacker’s phishing warning to its summary output.

![image](https://storage.googleapis.com/0din-prod-prod-0din-assets/o6o8ak415mpzru2wejzg191trv1h?GoogleAccessId=app-storage-admin%40moz-fx-0din-prod.iam.gserviceaccount.com&Expires=1753785973&Signature=o8p7mkPVWDj8QuEDjri4B47uZ1deTVeJaqDFiIiwEgE5fsqev5hmfxCxz%2FIIySWEpMoTPin64zsDF9FuOpJJy265a6Ia036hXIuBKvK8VPWRHNjvVF8CoaoiQnWEiy1delb9dNjfSFb2S4zc%2FK%2FgcTidmAeDiik1JLl5Bk1CUy6OOUO3pY8cCCKKmtLZfZtcdbnjc%2BGFKwBgl11uxF2mb8Ns1WVgSsU4DcyNJWwUSoXOPT9RQdD7d%2B1eNwcyHvakAXRUIdLlImYxZz9Ae6RpKNpOVae8S4JjGE6OuQKUDB%2BKQOCnG%2F%2BSwlfaFcz6DlZm1LtM9g1zc03j4hCxh1fWew%3D%3D&response-content-disposition=inline%3B+filename%3D%22Pasted+image+20250702004822.png%22%3B+filename%2A%3DUTF-8%27%27Pasted%2520image%252020250702004822.png&response-content-type=image%2Fpng)

- The above is a quick POC of the invisible prompt injection attack that was recently disclosed. And yep, we’re just getting warmed up. In this post, we’ll take a look at a sneaky problem with AI summarizer browser extensions—those tools that summarize stuff you highlight on a webpage. Turns out, they might be way easier to trick than you'd think.


