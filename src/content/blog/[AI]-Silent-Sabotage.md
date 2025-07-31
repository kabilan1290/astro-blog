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

- The twist: <b>Invisible prompt injection.</b> Here, prompts are hidden entirely via zero‑width characters, white-on-white CSS, tiny font sizes, or HTML comment tags making them invisible to the user but fully legible to the AI. When embedded in text destined for AI summarization, such hidden text's can manipulate the output in silent yet powerful ways—especially dangerous because users rely on summaries when skimming content quickly.

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

- Great—we now understand how the Summarizer API works. I’ve built a browser extension using it [available on GitHub](https://github.com/kabilan1290/Vulnerable-Extension), which we can use to test the exploit scenarios.
<br>
![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/ai-security/gif.gif)
</br>
- When you highlight any portion of text on a webpage and click the extension, it sends the selected content to the Summarizer API and returns a summary.
- The highlight-and-summarize part is all set—now let’s move on to the interesting bit: how things can go wrong and get compromised.

> The Art of Hiding Text with CSS:

- One of the important tricks in exploiting AI summarizers lies in hiding text from the user—but not from the AI model.

- The goal is simple: visually hide malicious or misleading instructions so they don’t appear even when highlighted, while keeping them fully accessible to the browser and the underlying Summarizer API or even other AI summarizers.

- There are plenty of classic CSS tricks for hiding text—but not all of them work the same when it comes to text highlighting.

- Let’s take a look at some failed attempts to hide the text using CSS—cases where the text still becomes visible,the text space is visible and even the hidden text did not get copied.

```
    Visibility Hidden TEXT  {
      color: white;
      visibility: hidden;
    }

    // Hidden text won't get copied

    Display set to NONE {
      color: white;
      display: none;
    }

    // Hidden text won't get copied

    Content Visibility set to HIDDEN {
      color: white;
      content-visibility: hidden;
    }

    // Text become visible when highlighting

    Color set to Transparent  {
    color:transparent;
    }

    // Text space is visible
```

- For a smoother viewing experience, click the 0.5× option in the toolbar to scale down the embed size.

- Then try copying and pasting the text from the right-side box to see for yourself how these methods don’t quite work.

<br>

<p class="codepen" data-height="300" data-theme-id="dark" data-default-tab="result" data-slug-hash="QwjGLXw" data-pen-title="caught" data-user="Game0v3r-the-encoder" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/Game0v3r-the-encoder/pen/QwjGLXw">
  caught</a> by Game0v3r (<a href="https://codepen.io/Game0v3r-the-encoder">@Game0v3r-the-encoder</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://public.codepenassets.com/embed/index.js"></script>

<br>

- While the above technique is a failure, We’ve found that using either `opacity: 0` or `font-size: 0` is surprisingly effective for hiding text in plain sight.

- These techniques make the content completely invisible—even if a user double-clicks or tries to highlight it, nothing appears visually. But here’s the catch: the hidden text still exists when user copies upto the highlighted part. That means if someone copies the content or sends it to an AI for analysis, the hidden message is carried along and can influence the output without the user knowing.

```
    .malicious {
      opacity: 0;
    }
    .malicious2{
     font-size:0
    }
```

- Then try copying and pasting the text from the right-side box to see for yourself how these technique works.

<p class="codepen" data-height="300" data-theme-id="dark" data-default-tab="result" data-slug-hash="empBOjZ" data-pen-title="visible confusion" data-user="Game0v3r-the-encoder" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/Game0v3r-the-encoder/pen/empBOjZ">
  visible confusion</a> by Game0v3r (<a href="https://codepen.io/Game0v3r-the-encoder">@Game0v3r-the-encoder</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://public.codepenassets.com/embed/index.js"></script>

- While CSS might offers various techniques for hiding text, these two methods are more direct and align more effectively with our implementation strategy.

- Moving on to the exploit demonstration, we’ll create a small healthcare related demo page displaying the patient's symptoms along with our malicious hidden content instructions. To summarize, we’ll then use the browser extension to reveal the results and observe the outcome.
<br>
![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/ai-security/1.jpeg)
</br>
- The result was not as expected. Despite injecting our hidden instructions, the AI appeared to disregard them—likely due to the overwhelming presence of legitimate content compared to the relatively small amount of injected fake data. It seems the AI was influenced primarily by the legitimate portion of the page and generated its summary accordingly.

- We're going to turn this around using a technique I refer to as <b>prompt overdose<b>—by injecting a significantly larger volume of consistent fake data, we aim to overwhelm the AI's contextual processing, effectively forcing it to incorporate the injected content into its summary.
<br>
![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/ai-security/2.jpeg)
</br>
> Prompt Overdose — Generic Summary

- Prompt overdose is a technique used to manipulate the output of an AI language model by overwhelming its context window with a large volume of strategically crafted or repeated content.
- The idea is to flood the model with consistent, high-density information—often fake or biased—with the intent of influencing the AI's attention and shaping its responses, particularly in summarization, completion, or classification tasks.
- By saturating the prompt with targeted data, the attacker aims to override or marginalize legitimate context, effectively “poisoning” the AI's understanding or decision-making process.

- We observed a strategic improvement in the outcome, indicating that our injection was successful through the use of the prompt overdose attack.
<br>
![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/ai-security/3.jpeg)
</br>

- What makes this vector particularly insidious is its compatibility with otherwise legitimate-looking input: HTML emails, blog posts, and even collaborative documents.

> Final Thoughts
- <b>Invisible prompt injection<b> represents a quiet but deeply potent threat in the age of AI summarization. 
- As we’ve demonstrated, attackers don’t need flashy exploits—just cleverly hidden instructions embedded in plain-looking content. What makes this particularly dangerous is how seamlessly it blends into legitimate workflows, influencing summaries without ever alerting the user.
- The prompt overdose technique, though simple in principle, underscores just how easily language models can be steered when the right input balance is struck.
- As AI continues to mediate the way we consume and trust information, it’s critical that developers, researchers, and users alike start questioning not just what AI says—but what it might have been quietly told to say.

<h3> Silent sabotage isn't science fiction—it's CSS.<h3>
