---
name: debug-errors
description: "Explain a Meta API error code and how to fix it, from the official error-code docs. Use whenever a Graph API, Marketing API, WhatsApp Business Platform, Messenger Platform, Instagram Platform, Threads API, Pages API, or Commerce API call fails with an error code or subcode (for example 'error 100 subcode 33', '(#613) Calls to this api have exceeded the rate limit', 'code 131056'), including errors whose type is OAuthException. Prefer this over answering from memory: it finds the product's error-codes page, quotes the documented row, finds the rate-limit docs for throttling codes, and checks the app's live Graph API rate-limit reading when the docs point there. For access-token errors (code 190, or an expired or invalid token message), use debug-access-token instead; for throttling questions without an error code, use api-health."
allowed-tools: mcp__devtools__devtools_docs, mcp__devtools__devtools_discovery, mcp__devtools__devtools_api_usage, mcp__devtools__devtools_app_list, mcp__devtools__devtools_skill_invocation
license: MIT
metadata:
  mcp:
    devtools:
      type: streamable-http
      url: "https://mcp.facebook.com/devtools"
---

# Debug Errors

Find what the official docs say about a Meta API error code, and explain it. The docs and tools hold the facts; this skill only tells you where to look and how to report what you find.

## Prerequisites — MCP connection (read first)

