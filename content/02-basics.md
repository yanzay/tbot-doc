+++
title = "Basics"
weight = 1
type = "docs"
slug = "basics"
+++

# Telegram Bot Basics

## Register a Bot

First of all, you need to create a bot in Telegram. There is a special bot for this purpose - [@BotFather](https://t.me/BotFather). Open chat with BotFather and press “Start” to start chat with it. It will show you help message to guide through the steps:
- Click `/newbot` link in the help message, or send `/newbot` message to chat.
- Then BotFather will ask you for the name of your bot. You can choose anything you like, for example, MyTest.
- After the name, BotFather will ask for a username of the bot. It must end with “bot”, so users can differentiate it from real people.

You’re done! BotFather will provide authentication token for the bot, just provide it to tbot on initialization with `tbot.New()`.

## Get Updates from Telegram

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

## Use Telegram Bots API Client

To send messages, files, buttons, actions to the User we use API client. You can get preconfigured client from bot using `Client()` method:

```go
bot := tbot.New(token)
client := bot.Client()
```

This client provides wrappers for all API methods available on Telegram Bots API platform like `SendMessage`, `SendAction`, etc.

```go
client.SendMessage(chatID, "hello from tbot!")
```
