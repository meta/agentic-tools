---
name: search-code-examples
description: "Find working code examples for a Meta developer API (Graph API, Marketing API, WhatsApp Business Platform, Instagram Platform, Pages API, Messenger, webhooks) and return them verbatim from the official docs, with the page and section they came from. Use whenever a user asks for a code sample, snippet, cURL command, request/response payload, or 'how do I call X in Python/Node/PHP/cURL' for a Meta API. Prefer this over writing the call from memory: it quotes the documented request exactly, so field names, API versions, and endpoints are right. For setting up a new integration end to end, use api-integration."
allowed-tools: mcp__devtools__devtools_discovery, mcp__devtools__devtools_docs, mcp__devtools__devtools_skill_invocation
license: MIT
metadata:
  mcp:
    devtools:
      type: streamable-http
      url: "https://mcp.facebook.com/devtools"
---

# Search Code Examples

Find code examples in Meta developer documentation and return them exactly as documented. The docs hold the code; this skill only tells you where to look and how to report what you find.

## Prerequisites — MCP connection (read first)

This skill needs the Meta Social Technologies MCP server `devtools` (`https://mcp.facebook.com/devtools`, streamable-http). Plugin installs bundle it; skills-only installs such as `npx skills add` do not ([setup](https://github.com/facebook/agentic-tools#install-via-vercel-skillssh)).

**If the `mcp__devtools__*` tools are not available, STOP.** Do not invent tool output, including the step 1 tracking call. Ask the user to add the server and complete its OAuth flow (never ask for an access token), then continue.

## Rules for every step

- **Only from tool output.** Every line you present as documentation must come from text a tool returned in this session, character for character. Anything you change or write yourself is labelled as yours.
- **Read whole sections.** `devtools_docs` `full_document` returns one part at a time. While `offset` + `content_bytes` < `total_bytes`, call again with `offset` set to that sum, until you have the operation's section with all its code blocks and its response, or you reach the end.
- **Unreadable pages.** For a result with `agent_ready` `false`, or a page that fails as not agent-ready (`DOC_NOT_AGENT_READY`), don't call `full_document` again; use its snippet and give the user the URL.
- **Failed calls.** If any other call fails, say which search or page failed and carry on with what you have.
- **Duplicates.** The same page can appear under `/docs/` and `/documentation/`, with or without a trailing slash, or with an encoded fragment such as `%23`. Count it once, preferring a `/documentation/` URL from the results. Skip "This page has moved" stubs; follow them only to a page in the results.
- **Access tokens.** Never ask for or paste a real access token. Keep the docs' placeholder.

## Workflow

1. **Start tracking.** Before any other work, call `devtools_skill_invocation` with action `start` and `skill_name` `search-code-examples`. Pass `skill_name` `search-code-examples` on every `devtools_*` call after that.

2. **Pin down the request:** the operation ("send a WhatsApp template message", "create an ad set with a daily budget") and the language, if any. If the user names only a product, or the request could mean two different calls, ask, or answer the likelier one and name the other.

3. **Find pages.** Call `devtools_discovery` `search_docs` with the product and the operation, and `max_results` 8. Pick up to 3 pages, judged by title and snippet: guides and get-started pages first, then endpoint reference pages. Skip overviews, policies, changelogs and blog posts. If nothing is worth reading, go to step 7.

4. **Read the pages** in parallel, as the rules say. If none has code for the operation in the requested language, or none has a response, read the most relevant reference page from the results too.

5. **Pick the examples.** Code is in fenced blocks.
   - **Decide the language from the content,** since labels are often wrong or missing: a block starting with `curl` is cURL, one starting with `{` or `[` is JSON, one starting with `GET https://...` is an HTTP request. Otherwise use the label, or a heading just above that names a language (`### PHP SDK`).
   - Keep blocks in the requested language, plus the request body, response or webhook payload for the operation. Use the nearest heading that names an operation to tell which operation a block belongs to.
   - **From a snippet** (an unreadable page): search snippets lose their line breaks and are often cut off with `…` or `...`, so never present one as code. Say what it shows about the call, say it came from a search snippet, and give the page URL.

6. **Answer.** For each example:
   - The code, copied exactly, in a fence labelled with the language you decided on.
   - Where it came from: page title, section heading, URL.
   - What it does, in one line from the surrounding docs.
   - What to replace: placeholders and sample values (IDs, phone numbers, dates, names), and anything the docs say must exist first (for example an approved template).
   - **If it won't work as written** (broken JSON, a shell command with a missing line continuation, a URL split inside quotes, a missing auth parameter the page requires), say so after the verbatim copy and show only the corrected lines, labelled as your correction. Never fill in `...` placeholders; point to the page's parameter list instead. Don't "correct" responses or webhook payloads.
   - Give request then response, call by call for a multi-call flow. If the pages you read show no response, say so.

   **Not in the requested language:** say the pages you checked have no example in it, show the closest documented example, and offer a translation. Label any translation as yours, and keep its URL, version, fields and values.

   **Body and command on different pages:** you may combine a complete documented body with a complete documented command for the same endpoint and method. Label the result **combined from two documented examples**, cite both, and say if their API versions differ.

7. **Nothing found:** this means no page you read has code for the operation in any language. Code only in another language is covered in step 6. Repeat steps 3 to 6 once, with one more search using different wording or the endpoint path (`/messages`, `/act_{id}/adsets`). That is the last search, even if a page points to others that the results don't include. If there's still nothing, say so, list the pages you read and any results you skipped or couldn't read, and point to the most relevant page from the results. Don't write the call yourself.

8. **End tracking.** Call `devtools_skill_invocation` with action `end` and `skill_name` `search-code-examples`.

## When to suggest other skills

- The user needs the app or permissions set up first → `/api-integration`
- The user wants to subscribe to webhooks → `/webhook-setup`
