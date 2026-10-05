# ChatGPT MCP Server

<!-- mcp-name: com.hasdata/chatgpt -->

A hosted Model Context Protocol (MCP) server that gives Claude, Cursor, Windsurf and any other MCP client one ChatGPT tool. Send a prompt anonymously and get the answer back as markdown together with the web pages ChatGPT consulted, as structured JSON, with no OpenAI account and no API key of your own.

**1,000 free credits every month, no card required**, which is 100 prompts.

```
https://mcp.hasdata.com/mcp?apis=chatgpt
```

[![Glama score](https://glama.ai/mcp/servers/HasData/chatgpt-mcp/badges/score.svg)](https://glama.ai/mcp/servers/HasData/chatgpt-mcp)
[![tool contract](https://github.com/HasData/chatgpt-mcp/actions/workflows/contract.yml/badge.svg)](https://github.com/HasData/chatgpt-mcp/actions/workflows/contract.yml)
[![MCP](https://img.shields.io/badge/MCP-remote%20%7C%20streamable%20HTTP-6366f1?style=flat-square)](https://modelcontextprotocol.io)
[![Tools](https://img.shields.io/badge/tools-1-10b981?style=flat-square)](#tools)
[![npm](https://img.shields.io/npm/v/@hasdata/chatgpt-mcp?style=flat-square&logo=npm&label=npm&color=cb3837)](https://www.npmjs.com/package/@hasdata/chatgpt-mcp)
[![PyPI](https://img.shields.io/pypi/v/hasdata-chatgpt-mcp?style=flat-square&logo=pypi&logoColor=white&label=PyPI&color=3775a9)](https://pypi.org/project/hasdata-chatgpt-mcp/)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)

## Contents

- [What you need](#what-you-need)
- [Quick start](#quick-start)
- [Example prompts](#example-prompts)
- [Tools](#tools)
- [Errors and failure paths](#errors-and-failure-paths)
- [Pricing, free tier and limits](#pricing-free-tier-and-limits)
- [How it compares](#how-it-compares)
- [FAQ](#faq)
- [HasData links](#hasdata-links)
- [Development](#development)
- [Contributing](#contributing)
- [License](#license)

## What you need

An MCP client and a HasData API key from the [dashboard](https://app.hasdata.com/sign-up?utm_source=github&utm_medium=syndication&utm_campaign=chatgpt-mcp), free to create with no card, and the free tier covers 100 calls a month at the 10-credit rate. This is a remote server, so the simplest path is a URL and an `x-api-key` header, with no container to run and no OpenAI billing anywhere in the flow. A client that only speaks stdio reaches it through a thin launcher, published as `@hasdata/chatgpt-mcp` on npm and `hasdata-chatgpt-mcp` on PyPI, shown below.

## Quick start

The server URL is the same for every client. We run it hands-on in Claude Code and Claude Desktop. The other blocks follow each client's own documented format for a remote server.

| Field | Value |
| :--- | :--- |
| URL | `https://mcp.hasdata.com/mcp?apis=chatgpt` |
| Transport | HTTP, streamable |
| Auth header | `x-api-key: HASDATA_API_KEY` |

Clients with OAuth support can add the same URL as a connector and sign in without putting a key in a config file.

<details>
<summary><b>Claude Code</b></summary>

```bash
claude mcp add --transport http chatgpt "https://mcp.hasdata.com/mcp?apis=chatgpt" \
  --header "x-api-key: HASDATA_API_KEY"
```

</details>

<details>
<summary><b>Claude Desktop</b></summary>

```json
{
  "mcpServers": {
    "chatgpt": {
      "type": "http",
      "url": "https://mcp.hasdata.com/mcp?apis=chatgpt",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>Cursor</b></summary>

```json
{
  "mcpServers": {
    "chatgpt": {
      "type": "streamable-http",
      "url": "https://mcp.hasdata.com/mcp?apis=chatgpt",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>VS Code</b></summary>

```json
{
  "servers": {
    "chatgpt": {
      "type": "http",
      "url": "https://mcp.hasdata.com/mcp?apis=chatgpt",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

## Example prompts

Prompts, not code. Paste one in and the agent picks the tool itself. Each is annotated with the calls it takes, because every successful call costs 10 credits.

> Ask ChatGPT what the current state of the Artemis program is, and list the sources it used.

*One call, 10 credits. The answer and the sources come back together.*

> Ask ChatGPT the same question in German.

*One call, 10 credits. The answer follows the language of the prompt, so write the prompt in the language you want back.*

> Ask ChatGPT what happened in AI this week, with Berlin as the current date.

*One call, 10 credits. `timezone` is what fixes the meaning of "this week".*

> Put the same question to ChatGPT three times and show where the answers differ.

*Three calls, 30 credits. Each request is a fresh conversation, so there is no carry-over between them.*

## Tools

| Tool | What it returns |
| --- | --- |
| `hasdata_chatgpt_chat_getChatgptAnswer` | The answer as markdown, the web sources ChatGPT consulted with their domain and title, the conversation title it generated, the model that answered, and whether a web search was used. 10 credits a call |

One tool, 10 credits per successful call.

### Ask ChatGPT

[`hasdata_chatgpt_chat_getChatgptAnswer`](https://docs.hasdata.com/apis/chatgpt/chat?utm_source=github&utm_medium=syndication&utm_campaign=chatgpt-mcp)

| Parameter | Type | Required | Notes |
| :--- | :--- | :--- | :--- |
| `prompt` | string | yes | The question or instruction, up to 8000 characters |
| `timezone` | string | | IANA zone name such as `Europe/Berlin`. It sets what ChatGPT treats as today |

Everything lands under `conversation`. The answer is in `answer` as markdown, with `title` holding the name ChatGPT generated for the thread, `model` the model that answered, and `usedWebSearch` saying whether it went to the web at all. `finishReason` and `complete` tell a truncated answer from a finished one.

```json
{
  "conversation": {
    "answer": "As of **October 5, 2026**, NASA's Artemis program has made substantial progress…",
    "title": "Artemis developments cited",
    "model": "gpt-5-6",
    "usedWebSearch": true,
    "finishReason": "stop",
    "complete": true,
    "sources": [
      { "title": "Artemis News - NASA", "url": "https://www.nasa.gov/artemis-news/", "domain": "www.nasa.gov" }
    ]
  }
}
```

## Errors and failure paths

Your client almost never sees an HTTP error code from a tool call. The MCP layer answers 200 and puts the failure inside the result, with `isError` set to `true` and the reason as text.

**`sources` is uneven, and partly absent.** In a measured answer carrying twenty sources, every one had `title`, `url` and `domain`, eleven had `publishedDate` and only five had `snippet`. Read each field defensively rather than assuming the shape of the first element holds for the rest.

**No web search means no sources.** `usedWebSearch` is false when ChatGPT answers from the model alone, and `sources` is then empty or missing. A prompt about a stable fact often takes that path, so ask for current information when citations are the point.

**Every call is a fresh conversation.** There is no memory between requests, so a follow-up has to carry its own context in the prompt. `conversationId` identifies the thread that answered, not a thread you can continue.

**A truncated answer still succeeds.** Check `complete` and `finishReason` before treating `answer` as the whole response.

Each successful call spends credits from the connected account. A call that fails validation is not billed.

## Pricing, free tier and limits

The ChatGPT tool costs **10 credits per successful call**. Answer length does not change the price.

The free tier is **1,000 credits every month with no card**, which is 100 prompts.

Paid plans start at **$59 a month** for 200,000 credits, which is 20,000 calls. The unit price falls on larger plans. Current numbers are on the [plans page](https://hasdata.com/prices?utm_source=github&utm_medium=syndication&utm_campaign=chatgpt-mcp).

## How it compares

| | OpenAI API | This server |
| :--- | :--- | :--- |
| Account | Your own OpenAI account and billing | One HasData key |
| What you get | The model's answer | The answer plus the web sources behind it |
| Web search | A separate tool you wire up | Included, with `usedWebSearch` reporting whether it ran |
| Output | Your own schema | Parsed JSON with the sources already split out |

The two are not substitutes. The OpenAI API is the right call when you need system prompts, tools, streaming and conversation state. This server is the right call when you want what ChatGPT publicly answers, with its citations, and no account of your own.

## FAQ

### Do I need an OpenAI account or an API key?

No. The server answers anonymously through HasData, and the only credential involved is your HasData key.

### Which model answers?

Whichever ChatGPT serves anonymously at the time. The response reports it in `model`, so read it rather than assuming. A measured call returned `gpt-5-6`.

### Can I continue a conversation?

No. Each request is a fresh thread with no memory of earlier ones. Carry the context in the prompt instead.

### What language will the answer be in?

The language of the prompt. Ask in German and the answer comes back in German, or ask explicitly for a language in the prompt.

### Is HasData affiliated with OpenAI?

No. HasData is an independent web data provider and is not affiliated with, endorsed by or sponsored by OpenAI. All trademarks belong to their owners.

### Compliance and personal data

The server reads what ChatGPT answers publicly to an anonymous visitor. It does not log in, and it does not reach private conversations or account data.

## HasData links

| | |
| :--- | :--- |
| Product page and request builder | [ChatGPT Scraper API](https://hasdata.com/apis/chatgpt-api?utm_source=github&utm_medium=syndication&utm_campaign=chatgpt-mcp) |
| Endpoint documentation | [ChatGPT Scraper API docs](https://docs.hasdata.com/apis/chatgpt/chat?utm_source=github&utm_medium=syndication&utm_campaign=chatgpt-mcp) |
| Server documentation | [MCP server docs](https://docs.hasdata.com/mcp-server?utm_source=github&utm_medium=syndication&utm_campaign=chatgpt-mcp) |
| Every tool in one server | [HasData/hasdata-mcp](https://github.com/HasData/hasdata-mcp) |
| Client walkthroughs | [MCP clients and integrations](https://hasdata.com/integrations/mcp?utm_source=github&utm_medium=syndication&utm_campaign=chatgpt-mcp) |
| Plans and credit costs | [Plans and credit costs](https://hasdata.com/prices?utm_source=github&utm_medium=syndication&utm_campaign=chatgpt-mcp) |

## Development

```bash
npm install
npm test
```

The tests in `test/` assert the tool contract, the part that can break without a commit here. They check that `?apis=chatgpt` returns the one expected tool, that its name and required parameter have not changed, that it carries a description, and that a real call still puts the answer under `conversation`.

## Contributing

The parameter table and the sample above were read from the live schema and from real calls rather than from documentation. A correction is welcome when a field or a failure mode has changed. Open an issue with the response you saw.

## License

MIT
