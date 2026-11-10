---
name: app-setup
description: "Set up a Meta app for a product integration end to end — create or pick the app, connect it to a business portfolio, and add the use cases the product needs. Use when starting a new integration from scratch, or when an app is missing its business link or a required use case. Other product skills can run this first and continue with the app_id and business_id it reports."
allowed-tools: mcp__devtools__devtools_app_create, mcp__devtools__devtools_app_list, mcp__devtools__devtools_business_list, mcp__devtools__devtools_business_create, mcp__devtools__devtools_app_business_manage, mcp__devtools__devtools_app_use_cases, mcp__devtools__devtools_app_use_cases_manage, mcp__devtools__devtools_skill_invocation
license: MIT
---

# App Setup

Take a developer from "I need an app for this integration" to an app that is owned by a business portfolio and has the use cases the integration needs.

## Rules

- **Get the developer's agreement before every write.** `devtools_app_create`, `devtools_business_create`, `devtools_app_business_manage` and `devtools_app_use_cases_manage` ask for confirmation. If the client supports elicitation, the server prompts the developer directly and executes on the first call, so say what you are about to do before calling. Otherwise the first call returns a summary and a `confirm_token`; show the summary, and after the developer agrees, repeat the call with the same arguments plus that `confirm_token`.
- **Pass the skill name on every call.** Every `devtools_*` call passes `skill_name` `app-setup`, or the calling skill's name where [When another skill runs this one](#when-another-skill-runs-this-one) says so.
- **Verify every write with its read before a dependent write or a retry.** A write can report an error and still take effect. Check with `devtools_app_list`, `devtools_business_list` or `devtools_app_use_cases` before continuing.
- **Create at most one app.** There is no delete-app tool, and a duplicate app counts against the developer's app limits. If the developer names an existing app that is not listed, treat it as ungranted and reauthorize instead of creating a replacement.
- **Nothing here can be undone over MCP.** No tool deletes an app, unlinks a business or removes a use case. Reversal is a step in the App Dashboard.
- **Let the server explain a refusal.** When a call is refused, show the developer the error's `message` as returned; it says what they need to do. Retry only if the message says to, and never create a replacement app or portfolio to get around a refusal. Then close tracking and stop.
- **Close tracking whenever you stop.** Call `devtools_skill_invocation` with action `end` before you finish or hand control back for good. Asking the developer a question is not stopping.

## Workflow

