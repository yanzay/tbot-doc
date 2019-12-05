+++
title = "Basics"
weight = 1
type = "docs"
slug = "basics"
# bookFlatSection = false
# bookToc: 6
# bookHidden: false
+++

# Telegram Bot Basics

## Register a Bot

TBD

## Interact with Telegram

Telegram bots can interact with Telegram servers in two different ways: using long polling or webhooks. In this section we will review both ways and how to implement them in tbot.

### Long polling

In long polling setup tbot act as a client, asking to Telegram server for updates and waiting when they're available. Example scenario in following diagram:

{{< mermaid >}}
sequenceDiagram
    tbot->>Telegram: Hello Telegram, any updates for me?
    Telegram-->>tbot: Yes, here they are!

    tbot->>Telegram: Great! And now?
    activate Telegram
    Note right of Telegram: Hm... Wait a min...
    Telegram-->>tbot: Found one!
    deactivate Telegram
{{< /mermaid >}}

tbot will use long polling by default, when you create new instance of `Bot`:

```go
bot := tbot.New(token)
```

This kind of setup is really handful during development and for moderate load bots. If your bot is suppose to be highly interactive or be used by a lot of users, the best production grade setup is using webhooks.

### Webhooks

In the webhooks setup tbot acts like a server, listening for a requests from Telegram with updates. 

It looks like on the following diagram:

{{< mermaid >}}
sequenceDiagram
    tbot->>Telegram: Hi! I'm listening on https://example.com
    activate Telegram
    Telegram->>tbot: User is typing!
    Telegram->>tbot: User sent you a message!
    Telegram->>tbot: User sent you a picture!
    Telegram->>tbot: User pressed the button!
    deactivate Telegram
{{< /mermaid >}}

To setup webhook just pass it's configuration on initializing:

```go
bot := tbot.New(token, tbot.WithWebhook("https://example.com", ":8080"))
```

`tbot.WithWebhook` function takes two parms:
- public URL of web service
- address for server to listen on

{{< hint warning >}}
**Note:** you need **HTTPS** endpoint with a valid certificate. You may want to use service like LetsEncrypt to get one.
{{< /hint >}}
