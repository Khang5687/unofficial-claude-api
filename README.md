
# unofficial-claude2-api

## Table of Contents

- [Installation](#how-to-install)

  - [Requirements](#requirements)

- [Example Usage](#example-usage)

  - [Tips](#tips) *(chat-history/sessions/proxies)*

- [Troubleshooting](#troubleshooting)

## What is this?

This unofficial Python API provides access to the conversational capabilities of Anthropic's Claude AI through a simple chat messaging interface.

While not officially supported by Anthropic, this library can enable interesting conversational applications.

It allows for:

- Creating chat sessions with Claude and getting chat IDs.
- Sending messages to Claude containing up to 5 attachment files (txt, pdf, csv, etc...) 10 MB each.
- Retrieving chat message history.
- Deleting old chats when they are no longer needed.

### Some of the key things you can do with Claude through this API

- Ask questions about a wide variety of topics. Claude can chat about current events, pop culture, sports,
and more.

- Get helpful explanations on complex topics. Ask Claude to explain concepts and ideas in simple terms.

- Generate summaries from long text or documents. Just give the filepath as an attachment to Claude and get back a concise summary.

- Receive thoughtful responses to open-ended prompts and ideas. Claude can brainstorm ideas, expand on concepts, and have philosophical discussions.

## How to install

```shell
pip install unofficial-claude2-api
```

## Uninstallation

```shell
pip uninstall unofficial-claude2-api
```

## Requirements

### These requirements are needed to auto retrieve session cookie and UserAgent using selenium

- Firefox installed, and with at least one profile logged into [Claude](https://claude.ai/chats).
- [geckodriver](https://github.com/mozilla/geckodriver/releases) installed inside a folder registered in PATH environment variable.

***( scrolling through this README you'll find also a manual alternative )***

## Example Usage

```python
from .client import (
    ClaudeAPIClient,
    SendMessageResponse,
    MessageRateLimitError,
)
from .session import SessionData, get_session_data

# Wildcard import will also work the same as above
# from claude2_api import *

# List of attachments filepaths, up to 5, max 20 MB each
FILEPATH_LIST = [
    "test1.txt",
    "test2.txt",
]

# This function will automatically retrieve a SessionData instance using selenium
# Omitting profile argument will use default Firefox profile
data: SessionData = get_session_data()

# Initialize a client instance using a session
# Optionally change the requests timeout parameter to best fit your needs...default to 240 seconds.
client = ClaudeAPIClient(data, timeout=240)

# Create a new chat and cache the chat_id
chat_id = client.create_chat()
if not chat_id:
    # This will not throw MessageRateLimitError
    # But it still means that account has no more messages left.
    print("\nMessage limit hit, cannot create chat...")
    quit()

try:
    # Used for sending message with or without attachments
    # Returns a SendMessageResponse instance
    res: SendMessageResponse = client.send_message(
        chat_id, "Hello!", attachment_paths=FILEPATH_LIST
    )
    # Inspect answer
    if res.answer:
        print(res.answer)
    else:
        # Inspect response status code and error string
        print(f"\nError code {res.status_code}, response -> {res.error_response}")
except MessageRateLimitError as e:
    # The exception will hold these informations about the rate limit:
    print(f"\nMessage limit hit, resets at {e.reset_date}")
    print(f"\n{e.sleep_sec} seconds left until -> {e.reset_timestamp}")
    quit()
finally:
    # Perform chat deletion for cleanup
    client.delete_chat(chat_id)

# Get a list of all chats ids
all_chat_ids = client.get_all_chat_ids()
# Delete all chats
for chat in all_chat_ids:
    client.delete_chat(chat)

# Or by using a shortcut utility
#
# client.delete_all_chats()
```

## Tips

### How to access specific chat data  ( retrieving chat conversations )

```python
# A convenience method to access a specific chat conversation is
chat_data = client.get_chat_data(chat_id)
```

`chat_data` will be the same json dictionary returned by calling
`/api/organizations/{organization_id}/chat_conversations/{chat_id}`

Here's an example of this json:

```json
{
    "uuid": "<ConversationUUID>",
    "name": "",
    "summary": "",
    "model": null,
    "created_at": "1997-12-25T13:33:33.959409+00:00",
    "updated_at": "1997-12-25T13:33:39.487561+00:00",
    "chat_messages": [
        {
            "uuid": "<MessageUUID>",
            "text": "Who is Bugs Bunny?",
            "sender": "human",
            "index": 0,
            "created_at": "1997-12-25T13:33:39.487561+00:00",
            "updated_at": "1997-12-25T13:33:40.959409+00:00",
            "edited_at": null,
            "chat_feedback": null,
            "attachments": []
        },
        {
            "uuid": "<MessageUUID>",
            "text": "<Claude response's text>",
            "sender": "assistant",
            "index": 1,
            "created_at": "1997-12-25T13:33:40.959409+00:00",
            "updated_at": "1997-12-25T13:33:42.487561+00:00",
            "edited_at": null,
            "chat_feedback": null,
            "attachments": []
        }
    ]
}
```

### How to avoid using selenium ( faster loading )

If for whatever reason you'd like to avoid auto session gathering using selenium,
you just need to manually create a `SessionData` class for `ClaudeAPIClient` constructor, like so...

```python
from claude2_api.session import SessionData

cookie_header_value = "The entire Cookie header value string when you visit https://claude.ai/chats"
user_agent = "User agent to use, required"

data = SessionData(cookie_header_value, user_agent)
```

### How to set HTTP/S proxy

***NOTE ( Only proxies with no user/passwd authentication are supported )***

If you'd like to set a proxy for all requests, follow this example:

```py
from claude2_api.client import HTTPProxy, ClaudeAPIClient

# Create HTTPProxy instance
http_proxy = HTTPProxy(
    "the.proxy.ip.addr",    # Proxy IP
    8080,                   # Proxy port
    use_ssl=False           # Set to True if proxy uses HTTPS schema
)

# session = ...
# Give the proxy instance to ClaudeAPIClient constructor, along with session data.
client = ClaudeAPIClient(session, proxy=http_proxy)
```

______

## TROUBLESHOOTING

\****This bug should be already fixed after version 0.2.2***\*

This api will sometime return a 403 status_code when calling `send_message`, when this happens it is recommeded to look for these things:

- Check if your IP location is allowed, should be in US/UK, other locations may work sporadically.

- Don't try to send the same prompt/file over and over again, instead wait for some time, and change input.

## DISCLAIMER

This repository provides an unofficial API for automating free accounts on [claude.ai](https://claude.ai/chats).
Please note that this API is not endorsed, supported, or maintained by Anthropic. Use it at your own discretion and risk. Anthropic may make changes to their official product or APIs at any time, which could affect the functionality of this unofficial API. We do not guarantee the accuracy, reliability, or security of the information and data retrieved using this API. By using this repository, you agree that the maintainers are not responsible for any damages, issues, or consequences that may arise from its usage. Always refer to Anthropic's official documentation and terms of use. This project is maintained independently by contributors who are not affiliated with Anthropic.

## DONATING

A huge thank you in advance to anyone who wants to donate :)

[![Buy Me a Pizza](https://img.buymeacoffee.com/button-api/?text=1%20Pizza%20Margherita&emoji=🍕&slug=st1vms&button_colour=0fa913&font_colour=ffffff&font_family=Bree&outline_colour=ffffff&coffee_colour=FFDD00)](https://www.buymeacoffee.com/st1vms)
