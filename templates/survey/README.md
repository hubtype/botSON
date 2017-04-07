#Survey Template

##A sequence of questions with different ways to get the answers

###Description:

This bot shows diverse ways to display inputs to the user. These are `quick replies`, `message buttons`, `buttons in 
lists` _(facebook)_ and a succesion of `message buttons` _(placeholder for other messaging apps)_. 

Also, in the end is shown how to access variables inside the bot using [Jinja](http://jinja.pocoo.org/).

###States:

- `initial`: State where the bot is initialized when a user talks to it.

- `survey_quick_replies`: Quick reply demonstration. The response is saved in _answer1_.

- `survey_buttons`: Message buttons demonstration. Each button activates a trigger. There the response is saved in 
_answer2_ and the state is redirected to `survey_facebook_list` or `survey_alternative_list` depending of the messaging app.

- `survey_facebook_list`: List with buttons demonstration. Buttons works the same way in different containers. The 
response is saved in _answer3_.

- `survey_alternative_list`: This shows how to obtain a similar result of list in other messaging apps like telegram.

- `survey_results`: This state outputs the content of the 3 answers using Jinja templating.