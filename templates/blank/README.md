# Hello World!

## The minimal bot so you can start from scratch

### Description:

This bot contains the minimum required for the bot to start with no errors and work properly.
When the user says anything the bot starts at the `initial` state and output "Hello world!".

Note that in other states the `output` phase always is done **before** the `input`.

### States:

- `initial`: The bot only needs the declaration of this state with a `label`, an `input` and a `next_step`. For a more 
minimalistic bot that will do nothing the `output` is not needed.

### Flow:

- `User:` _any message_
- `Bot:` Hello world!
