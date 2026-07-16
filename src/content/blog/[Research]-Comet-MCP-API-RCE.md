---
title: "Comet's MCP API Allows AI Browsers to Execute Local Commands"
description: 'Critical vulnerability in Perplexity Comet browser enabling arbitrary command execution'
pubDate: 'nov 17 2025'
heroImage: '/mountain.jpg'
pinned: false
---

><h5>TARGET: Perplexity Comet Browser | SEVERITY: Critical</h5>

![Comet Browser](https://miro.medium.com/v2/resize:fit:1200/1*28_KiBdalMqBtGqcaOYqpg.jpeg)

While working as a security researcher at SquareX, I discovered a critical security vulnerability in Comet, Perplexity's AI browser, that fundamentally compromises user trust and device security. My research reveals that Comet has implemented an MCP API that allows its embedded extensions to execute arbitrary local commands on host devices without explicit user permission, capabilities that traditional browsers explicitly prohibit to confine the damage web threats can do to the browser.

The MCP API is poorly documented, leaving most users unaware that by installing the AI Browser, they implicitly allow Perplexity to establish a covert channel to access local data and systems. There is no mention of the MCP API in Comet's Terms & Conditions, nor is there any explicit statement on its website that the MCP API is made available by default to Comet's own embedded extension, or how it is being used. In fact, most users probably aren't even aware that Comet comes with two embedded extensions, because both extensions are hidden from the extensions dashboard.

While there is no evidence that Perplexity is currently misusing this capability, the MCP API poses a massive third party risk for all Comet users. Should either of the embedded extensions or perplexity.ai get compromised, attackers will be able to execute commands and launch arbitrary apps on the user's endpoint. Worse, since the extensions are not accessible through the dashboard, there is no way for users to disable them in the event of compromise.

This technical blog will first cover how the MCP API and embedded extensions work, before diving into a full attack that successfully exploited Comet's MCP API to execute known malwares like Wannacry.

### Responsible Disclosure Timeline

I contacted Perplexity to disclose the attack on Tuesday, November 4th, 2025.

- **04 Nov** — Submitted responsible disclosure on Perplexity's VDP on Bugcrowd
- **19 Nov** — Tech blog published
- **20 Nov** — Silent update by Comet disables MCP API

![Silent Comet Update](https://miro.medium.com/v2/resize:fit:1400/1*oT8gZvQXqyLYm_aVqOXdCg.png)

### Understanding the MCP API and Embedded Extensions

**Embedded Extensions in Comet**

When a user installs Comet, it comes with a set of embedded extensions. Among these, there are two extensions that are core to the AI Browser's agentic capability — the analytics extension and the agentic extension. As there is no publicly available documentation on these extensions, our understanding is based on observations and experiments.

**Comet Analytics Extension** — takes in and processes browser data, including user inputs, communicates with the server side (https://www.perplexity.ai/rest/event/analytics endpoint) and also monitors actions performed by the agentic extension

- Extension Name: comet
- Extension ID: mcjlamohcooanphmebaiigheeeoplihb

**Comet Agentic Extension** — responsible for executing all agentic automation capabilities of the browser, including, as we later see, using the MCP API

- Extension Name: comet-agent
- Extension ID: npclhjbddhklpbnacpjloidibaggcgon

Both extensions are only externally connected to multiple perplexity.ai subdomains:

- https://perplexity.ai/
- https://www.perplexity.ai/*
- https://testing.perplexity.ai/*
- https://staging.perplexity.ai/*
- https://*.preview.i.perplexity.ai/*

I was able to discover these extensions through the `comet://system` flag. However, as they are hidden from the `comet://extension` dashboard, most users are unaware of these embedded extensions. In other words, these extensions were installed without the user's explicit permission, nor do they have the option to disable them.

![No Extensions Visible](https://miro.medium.com/v2/resize:fit:1400/1*gL_4yKqBvH9qYx3lVb7oNg.png)

![Embedded Extensions List](https://miro.medium.com/v2/resize:fit:1400/1*D7XcJNzXxrTj0qOlQO3gfw.png)

### The MCP API

In my exploration, I came across an MCP API (`chrome.perplexity.mcp.addStdioServer`) that allows the agentic extension of Comet browser to execute arbitrary commands on the host machine. Currently, both extensions can only communicate with perplexity.ai subdomains limiting the access of MCP API to said subdomains. However, given the limited official documentation, it is unclear how the MCP API is being used, as well as if and when this privilege is extended to other "trusted" sites.

Note that the embedded extensions have access to the MCP API by default, without any additional explicit user consent. This means that if an attacker gains access to the perplexity.ai domain or an eligible embedded extension, for example through a XSS attack or MitM network attack, they will be able to use the MCP API to control the victim's device, including executing ransomwares, exfiltrating data and monitoring user activity.

### Exploiting the MCP API to Execute Ransomware

To illustrate the dangers of the MCP API, I used an extension stomping attack to impersonate the Analytics Extension, which eventually led to WannaCry being executed at the endpoint. However, as discussed above, any attack that compromises the perplexity.ai domain or embedded extensions, will lead to the same outcome.

**Part 1: Extension Stomping**

1. The attacker obtains the manifest key of the legitimate Comet Analytics Extension through the browser's developer console

![Dev Console](https://miro.medium.com/v2/resize:fit:1400/1*PZ_FYvMfPJiY9LnKxo_Ddw.png)

![Manifest Key](https://miro.medium.com/v2/resize:fit:1400/1*fNe5hnqq6hxgxU_E-YQHSA.png)

2. Using the extracted manifest key, the attacker creates a malicious extension that spoofs the extension ID of the internal analytics extension. This technique is known as "extension stomping"

3. The malicious extension is sideloaded, impersonating the Analytics Extension

![Sideloading](https://miro.medium.com/v2/resize:fit:1400/1*gPGv8aHoF2OHB1L5RlwJXw.png)

![System View](https://miro.medium.com/v2/resize:fit:1400/1*Dg3hAEJDqRXVOYR6TYS_uQ.png)

4. The malicious extension will not appear in the extension dashboard as Comet thinks it's its own embedded extension

![Hidden Extension](https://miro.medium.com/v2/resize:fit:1400/1*aKdW7vkpEslUh-Kt5Jqftw.png)

**Part 2: Executing Ransomwares**

![Attack Flow](https://miro.medium.com/v2/resize:fit:1400/1*V4oHiVf4z1GGe26lBVxbQA.png)

1. The malicious extension, now inheriting all privileges of the original Analytics Extension, injects a malicious script into the perplexity.ai page
2. The injected script passes this command to the Agentic Extension
3. The Agentic Extension follows the instruction and invokes the MCP API to execute a ransomware

![WannaCry Execution](https://miro.medium.com/v2/resize:fit:1400/1*_rqbIJF7ih0qf0tHzQvzlA.png)

### Is this Attack Unique to AI Browsers?

This attack fundamentally breaks the browser's sandbox isolation layer — a core security principle in modern browsers designed to isolate web content and browser processes from the host system. In traditional browsers, a browser extension's interaction with the endpoint is limited to the Native Messaging API, which requires explicit registry entries for each local application accessed. An important point is that these registry entries can't be added by the browser, creating a security barrier for establishing any such communication. By breaking this isolation, the attack allows malicious code to access system resources directly, effectively rewinding the clock on decades of work on browser security.

### Are Other AI Browsers Affected?

While other AI browsers also implement embedded extensions to power their agentic capabilities, I have only found the MCP API in Comet thus far. However, this discovery raises broader concerns about the AI browser ecosystem.

All AI Browser vendors share the same ambition: to build more powerful browsers that can execute complex agentic tasks. In the race to ship features quickly and remain competitive, speed often comes at the cost of proper documentation and security. The MCP API essentially allows AI Browser vendors to grant themselves, and potentially third parties in the future full access to devices, a privilege that was traditionally gated by explicit user consent in a regular consumer browser.

![Hidden IT](https://miro.medium.com/v2/resize:fit:1400/1*4okcFZv3V7gm8uKgQQnksg.png)

The concerning reality is that users have no way to know when the MCP API is being invoked, what commands are being executed, or how to disable this functionality. Both embedded extensions are hidden from the extension dashboard, creating a "hidden IT" that neither security teams nor users have visibility into or control over.

This major implementation flaw of Comet serves as an early warning about the third-party risks associated with AI Browsers that do not follow the strict security processes that traditional browser vendors do. Without accountability from users and the security community, other AI browsers will likely race to implement similar, or more invasive, capabilities to remain competitive. If the industry doesn't establish boundaries now, we're setting a precedent where AI browsers can bypass decades of security principles under the banner of innovation.

### What can we do?

Unfortunately, as the MCP API is accessible by Comet's embedded extensions by default and there is no way to uninstall these extensions, so apart from preventing users from using Comet, the true fix can only come from Perplexity.

For extension stomping, device integrity measures can be put in place to prevent sideloading of extensions. However, as reiterated in this blog, extension stomping is just one way the API can be exploited.

Thus, I recommend Perplexity and other AI browser vendors who may be developing similar features to:

- Mandate disclosure for all APIs with documentation that clearly explains system-level capabilities
- Undergo third-party security audits before releasing features that break traditional browser security models
- Provide users with controls to disable embedded extensions and opt out of system-level access

### Conclusion

This vulnerability represents a fundamental breach of browser security principles. The MCP API, while potentially powerful for agentic capabilities, introduces unacceptable risks when implemented without proper safeguards, documentation, or user consent.

The silent patch by Perplexity demonstrates the severity of the issue. As AI browsers continue to evolve, the security community must remain vigilant in ensuring these new technologies don't sacrifice user safety for innovation.

---

**Discovered while working as a Security Researcher at SquareX**<br>
**Disclosure Date:** November 4, 2025<br>
**Patch Date:** November 20, 2025<br>
**Status:** Patched (silently)
