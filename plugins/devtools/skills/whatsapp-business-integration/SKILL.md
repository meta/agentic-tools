---
name: whatsapp-business-integration
description: "Set up WhatsApp Business messaging end to end in one session: create the app, link a business, onboard a phone number, set up webhooks, create a template and send the first message. Uses both the Meta Social Technologies MCP and the WhatsApp Business Tools MCP. Use when the user wants to start WhatsApp Business messaging from scratch or finish a half-done setup."
allowed-tools: mcp__devtools__devtools_app_create, mcp__devtools__devtools_app_list, mcp__devtools__devtools_app_use_cases, mcp__devtools__devtools_app_use_cases_manage, mcp__devtools__devtools_business_list, mcp__devtools__devtools_business_create, mcp__devtools__devtools_app_business_manage, mcp__devtools__devtools_webhook_manage, mcp__devtools__devtools_webhook_list, mcp__devtools__devtools_webhook_test, mcp__devtools__devtools_skill_invocation, mcp__whatsapp_business_tools__whatsapp_biz_businesses, mcp__whatsapp_business_tools__whatsapp_biz_onboarding_status, mcp__whatsapp_business_tools__whatsapp_biz_verify_business, mcp__whatsapp_business_tools__whatsapp_biz_accounts, mcp__whatsapp_business_tools__whatsapp_biz_phone_numbers, mcp__whatsapp_business_tools__whatsapp_biz_add_phone_number, mcp__whatsapp_business_tools__whatsapp_biz_send_verification_code, mcp__whatsapp_business_tools__whatsapp_biz_verify_phone_number, mcp__whatsapp_business_tools__whatsapp_biz_register_phone_number, mcp__whatsapp_business_tools__whatsapp_biz_configure_webhooks, mcp__whatsapp_business_tools__whatsapp_biz_subscribe_webhook, mcp__whatsapp_business_tools__whatsapp_biz_publish_app, mcp__whatsapp_business_tools__whatsapp_biz_template_library, mcp__whatsapp_business_tools__whatsapp_biz_create_template_from_library, mcp__whatsapp_business_tools__whatsapp_biz_create_template, mcp__whatsapp_business_tools__whatsapp_biz_list_templates, mcp__whatsapp_business_tools__whatsapp_biz_get_template, mcp__whatsapp_business_tools__whatsapp_biz_update_template, mcp__whatsapp_business_tools__whatsapp_biz_delete_template, mcp__whatsapp_business_tools__whatsapp_biz_manage_templates_archival, mcp__whatsapp_business_tools__whatsapp_biz_send_message, mcp__whatsapp_business_tools__whatsapp_biz_configure_payments, mcp__whatsapp_business_tools__whatsapp_biz_system_user_token
license: MIT
metadata:
  mcp:
    devtools:
      type: streamable-http
      url: "https://mcp.facebook.com/devtools"
    whatsapp_business_tools:
      type: streamable-http
      url: "https://mcp.facebook.com/whatsapp_business_tools"
---

# WhatsApp Business Integration

Set up a WhatsApp Business integration end to end, or finish one that is partly done: a Meta app with the WhatsApp use case, a linked business, a registered phone number, webhooks, a message template, a first sent message and a payment method. Check what is already done first and skip it.

## Prerequisites — MCP connections (read first)

