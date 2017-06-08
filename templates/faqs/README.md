# FAQs

## A chatbot that answers simple questions from users

### Description:

This bot shows how to set up a simple FAQ bot that shows a `carrousel` with a few relevant cases and a link in a button.

Also tells how the `input_failure` state works.

### States:

- `initial`: State where the bot is initialized when a user talks to it.

- `main_menu`: Quick reply demonstration for deciding the next state.

- `human_agent`: Explanation of the human agent call.

- `faqs`: Carrousel demonstration.

- `show_main_menu`: Another quick reply.

- `input_failure`: Modified input_failure example. Acts like any other state, the difference is that it is triggered 
if the user input is not as expected.