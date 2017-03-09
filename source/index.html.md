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
  "states": [
    {
      "label": "initial",
      "input": {
        "variable": "first_text_from_user",
        "action": "free_text"
      },
      "output": "Hello World!",
      "next_step": "exit"
    }
  ]
}
```

botSON is a JSON based programming language to create chatbots for the [Hubtype](https://hubtype.com) platform. The goal of botSON is to be a simple yet powerful tool to build conversational flows that the user navigates by feeding in text or media. In a simplified way, chatbots are defined as finite state machines, at each state the chatbot outputs some text and waits for the user input that makes the bot transition to another state. A [context](#context) is kept for every bot session (the extent of a user's conversation) which can be updated at every step with variables and external http calls.

Additionally, botSON supports [triggers](#triggers), which are high priority actions that the user can invoke regardless of the current state of the conversation.

botSON is messenger agnostic, meaning that it's not designed specifically for a single messaging app but it works with any messaging channel supported by Hubtype, however not all interactive elements are supported by all channels (like [carrousels](#carrousel)).

#Getting Started

##Creating a state

```json
{
  "states": [
    {
      "label": "initial",
      "input": {
        "variable": "first_text_from_user",
        "action": "free_text"
      },
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

A bot is composed by states. Each state has a `label`, a `next_step` and usually an `output` or `input` (or both). When the bot enters a state, the first thing it does is to send the `output` (if there's one) to the user, then awaits for the user input if there's an `input` section. Finally it jumps to the state defined in the expression `next_step`. The state 'exit' indicates the end of the session, if the user talks again then a new session will start from the 'initial' state.

There's only one mandatory state: 'initial', which indicates the beginning of the conversation flow. The initial state must have an `input` section that captures the first interaction from user. The 'initial' state is different from the other states as the `input` statement is evaluated before the `output`, while the usual execution is `output` first, then `input`.

When the bot enters a state with no `input`, it just sends the `output` (if any) and jumps to `next_state` immediatelly. It is possible to chain several state jumps without any `input`, the bot will just send all the outputs consecutively until it reaches a state with an `input` statement, then it will pause until the user sends a new message.

##Basic bot

```json
{
    "input_retry": 3,
    "states": [
        {
            "label": "initial",
            "input": {
                "variable": "first_text_from_user",
                "action": "free_text"
            },
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
                "action": "get_in_set",
                "action_parameters": ["RED", "BLUE", "GREEN"],
                "variable": "user_choice"
            },
            "next_step": "result"
        },
        {
            "label": "result",
            "output": "You're choice was {{user_choice}}",
            "next_step": "choice"
        },
        {
            "label": "input_failure",
            "output": "You have put an unexpected input 3 times!",
            "next_step": "choice"
        }
    ]
}
```

A complete bot usually never ends, it loops back to previous states. Also, it can use variables and other forms of `input` and `output` other than simple text.

In this case, after the first text of the user, this bot will show three quick replies: 'Red', 'Blue' and 'Green'. The user can click them or write itself the response. 
 
Then, the bot expects an input string that can be 'RED', 'BLUE' or 'GREEN', store it in the variable `user_choice` and go to the next state `result`.

* If the user wrote anything else, the chatbot would reply with the possible options in a message like "Please, choose one of the following values: RED, BLUE" at most `input_retry` times. If it fails all 3 times it will go to the state `input_failure`.

Finally, in the state `result` the bot sends a text message saying what was the pick of the user and returns to the `choice` state.

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

Specifies the number of input failures before the machine goes to the `input_failure` state. By default is set to 3.
An input failure is given when the [type of input](#input-actions) doesn't match the required data. 

##name

The name of the bot.

##language

Indicates the language the bot will use. Currently this field is not used.

<aside class="softwarn">
Currently this field is not used.
</aside>

##defaults

Defines the default values that are used throughout the chatbot flow in different states:

* **requests**

  Currently only supports default `headers` to be used in every url request.

* **context**

  Variables that will be always available at any state of the bot. They are recalculed every time there is an input, 
  so, if a bot definition is changed, previous conversations will have updated variables.

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

Triggers are matched with regular expressions. If the regexp contains named groups, those will be added as variables
with the corresponding matched value. When a user input is captured by a trigger, it is not consumed by the current
state.

Triggers must have valid `next_step` and `match` attributes. If `next_step` is `null` the bot will consume that input 
without any change, which is useful if you want to ignore some words. See [next_step](#next_step) reference for more 
info.

Usually triggers are used to capture button clicks (what Facebook calls 'Postbacks') and general commands like "help".

##states

This field contains an array of all the states of the bot.

When the bot enters a state, the first thing it does is to update the context, then it writes all the outputs
(if any) and finally wait for a user input (if defined).

```json
{
  "states": [
    {
      "label": "initial",
      "input": {
        "action": "free-text",
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

It is required a state named `initial`, that is the first state in the execution flow. `initial` will only start 
after the user sends it's first input. Then, it will update the context, write the outputs and go to `next_step`. It 
is recomended that no state go to `initial`.

###label

The state name.

###context 

*(Optional)*

Variables to be assigned when entering the state. See context.

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

#Context

> The context is just a set of variables that can contain any JSON type

```json

{
  "user": {
    "name": "John",
    "last_name": "Doe"
  },
  "is_customer": true
}
```

The context is a set of variables that is carried along during the whole session. Everytime the bot transitions to a new state, the context is updated with all the variables defined in the new state `context` statement.

#Output types

##Text

```json
{
  "type": "text",
  "data": "Done!"
}
```
| Field     | Value           |   |
| --------- |:-------------:| -----:|
| type      | "text" | |
| data      | String      | *ASCII and emojis*|

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
| type      | "web_url", "postback" or "phone_number" | |
| title  | String  |   |
| webview_height_ratio | String | *Possible in web_url* |
| messenger_extensions | String | *Possible in web_url* |
| fallback_url         | String | *Possible in web_url* | 
| url  | String  | *mandatory if type is "web_url"*  |
| payload  | String  | *mandatory if type is "postback" or "phone_number"* |

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

#Input actions

`"Action"` specifies what is the expected action and `"data"` will be stored in the specified variable. To access the 
variable see [Templating](#templating).

##Free Text

```json
{
  "action": "free_text",
  "variable": "text_var_name"
}
```

Accepts any text from the user and stores it in the variable.

##Get Integer

```json
{
  "action": "get_int",
  "variable": "name_of_int_var"
}
```

Accepts any integer as input , other kinds of inputs are rejected.

##Get in Set

```json
{
  "action": "get_in_set",
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
  "action": "get_in_set_fuzzy",
  "variable": "object_user_choice",
  "action_parameters": ["object1", "object2"]
}
```

Same as get in set, but it does not require a perfect match.

| Field     | Value           |
| --------- |:-------------:|
| action_parameters | Array of strings |

##Get yes or no

```json
{
  "action": "get_yes_no",
  "variable": "confirm_var_name"
}
```

The accepted outputs are `yes`, `si`, `sí` and `no` in lowercase or uppercase.

##Get from url

```json
{
  "action": "get_from_url",
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
  "action":"get_name",
  "variables":"confirm_get_name"
}
```

The user input should be a text with at most 3 words.

##Get email

```json
{
  "action": "get_email",
  "variable": "user_email"
}
```

Gets the email and check if it is well writed. **something@someother**

##Get age

```json
{
  "action":"get_age",
  "variable":"age"
}
```

Saves a number representing the age. Age should match 1 <= age < 120.

##Get location

```json
{
  "action":"get_location",
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
  "action": "get_image",
  "variable": "the_image_url"
}
```
It will save in the variable the url given by the provider.

<aside class="softwarn">
NOTE: The get image input is only available on Facebook. Currently other platforms will ignore that output.
</aside>

#Templating

See [Jinja](http://jinja.pocoo.org/).

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

#AI integration
You are able to use watson or api.ai bots to manage the botSON flow. 
```json
{
  "ai_backends":{
    "watson": {
      "username": "(id)",
      "password": "(pass)",
      "workspace_id": "(w_id)"
    },
    "api_ai":{
    "token":"(token)"
  }
}
}
```
To start just configure the credentials

```json
{
  "input":{
    "action":"get_intent",
    "variable":"variable_name"
  }
}
```
```json
{
  "context": {
    "ai_result_name_var": "ai_conversation(ai_backends, _input)"
  }
}
```
Then you can call the ai in two different ways.
<br><br>The object return has `intent`, `entities` and `raw`. Where raw is 
the full API response. The object saved in the variable will have the following structure.

| Field     | Value           |   |
| --------- |:-------------:| -----:|
| intent      | String | |
| entities    | String | |

<!-- | raw         | Object | *Depending on your AI the object <br>could have different structures* |
-->