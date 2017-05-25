# IBM Watson Template

## An example of how to integrate with IBM Watson and use the intents to navigate the conversation flow

### Description:

This is a basic template for integrating Watson to botSON. First, Watson account details have to be specified for the
bot to connect.
 
In the bot is shown how to do a call to Watson and decide the next state with the response.

### States:

- `initial`: State where the bot is initialized when a user talks to it.

- `get_an_intent`: Call to Watson. It will send the message of the user to it and save the response in the variable 
'response'.

- `search_products`: If Watson decides that the intent of the user is 'search' it will go here. This is a placeholder.

- `dont_understand`: Else the user will go here.
