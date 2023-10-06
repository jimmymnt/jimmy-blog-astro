---
title: "Slack API notification [Incoming Webhook]"
publishDate: "06 October 2023"
description: "Send data into Slack using this GitHub Action!"
tags: ["github-actions", "slack", "notification"]
---

# Slack Send GitHub Action

Send data into Slack using this GitHub Action!

## Sending Variables

You can send GitHub-specific data related to GitHub Action workflow events using [GitHub Contexts](https://docs.github.com/en/actions/learn-github-actions/contexts) and [Variables](https://docs.github.com/en/actions/learn-github-actions/variables) that GitHub Actions provides.

For examples on how to leverage this in your workflows, check out the [example workflows we have](https://github.com/slackapi/slack-github-action/tree/main/example-workflows).

## How to Send Data to Slack

This package has three different techniques to send data to Slack:

1) Send data to Slack's Workflow Builder (requires a paid Slack instance).
2) Send data via a Slack app to post to a specific channel (use an existing custom app or create a new one).
3) Send data via a Slack Incoming Webhook URL (use an existing custom app or create a new one).

**Today, we are going to focus on "Send data via a Slack Incoming Webhook URL".**

### Technique: Slack Incoming Webhook

This approach allows your GitHub Actions job to post a message to a Slack channel or direct message by utilizing [Incoming Webhooks](https://api.slack.com/messaging/webhooks).

Incoming Webhooks conform to the same rules and functionality as any of Slack's other messaging APIs. You can make your posted messages as simple as a single line of text, or make them really useful with [interactive components](https://api.slack.com/messaging/interactivity). To make the message more expressive and useful use [Block Kit](https://api.slack.com/block-kit) to build and test visual components.

#### Setup
* Log in to your Slack workspace.
* [Create a Slack App][apps] for your workspace (alternatively use an existing app you have already created and installed).
* Add the [`incoming-webhook`](https://api.slack.com/scopes/incoming-webhook) bot scope under **OAuth & Permissions**.
* Install the app to your workspace (you will select a channel to notify).
* Activate and create a new webhook under **Incoming Webhooks**.
* Copy the Webhook URL from the Webhook you just generated [add it as a secret in your repo settings][repo-secret] named `SLACK_WEBHOOK_URL`.
  * In your GitHub repository, go to `Settings" > "Secrets` and add a secret for your Slack Webhook URL. You can name it something like `SLACK_WEBHOOK_URL`. This secret will be used in your workflow to authenticate with the incoming webhook.

#### Usage

Create a `.github/workflows/slack-notification.yml` file in your repository to define your GitHub Actions workflow. Here's an example YAML file:

##### Using `payload`
```yaml
name: Send Slack Notification

on:
  push:
    branches:
      - main  # Change this to your desired branch

jobs:
  send-notification:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Send Slack Notification
        uses: slackapi/slack-github-action@v1.24.0
        with:
          channel-id: 'slack-channel-id'
          payload: |
            {
              "text": "This is the message is sent via Slack Notification in Github Actions"
            }
          env:
            SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
            SLACK_WEBHOOK_TYPE: INCOMING_WEBHOOK
```
**NOTE:**
```text
Replace 'slack-channel-id' with the actual Slack channel ID.
```

In this workflow:

* It triggers on pushes to the `main` branch. You can adjust the trigger event and branch as needed.
* It checks out your code using actions/checkout.
* It sends a Slack notification to the specified channel using the provided webhook URL.

##### Using `payload-file-path`
As you can see if your payload JSON is much more complicated, above format will not easy for you to maintain or edit if needed.
Rather using `payload` property, we can use `payload-file-path` to provide the path of the JSON file format, which will be a much easier way for us to perform any opening wide next time.

First, we need to create a file as JSON type at `.github/resources`. Lets create a file called `payload-example.json`, and put the content of the message inside that file.

E.g:
`./github/resources/payload-example.json`
```json
{
  "text": "This is the message is sent via Slack Notification in Github Actions"
}
```

```yaml
name: Send Slack Notification

on:
  push:
    branches:
      - main  # Change this to your desired branch

jobs:
  send-notification:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Send Slack Notification
        uses: slackapi/slack-github-action@v1.24.0
        with:
          channel-id: 'slack-channel-id'
          payload-file-path: ./github/resources/payload-example.json
          env:
            SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
            SLACK_WEBHOOK_TYPE: INCOMING_WEBHOOK
```

## Conclusion
That's it! You've set up GitHub Actions to send Slack notifications using an incoming webhook. This allows you to easily notify your team about important events and updates in your GitHub repository.


[create-webhook]: https://slack.com/intl/en-ca/help/articles/360041352714-Create-more-advanced-workflows-using-webhooks
[job-step]: https://docs.github.com/en/actions/learn-github-actions/workflow-syntax-for-github-actions#jobsjob_idsteps
[repo-secret]: https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets#creating-encrypted-secrets-for-a-repository
[apps]: https://api.slack.com/apps