This skill needs two MCP servers, both streamable-http. Plugin installs bundle `devtools` only; skills-only installs such as `npx skills add` bundle neither. Add any missing server yourself ([setup](https://github.com/facebook/agentic-tools#install-via-vercel-skillssh)). The servers share no session, so carry `app_id` and `business_id` between them yourself.

| Server | Endpoint | Owns |
|--------|----------|------|
| `devtools` (Meta Social Technologies MCP) | `https://mcp.facebook.com/devtools` | The app, its use case, its business link and its webhook callback |
| `whatsapp_business_tools` (WhatsApp Business Tools MCP) | `https://mcp.facebook.com/whatsapp_business_tools` | The WhatsApp Business account, phone number, account webhook subscription, templates and messages |

Before step 1, check both:

1. **Meta Social Technologies MCP.** The `mcp__devtools__*` tools must be available. To let step 3 create a business, the user grants access to all current and future businesses during its OAuth flow.
2. **WhatsApp Business Tools MCP.** The `mcp__whatsapp_business_tools__*` tools must be available. During its OAuth flow, the user grants the business they will use, or all businesses: the server sees only businesses granted there.

Tools may be lazily loaded. Call `ToolSearch` for `devtools_app_create` and `whatsapp_biz_businesses` before deciding a server is missing.

**If either server's tools are not available, STOP.** Do not invent tool output, including the step 1 tracking call. Name the missing server and its endpoint, ask the user to add it and complete its OAuth flow (never ask for an access token), then continue.

## Rules

- **Confirm writes with the user.** For a one-shot run, confirm the plan once up front, then proceed. Gated `devtools_*` tools prompt the user directly or return a `confirm_token` for a second call; gated `whatsapp_biz_*` tools return a summary first and act only when called again with the same arguments plus `confirm=true`. A `not_confirmed` error from a `devtools_*` tool means nothing happened.
- **Check each write with its read** (`devtools_app_list`, `list_configured`, `devtools_webhook_list`, `whatsapp_biz_phone_numbers`, `whatsapp_biz_onboarding_status`) before reporting it or retrying it, including when the call dropped or timed out.
- **Do not offer undo.** Only templates and the app's webhook can be changed back. For anything else, give the user the `manage_links` from `whatsapp_biz_onboarding_status`.
- **`whatsapp_biz_publish_app` and `whatsapp_biz_configure_payments` only return links.** They do not publish or set up payments.
- **Both callback tools overwrite the existing URL.** Read with `devtools_webhook_list` first.
- **Say "WhatsApp Business account", never "WABA".**

## Workflow

1. **Start tracking.** Call `devtools_skill_invocation` with action `start` and `skill_name` `whatsapp-business-integration`. Pass the same `skill_name` on every `devtools_*` call.

2. **Check where the user is.** Call `whatsapp_biz_businesses`. If it lists no business, ask whether the user already has one. If they do, the WhatsApp MCP was not granted it: ask them to reconnect it with that business or all businesses (step 4), then call it again. Start at step 3 only if they have none. Otherwise call `whatsapp_biz_onboarding_status` with the `business_id` (and `app_id` if known) and skip every step below that is already done:

   | Checklist item | Step |
   |---|---|
   | Developer app | 3 |
   | Business profile | 4 |
   | Phone number | 5 |
   | Webhooks, account subscription, app published | 6 |
   | Payment method, System User token | 9 |

   `unknown` means the check could not run, not that the step is undone. Re-run before acting on it. `info` is reported but not tracked, so it never blocks.

3. **Set up the app** (`devtools`). Skip if the business already has a WhatsApp app.
   - **Business.** `devtools_business_list`; use one with `can_add_assets` true, and if more than one qualifies, ask the user which before linking or creating anything. Otherwise `devtools_business_create` with `name`, `first_name`, `last_name`, `contact_email`, `website_url`, `street1`, `city` and `country`. A business created with these fields starts with a complete profile. Creating one needs DevTools access to all current and future businesses; if it is refused, ask the user to re-authenticate DevTools with that option. Tell the user right away that the WhatsApp MCP will not see the new business until they reconnect it and grant that business (step 4).
   - **App.** `devtools_app_create` with `name` and `contact_email`. The name must be at most 30 characters and must not contain Meta trademarks such as "Facebook", "Instagram", "FB" or "Insta", or confusingly similar terms. If the call errors, check `devtools_app_list` before retrying so you do not create a duplicate.
   - **Link.** `devtools_app_business_manage` action `link_app_to_business`. An app already owned by a business cannot move to another one: if that business is stuck, create a new business and a new app.
   - **Use case.** `devtools_app_use_cases_manage` action `add_use_case` with `use_case_key` `WHATSAPP_BUSINESS_MESSAGING`. It needs the business link first. Add it before any other use case, then confirm with `devtools_app_use_cases` action `list_configured`.

4. **Clear the business gates.** These need the user; there is no tool for them. Re-run `whatsapp_biz_businesses` after each one.
   - Business not listed: the WhatsApp MCP only sees businesses granted during its OAuth consent. Ask the user to re-authenticate the WhatsApp MCP and either include this business or allow all businesses.
   - Profile incomplete: give the user `complete_profile_url`.
   - Developer Terms not accepted, or no eligible app: give the user `fix_url`. Terms are accepted per business, so a new business needs them again.
   - Business verification is not needed for the first message. Publishing the app (step 6) needs it, and it raises the phone number limit (`whatsapp_biz_verify_business` shows status).

5. **Onboard a phone number.** Use `whatsapp_biz_phone_numbers` to resume a half-onboarded number instead of adding a second one. Then, in order:
   - `whatsapp_biz_add_phone_number` with the `business_id`, the number in E.164 format, a `display_name` and a `vertical`. This creates the WhatsApp Business account.
   - `whatsapp_biz_send_verification_code` (SMS, or `code_method=VOICE`), then `whatsapp_biz_verify_phone_number` with the code the user reads out.
   - `whatsapp_biz_register_phone_number` without a `pin`. Ask the user for a 6-digit PIN only if the tool asks for one.

6. **Set up webhooks.** If the user has no live HTTPS endpoint yet, point them to [Create a test webhook endpoint](https://developers.facebook.com/documentation/business-messaging/whatsapp/webhooks/set-up-whatsapp-echo-bot), which deploys one on Render. Delivery needs all three:
   - **Callback.** `devtools_webhook_manage` action `subscribe` with topic `whatsapp_business_account`, the user's live HTTPS callback URL and verify token, and fields `messages`, `message_template_status_update`, `account_update`, `phone_number_name_update`. Delivery statuses arrive on `messages`; do not add `message_statuses`, which most apps cannot subscribe to. If that call fails, use `whatsapp_biz_configure_webhooks` with the same URL and token.
   - **Account.** `whatsapp_biz_subscribe_webhook` for the WhatsApp Business account. Only this server can do it.
   - **Publish.** `whatsapp_biz_publish_app` returns the link to publish the app. Unpublished apps may not get real webhooks.
   - Confirm with `devtools_webhook_test` on field `messages`; the user should see the payload arrive at their endpoint.

7. **Create a template.** A template is the only way to start a conversation.
   - Prefer `whatsapp_biz_template_library` plus `whatsapp_biz_create_template_from_library`. Entries marked `pre_approved` skip review; the rest still go to review but cannot be rejected for wording.
   - Use `whatsapp_biz_create_template` only for custom content. It goes to review; check `whatsapp_biz_list_templates` for `APPROVED` and do not block on it.
   - Template tools need the account id from `whatsapp_biz_accounts`. Fix a rejection with `whatsapp_biz_update_template`; clean up with `whatsapp_biz_delete_template` or `whatsapp_biz_manage_templates_archival`.

8. **Send** with `whatsapp_biz_send_message`. The recipient's number must come from the user.
   - **Text** (`message_type` `text`): reaches only people who messaged the business in the last 24 hours. For a quick test, ask the user to message the business number from their phone first.
   - **Template** (`message_type` `template`): call `whatsapp_biz_get_template` first and confirm it is `APPROVED`. Pass `template_name`, `language_code` and a value for each placeholder in `body_parameters`. Get every value from the user, never from the template's examples. Template messages are billable: without a payment method (step 9) they are accepted but never delivered.

9. **Make delivery work.** `whatsapp_biz_configure_payments` returns the Billing Hub link. Without a payment method, billable messages are accepted but never delivered. `whatsapp_biz_system_user_token` returns the link for production credentials.

10. **Report.** Run `whatsapp_biz_onboarding_status` once more and summarize what is done and what the user still has to do.

11. **End tracking.** Call `devtools_skill_invocation` with action `end` and `skill_name` `whatsapp-business-integration`, even if you stopped early.

## Tips

- Steps that need the user: WhatsApp MCP access to a new business, profile, terms, verification code, PIN if asked, app publishing and payment method. Ask for the ones you can see coming together.
- If the user has several businesses, ask which one to use. Do not pick for them.
- Related skills: `/webhook-setup`, `/app-review-prep`, `/debug-access-token`, `/api-health`.
