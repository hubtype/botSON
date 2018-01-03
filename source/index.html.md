---
title: botSON Reference

toc_footers:
  - <a href='https://hubtype.com/'>Hubtype</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

search: true

---

#Introduction

> This is the simplest possible bot in botSON.

```json
{
  "version": "1.0",
  "states": [
    {
      "label": "initial",
      "output": "Hello World!",
      "next_step": "exit"
    }
  ]
}
```

botSON is a JSON based programming language to create chatbots for the [Hubtype](https://hubtype.com) platform. The goal of botSON is to be a simple yet powerful tool to build conversational flows that the user navigates by feeding in text or media. In a simplified way, chatbots are defined as finite state machines, at each state the chatbot outputs some text and waits for the user input that makes the bot transition to another state. You can optionally incorporate Natural Language Understanding to your bot by integrating services like IBM Watson or API.ai. A [context](#context) is kept for every bot session (the extent of a user's conversation) which can be updated at every step with variables and external http calls.

Additionally, botSON supports [triggers](#triggers), which are high priority actions that the user can invoke regardless of the current state of the conversation.

botSON is messenger agnostic, meaning that it's not designed specifically for a single messaging app but it works with any messaging channel supported by Hubtype, however not all interactive elements are supported by all channels (like [carrousels](#carrousel)).

We encourage you to watch the screencasts:

* Part 1: [Scaffolding and publishing](https://www.youtube.com/watch?v=h53SjVGMwos)

#Getting Started

##Creating a state

```json
{
  "version": "1.0",
  "states": [
    {
      "label": "initial",
      "next_step": "helloworld"
    },
    {
      "label": "helloworld",
      "output": "Hello World!",
      "next_step": "exit"
    }
  ]
}
```

A bot is composed by states. Each state has a `label`, a `next_step` and usually an `output` or `input` (or both). When the bot enters a state, the first thing it does is to send the `output` (if there's one) to the user, then awaits for the user input if there's an `input` section. Finally it jumps to the state defined in the expression `next_step`. The state 'exit' indicates the end of the session, if the user talks again then a new session will start from the state defined in 'initial_state'.

When the bot enters a state with no `input`, it just sends the `output` (if any) and jumps to `next_state` immediatelly. It is possible to chain several state jumps without any `input`, the bot will just send all the outputs consecutively until it reaches a state with an `input` statement, then it will pause until the user sends a new message.

##Basic bot

```json
{
    "input_retry": 3,
    "version": "1.0",
    "states": [
        {
            "label": "first_state",
            "next_step": "choice"
        },
        {
            "label": "choice",
            "output": {
                "type": "text",
                "data": "Here you have to choose:",
                "keyboard": [
                    {"label": "Red", "data": "RED"},
                    {"label": "Blue", "data": "BLUE"},
                    {"label": "Green", "data": "GREEN"}
                ]
            },
            "input": {
                "type": "in_keyboard",
                "variable": "user_choice"
            },
            "next_step": "result"
        },
        {
            "label": "result",
            "output": "You're choice was {{user_choice.data}}",
            "next_step": "choice"
        },
        {
            "label": "input_failure",
            "output": "I don't understand what you're trying to tell me, it seems I'm not smart enough for you =(",
            "next_step": "choice"
        }
    ]
}
```

A complete bot usually never ends, it loops back to previous states. Also, it can use variables and other forms of `input` and `output` other than simple text.

In this case, after the first text of the user, this bot will show three quick replies: 'Red', 'Blue' and 'Green'.

The bot will wait for the user to click on one of these quick replies and then it will store the key pressed in the context variable `user_choice`. For example, if the user clicks on "Red", the bot will set `user_choice` to `{"label": "Red", "data": "RED"}` and then jump to the next state `result`.

If the user wrote any random sentence, the chatbot would ignore that input and prompt the keyboard again. If the user fails to give an apropiate answer 3 times (this can be configured using the `input_retry` setting) it will go to the state `input_failure` which is defined under the hood by default but it can be overridden.

Finally, in the state `result` the bot sends a text message saying what was the pick of the user and returns to the `choice` state.

##botSON code
```json
{
  "something": "this is a multiline
    line because
    botSON is cooler than JSON"
}
```
botSON and JSON are slightly different, botSON allows multiline strings while JSON don't. 
<aside class="softwarn">
Please keep in mind that if there is some error in those lines the bug report will point to the first line.
</aside>

#Top level properties

##input_retry

```json
{
  "input_retry": 2,
  "name": "The Weather Bot",
  "defaults": {
    "requests": {
      "headers": {
        "Authorization": "Bearer mytoken"
      }
    },
    "context": {
      "API_URL": "https://api.example.com",
      "foo": "bar"
    }
  },
  "triggers": {},
  "states": []
}
```

Specifies the number of input failures before the machine goes to the 'input_failure' state. By default is set to 3.
An input failure is given when the [type of input](#input-actions) doesn't match the required data. 

##name

The name of the bot.

##version

Version of the BotSON language. We encourage you to explicitly set this setting to "1.0" as it's the version this documentation refers to. If not set, the default value in "0.1", a version that is NOT compatible with "1.0".

##language

Indicates the language the bot will use.

<aside class="softwarn">
Currently this field is not used.
</aside>

##initial_state

Label of the first state of the bot. Defaults to "initial".

##defaults

Defines the default values that are used throughout the chatbot flow in different states:

* **requests**

  Currently only supports default `headers` to be used in every url request.

* **delay**:

  Time (in seconds) to delay sending any outputs. It can be a decimal number. This parameter can be overridden in each output. Defaults to 0.

* **typing**

  Time (in seconds) that the bot will display the "typing..." effect before sending this output. It can be a decimal number. If the typing effect is not supported by the messenger it will result in extra delay time. This parameter can be overridden in each output. Defaults to 0.

* **context**

  Variables that will be always available at any state of the bot. They are recalculated every time there is an input, so, if a bot definition is changed, previous conversations will have updated variables.

##literals

TODO

##session_timeout

Time (in minutes) after the last user interaction to wait before the session is closed automatically. The default is `null` which means the session will never be closed automatically, only if a state with `"next_step": "exit"` is reached.

##keep_session

TODO

##keep_trace

TODO

##early_typing

TODO

##analytics

TODO

##triggers

```json
{
  "triggers": {
    "payload":[
      {
        "match": "^WATCH_VIDEO_(?P<video_id>[a-z0-9-]+)$",
        "context": {
          "video_url": "videos/{{video_id}}.mp4"
        },
        "next_step": "watch_video"
      }
    ],
    "text":[
      {
        "match": "trigger_(?P<id>\\w+)",
        "next_step": "trigger_{{id}}"
      },
      {
        "match": "help",
        "next_step": "trigger_help"
      }
    ]
  }
}
```

Triggers are commands that are available at any point during the bot execution, regardless of its current state.

* `text` triggers can be fired by user inputs or payloads

* `payload` triggers can not be fired by user messages. *(Higher priority than text triggers.)*

Triggers are matched with regular expressions. If the regexp contains named groups, those will be added as variables with the corresponding matched value. When a user input is captured by a trigger, it is not consumed by the current state.

Triggers must have valid `next_step` and `match` attributes. If `next_step` is `null` the bot will consume that input without any change, which is useful if you want to ignore some words. See [next_step](#next_step) reference for more info.

Usually triggers are used to capture button clicks (what Facebook calls 'Postbacks') and general commands like "help".

##states

This field contains an array of all the states of the bot.

When the bot enters a state, the first thing it does is to update the context, then it writes all the outputs (if any) and finally wait for a user input (if defined). Finally, it jumps to the state defined in the `next_step` expression.

```json
{
  "version":"1.0",
  "initial_state":"first_state",
  "states": [
    {
      "label": "first_state",
      "input": {
        "type": "free-text",
        "variable": "first_input_from_user"
      },
      "output": {
        "type": "text",
        "data": "data_of_output"
      },
      "next_step": "next_state_in_flow"
    },
    {
      "label": "next_state_in_flow",
      "context": {
        "variable_name": "variable_value"
      },
      "output": [
        {
          "type": "text",
          "data": "data_of_output_in_array"
        },
        {
          "type": "text",
          "data": "another_output_in_array"
        },
        {
          "type": "text",
          "data": "variable_name value: {{variable_name}}"
        }
      ],
      "next_step": "next_state_in_flow"
    }
  ]
}
```

> Default input_failure

```json
{
    "label": "input_failure",
    "output": "input_failure",
    "next_step": "exit"
}
```

> Default external_request_failure

```json
{
    "label": "external_request_failure",
    "output": "external_request_failure",
    "next_step": "exit"
}
```

> Default fallback_instruction

```json
{
    "label": "fallback_instruction",
    "output": "fallback_instruction",
    "next_step": "exit"
}
```

> Default loop_overflow

```json
{
    "label": "loop_overflow",
    "output": "loop_overflow",
    "next_step": "exit"
}
```

It is required to set some 'initial_state', that is the first state in the conversation flow. 'initial_state' will only start after the user sends it's first input. Then, it will update the context, write the outputs and go to `next_step`.

###label

The state name.

###context 

*(Optional)*

Variables to be assigned when entering the state. See [context](#context).

###output 

*(Optional)*

The bot will send in order all outputs defined here. It has different possible [data types](#output-types).

###input 

*(Optional)*

Input of the bot. It can come from the user or as a response to url calls. The field "actions" specifies the expected 
[type of input](#input-actions).

###next_step

Name of the next state in the flow of the bot. For a conditional next_step (i.e. jumping to different states 
depending of variables) see [templating](#templating).


### Implicit states

There are some implicit states that are always defined under the hood. You can always redefine these states in your code in order to customize their default behaviour.

* `input_failure`: The chatbot jumps to this state when the user fails to give an appropiate answer after N attempts. This can be controlled using the global setting `input_retry`.

* `external_request_failure`: The chatbot jumps to this state when it fails to make an http request. This can be triggered for a variety of reasons, for example, if the request took too long or the response code was ouside the 2XX range.

* `fallback_instruction`: The chatbot jumps to this state when it encounters a `next_step` statement that points to a label that doesn't exist. This can happen when you define a dynamic `next_step` using some templating expression like `my_state_{{option}}` and the variable `option` is not defined.

* `loop_overflow`: The chatbot jumps to this state when it's been jumping from state to state N times without finding any "input" statement which makes the bot to pause and wait for user input. This is needed in order to prevent infinite loops where state A has a next_step to state B and vice versa. You can set the max number of iterations using the global setting `max_loop`, but it can't be higher than 50.


#Context

> The context is just a set of variables that can contain any JSON type

```json
{
  "context": {
    "user": {
      "name": "John",
      "last_name": "Doe"
    },
    "is_customer": true
  }
}
```

> Context with a forced order of execution

```json
{
  "context": [
    {"first_var": 1},
    {"second_var": "This depends on the {{first_var}}"}
  ]
}
```
The context is a set of variables that is carried along during the whole session. Everytime the bot transitions to a new state, the context is updated with all the variables defined in the new state `context` statement. Context variables can then be used to build the outputs dynamically and in other places by using the [templating syntax](#templating).

<aside class="softwarn">
NOTE: JSON objects have no ordering by definition. So if you define a set of variables in the context, we can't guarantee they'll be set in the order you write them. If you need to force a predefined order you can use the array notation (see example on the right).
</aside>

##Context actions

> In this example, 'new_user' will contain the json response of the call.

```json

{
  "new_user": "{{http('http://my-api.example.com/new_user', params={'q': 'apples'})}}"
}
```

> In this example, 'available_queues' will contain an array of IDs: the queues that are open in Hubtype DESK.

```json

{
  "available_queues": "{{active_queues()}}"
}
```

> Here we create a note so the human agent that is going to attend the conversation can quickly see a summary of the customer.

```json

{
  "content": "User name: {{user.name}}. Age: {{user.age}}. Order number: {{order_num}}.",
  "note": "{{add_note(content)}}"
}
```

> Finally, we create a case in one of the available queues or in the default one if there are none available.

```json

{
  "case": "{{create_case(available_queues[0] if len(available_queues) else 'Default queue')}}"
}
```

Context actions are used to interact with the outside world. Probably the most handy action is `http`, which allows you to make calls to any standard JSON API. Here's a complete list of available actions:

* **http**: Makes a starndard http request to a url and retrieves the response in json. The expected parameters are:

| Action     | Type           |   |
| --------- |:-------------:| -----:|
| url      | String          |  |
| method | String (GET, POST, PUT, PATCH, DELETE, HEAD)        | Optional. Defaults to GET |
| headers | Object        | Optional |
| params | Object        | Optional. Query parameters (GET params) |
| data | Object        | Optional. Form data to send along with the request |
| json | Object        | Optional. JSON data to send in the body of the request |

* **active_queues**: Returns the IDs of the queues in status "Open" from your organization at Hubtype DESK. Useful when your chatbot has qualified a lead and has to handover the conversation to a human agent.

* **add_note**: It creates a note in the conversation. You can use this to make a summary of the user details before a human agent handover.

| Action     | Type           |   |
| --------- |:-------------:| -----:|
| text      | String          | Content of the note |

* **create_case**: It creates a case in Hubtype DESK so that a human agent gets over the conversation. While the case is open, the chatbot won't respond to any user input. Only when the agent resolves the case the chatbot will continue its flow.

| Action     | Type           |   |
| --------- |:-------------:| -----:|
| queue      | String          | ID or name of the queue that the case will be created on |

<aside class="softwarn">
NOTE: active_queues are the only ones that some user can be transferred
</aside>

##Special context variables

There are some special variables in the context that the bot populates automatically when a new session starts or when certain events occur:

###User
```json
{
  "user":{
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "name": "John Doe",
    "provider": "facebook | telegram | twitter...",
    "username": "j_doe",
    "provider_id": "12345677"
  }
}
```
This variable contains the main information about the provider from which the user contacts. It can be useful to prevent sending some kind of output that is not supported on specifics providers.

###Organization
```json
{
  "organization": "ACME"
}
```
This is the name of your organization at Hubtype.

###Bot
```json
{
  "bot": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "name": "Charlie Bot"
  }
}
```
Contains a unique identifier set when you created the bot. Also the name of the bot stated in the "name" attribute.

###Trace
```json
{
  "_trace":["initial", "welcome_message", "show_products", "buy_item"]
}
```
List of states for which the user has passed. It can be useful to retrieve behaviour data about your users.

###First text

```json
{
  "first_text": "Hey there!"
}
```

First text message from the user

###Last session
```json
{
  "last_session":{
    "last_state": "buy_item",
    "created_at": "2017-10-24T15:05:31.949553Z",
    "last_interaction": {
      "created_at": "2017-01-24T16:05:31.949553Z",
      "_input": "Bye bye!"
    }
  }
}
```

All information you need about the last time this user contacts the bot.

###Last keyboard
```json
{
  "_last_keyboard": [
    {"label": "Yeah", "data": "yes"},
    {"label": "Nope", "data": "no"}
  ]
}
```

Array with the different options of last shown keyboard

###Hubtype case typification and status
```json
{
  "_hubtype_case_typification": "timetable",
  "_hubtype_case_status": "result_ok"
}
```

The typification and status of a case originated from this bot given by the human agent that resolved the case.

#Outputs

> Valid output expressions

```json
{
    "output": "This is a simple text"
}
{
    "output": ["This", "is", "a", "sequence", "of", "messages"]
}
{
    "output": {
        "type": "text",
        "delay": 0.5,
        "typing": 1,
        "data": "This is a text in its expanded form"
    }
}
```

Outputs are the messages that the bot sends to the user when it reaches the current state. Outputs can be defined in a variety of different ways: a simple text string, an object representing the expanded message format (see types of messages in the next section) or an array of messages (either in the simple text format or the expanded object).

All outputs accept the following parameters:

| Field     | Value           |   |
| --------- |:-------------:| -----:|
| delay      | Number | Time (in seconds) to delay sending this output. It can be a decimal number. Defaults to 0. |
| typing      | Number | Time (in seconds) that the bot will display the "typing..." effect. It can be a decimal number. Defaults to 0. |

<aside class="softwarn">
NOTE: If the typing effect is not supported by the messenger it will result in extra delay time.
</aside>

`delay` and `typing` defined in the output will override the global settings in the `defaults` section.

Next, we will define all the available output types:

##Text

```json
{
  "type": "text",
  "data": "Done! 😉"
}
```
| Field     | Value           |   |
| --------- |:-------------:| -----:|
| type      | "text" | |
| data      | String      | *Any UTF-8 encoded string*|

##Image

```json
{
  "type": "image",
  "data": "http://this.is.the/url/of/the/image.jpg"
}
```

| Field     | Value           |   |
| --------- |:-------------:| -----:|
| type      | "image" | |
| data      | String      | *The image URL*|

##Video

```json
{
  "type": "video",
  "data": "http://this.is.the/url/of/the/video.mp4"
}

```

| Field     | Value           |   |
| --------- |:-------------:| -----:|
| type      | "video" | |
| data      | String      | |

##Audio

```json
{
  "type": "audio",
  "data": "http://this.is.the/url/of/the/audio.mp3",
  "caption": "Some video subtitle"
}

```

| Field     | Value           |   |
| --------- |:-------------:| -----:|
| type      | "audio" | |
| data      | String      | |
| caption   | String      | *optional*|

##Document

```json
{
  "type": "document",
  "data": "http://this.is.the/url/of/the/document.pdf",
  "caption": "Some document subtitle"
}
```
| Field     | Value           |   |
| --------- |:-------------:| -----:|
| type      | "image" | |
| data      | String      | |
| caption   | String      | *optional*|

##Location

```json
{
  "type": "location",
  "latitude": 41.412255,
  "longitude": 2.2079313,
  "title": "My Home",
  "address": "Hollywood Boulevard 32",
  "url": "http://any.url.com"
}
```

| Field     | Value           |   |
| --------- |:-------------:| -----:|
| type      | "location" | |
| latitude  | Number      |    |
| longitude | Number      |    |
| title     | String      |  *optional*<br>*32 chars max*  |
| address   | String      |  *optional*  |
| url       | String      |  *optional*  |

<aside class="softwarn">
NOTE: The Location Output is not available on Facebook.
</aside>

##Contact

```json
{
  "type": "contact",
  "first_name": "John",
  "last_name": "Doe",
  "phone_number": "678909909",
  "vcard": "vcard:data"                 
}
```

| Field       | Value           |   |
| ---------   |:-------------:| -----:|
| type        | "contact" | |
| first_name  | String      |    |
| last_name   | String      |  *optional*  |
| phone_number| Number      |  *optional*  |
| vcard       | String      |  *optional*  |

##Buttonmessage
```json
{
  "type": "buttonmessage",
  "text": "Pick one option",
  "buttons": [
    {
      "type":"postback",
      "title":"Option 1",
      "payload":"POSTBACK_1"
    },
    {
      "type":"postback",
      "title":"Option 2",
      "payload":"POSTBACK_2"
    },
    {
      "type":"web_url",
      "title":"Web url",
      "url":"https://www.google.es/"
    },
    {
      "type" : "phone_number",
      "title" : "Call",
      "payload" : "{% raw %}+44 7700 900200{% endraw %}"
    }
  ]
}
```

A collection of at most 4 buttons displayed vertically.

###Button:

| Field     | Value           |   |
| --------- |:-------------:| -----:|
| type                 | "web_url", "postback" or "phone_number" | |
| title                | String  |   |
| webview_height_ratio | String | *Possible in web_url* |
| messenger_extensions | String | *Possible in web_url* |
| fallback_url         | String | *Possible in web_url* | 
| url                  | String | *mandatory if type is "web_url"*  |
| payload              | String | *mandatory if type is "postback" or "phone_number"* |
| next_step            | String | *optional substitute of payload when type is "postback"* |

<aside class="softwarn">
NOTE: The messagebutton output is only available on Facebook and Telegram. Also, phone_number button type only is 
available on Facebook. Currently other platforms will ignore that output.
</aside>

##Carrousel

```json
{
  "type": "carrousel",
  "elements":[
    {
      "title": "Title Element 1",
      "subtitle": "My description",
      "image_url": "https://www.cover.image.jpg",
      "buttons":[
        {
          "type": "postback",
          "title": "My Button",
          "payload": "EXTRA_DATA"
        }
      ]
    },
    {
      "title": "Title Element 2",
      "subtitle": "My Description",
      "image_url": "https://www.cover.image.jpg",
      "buttons":[
        {
          "type": "web_url",
          "title": "Darse de baja",
          "url": "https://www.some.web.com"
        }
      ]
    }
  ]
}
```

A rich interactive message that displays a horizontal scrollable carousel of items, each composed of an image 
attachment, short description and buttons to request input from the user.

<img width="300" src="/images/carrousel.png">

| Field     | Value           |   |
| --------- |:-------------:| -----:|
| type      | "carrousel" | |
| elements  | Array of Elements  | *10 elements max (will truncate)* |

###Element:

| Field     | Value           |   |
| --------- |:-------------:| -----:|
| title      | String | |
| subtitle  | String  | |
| image_url  | String  | |
| [buttons](#buttonmessage)  | Array of Buttons  | *3 elements max (will truncate)* |

<aside class="softwarn">
NOTE: The carrousel output is only available on Facebook. Currently other platforms will ignore that output.
</aside>

##List
```json
{
    "type": "list",
    "elements": [{
        "title": "First title",
        "subtitle": "Some subtitle",
        "image_url": "https://www.url/g/image.png",
        "buttons": [{
            "type" : "phone_number",
            "title" : "Call",
            "payload" : "{% raw %}+44 7700 900200{% endraw %}"
        }]
        },
        {
        "title": "Second title",
        "subtitle": "Oh my god",
        "image_url": "https://ToBeOrNot.ToBe.an.url/image.jpg",
        "buttons": [{
            "type": "web_url",
            "title": "Go details",
            "url": "http://www.details.url/#section"

        }]
    }]
}
```

| Field     | Value           |   |
| --------- |:-------------:| -----:|
| [elements](#carrousel) | Array of Elements  | *min 2 elements <br> max 4 elements*   |

<aside class="softwarn">
NOTE: The carrousel output is only available on Facebook. Currently other platforms will ignore that output.
</aside>

##Quick Replies

```json
{
  "type": "text",
  "data": "What do you need?",
  "keyboard": [
    {
      "label": "Option One",
      "data": "option_1"
    },
    {
      "label": "Option Two",
      "data": "option_2"
    },
    {
      "label": "Option Three",
      "data": "option_3"
    }
  ]
}
```

Quick Replies are a convenient way to let users know the available options to type. Also, they save the end-user the 
effort to type all words and let them navigate through options much faster.

Any message type accepts a `"keyboard"` field with an array of (`"label"`, `"data"`) pairs.

<aside class="softwarn">
NOTE: Telegram only uses 'label', not 'data'.
</aside>

<aside class="notice">
Note that using get_in_keyboard input type you can access both 'label' and 'data' from the selected key, even if the 
bot is on Telegram. 
</aside>

##Receipt

```json
{
  "type": "receipt",
  "recipient_name": "{{user.name}}",
  "order_number": "123",
  "currency": "EUR",
  "payment_method": "Visa",
  "summary": {
    "total_cost": 10.0
  }
}
```

```json
{
  "type": "receipt",
  "recipient_name": "{{user.name}}",
  "order_number": "1010101010",
  "currency": "EUR",
  "payment_method": "MasterCard",
  "summary": {
    "total_cost": 30.96,
    "subtotal": 27.4,
    "total_tax": 3.56,
    "shipping_cost": 0
  },
  "merchant_name": "Hubtype",
  "timestamp": "10/3/17",
  "order_url": "https://hubtype.com/",
  "elements": [
    {
      "title": "item1",
      "price": 3,
      "quantity": 2
    },
    {
      "title": "item2",
      "price": 1,
      "subtitle": "this is a subtitle",
      "currency": "USD"
    },
    {
      "title": "kitten",
      "subtitle": "adorable",
      "quantity": 1,
      "price": 10,
      "currency": "EUR",
      "image_url": "http://i.telegraph.co.uk/multimedia/archive/02830/cat_2830677b.jpg"
    }
  ],
  "address": {
    "street_1": "Street1",
    "city": "city",
    "postal_code": "010101",
    "state": "state",
    "country": "country",
    "street_2": "Street2"
  },
  "adjustments": [
    {
      "name": "Special discount",
      "amount": 10
    }
  ]
}
```

This output is used to send receipts to users. It can be simple with not much information or very detailed. The user 
will see a summarised version that will be expanded with all the information when the user click it.

<img width="300" src="/images/receipt.png">

Simple receipt, with minimum information. (code example produces this receipt)

<img width="300" src="/images/receipt2.png">

Detailed receipt that uses all possible fields.

<img width="300" src="/images/receipt2_expanded.png">

Previous receipt expanded when clicked by the user.

| Field          | Value  ||
| -------------  |:------:| -----:|
| recipient_name | String ||
| merchant_name  | String | *Optional* |
| order_number   | String ||
| currency       | String ||
| payment_method | String ||
| timestamp      | String | *Optional* |
| order_url      | String | *Optional* |
| elements       | Array of ReceiptElement | *Optional<br>Facebook doesn't<br>guarantee the order* |
| address        | Address | *Optional* |
| summary        | Summary ||
| adjustments    | Array of Adjustment | *Optional* |

###ReceiptElement
| Field    | Value  |   |
| -------- |:------:| -----:|
| title    | String ||
| subtitle | String | *Optional* |
| quantity | Number | *Optional* |
| price    | Number | *Can be 0* |
| currency | String | *Optional* |
| image_url | Link  | *Optional* |

###Address
| Field   | Value  |   |
| ------- |:------:| -----:|
| street1 | String ||
| street2 | String | *Optional* |
| city    | String ||
| postal_code | String ||
| state   | String ||
| country | String ||

###Summary
| Field         | Value  |            |
| ------------- |:------:| ----------:|
| subtotal      | Number | *Optional* |
| shipping_cost | Number | *Optional* |
| total_tax     | Number | *Optional* |
| total_cost    | Number ||

###Adjustment
| Field  | Value  |   |
| ------ |:------:| -----:|
| name   | String ||
| amount | Number ||

<aside class="softwarn">
NOTE: The receipt output is only available on Facebook. Currently other platforms will ignore that output.
</aside>

#Input actions

`"type"` specifies what is the expected action and `"data"` will be stored in the specified variable. To access the
variable see [Templating](#templating).

##Free Text

```json
{
  "type": "free_text",
  "variable": "text_var_name"
}
```

Accepts any text from the user and stores it in the variable.

##Get Integer

```json
{
  "type": "int",
  "variable": "name_of_int_var"
}
```

Accepts any integer as input , other kinds of inputs are rejected.

##Get in Set

```json
{
  "type": "in_set",
  "variable": "name_of_option",
  "action_parameters": ["op1", "op2"]
}
```

Only accepts these parameters as answer. It will ask at most `input_retry` times.

| Field     | Value           |
| --------- |:-------------:|
| action_parameters | Array of strings |

##Get in Set Fuzzy

```json
{
  "type": "in_set_fuzzy",
  "variable": "object_user_choice",
  "action_parameters": ["object1", "object2"]
}
```

Same as get in set, but it does not require a perfect match.

| Field             | Value            |
| ----------------- |:----------------:|
| action_parameters | Array of strings |

##Get in keyboard

```json
{
  "label": "in_keyboard_example",
  "output": {
    "type": "text",
    "data": "Select one option:",
    "keyboard": [
      {"label": "Item 1", "data": "1"},
      {"label": "Item 2", "data": "2"}
    ]
  },
  "input": {
    "type": "in_keyboard",
    "variable": "key_user_choice"
  },
  "next_step": "another_state"
}
```

Puts in the variable the key object (*label, data*) of the key pressed by the user. There must be a 
[keyboard](#quick-replies) present.

You can access both variables using jinja: `{{key_user_choice.label}}` and `{{key_user_choice.data}}`

##Get yes or no

```json
{
  "type": "yes_no",
  "variable": "confirm_var_name"
}
```

The accepted outputs are `yes`, `si`, `sí` and `no` in lowercase or uppercase.

##Get from url

```json
{
  "type": "from_url",
  "variable": "var_name_to_save",
  "action_parameters": {
    "url": "{{BASE_URL}}path/search/",
    "params": {
      "param1": "{{_input}}",
      "param2": "val_2"
    }
  }
}
```

It makes a call to the url and stores the response json object in the variable name.

| Field     | Value           |
| --------- |:-------------:|
| url | Get url |
| params | Array of parameters for the get call |

##Get name

```json
{
  "type":"name",
  "variables":"confirm_get_name"
}
```

The user input should be a text with at most 3 words.

##Get email

```json
{
  "type": "email",
  "variable": "user_email"
}
```

Gets the email and check if it is well writed. **something@someother**

##Get age

```json
{
  "type":"age",
  "variable":"age"
}
```

Saves a number representing the age. Age should match 1 <= age < 120.

##Get location

```json
{
  "type":"location",
  "variable":"location"
}
```

The variable location will contain the components: `latitude` , `longitude`, `title`, `address`, `url`. For field 
info see [location](#location).

<aside class="softwarn">
NOTE: The get image input is only available on Facebook. Currently other platforms will ignore that output.
</aside>

##Get image
```json
{
  "type": "image",
  "variable": "the_image_url"
}
```
It will save in the variable the url given by the provider.

<aside class="softwarn">
NOTE: The get image input is only available on Facebook. Currently other platforms will ignore that output.
</aside>

#Templating

> Basic templating

```json
{
  "output": "Good morning {{user.name}}"
}
```

> Templating with array expansion

```json
{
  "output": "[{% for t in toppings %}
      {
        'type': 'text',
        'data': 'Topping {{loop.index}}: {{t.name}}',
      },
    {% endfor %}]"
}
```

Variables stored in the context can be used to construct outputs dynamically, to define new context variables or to determine the `next_step` on runtime.

We use [Jinja](http://jinja.pocoo.org/) templating framework under the hood, we encourage you to check out their docs to learn all its features.

Usually, templating engines transform text strings into new text strings, however, BotSON templating engine is able to expland strings into more complex structures like arrays or objects.

## Filters


```json
{
  "context": {
    "quote": "Ceci n'est pas une pipe. Les Deux Mystères."
  },
  "output": [
    "{{quote | slugify}} == ceci-nest-pas-une-pipe-les-deux-mysteres",
    "{{quote | type}} == <class 'str'>",
    "{{quote | escape_quotes}} == Ceci n\\'est pas une pipe. Les Deux Mystères.",
    "{{quote | ascii}} == Ceci n'est pas une pipe."
  ]
}
```

Filters are used to modify the variables inside your template and are expressed with a pipe symbol (|). Multiple filters can be chained, the output of one filter is applied to the next. You can use all the builtin filters supported by Jinja like `join`, `length` or `random`. Checkout the full list [here](http://jinja.pocoo.org/docs/2.10/templates/#builtin-filters).

Additionally, BotSON defines the following custom filters:

| Filter |  |
| --------- | ------------- |
| slugify | Returns the slug of any string  |
| type | Returns the type of a variable |
| escape_quotes | Escapes single quotes |
| ascii | Returns the closest ascii codification of any unicode string |

## Dates

```json
{
    "context": {
        "a1": "{{date()}}",
        "a2": "{{date('tomorrow')}}",
        "a3": "{{date('1984/05/05')}}",
        "a4": "{{date('Europe/Paris')}}"
    },
    "output": [
        "{{date(a1) - date() < date_interval(minutes=1)}} == True",
        "{{date(a2).start_of('week') < date()}} == True",
        "{{date(a3).year}} == 1984",
        "{{date(_input).to_datetime_string()}} == 2000-08-25 12:00:00"
    ]
}
```

It's possible to get the current time and operate with dates and time intervals using the functions `date` and `date_interval`. Under the hood we use the [Pendulum](https://pendulum.eustace.io/) library, we encourage you to check out their documentation to learn all the options available.

#Error types

There may be errors during the parsing and execution of a bot. They are of different types inheriting from 
`BotException`, and are mainly classified as JSON or BotSON related.

| BotException |  |
| --------- | ------------- |
| JSONParseException | Raises when the JSON is invalid |
| BotSONException | Raises otherwise |

##JSONParseException

| Attributes | Contains |
| --------- | ------------- |
| message | Reason why the json is invalid |
| line | Line of the problem in the json |
| column | Column of the problem in the json |

##BotSONException

| Types |  |
| --------- | ------------- |
| BotSONParseException | Raises during parsing |
| BotSONExecException | Raises during execution |


| Attributes | Contains |
| --------- | ------------- |
| message | Explanation of the error |
| trace | Approximate path to the error in the json. _Note: in states and trigger arrays it will use label/match value instead of index number_  |

#Jobs

TODO

#AI integration

You are able to use watson or api.ai bots to manage the botSON flow.

##Configuration

```json
{
  "ai_backends": {
    "watson": {
      "username": "<WATSON_USERNAME>",
      "password": "<WATSON_PASSWORD>",
      "workspace_id": "<WATSON_WORKSPACE_ID>"
    },
    "api_ai": {
      "token": "<API.AI_TOKEN>"
    }
  }
}
```

> Get the user intent using AI and fetch an API. Here we assume our AI backend has a couple of intents defined: "search" and "buy", and 1 entity: "item".

##AI calls

```json
[
    {
        "label": "search_menu",
        "output": "What do you want to do?",
        "input": {
            "type": "intent",
            "variable": "ai_result"
        },
        "next_step": "{{ai_result.intent}}"
    },
    {
        "label": "search",
        "context": {
            "search_result": {
                "url": "http://my-api.my-company.com/search",
                "params": { "query": "{{ai_result.item}}" }
            }
        },
        "output": "I've searched for {{ai_result.item}} and found {{search_result | length}} results.",
        "next_step": "exit"
    },
    {
        "label": "buy",
        "output": "Are you sure you want to buy {{ai_result.item}}?",
        "next_step": "exit"
    }
]
```

> Manually invoke the AI backend with an arbitrary input

```json
{
  "context": {
    "ai_result": "{{ai_conversation(ai_backends, 'hello world')}}"
  }
}
```

You are able to use watson or api.ai bots to manage the botSON flow. 

To start just configure the credentials

Then you can call the ai in two different ways.
<br><br>The object return has `intent`, `entities`. The object saved in the variable will have the following fields:

| Field     | Value           |   |
| --------- |:-------------:| -----:|
| intent      | String | |
| entities    | String | |
| output_text | String | Verbose response of the AI in that intent|

<!-- | raw         | Object | *Depending on your AI the object <br>could have different structures* |
-->
