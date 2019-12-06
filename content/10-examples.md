+++
title = "Examples"
weight = 1
type = "docs"
slug = "examples"
+++

# Examples

## Basic

```go
package main

import (
	"log"
	"os"

	"github.com/yanzay/tbot/v2"
)

func main() {
	bot := tbot.New(os.Getenv("TELEGRAM_TOKEN"))
	c := bot.Client()
	bot.HandleMessage(".*yo.*", func(m *tbot.Message) {
		c.SendMessage(m.Chat.ID, "hello!")
	})
	err := bot.Start()
	if err != nil {
		log.Fatal(err)
	}
}
```

## Webhook

```go
package main

import (
	"log"
	"os"

	"github.com/yanzay/tbot/v2"
)

func main() {
	bot := tbot.New(os.Getenv("TELEGRAM_TOKEN"),
		tbot.WithWebhook("https://test.com", ":8080"))
	c := bot.Client()
	bot.HandleMessage("ping", func(m *tbot.Message) {
		c.SendMessage(m.Chat.ID, "pong")
	})
	log.Fatal(bot.Start())
}
```

## Cowsay

```go
package main

import (
	"fmt"
	"os"
	"strings"
	"unicode/utf8"

	"github.com/yanzay/tbot/v2"
)

func main() {
	token := os.Getenv("TELEGRAM_TOKEN")
	bot := tbot.New(token)
	c := bot.Client()
	bot.HandleMessage("cowsay .+", func(m *tbot.Message) {
		text := strings.TrimPrefix(m.Text, "cowsay ")
		cow := fmt.Sprintf("```\n%s\n```", cowsay(text))
		c.SendMessage(m.Chat.ID, cow, tbot.OptParseModeMarkdown)
	})
	bot.Start()
}

func cowsay(text string) string {
	lineLen := utf8.RuneCountInString(text) + 2
	topLine := fmt.Sprintf(" %s ", strings.Repeat("_", lineLen))
	textLine := fmt.Sprintf("< %s >", text)
	bottomLine := fmt.Sprintf(" %s ", strings.Repeat("-", lineLen))
	cow := `
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
               ||----w |
               ||     ||
	`
	resp := fmt.Sprintf("%s\n%s\n%s%s", topLine, textLine, bottomLine, cow)
	return resp
}
```

## Middlewares

```go
package main

import (
	"log"
	"os"
	"time"

	"github.com/yanzay/tbot/v2"
)

func stat(h tbot.UpdateHandler) tbot.UpdateHandler {
	return func(u *tbot.Update) {
		start := time.Now()
		h(u)
		log.Printf("Handle time: %v", time.Now().Sub(start))
	}
}

func main() {
	bot := tbot.New(os.Getenv("TELEGRAM_TOKEN"))
	c := bot.Client()
	bot.Use(stat) // add stat middleware to bot
	bot.HandleMessage("", func(m *tbot.Message) {
		c.SendMessage(m.Chat.ID, "hello!")
	})
	err := bot.Start()
	if err != nil {
		log.Fatal(err)
	}
}
```

## Get File

```go
package main

import (
	"io"
	"log"
	"net/http"
	"os"

	"github.com/yanzay/tbot/v2"
)

func main() {
	token := os.Getenv("TELEGRAM_TOKEN")
	bot := tbot.New(token)
	client := bot.Client()
	bot.HandleMessage("", func(m *tbot.Message) {
		// here we check if message contains Document
		// you could also check for other types of files:
		// Audio, Photo, Video, etc.
		if m.Document != nil {
			doc, err := client.GetFile(m.Document.FileID)
			if err != nil {
				log.Println(err)
				return
			}
			url := client.FileURL(doc)
			resp, err := http.Get(url)
			if err != nil {
				log.Println(err)
				return
			}
			defer resp.Body.Close()
			out, err := os.Create(m.Document.FileName)
			if err != nil {
				log.Println(err)
				return
			}
			defer out.Close()
			io.Copy(out, resp.Body)
		}
	})
	log.Fatal(bot.Start())
}
```

## Send Image

```go
package main

import (
	"fmt"
	"log"
	"os"

	"github.com/yanzay/tbot/v2"
)

func main() {
	bot := tbot.New(os.Getenv("TELEGRAM_TOKEN"))
	c := bot.Client()
	bot.HandleMessage("image", func(m *tbot.Message) {
		_, err := c.SendPhotoFile(m.Chat.ID, "image.png", tbot.OptCaption("this is image"))
		if err != nil {
			fmt.Println(err)
		}
	})
	err := bot.Start()
	if err != nil {
		log.Fatal(err)
	}
}
```

## Polls

```go
package main

import (
	"fmt"
	"os"

	"github.com/yanzay/tbot/v2"
)

var client *tbot.Client

func main() {
	bot := tbot.New(os.Getenv("TELEGRAM_TOKEN"))
	client = bot.Client()
	// listen poll message and send poll
	bot.HandleMessage("poll", sendPoll)
	// handle poll updates, just print on the screen
	bot.HandlePollUpdate(func(p *tbot.Poll) {
		fmt.Println("Poll update received:")
		fmt.Println(p.Question)
		for _, opt := range p.Options {
			fmt.Println(opt.Text, opt.VoterCount)
		}
	})
	bot.Start()
}

func sendPoll(m *tbot.Message) {
	options := []string{
		"Perfect",
		"Good",
		"So so",
	}
	client.SendPoll(m.Chat.ID, "How are you?", options)
}
```

## Inline Buttons

```go
package main

import (
	"fmt"
	"os"

	"github.com/yanzay/tbot/v2"
)

type application struct {
	client  *tbot.Client
	votings map[string]*voting
}

type voting struct {
	ups   int
	downs int
}

func main() {
	token := os.Getenv("TELEGRAM_TOKEN")
	bot := tbot.New(token)
	app := &application{
		votings: make(map[string]*voting),
	}
	app.client = bot.Client()
	bot.HandleMessage("/vote", app.votingHandler)
	bot.HandleCallback(app.callbackHandler)
	bot.Start()
}

func (a *application) votingHandler(m *tbot.Message) {
	buttons := makeButtons(0, 0)
	msg, _ := a.client.SendMessage(m.Chat.ID, "Please vote", tbot.OptInlineKeyboardMarkup(buttons))
	votingID := fmt.Sprintf("%s:%d", m.Chat.ID, msg.MessageID)
	a.votings[votingID] = &voting{}
}

func (a *application) callbackHandler(cq *tbot.CallbackQuery) {
	votingID := fmt.Sprintf("%s:%d", cq.Message.Chat.ID, cq.Message.MessageID)
	v := a.votings[votingID]
	if cq.Data == "up" {
		v.ups++
	}
	if cq.Data == "down" {
		v.downs++
	}
	buttons := makeButtons(v.ups, v.downs)
	a.client.EditMessageReplyMarkup(cq.Message.Chat.ID, cq.Message.MessageID, tbot.OptInlineKeyboardMarkup(buttons))
	a.client.AnswerCallbackQuery(cq.ID, tbot.OptText("OK"))
}

func makeButtons(ups, downs int) *tbot.InlineKeyboardMarkup {
	button1 := tbot.InlineKeyboardButton{
		Text:         fmt.Sprintf("👍 %d", ups),
		CallbackData: "up",
	}
	button2 := tbot.InlineKeyboardButton{
		Text:         fmt.Sprintf("👎 %d", downs),
		CallbackData: "down",
	}
	return &tbot.InlineKeyboardMarkup{
		InlineKeyboard: [][]tbot.InlineKeyboardButton{
			[]tbot.InlineKeyboardButton{button1, button2},
		},
	}
}
```
