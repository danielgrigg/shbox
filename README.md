# shbox

# Intro

So this is essentially a Bash SDK for Box's API. It's limited,
but you can currently:

* authenticate automatically (without browser annoyances)
* list folders
* get thumbnail previews
* get files (partial support)

# Using

Simply source the script in your shell, ie:

. shbox.sh

The output instructions should get you going from there. 
An example session:

$ . shbox.sh
> _help info_

> box\_folder\_items | head
{
  "order": [
    {
      "direction": "ASC",
      "by": "type"
    },
    {
      "direction": "ASC",
      "by": "name"
    }
  ...
}

# Wtf?
Why did I write this? A) I could. B) no one seems to have done it
and I personally find it useful using curl etc for playing with API.
The automated oauth flow means you can use the script as the basis for
automating the flow in other languages, which is useful if you're into
automated testing etc.


Copyright (c) Daniel Casimir Grigg 2013