1. **Start tracking.** Before any other work, call `devtools_skill_invocation` with action `start` and `skill_name` `app-setup`. Pass `skill_name` `app-setup` on every `devtools_*` call in the following steps. When another skill runs this one, follow [When another skill runs this one](#when-another-skill-runs-this-one) instead.

2. **Collect what is needed.** Ask for anything the developer has not said:
   - **App** — an existing app (name or ID), or a name and contact email for a new one. A new app's name must be at most 30 characters and must not contain Meta trademarks such as "Facebook", "Instagram", "FB" or "Insta", or confusingly similar terms.
   - **Use cases** — the product the app is for (for example WhatsApp messaging, Instagram, Marketing API). Map it to `use_case_key` values in step 6.
   - **Business portfolio** — which portfolio should own the app, if they have one in mind. To create one, collect its name; the creator's first and last name; a contact email; a public `website_url`; `street1`; optional `street2`; `city`; optional `state` and `postal_code`; and a two-letter ISO `country` code.

3. **Resolve the app.**
   - For an existing app, call `devtools_app_list` with action `list` and limit `100`, following every returned `cursor` until exhausted, then match the name or ID. If several match, show the candidates and ask. If a named existing app is absent from the complete list, ask the developer to reauthorize the connection and select that app; do not create a replacement. Note its `owning_business_id`. If `owning_business_granted` is false, the app belongs to a portfolio the developer did not grant; ask them to reauthorize and select that portfolio instead of treating the app as unowned. Close tracking before either handoff.
   - For a new app, call `devtools_app_create` with action `create_app`, `name` and `contact_email`. If it returns an error, paginate `devtools_app_list` with action `list` and limit `100` to look for the app before doing anything else. Never start another create after one has been confirmed; when the first call only returns a `confirm_token`, repeating the same call with unchanged arguments plus that token completes the original create.

4. **Resolve the business portfolio.**
   - If the app already has an `owning_business_id`, that portfolio owns it, and an app cannot move to another portfolio over MCP. If the developer wants a different portfolio, ask whether to use the one that owns the app instead. If they agree, or did not name one, use it as the `business_id` and go to step 6. If not, resolve the portfolio they want below; step 5's link is refused for an app another portfolio owns, and the refusal says what to do.
   - Otherwise, or when the developer wants a different portfolio, call `devtools_business_list` with action `list` and limit `100`, following every returned `cursor` until exhausted. Only a portfolio with `can_add_assets` true can own the app. If several qualify, ask which one.
   - If the complete list has portfolios but none has `can_add_assets` true, do not create a replacement. Ask which portfolio the developer wants and go on to step 5: the link is refused before the developer is asked to confirm, and the refusal says what they need.
   - If the portfolio the developer expects is missing from the complete list, it was not granted when they signed in; ask them to reauthorize the connection and select it, close tracking, and stop. Do not create a replacement.
   - Only if the complete list is empty and the developer confirms they have no portfolio, tell them that the new portfolio will use business category `OTHER` and time zone `America/Los_Angeles`, then call `devtools_business_create` with action `create_business`, `name`, `first_name`, `last_name`, `contact_email`, `website_url`, `street1`, optional `street2`, `city`, optional `state` and `postal_code`, and two-letter ISO `country`. Pass the same values unchanged on a confirm-token follow-up.

5. **Link the app to the portfolio.** Call `devtools_app_business_manage` with action `link_app_to_business`, `app_id` and `business_id`. `already_linked` true means there was nothing to do. If the link is refused, for example because the portfolio needs a two-factor step this tool cannot complete, follow the refusal rule; once the developer has done what the message asks, running app-setup again picks up the linked app. After a link that was not refused, paginate `devtools_app_list` with action `list` and limit `100` until the app is found, and require the expected `owning_business_id` before step 6.

6. **Add the use cases.**
   - First call `devtools_app_use_cases` with action `list_configured` and `app_id`, and skip any requested use case that is already configured. Then call `devtools_app_use_cases` with action `list_available` and `app_id`, and pick the `use_case_key` values that match the product. Do not guess keys that are not in the list.
   - Add the product's main use case first with `devtools_app_use_cases_manage`, action `add_use_case`, `app_id` and `use_case_key`. Each use case added can make others non-combinable and drop them from `list_available`, so add the most important one before any extras, and call `list_available` again with `app_id` before each further addition. If `add_use_case` returns a `warning` or `next_step`, keep it with that use case.
   - Confirm with `devtools_app_use_cases` action `list_configured` and `app_id`. If an addition errored, this tells you whether it took effect.

7. **Report.** Give the developer:
   - The `app_id` and `business_id`.
   - Every use case now configured, with any `warning` or `next_step` it returned, and any that could not be added, with the reason.
   - Anything they still have to do in the App Dashboard, with the link.

8. **End tracking.** After completing all preceding steps, call `devtools_skill_invocation` with action `end` and `skill_name` `app-setup`.

## When another skill runs this one

Product skills such as `whatsapp-business-integration` can run app-setup for their app phase.

- Take the app, portfolio and use cases the calling skill passes, and ask only for what is missing.
- Call `devtools_skill_invocation` `start` and `end` with `skill_name` `app-setup` as usual, and leave the calling skill's tracking open. On every other `devtools_*` call, pass the calling skill's `skill_name` instead of `app-setup`, so its funnel keeps these calls.
- Instead of the step 7 report, hand back the `app_id`, the `business_id`, whether the portfolio was created in this run, and the configured use cases with any `warning` or `next_step` each returned. The calling skill shows those to the developer before it continues. Then end tracking (step 8) and continue with the calling skill's next step.

## Tips

- A new app starts in development mode. Use cases add permissions and products, but going live is a separate App Dashboard step.
- `WHATSAPP_BUSINESS_MESSAGING` is the WhatsApp Cloud API use case. `WHATSAPP_LOGIN` is unrelated; it is a Facebook Login use case.
