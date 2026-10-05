---
name: sourced-answer
description: Ask ChatGPT a question and report the answer with the sources it actually cited
---

Put a question to ChatGPT and come back with the answer and the evidence behind it.

Ask me for the question if I have not given one. Ask which timezone to assume when the question involves "today", "this week" or anything else that depends on the date.

Then:

1. Call `hasdata_chatgpt_chat_getChatgptAnswer` with the question as `prompt`, and `timezone` set when the date matters. Write the prompt in the language the answer should come back in.
2. Read `conversation.complete` and `conversation.finishReason` first. If the answer was cut off, say so before quoting it.
3. Report the answer, then the sources as a list of domain and title, with the publication date where the source carries one. Do not invent dates for the ones that do not.
4. Say whether `usedWebSearch` was true. If it was false there are no sources, and the answer came from the model alone, which the reader needs to know.

Count the sources and say how many carried a snippet and how many a date. Those fields are present on some and missing on others, and a reader who sees five snippets under twenty sources should know that is the API rather than your summarising.

Each successful call spends credits, so ask once with a well-formed question rather than iterating on phrasing.
