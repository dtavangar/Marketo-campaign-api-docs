
# How to Trigger a Smart Campaign in Marketo Using the REST API and Tokens


## Overview

This tutorial walks you through how to trigger a Smart Campaign in Marketo using the REST API and personalize the email using My Tokens. This use case is ideal for customer-triggered notifications like webinar reminders, onboarding steps, or post-purchase follow-ups.

## Use Case

A lead registers for a webinar through an external platform (e.g., custom app, Pendo, or Eventbrite). You want to automatically:

- Trigger a reminder email from Marketo
- Personalize it with:
  - The lead’s first name
  - Webinar title
  - A unique join link

This can be done using the REST API and My Tokens.

## Step 1: Create the Smart Campaign

1. Go to **Marketing Activities** and create a new Smart Campaign (e.g., `Send Webinar Reminder`)
2. In the **Smart List** tab, add the trigger:
   - `Campaign is Requested`
   - Set **Source** to: `Web Service API`

![Smart List trigger setup](/help/assets/trigger-campaign/smart-list-trigger.png)

## Step 2: Define the Email Content

Create or edit an email asset that references both lead and My Tokens.

Make sure to **insert the tokens directly into the email content**, like this:

```html
Hi {{lead.First Name}},

You're registered for **{{my.WebinarTitle}}**.

Join here: {{my.JoinLink}}
```

> **Note:** If you're using a token to dynamically inject an image URL (e.g., `{{my.WebinarImage}}`), you must wrap the token in an HTML image tag:
>
> ```html
> <img src="{{my.WebinarImage}}" alt="Webinar banner" />
> ```
>
> Marketo will not render the image unless the token is placed inside a valid image tag.

![Email editor showing token usage](/help/assets/trigger-campaign/email-editor.png)

## Step 3: Add Tokens to the Campaign

1. Go to the **My Tokens** tab of the Smart Campaign
2. Add the necessary **Text Tokens**, such as:
   - `{{my.WebinarTitle}}`
   - `{{my.JoinLink}}`
   - `{{my.WebinarImage}}` (optional image URL)

Leave the values blank — they will be injected dynamically at runtime through the API.

> ✅ **Important:** Defining a token in the campaign does not automatically include it in your email. You must manually insert each token in the email editor as shown above.

![My Tokens tab in campaign](/help/assets/trigger-campaign/tokens-tab.png)

## Step 4: Approve and Activate the Campaign

Ensure the Smart Campaign is **approved and active**. Inactive campaigns cannot be triggered by API calls.

![Smart Campaign status set to Approved](/help/assets/trigger-campaign/campaign-status-approved.png)

## Step 5: Trigger the Campaign via REST API

Use the following endpoint:

```
POST /rest/v1/campaigns/{campaignId}/trigger.json
```

### Example Request Body:

```json
{
  "input": [
    {
      "id": 12345,
      "type": "Lead"
    }
  ],
  "tokens": [
    {
      "name": "{{my.WebinarTitle}}",
      "value": "Scaling Customer Engagement in 2025"
    },
    {
      "name": "{{my.JoinLink}}",
      "value": "https://webinars.company.com/join/abc123"
    },
    {
      "name": "{{my.WebinarImage}}",
      "value": "https://company.com/images/webinar-banner.png"
    }
  ]
}
```

Replace `12345` with the correct Lead ID from your Marketo instance.

## Authorization

All Marketo REST API requests require an OAuth 2.0 Bearer token.

To retrieve your access token:

```
GET /identity/oauth/token?grant_type=client_credentials&client_id=XXX&client_secret=YYY
```

Include the token in the header of your API requests:

```
Authorization: Bearer YOUR_ACCESS_TOKEN
```

## Best Practices

- Add fallback/default values to your tokens for testing and QA
- Use `{{lead.token}}` for lead fields, and `{{my.token}}` for campaign-scoped dynamic values
- Marketo supports up to **100 leads per request**
- Leads must meet the Smart List criteria — otherwise, they are silently skipped

## Summary

With this approach, you can personalize communications using Smart Campaigns that are triggered from external platforms via API. This is useful for scenarios like webinar registration confirmations, onboarding emails, and transactional notifications — all while injecting real-time data using My Tokens.
