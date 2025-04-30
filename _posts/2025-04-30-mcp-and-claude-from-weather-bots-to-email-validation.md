---
layout: post
title: 'MCP & Claude: From Weather Bots to Email Validation'
date: 2025-04-30 16:00 +0300
categories: [Development]
tags: [mcp,ai]
image:
  path: https://images.unsplash.com/photo-1525338078858-d762b5e32f2c?fm=jpg&q=60&w=3000&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
---
Hello 👋

As everyone working with AI knows, MCP has been a popular topic lately. I wanted to learn about this topic and create a few applications. I was looking into it in my spare time after work, but hadn't made much progress. The innovation week we have at the company after each cycle was an opportunity for me. During this innovation week, I set out to learn about MCP and develop at least one application.

When I first encountered Claude’s Model Context Protocol (MCP) support, it looked simple enough. A quick demo tool that checks the weather? Nice. It made me think: “This could be useful. Maybe I’ll try building something small of my own.”

That “something small” turned into a week-long adventure full of surprises, frustrations, and unexpected wins. I started with Claude’s weather tool, built a football match API on top of it, and then dove head-first into a CSV email validator. What I thought would be a couple of lines of code turned into a masterclass in JSON-RPC, tool registration, prompt design, and Claude’s tendency to... make things up.

## Step 1: The Weather Tool (from MCP Docs)
My MCP journey began with the example tool provided in the [official Model Context Protocol documentation](https://github.com/modelcontext/spec). It was a weather bot — a small service that, when asked, could return the weather in a given city using a public API.

I didn’t build it myself, but running it was my first taste of how Claude interacts with tools using JSON-RPC, how `zod` is used to validate input schemas, and how responses are structured in a way Claude can understand and present back to the user.

It was clean, simple, and gave me just enough confidence to try building my own tool. That first step was important: it showed me how Claude calls a tool, what metadata it expects, and what it means when a tool “appears” in Claude’s UI.

Of course, I didn’t yet know that getting Claude to call *my* tool — and use the real data I gave it — would become a battle of prompt phrasing, schema formatting, and taming Claude’s sometimes overly helpful hallucinations.

## Step 2: Building My Own – The Football Tool
After playing around with the weather example, I wanted to try something of my own. I’m a football fan, so I decided to build a tool that could fetch the next match for any Premier League team using the [Football Data API](https://www.football-data.org/).

This was my first custom MCP tool, and it taught me a lot:

- I used `fetch` to query real-time data from the API
- I created a simple schema where Claude could pass in a team name like `"Liverpool"` or `"Arsenal"`
- I returned a friendly summary: next match date, teams, and venue

Most importantly, I learned how to register a tool using the method I preferred:

```js
const server = new McpServer({
  name: "football",
  version: "1.0.0",
  capabilities: {
    tools: {},
    resources: {},
  },
});
```

Then, I added the tool with `server.tool(...)` instead of defining tools inline. This gave me more control and matched the structure I wanted to follow going forward.

Claude responded beautifully. I could upload a file or just type “What’s the next match for Tottenham?” and the tool did its job. It was a win — both technically and emotionally.

I felt like I understood the flow. And then I tried to build a tool that reads a CSV file…

//image

## Step 3: The Email Checker Tool (aka The Boss Fight)

After the football tool, I was feeling confident. So I challenged myself to build something more complex: a tool that would read a CSV file, find rows with invalid or missing email addresses, and return a report.

Sounds simple, right?

It wasn’t.

### What I wanted:

- Let Claude users upload a CSV
- Check the `email` column for missing or invalid entries
- Return structured output with error rows

### What actually happened:

I spent days untangling edge cases and weird behaviors. Here are just a few:

#### 1. Tool didn’t show up in Claude
I initially defined my tool like this:

```js
const server = new McpServer({
  name: "email-checker",
  version: "1.0.0",
  tools: [ ... ],
});
```

But Claude wouldn’t list it. After some digging, I realized Claude only recognizes tools when they're registered through the capabilities.tools structure or added dynamically via server.tool(...). Once I switched to the football-style setup, it finally worked.

#### 2. Console logs crashed the JSON parser
My tool used `console.log()` to show debug info like “⚙️ Tool called” or “📥 Received file from Claude.”

Claude interpreted this as invalid JSON (yep — even emoji caused JSON.parse errors).

Switching to `console.error()` helped, but sometimes even that made Claude silently ignore the response.

In the end, I just removed logs entirely unless I was running locally.

#### 3. Claude ignored my file and made one up
This was the most frustrating part. I uploaded a real emails.csv, but Claude decided to “help” by generating its own content instead. Every time I asked it to run the tool, it sent a fake CSV string like:

```csv
id,email
1,test@example.com
2,fake@demo.com
```

Meanwhile, my actual file had missing values and real formatting — things I needed the tool to analyze.

#### The fix? Prompt engineering.
I had to explicitly instruct Claude to use the uploaded file as-is. Here’s the one prompt that consistently worked:

> “Please call the check-emails tool and pass the file using the file argument, with the full CSV content. Do not simulate or summarize it — use the actual uploaded file’s content.”
>

Only then did Claude finally send the correct `file.content`, and the tool behaved as expected.

## What I Learned
Every tool I built taught me something new — not just about MCP or Claude, but about debugging, designing for unpredictability, and the importance of developer empathy when tools behave oddly.

Here are the key lessons I walked away with:

### MCP is simple in theory, sensitive in practice
MCP’s structure is straightforward: define a name, a version, a tool with an input schema and an execute method. But if you misplace a tool inside the wrong part of the server setup — say, using tools: [...] instead of capabilities.tools or server.tool() — Claude won’t see it. And it won’t tell you why.

### stdout ≠ stderr (and Claude cares a lot)
Claude listens to stdout for JSON-RPC responses — so if your tool prints logs with console.log(...), it might break JSON parsing. Emoji? Multiline logs? You’re in trouble. Use console.error(...) or remove logs entirely when testing in Claude.

### Prompt engineering matters — a lot
Claude is smart, but it's also imaginative. If you're not specific, it might invent file content, misinterpret schema expectations, or summarize rather than relay. You need to tell it *exactly* what to do. I learned to write prompts like a lawyer:

> “Use the uploaded file. Pass it via the file argument. Do not simulate, summarize, or replace the content.
>

### Logging is life
When I wasn’t sure what Claude was actually sending my tool, I started logging incoming arguments. That's how I spotted empty strings, fake data, or missing keys. Don’t trust the prompt output alone — verify it on your end.

### The smallest detail can break everything
A misplaced emoji, a missing await, a capital letter in a tool name… all of these broke things silently at one point. Debugging tools that talk to AI requires the patience of a saint — or at least a strong coffee habit.

## Closing Thoughts
What started as a simple experiment with Claude’s weather tool ended up being one of the most intense debugging experiences I’ve had in a while.

I didn’t expect to wrestle with output streams, prompt phrasing, or a language model’s tendency to make things up. But I did. And while it was frustrating at times (okay, *a lot* of the time), it was also deeply rewarding.

I learned how Claude sees the world — not just how to call a tool, but how it interprets intent, how it guesses context, and how it sometimes needs to be guided very, very precisely. I also learned that building tools for AI is less about "writing code" and more about "creating contracts between code and language."

If you’re planning to build your own MCP tools:

👉 start small,

👉 test everything locally first,

👉 and assume nothing — especially when Claude tells you “Done!” 😅

Would I go through this again?

Yes — because now I can build tools that really work, and I know how to make Claude behave when it doesn’t.

❤️