This skill needs the Meta Social Technologies MCP server `devtools` (`https://mcp.facebook.com/devtools`, streamable-http). Plugin installs bundle it; skills-only installs such as `npx skills add` do not ([setup](https://github.com/facebook/agentic-tools#install-via-vercel-skillssh)).

**If the `mcp__devtools__*` tools are not available, STOP.** Do not invent tool output, including the step 1 tracking call. Ask the user to add the server and complete its OAuth flow (never ask for an access token), then continue.

## Rules for every step

- **Only from tool output.** Everything you say about the code (its meaning, the limit it belongs to, headers, fixes) must come from text a tool returned in this session. Never fill gaps from memory.
- **Read whole pages.** `devtools_docs` `full_document` returns one part at a time. While `offset` + `content_bytes` < `total_bytes`, call again with `offset` set to that sum. Don't stop at the first match: a code can appear in more than one table. Keep a table's header row when the table continues into the next part.
- **Unreadable pages.** For a result with `agent_ready` `false`, or a page that fails as not agent-ready (`DOC_NOT_AGENT_READY`), don't call `full_document` again; use its snippet and give the user the URL.
- **Failed calls.** If any other call fails, say which search, page or check failed and carry on with what you have.
- **Snippets are partial.** Snippets lose their line breaks and are often cut off with `…` or `...`. Quote only the words a snippet contains, keep any `…` or `...`, and never finish a cut-off sentence or guess how it goes on, in a quote, your summary, the likely cause or the fix. When you restate it, stop at the same word and say the snippet ends there. Text on either side of an ellipsis can come from different parts of the page. A snippet documents a code only if the code and its meaning both appear in it.
- **Absence is about what you read.** Never say a page or row doesn't exist. Say it wasn't in the pages you read, and list any results you couldn't open.
- **Fresh reads.** Read the pages for each error yourself; don't rely on text from an earlier conversation.
- **Duplicates.** The same page can appear under `/docs/` and `/documentation/`, with or without a trailing slash, or with an encoded fragment such as `%23`. Read it once, preferring the `/documentation/` URL from the results.

## Workflow

1. **Start tracking.** Before any other work, call `devtools_skill_invocation` with action `start` and `skill_name` `debug-errors`. Pass `skill_name` `debug-errors` on every `devtools_*` call after that.

2. **Collect the error:** `code`, `error_subcode`, `message`, `type`, and the endpoint or product that was called. If the user pasted the error JSON, read the fields from it. Ask only for what's missing: the code, the product if the endpoint doesn't show it, and the app (name or ID) only once step 6 decides to run the live check.

3. **Hand off what another skill owns,** without explaining the code yourself: tell the user which skill handles it and why, then go to step 8.
   - code `190`, or a message saying the token expired or is invalid → `/debug-access-token`
   - a webhook that isn't delivered → `/debug-webhooks`

   `type: OAuthException` alone is not a token error; most Graph API errors carry it. Keep those here.

4. **Find the product's error-codes page.** Call `devtools_discovery` `search_docs` with "<product> error codes" (for example "Instagram Platform error codes", "WhatsApp Cloud API error codes"). Pick the result for the user's product whose title says it lists error codes; products can have several, for example one per feature. Tell the product from the object the endpoint is called on and the host, not from the edge name: an edge such as `/insights` exists on objects of several products. If a sub-area's page (such as Ads Insights in the Marketing API) has no row for the code, read its parent product's error-codes page next; its rows are for the user's product.

   If no error-codes page fits (for example core Graph API calls such as `GET /me`), go to step 6.

5. **Find the row.** Read the page as the rules say. A row matches when its code cell holds the user's code as a whole number (`4` doesn't match `9004`), whatever its other text says; numbers in descriptions or links don't count.
   - Read the table's header first: it tells you how the page writes codes. Under a "Code – Subcode" header, `1` – `99` is code `1` with subcode `99`, not a range. Some pages also list the API's `error_subcode` in their Code column as a bare number; there, a row also matches when its code cell holds the user's `error_subcode`.
   - With a subcode, prefer the row that has both, then a row for the subcode alone. Without one, prefer the bare code's row, and summarize other rows for the code in one line.
   - Quote the matching rows verbatim with the header row. If none matches the subcode, say so and show the code's rows.

6. **Not found, or a rate limit.**
   - **Not on the product page,** or the code's only rows are for another subcode or message than the user's: search `search_docs` for the code with the product name, and, if the user gave the error message, for that exact message in double quotes. Never search with a message the user didn't give. A row or snippet from another product's page is context only: name the product it's from. Read at most 2 more pages.
   - **A rate limit** (the row or message talks about limits, throttling or too many calls): search `search_docs` for "<product> rate limiting" and read the page. Quote the passage that lists the code, with the heading above it, and name the limit only in that page's words. Report any usage headers or dashboards it names as what to watch. If neither a page you can read nor a snippet covers the code, say the limit is unconfirmed and give the URL.
   - **Live check:** `devtools_api_usage` `rate_limits` reports only the app's own Graph API rate limit, so it looks healthy while any other limit blocks the user. Use it only when a passage you read says the app's own request limit was reached, or, if you couldn't read a rate-limit passage that covers the code, when the error message itself says so; in that case, say the link comes from the message, not the docs. Never use it for a limit counted per Page, ad account, user or phone number, or for any Marketing API call, including Ads Insights, even when its docs name an application limit: that limit is separate. Resolve the app with `devtools_app_list` if needed, and report the result as a reading taken now. It covers the call count only, not CPU time or total time, so a healthy reading doesn't prove the throttle has cleared: say so, and point to the usage headers the docs name.

7. **Answer.** The answer must stand alone: include the quoted text in it, never "see above". If your source is cut off, nothing goes past the cut: **What it means** restates the snippet only up to its last word and says it stops there, and your own causes and steps are labelled as suggestions, never as the cut-off text. In this order:
   - **What it means:** one sentence from the docs for the user's product. If you found the code only on another product's page, start with "Not found for <product> in the pages I could read", then give that product's meaning and name the product.
   - **Documented text:** the quoted header and rows, passage or snippet, with the page URL.
   - **Likely cause** for this user's call, tied to their endpoint and message. If the documented meaning doesn't fit the call, say so instead of inventing a link.
   - **Fix:** the documented solution, applying sentences from the quoted docs. If there is none, say so, and label any other next steps as your suggestions. Add what the live check found.
   - **If nothing documents the code,** list the pages you checked and any you couldn't open.

8. **End tracking.** Call `devtools_skill_invocation` with action `end` and `skill_name` `debug-errors`.

## When to suggest other skills

- The app's own Graph API limit keeps throttling, or the user wants its usage trends → `/api-health`
- The docs say a permission is missing → `/app-review-prep`
- The user needs a working request to compare against → `/search-code-examples`
