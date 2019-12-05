+++
title = "Quick Start"
weight = 1
type = "docs"
slug = "quick-start"
# bookFlatSection: false
# bookToc: 6
# bookHidden: false
+++

# Quick Start

## Installation

```bash
go get github.com/yanzay/tbot/v2
```

## Usage

Simple usage example:

```go
package main

import (
	"log"
	"os"
	"time"

	"github.com/yanzay/tbot/v2"
)

func main() {
	bot := tbot.New(os.Getenv("TELEGRAM_TOKEN"))
	c := bot.Client()
	bot.HandleMessage(".*yo.*", func(m *tbot.Message) {
		c.SendChatAction(m.Chat.ID, tbot.ActionTyping)
		time.Sleep(1 * time.Second)
		c.SendMessage(m.Chat.ID, "hello!")
	})
	err := bot.Start()
	if err != nil {
		log.Fatal(err)
	}
}
```
