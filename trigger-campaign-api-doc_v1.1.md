
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

1. Go to **Marketing Activities**, and under your **Programs** folder, create a new **Smart Campaign** called `Send Webinar Reminder`.
2. In the **Smart List** tab, add the trigger:
   - `Campaign is Requested`
   - Set **Source** to: `Web Service API`

![Smart List trigger setup](/help/assets/trigger-campaign/01CampaignIsRequested.png)

## Step 2: Define the Email Content

Create or edit an email asset that references both lead and My Tokens.

Make sure to **insert the tokens directly into the email content**, as shown below:

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
> **Important:** Marketo will not render the image unless the token is placed inside a valid image tag. 

![Email editor showing token usage](/help/assets/trigger-campaign/02AddtokensToEMail.png


## Step 3: Add Tokens to the Program
To pass values dynamically via API, the tokens must already exist in Marketo. You’ll need to create them under the **My Tokens** tab of your Program.

1. Go to the **My Tokens** tab of your parent Program.
2. Drag in a **Text token** from the right-side panel for each dynamic value.
  - `{{my.WebinarTitle}}` — Text token
  - `{{my.JoinLink}}` — Text token
  - `{{my.WebinarImage}}` — Text token (this will be used as the `src` in an `<img>` tag)

![My Tokens tab in campaign](/help/assets/trigger-campaign/03MyTokens.png)


## Step 4: Set Campaign Qualification Rules and Activate Campaign

Configure the **qualification rules** to control how often a lead can run through the Smart Campaign.

Once configured, click **Activate** in the Smart Campaign to activate the Smart Campaign and able to receive API requests.

![Smart Campaign qualification rule](https://experienceleague.adobe.com/en/docs/marketo/using/product-docs/core-marketo-concepts/smart-campaigns/using-smart-campaigns/media_1391fa7996ec631e24efbaa374bd5a65863eb6d4f.png?width=600&format=png&optimize=medium)



## Step 5: Trigger the Campaign via REST API


###  Find the Campaign ID

To trigger a Smart Campaign via API, you'll need the **campaign ID**:

1. Find and click on the Smart Campaign you want to trigger.
2. Look at the URL in your browser. It will look something like this: `https://app-XXX.marketo.com/#/classic/SC*1234*A1ZN38`
3. The 4 digits after `SC` is your campaign ID, in the above example the Smart Campaign ID is '1234'



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
