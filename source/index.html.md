---
title: botSON Reference

toc_footers:
  - <a href='https://hubtype.com/'>Hubtype</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

search: true
---

#Deprecation Notice

botSON is no longer maintained. Please refer to [Botonic](https://botonic.io) to learn about the latest and recommended framework for building bots for the [Hubtype](https://hubtypee.com) platform. Botonic is open-source, based on [React](https://reactjs.org/) and allows for way more features, flexibility and ease of use.

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

- Part 1: [Scaffolding and publishing](https://www.youtube.com/watch?v=h53SjVGMwos)

#Examples

##Dynamic carrousel

```
{
  "name": "Clothes Bot",
  "triggers": {},
  "version": "1.0",
  "literals": { #we store our secret api key
      "api_key": "uid8900-40385330-57"
  },
  "states": [
    {
      "label": "initial",
      "output":
      {
          "type": "text",
          "data": "Hey, what clothes are you interested in?",
          "keyboard": [
            {"label": "Mens shirt", "data": "mens-shirts"},
            {"label": "Womens shirt", "data": "womens-tops"}
          ]
      },
      "input": {
          "variable": "user_response",
          "action": "get_in_keyboard"
      },
      "next_step": "create_carrousel"
    },
    {
      "label": "create_carrousel",
      "context":[
          {"items": { #we get all the items id's related with the user_response
                  "url": "http://api.shopstyle.com/api/v2/products?pid={{api_key}}&fts={{user_response.data}}&offset=0&limit=5",
                  "method": "GET",
                  "params": {
                  }
              }
          },
          {"full_items": "[
              {# for every item, we store all the information in 'full_items' #}
              {% for item in items.products %}
                  {{http('http://api.shopstyle.com/api/v2/products/' +
                      item.id|string +
                      '?pid=' +
                      api_key|string
                  )}},
              {% endfor %}
          ]"
          }
      ],
      "output": [
          {
              "type": "carrousel", #for every item, we display his information in a carrousel
              "elements": "[
                  {% for item in full_items %}
                      {
                          'title': '{{item.name}}',
                          'subtitle': '{{item.priceLabel}}',
                          'image_url': '{{item.image.sizes.Best.url}}',
                          'buttons':[
                              {
                                  'type': 'web_url',
                                  'title':'Open product',
                                  'url': '{{item.clickUrl}}'
                              }
                          ]
                      },
                  {% endfor %}
              ]"
          }
      ],
      "next_step": "exit"
    }
  ]
}
```

This is a simple bot, that displays five men or woman shirts and its price, and we are getting the clothes information from the Shopstyle API.

First of all, we ask what are the user interests. When we get it, we generate the [carrousel](#carrousel) of the products.

When we want to make REST API request, we create the `context`, and we put all the logic in there.
In order to get all the information, we store all the items making a request with the user prefence and our API_KEY.

The request goes against a specific `url`, with his specific `method`, and if necessary with the `params`.

For getting all the information of every product, we need to make a call to the API for all the shirt ids. The id of a product, it's in JSON format, and we can do `item.id` or `item['id']` to access it.

When we have all the product information, we can display the [carrousel](#carrousel).

The [carrousel](#carrousel) can have at most ten elements, where each element will represent one product and will have his name, the price, an image and a link to the product page.

For access to this values, we do `item.name` , `item.priceLabel`, etc.

<p style="text-align:center">
  <img src="/videos/dynamic_carrousel.gif" align="middle" height="600px" width="400px"/>
</p>

##Location Bot

```
{
  "name": "Location bot",
  "input_retry": 3,
  "triggers": {},
  "version": "1.0",
  "states": [
      {
        "label": "initial",
        "output":
        {
            "type": "text",
            "data": "Hey, where I have to pick you up?",
            "keyboard": [
                {
                    "location": ""
                }
            ]
        },
        "input":{
            "action":"get_location",
            "variable":"location"
        },
        "next_step": "localizacion_obtenida"
      },
      {
        "label": "localizacion_obtenida",
        "output": [
          {
            "type": "text",
            "data": "Okai, I will pick up right now: \nLocation = long: {{location.latitude}} , lat: {{location.longitude}}"
          }
        ],
        "next_step": "initial"
      }
  ]
}

```

Being able to manage location is a powerfull tool nowadays.

This bot ask for your location, and with a simple click, the user can send his location to the bot.
In the state `initial`, we ask to the user where is he, and with the option `location` in the `keyboard`, we display a `keyboard` that will get the user location.

Then, in the `input`, we store the user location in the variable `location`.

<p style="text-align:center">
  <img src="/videos/bot_location.gif" height="600px" width="400px"/>
</p>

##Handover Bot

```
{
  "name": "Handover bot",
  "input_retry": 3,
  "triggers": {
      "text": [
          {
              "match": "^reset$",
              "next_step": "initial"
          }
      ]
  },
  "version": "1.0",
  "states": [
    {
      "label": "initial",
      "output":
      {
          "type": "text",
          "data": "Hey, how can I help you?"
      },
      "input":{
          "type":"text",
          "variable":"response"
      },
      "next_step": "transfer_case"
    },
    {
      "label": "transfer_case",
      "context": { #we create a new case in the queue q1.
          "case": {
            "action": "create_case",
            "queue": "q1"
          }
      },
      "output": {
          "type": "text",
          "data": "An agent will immediately take care of your case"
      },
      "input": {
          "variable": "trash",
          "action": "free_text"
      },
      "next_step": "exit"
    }
  ]
}
```

As bots aren't perfects, we can transfer a case to an agent in an easy way, let's have a look how this bot works.

This bot don't understand anything, so any input that we enter it will be redirected to an agent.
In order to do that, when we reach the state where we want to transfer the case, we declare in the `context` an object called `case`. This object will have two values, the `action` that must be 'create_case' and `queue` that it's the name or id of the queue where we will generate the case.

When the case it's resolved by the agent, the bot will continue to the `next_step` of the state.

<p style="text-align:center">
  <img src="/videos/bot_handover.gif" height="300px" width="600px"/>
</p>

##Failure Bot

```
{
  "name": "Failure bot",
  "input_retry": 3,
  "max_loop": 10,
  "triggers": {},
  "version": "1.0",
  "states": [
      {
            "label": "initial",
            "output": {
                "type": "text",
                "data": "Here you have to choose:",
                "keyboard": [
                    {"label": "State not defined", "data": "not_defined"},
                    {"label": "HTTP_REQUEST", "data": "http_request"},
                    {"label": "LOOP", "data": "loopA"}
                ]
            },
            "input": {
                "action": "get_in_keyboard",
                "variable": "user_choice"
            },
            "next_step": "{{user_choice.data}}" #next step depending on the user response
        },
        {
            "label": "http_request",
            "context":{ #we make a request to an invented link that doesn't exist.
                "image_test": {
                    "url": "www.randomurl.com/randomimage.jpg",
                    "method": "GET",
                    "params": {}
                }
            },
            "output": {
                "type": "image",
                "data": "{{url}}"
            },
            "next_step": "exit"
        },
        {
            "label": "loopA",
            "output": "A",
            "next_step": "loopB"
        },
        {
            "label": "loopB",
            "output": "B",
            "next_step": "loopA" #creates a bucle: loopA --> loopB --> loopA
        },
        {
            "label": "input_failure",
            "output": "You have failed answering the bot",
            "next_step": "exit"
        },
        {
            "label": "fallback_instruction",
            "output": "The bot don't recognize this state",
            "next_step": "exit"
        },
        {
            "label": "external_request_failure",
            "output": "There were an error with some http request",
            "next_step": "exit"
        },
        {
            "label": "loop_overflow",
            "output": "The bot enter to a loop",
            "next_step": "exit"
        }
  ]
}

```

In this bot, we can find all the possible states where the bot can be redirected when it gets lost or experience some error.

In the first demo, we see how the user enters 3 times some input that the bot isn't expecting, and it will display the same question as many times as you have declares in the `input_retry`. If the bot reaches the number of failures as declared, it will go into the `input_failure` state.

In the other demo, we can find the other three types of failure states.
The first one, the `fallback_instruction`, it's reached when the bot tries to enter into the state `not_defined`, and doesn't find it.

The second case is the `external_request_failure`, in this state we make a get request, and as we don't get any response, the bot goes into `external_request_failure`.

And the last one is reached when the user enters a loop. We are jumping from state `loopA` to `loopB` and vice versa. When it hits 10 times (defined at `max_loop`), the bot jumps to the state `loop_overflow`.

<p style="text-align:center">
  <img src="/videos/bot_input_failure.gif" height="600px" width="400px"/>
  <br><br>
  <img src="/videos/bot_failures.gif" height="600px" width="400px"/>
</p>

##Dynamic next state

```
{
    "name": "Next State Bot",
    "input_retry": 1,
    "triggers": {},
    "version": "1.0",
    "states": [
        {
            "label": "initial",
            "output": [
                {
                    "type": "text",
                    "data": "What do you prefer, rigth or left?",
                    "keyboard": [
                        {"label": "Right", "data": "right"},
                        {"label": "Left", "data": "left"}
                    ]
                }
            ],
            "input": {
                "type": "in_keyboard",
                "variable": "user_choice"
            },
            "next_step": [
                "{% if user_choice.data == 'right' %}",
                    "right_state",
                "{% elif user_choice.data == 'left' %}",
                    "left_state",
                "{% else %}",
                    "exit",
                "{% endif %}"
            ]
        },
        {
            "label": "right_state",
            "output": {
                "type": "text",
                "data": "Let's go right"
            },
            "next_step": "exit"
        },
        {
            "label": "left_state",
            "output": {
                "type": "text",
                "data": "Let's go left"
            },
            "next_step": "exit"
        }
    ]
}

```

When the `next_state` depends on the user response option, we can make a dynamic `next_state` definition.

In this bot, we can see how we store the user decision in `user_choice`. Depending on what the user has chosen, the bot will continue to one state or another.

If the user decided 'right', the bot will go to `right_state`, else if he decided 'left' it will got to `left_state`.
Otherwise, for avoiding possible errors, we say that if the data it's not right or left, it will go to `exit`.

<p style="text-align:center">
  <img src="/videos/bot_next_state.gif" height="600px" width="400px"/>
</p>

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
          { "label": "Red", "data": "RED" },
          { "label": "Blue", "data": "BLUE" },
          { "label": "Green", "data": "GREEN" }
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
      "output": "Your choice was {{user_choice.data}}",
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

<aside class="softwarn">
Currently this field is not used.
</aside>

##initial_state

Label of the first state of the bot. Defaults to "initial".

##defaults

Defines the default values that are used throughout the chatbot flow in different states:

###requests

Currently only supports default `headers` to be used in every url request.

###delay:

Time (in seconds) to delay sending any outputs. It can be a decimal number. This parameter can be overridden in each output. Defaults to 0.

###typing

Time (in seconds) that the bot will display the "typing..." effect before sending this output. It can be a decimal number. If the typing effect is not supported by the messenger it will result in extra delay time. This parameter can be overridden in each output. Defaults to 0.

###context

Variables that will be always available at any state of the bot. They are recalculated every time there is an input, so, if a bot definition is changed, previous conversations will have updated variables.

##literals

> Simple literals

```json
{
  "literals": {
    "text1": "Hello!",
    "text2": "Goodbye my friend!"
  },
  "states": [
    {
      "label": "initial",
      "output": "{{text1}}",
      "next_step": "exit"
    }
  ]
}
```

> Literals for multiple languages

```json
{
  "literals": {
    "text1": {
      "en": ["Hello!", "Hey there!", "Alohaaa"],
      "es": ["Hola!", "Hey que tal!", "Buenaaas"]
    },
    "text2": {
      "en": ["Goodbye my friend!", "bye byee"],
      "es": ["Adiós amigo!", "Hasta luegoo"]
    }
  },
  "defaults": {
    "context": {
      "lang": "en"
    }
  },
  "states": [
    {
      "label": "initial",
      "output": "{{text1[lang] | random}}",
      "next_step": "exit"
    }
  ]
}
```

When your bot grows you'll likely want to separate the content of your outputs from the logic of the states. One thing you could do is placing all your texts in a variable in the default context and then use `{{my_copys[1]}}` to dinamically build your output. While this is perfectly fine for small bots, this is generally a bad idea since the default context is evaluated every time the bot jumps from one state to another, so if you have hundreds or thousands of copys it will slow down your bot. Instead, `literals` is a static section where you can put as many variables as you want without incurring into a performance hit.

The way you organize your copys is entirely up to you, but we found that following a convention makes it easy to find them and to keep the code consistent even for really big chatbots. The example "Literals for multiple languages" on the right is the convention we recommend following.

##session_timeout

Time (in minutes) after the last user interaction to wait before the session is closed automatically. The default is `null` which means the session will never be closed automatically, only if a state with `"next_step": "exit"` is reached.

##keep_session

A boolean that indicates whether the variables from the context of the previous session should be copied to the new session.
Defaults to `false`.

##keep_trace

A boolean that indicates whether the `_trace` special variable should be kept between different sessions.
Defaults to `false`.

##early_typing

A boolean that indicates whether the bot should send a typing indicator as soon as it gets an input from the user. This is usually desirable, so that user gets instant feedback that his message is being processed, however if the bot does not generate an output it results in a weird effect where the typing indicator starts and then ends without any message being sent.

If set to `false`, the bot will delay sending the typing indicator until it knows for sure there's some message to send back. This effectively removes the weird effect, but it will delay the typing feedback for some milliseconds.

Defaults to `true`.

##triggers

```json
{
  "triggers": {
    "payload": [
      {
        "match": "^WATCH_VIDEO_(?P<video_id>[a-z0-9-]+)$",
        "context": {
          "video_url": "videos/{{video_id}}.mp4"
        },
        "next_step": "watch_video"
      }
    ],
    "text": [
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

- `text` triggers can be fired by user inputs or payloads

- `payload` triggers can not be fired by user messages. _(Higher priority than text triggers.)_

Triggers are matched with regular expressions. If the regexp contains named groups, those will be added as variables to the context with the corresponding matched value. When a user input is captured by a trigger, it is not consumed by the current state.

Triggers must have valid `next_step` and `match` attributes. If `next_step` is `null` the bot will consume that input without any change, which is useful if you want to ignore some words. See [next_step](#next_step) reference for more info.

Usually triggers are used to capture button clicks (what Facebook calls 'Postbacks') and general commands like "help".

##states

This field contains an array of all the states of the bot.

When the bot enters a state, the first thing it does is to update the context, then it writes all the outputs (if any) and finally wait for a user input (if defined). Finally, it jumps to the state defined in the `next_step` expression.

```json
{
  "version": "1.0",
  "initial_state": "first_state",
  "states": [
    {
      "label": "first_state",
      "input": {
        "type": "free_text",
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

_(Optional)_

Variables to be assigned when entering the state. See [context](#context).

###output

_(Optional)_

The bot will send in order all outputs defined here. It has different possible [data types](#output-types).

###input

_(Optional)_

Input of the bot. It can come from the user or as a response to url calls. The field "actions" specifies the expected
[type of input](#input-actions).

###next_step

Name of the next state in the flow of the bot. For a conditional next_step (i.e. jumping to different states
depending of variables) see [templating](#templating).

### Implicit states

There are some implicit states that are always defined under the hood. You can always redefine these states in your code in order to customize their default behaviour.

- `input_failure`: The chatbot jumps to this state when the user fails to give an appropiate answer after N attempts. This can be controlled using the global setting `input_retry`.

- `external_request_failure`: The chatbot jumps to this state when it fails to make an http request. This can be triggered for a variety of reasons, for example, if the request took too long or the response code was ouside the 2XX range.

- `fallback_instruction`: The chatbot jumps to this state when it encounters a `next_step` statement that points to a label that doesn't exist. This can happen when you define a dynamic `next_step` using some templating expression like `my_state_{{option}}` and the variable `option` is not defined.

- `loop_overflow`: The chatbot jumps to this state when it's been jumping from state to state N times without finding any "input" statement which makes the bot to pause and wait for user input. This is needed in order to prevent infinite loops where state A has a next_step to state B and vice versa. You can set the max number of iterations using the global setting `max_loop`, but it can't be higher than 50.

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
    { "first_var": 1 },
    { "second_var": "This depends on the {{first_var}}" }
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

> In this example, the variable 'queues' will contain an array of names: the queues that are open in Hubtype DESK.

```json
{
  "context": {
    "queues": {
      "action": "active_queues"
    }
  }
}
```

> Here we create a note so the human agent that is going to attend the conversation can quickly see a summary of the customer.

```json
{
  "context": {
    "note": {
      "action": "add_note",
      "text": "User name: {{user.name}}. Age: {{user.age}}. Order number: {{order_num}}."
    }
  }
}
```

> Finally, we create a case in one of the available queues or in the default one if there are none available.

```json
{
  "context": {
    "case": {
      "action": "create_case",
      "queue": "{{available_queues[0] if available_queues | length > 0 else 'Default queue'}}"
    }
  }
}
```

Context actions are used to interact with the outside world. Probably the most handy action is `http`, which allows you to make calls to any standard JSON API. Here's a complete list of available actions:

- **http**: Makes a starndard http request to a url and retrieves the response in json. The expected parameters are:

| Action  |                     Type                     |                                                        |
| ------- | :------------------------------------------: | -----------------------------------------------------: |
| url     |                    String                    |                                                        |
| method  | String (GET, POST, PUT, PATCH, DELETE, HEAD) |                              Optional. Defaults to GET |
| headers |                    Object                    |                                               Optional |
| params  |                    Object                    |                Optional. Query parameters (GET params) |
| data    |                    Object                    |     Optional. Form data to send along with the request |
| json    |                    Object                    | Optional. JSON data to send in the body of the request |

- **active_queues**: Returns the IDs of the queues in status "Open" from your organization at Hubtype DESK. Useful when your chatbot has qualified a lead and has to handover the conversation to a human agent.

- **add_note**: It creates a note in the conversation. You can use this to make a summary of the user details before a human agent handover.

| Action |  Type  |                     |
| ------ | :----: | ------------------: |
| text   | String | Content of the note |

- **create_case**: It creates a case in Hubtype DESK so that a human agent gets over the conversation. While the case is open, the chatbot won't respond to any user input. Only when the agent resolves the case the chatbot will continue its flow.

| Action |  Type  |                                                          |
| ------ | :----: | -------------------------------------------------------: |
| queue  | String | ID or name of the queue that the case will be created on |

<aside class="softwarn">
NOTE: active_queues are the only ones that some user can be transferred
</aside>

##Special context variables

There are some special variables in the context that the bot populates automatically when a new session starts or when certain events occur:

###User

```json
{
  "user": {
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
  "_trace": ["initial", "welcome_message", "show_products", "buy_item"]
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
  "last_session": {
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
    { "label": "Yeah", "data": "yes" },
    { "label": "Nope", "data": "no" }
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

| Field  | Value  |                                                                                                                |
| ------ | :----: | -------------------------------------------------------------------------------------------------------------: |
| delay  | Number |                     Time (in seconds) to delay sending this output. It can be a decimal number. Defaults to 0. |
| typing | Number | Time (in seconds) that the bot will display the "typing..." effect. It can be a decimal number. Defaults to 0. |

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

| Field | Value  |                            |
| ----- | :----: | -------------------------: |
| type  | "text" |                            |
| data  | String | _Any UTF-8 encoded string_ |

##Image

```json
{
  "type": "image",
  "data": "http://this.is.the/url/of/the/image.jpg"
}
```

| Field |  Value  |                 |
| ----- | :-----: | --------------: |
| type  | "image" |                 |
| data  | String  | _The image URL_ |

##Video

```json
{
  "type": "video",
  "data": "http://this.is.the/url/of/the/video.mp4"
}
```

| Field |  Value  |     |
| ----- | :-----: | --: |
| type  | "video" |     |
| data  | String  |     |

##Audio

```json
{
  "type": "audio",
  "data": "http://this.is.the/url/of/the/audio.mp3",
  "caption": "Some video subtitle"
}
```

| Field   |  Value  |            |
| ------- | :-----: | ---------: |
| type    | "audio" |            |
| data    | String  |            |
| caption | String  | _optional_ |

##Document

```json
{
  "type": "document",
  "data": "http://this.is.the/url/of/the/document.pdf",
  "caption": "Some document subtitle"
}
```

| Field   |  Value  |            |
| ------- | :-----: | ---------: |
| type    | "image" |            |
| data    | String  |            |
| caption | String  | _optional_ |

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

| Field     |   Value    |                              |
| --------- | :--------: | ---------------------------: |
| type      | "location" |                              |
| latitude  |   Number   |                              |
| longitude |   Number   |                              |
| title     |   String   | _optional_<br>_32 chars max_ |
| address   |   String   |                   _optional_ |
| url       |   String   |                   _optional_ |

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

| Field        |   Value   |            |
| ------------ | :-------: | ---------: |
| type         | "contact" |            |
| first_name   |  String   |            |
| last_name    |  String   | _optional_ |
| phone_number |  Number   | _optional_ |
| vcard        |  String   | _optional_ |

##Buttonmessage

```json
{
  "type": "buttonmessage",
  "text": "How can I help you?",
  "buttons": [
    {
      "type": "postback",
      "title": "Buy pizza",
      "next_step": "buy_pizza"
    },
    {
      "type": "postback",
      "title": "See menu",
      "payload": "SHOW_MENU"
    },
    {
      "type": "web_url",
      "title": "Visit website",
      "url": "https://www.google.es/"
    },
    {
      "type": "phone_number",
      "title": "Call us",
      "payload": "{% raw %}+44 7700 900200{% endraw %}"
    }
  ]
}
```

A text message followed by a collection of at most 4 buttons displayed vertically.

###Button:

| Field                |                  Value                  |                                                          |
| -------------------- | :-------------------------------------: | -------------------------------------------------------: |
| type                 | "web_url", "postback" or "phone_number" |                                                          |
| title                |                 String                  |                                                          |
| webview_height_ratio |                 String                  |                                    _Possible in web_url_ |
| messenger_extensions |                 String                  |                                    _Possible in web_url_ |
| fallback_url         |                 String                  |                                    _Possible in web_url_ |
| url                  |                 String                  |                         _mandatory if type is "web_url"_ |
| payload              |                 String                  |      _mandatory if type is "postback" or "phone_number"_ |
| next_step            |                 String                  | _optional substitute of payload when type is "postback"_ |

<aside class="softwarn">
NOTE: The messagebutton output is only available on Facebook and Telegram. Also, phone_number button type only is available on Facebook. Currently other platforms will ignore that output.
</aside>

##Carrousel

```json
{
  "type": "carrousel",
  "elements": [
    {
      "title": "Pepperoni",
      "subtitle": "Authentic soft New York style dough topped with tasty pepperoni",
      "image_url": "https://www.cover.image.jpg",
      "buttons": [
        {
          "type": "postback",
          "title": "Add to cart",
          "payload": "ADD_CART_PEPPERONI"
        }
      ]
    },
    {
      "title": "Tandoori Chicken",
      "subtitle": "Succulent chicken, Tandoori spices, fresh tomato and red onion",
      "image_url": "https://www.cover.image.jpg",
      "buttons": [
        {
          "type": "postback",
          "title": "Add to cart",
          "payload": "ADD_CART_TANDOORI"
        }
      ]
    }
  ]
}
```

A rich interactive message that displays a horizontal scrollable carousel of items, each composed of an image, short description and buttons to request input from the user.

<aside class="softwarn">
If you want the bot to stop and wait for the user to tap on a button, then you must add an "input" statement after the "output" where the carrousel is defined, otherwise the bot will just jump to the next step.
</aside>

<img width="300" src="/images/carrousel.png">

| Field    |       Value       |                                   |
| -------- | :---------------: | --------------------------------: |
| type     |    "carrousel"    |                                   |
| elements | Array of Elements | _10 elements max (will truncate)_ |

###Element:

| Field                     |      Value       |                                  |
| ------------------------- | :--------------: | -------------------------------: |
| title                     |      String      |                                  |
| subtitle                  |      String      |                                  |
| image_url                 |      String      |                                  |
| [buttons](#buttonmessage) | Array of Buttons | _3 elements max (will truncate)_ |

<aside class="softwarn">
NOTE: The carrousel output is only available on Facebook. Currently other platforms will ignore that output.
</aside>

##List

```json
{
  "type": "list",
  "elements": [
    {
      "title": "First title",
      "subtitle": "Some subtitle",
      "image_url": "https://www.url/g/image.png",
      "buttons": [
        {
          "type": "phone_number",
          "title": "Call",
          "payload": "{% raw %}+44 7700 900200{% endraw %}"
        }
      ]
    },
    {
      "title": "Second title",
      "subtitle": "Oh my god",
      "image_url": "https://ToBeOrNot.ToBe.an.url/image.jpg",
      "buttons": [
        {
          "type": "web_url",
          "title": "Go details",
          "url": "http://www.details.url/#section"
        }
      ]
    }
  ]
}
```

| Field                  |       Value       |                                      |
| ---------------------- | :---------------: | -----------------------------------: |
| [elements](#carrousel) | Array of Elements | _min 2 elements <br> max 4 elements_ |

<aside class="softwarn">
NOTE: The carrousel output is only available on Facebook. Currently other platforms will ignore that output.
</aside>

##Media Message

```
{
  "type": "mediamessage",
  "image": "https://ToBeOrNot.ToBe.an.url/image.jpg",
  "buttons": [
      {
          "type": "web_url",
          "title":"Go to image",
          "url": "http://www.details.es"
      }
  ]
}

{
  "type": "mediamessage",
  "video": "https://randompage.es/video.mp4",
  "buttons": [
      {
          "type": "web_url",
          "title":"Go to Video",
          "url": "http://www.videourl.es"
      }
  ]
}
```

It's composed of an image or a video, and followed by a button.

<aside class="softwarn">
IMPORTANT: The Media Message only accepts an image or a video, not the both in the same output. <br>
NOTE: The Media Message output is only available on Facebook. Currently other platforms will ignore that output.
</aside>

| Field                     |      Value       |                   |
| ------------------------- | :--------------: | ----------------: |
| image                     |      String      |                   |
| video                     |      String      |                   |
| [buttons](#buttonmessage) | Array of Buttons | _max of 1 button_ |

##Keyboard (Quick Replies)

```json
{
  "label": "select_restaurant_type",
  "output": {
    "type": "text",
    "data": "What's your budget for dinner?",
    "keyboard": [
      {
        "label": "$10 or less",
        "data": "cheap"
      },
      {
        "label": "$15 - $25",
        "data": "medium"
      },
      {
        "label": "$25 or more",
        "data": "expensive"
      }
    ]
  },
  "input": {
    "type": "in_keyboard",
    "variable": "budget"
  },
  "next_step": "show_{{budget.data}}"
}
```

> Location keyboard

```json
{
  "label": "ask_location",
  "output": {
    "data": "Please, share your location with me",
    "keyboard": [
      {
        "location": ""
      }
    ]
  },
  "input": {
    "type": "location",
    "variable": "location"
  },
  "next_step": "show_nearby_shops"
}
```

Quick Replies are a convenient way to let users know the available options to type. Also, they save the end-user the effort to type all words and let them navigate through options much faster.

Any message type accepts a `"keyboard"` field that typically consists of an array of (`"label"`, `"data"`) pairs. There's a special type of keyboard that allows you to request the user for a location. It will display a special button that, when tapped, will prompt a maps view so that the user can share her location.

<aside class="notice">
The "location" keyboard only works on Facebook Messenger.
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
  "timestamp": "1515671650",
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

| Field          |          Value          |                                                       |
| -------------- | :---------------------: | ----------------------------------------------------: |
| recipient_name |         String          |                                                       |
| merchant_name  |         String          |                                            _Optional_ |
| order_number   |         String          |                                                       |
| currency       |         String          |                                                       |
| payment_method |         String          |                                                       |
| timestamp      |         String          |                                            _Optional_ |
| order_url      |         String          |                                            _Optional_ |
| elements       | Array of ReceiptElement | _Optional<br>Facebook doesn't<br>guarantee the order_ |
| address        |         Address         |                                            _Optional_ |
| summary        |         Summary         |                                                       |
| adjustments    |   Array of Adjustment   |                                            _Optional_ |

###ReceiptElement
| Field | Value | |
| -------- |:------:| -----:|
| title | String ||
| subtitle | String | _Optional_ |
| quantity | Number | _Optional_ |
| price | Number | _Can be 0_ |
| currency | String | _Optional_ |
| image_url | Link | _Optional_ |

###Address
| Field | Value | |
| ------- |:------:| -----:|
| street1 | String ||
| street2 | String | _Optional_ |
| city | String ||
| postal_code | String ||
| state | String ||
| country | String ||

###Summary
| Field | Value | |
| ------------- |:------:| ----------:|
| subtotal | Number | _Optional_ |
| shipping_cost | Number | _Optional_ |
| total_tax | Number | _Optional_ |
| total_cost | Number ||

###Adjustment
| Field | Value | |
| ------ |:------:| -----:|
| name | String ||
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

| Field             |      Value       |
| ----------------- | :--------------: |
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

| Field             |      Value       |
| ----------------- | :--------------: |
| action_parameters | Array of strings |

##Get in keyboard

```json
{
  "label": "in_keyboard_example",
  "output": {
    "type": "text",
    "data": "Select one option:",
    "keyboard": [
      { "label": "Item 1", "data": "1" },
      { "label": "Item 2", "data": "2" }
    ]
  },
  "input": {
    "type": "in_keyboard",
    "variable": "key_user_choice"
  },
  "next_step": "another_state"
}
```

Puts in the variable the key object (_label, data_) of the key pressed by the user. There must be a
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

| Field  |                Value                 |
| ------ | :----------------------------------: |
| url    |               Get url                |
| params | Array of parameters for the get call |

##Get name

```json
{
  "type": "name",
  "variables": "confirm_get_name"
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
  "type": "age",
  "variable": "age"
}
```

Saves a number representing the age. Age should match 1 <= age < 120.

##Get location

```json
{
  "type": "location",
  "variable": "location"
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

| Filter        |                                                              |
| ------------- | ------------------------------------------------------------ |
| slugify       | Returns the slug of any string                               |
| type          | Returns the type of a variable                               |
| escape_quotes | Escapes single quotes                                        |
| ascii         | Returns the closest ascii codification of any unicode string |

## Dates

```json
{
  "context": {
    "a1": "{{date()}}",
    "a2": "{{date('tomorrow')}}",
    "a3": "{{date('1984/05/05')}}",
    "a4": "{{date('now', 'Europe/Paris')}}"
  },
  "output": [
    "{{date(a1) - date() < date_interval(minutes=1)}} == True",
    "{{date(a2).start_of('week') < date()}} == True",
    "{{date(a3).year}} == 1984",
    "{{date(a4).to_datetime_string()}} == current time in Paris"
  ]
}
```

It's possible to get the current time and operate with dates and time intervals using the functions `date` and `date_interval`. Under the hood we use the [Pendulum](https://pendulum.eustace.io/) library, we encourage you to check out their documentation to learn all the options available.

## Comments

```json
{
    "type": "carrousel", #This is a botSON comment
    "elements": "[
        {% for item in full_items %}
            {
                {# This is a JINJA comment #}
                'title': '{{item.name}}',
                'subtitle': '{{item.priceLabel}}',
                'image_url': '{{item.image.sizes.Best.url}}',
                'buttons':[
                    {
                        'type': 'web_url',
                        'title':'Open product',
                        'url': '{{item.clickUrl}}'
                    }
                ]
            },
        {% endfor %}
    ]"
}
```

We allow two types of comments. You can make a simple comment in the JSON code with a `#` followed by your comment, or if you want to put a comment inside a jinja template, it must be done with this syntaxis: `{# your comment #}`

#Error types

There may be errors during the parsing and execution of a bot. They are of different types inheriting from
`BotException`, and are mainly classified as JSON or BotSON related.

| BotException       |                                 |
| ------------------ | ------------------------------- |
| JSONParseException | Raises when the JSON is invalid |
| BotSONException    | Raises otherwise                |

##JSONParseException

| Attributes | Contains                          |
| ---------- | --------------------------------- |
| message    | Reason why the json is invalid    |
| line       | Line of the problem in the json   |
| column     | Column of the problem in the json |

##BotSONException

| Types                |                         |
| -------------------- | ----------------------- |
| BotSONParseException | Raises during parsing   |
| BotSONExecException  | Raises during execution |

| Attributes | Contains                                                                                                                              |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| message    | Explanation of the error                                                                                                              |
| trace      | Approximate path to the error in the json. _Note: in states and trigger arrays it will use label/match value instead of index number_ |

#Jobs

TODO

#Integrations

##AI/NLP

The `ai_backends` attribute allows you to easily integrate 3rd party services like Google's Dialogflow (former API.ai) or IBM Watson to add natural language understanding capabilities to your bot. Usually you'll want to pass the input from the user to the AI/NLP service and get an intent back, then use that intent to navigate to a particular state of your bot.

You must create an attribute named after the service you want to integrate, this attribute should contain an object with the credentials needed to talk to that service.

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

###AI calls

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

| Field       | Value  |                                           |
| ----------- | :----: | ----------------------------------------: |
| intent      | String |                                           |
| entities    | String |                                           |
| output_text | String | Verbose response of the AI in that intent |
| raw         | Object |          Raw response from the AI backend |

Available integrations:

| Service    | Availability | Credentials                        |
| ---------- | :----------: | ---------------------------------- |
| Dialogflow |  Available   | {token}                            |
| Watson     |  Available   | {username, password, workspace_id} |
| Wit.ai     | Coming Soon  |

##Analytics

```json
{
    "defaults": {
        "context": {
            "analytics": {
                "data": {
                    "my_id": "123" # This will be sent with all default events
                },
                "splunk": {
                    "url": "SPLUNK_INSTANCE_URL",
                    "token": "AUTH_TOKEN"
                }
            }
        }
    },
    "states": [
        ...
        {
            "label": "buy_item",
            "context": [
                {"_": "{{track(analytics, {'event': 'buy_item'})}}"}
            ],
            "output": "You have just bought an item",
            "next_step": "exit"
        }
    ]
}
```

The analytics attribute allows you to easily integrate 3rd party services like Mixpanel or Google Analytics to track events of your bot. You can use this to measure the activity of your bot's users and create different reports like retention or funnels.

You must create an attribute named after each of the services you want to integrate, this attribute should contain an object with the credentials needed to talk to that service. Then BotSON will automatically track several default events:

| Event        |          Data          |                   Description |
| ------------ | :--------------------: | ----------------------------: |
| user_message | BotSON encoded message | A message sent by an end-user |
| bot_mesage   | BotSON encoded message |     A message sent by the bot |
| bot_state    |      State label       |  The bot moved to a new state |

<p>You can set the attribute `defaults.analytics.data` in order to share data with all the events sent</p>
<p>Track your own events using the context function `track(analytics, data)`, where `data` is an object with arbitrary parameters that will be merged with the default `analytics.data`</p>

Available integrations:

| Service          | Availability | Credentials  |
| ---------------- | :----------: | ------------ |
| Splunk           |  Available   | {url, token} |
| Mixpanel         | Coming Soon  |
| Google Analytics | Coming Soon  |
