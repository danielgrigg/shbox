# shbox

So this is essentially a Bash SDK for Box's API. It's limited,
but you can currently:

* authenticate automatically (without browser annoyances)
* list folders
* get thumbnail previews
* get files (partial support)

Why did I write this? A) I could. B) no one seems to have done it
and I personally find it useful using curl etc for playing with API.
The automated oauth flow means you can use the script as the basis for
automating the flow in other languages, which is useful if you're into
automated testing etc.


Copyright (c) Daniel Casimir Grigg 2013
