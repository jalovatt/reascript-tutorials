# Building a GUI in Reaper (Part the Zeroth) 

**Note**: For this series, I'll be assuming a decent amount of familiarity with Lua, some of the "best practices" when writing a script, etc. There is a bit of a jump from my Lua Overview tutorials, and I'll try to fill that gap in eventually.

## In which we establish why user interfaces are necessary

So. 

It has come to [this][xkcd].

You've been playing with Reaper scripts for a while, you're having fun, and maybe you've even released some of them into the wild (you'd better be using [ReaPack] or I will hunt you down).

But something's missing... something to make your scripts feel like a Real Thing and not just another entry in Reaper's Action List, a place so huge and cavernous that you wouldn't be surprised to find the Ark of the Covenant hidden somewhere.

## User Interaction

Oh, right.

There's nothing wrong with a script just does one thing with a minimum of fuss, certainly, but at some point you'll come across an idea that can't be done on autopilot. Imagine that our buddy Justin has asked you to "write me a script that will chop the selected items up into pieces".

Great. That's easy enough to do. But how many pieces?

"5", he replies.

So you set to work, and a few days later send him a test version.

"Oh, I have items that are different lengths. I don't want the small items to end up in a bunch of small pieces. Can it have a minimum length?"

Sure. What?

"11.5 milliseconds."

...okay. Can do.

Off you go, and yet again you send Justin a script to test.

Again, he has a new idea. "When I have a take that I really like, I name the item **Llama**. Could it skip those items?"

Yes, yes it could.

Finally, after some back-and-forth with the API documentation you send him the finished script. He's happy with it, thank God.

A week or two later, he asks for another script - the same idea, but cutting them into 12 pieces, with no minimum length, and skipping items named "Alpaca".

You dig through the code for a few minutes, find the relevant variables, and swap them out. No sweat.

But then you get messages from a few people in his Reaper For Camelid Enthusiasts group on Facebook.

8 pieces. 3 pieces. 2 pieces.

200 milliseconds. 34. 11.6.

Dromedary. Bactrian. Vicuña.

That's the last straw - no way in HELL are you taking the time to look up the keyboard shortcut for `ñ`. These people can do it themselves.

"Yes, but none of us understand Lua. We're too busy with Facebook."

You kindly offer to put all of the variables at the very top of the file, with nice pretty comments saying `USER SETTINGS ARE HERE` so anyone who can open a text editor can change them.

Unfortunately, some of these folks aren't that computer-savvy. One of them accidentally saves the script as `.lua.txt`. One of them remembered a bit of C from his youth and accidentally put some `+=`s in for some reason.

This has to stop. There must be a better solution. Maybe there's a way to let the script ask _them_ so they don't have to edit it.

```txt
boolean retval, string retvals_csv = reaper.GetUserInputs(string title, integer num_inputs, string captions_csv, string retvals_csv)

Get values from the user.

If a caption begins with *, for example "*password", the edit field will not display the input text.

Maximum fields is 16. Values are returned as a comma-separated string. Returns false if the user canceled the dialog. To increase text field width, add an extra caption field, and specify extrawidth=xyz
```

_**BAM**_. Here we were, enjoying a nice little story, and then I had to go and bring programming into it.

As intimidating as that might look, it's not too bad. The bad part comes after, but we'll get there soon enough. Let's see what we're working with:

- It wants a `title`. Great.

- It needs to know how many values (`num_inputs`) we're asking for.

- It needs `caption`s for each entry, given as a `csv` or _comma-separated value_.

- It wants... values to return? A little unintuitive, but `retvals` is just the default values for each entry, again as a CSV.

When the user is done, `GetUserInputs` will return not one, but two values:

- `retval`, a Boolean telling us if they hit OK (`true`) or Cancel (`false`)

- `retvals`, a CSV containing whatever the user entered.

Our hypothetical script needs to prompt the user for three values:

```txt
Number of pieces:
Minimum length (ms):
Skip items named:
```

As defaults, we might as well use the values Justin originally gave us - 5, 11.5, and "Llama".

```lua
title = "Llamasplitter" -- This would make a great band name
captions = "Number of pieces:,Minimum length (ms):,Skip items named:"
defaults = "5,11.5,Llama"

retval, retvals_csv = reaper.GetUserInputs(title, 3, captions, defaults)
```

All good. If you paste that into a blank script and run it, you should see a nice... I was going to say "pretty", but on Windows it isn't really... you should see a dialog box asking you for stuff. Mission accomplished.

Almost. We still have to get the user's information out of that CSV. Remember when I said there was a bad part?

```lua
if not retval then return end
num_pieces, min_length, skip_name = string.match(retvals_csv, "([^,]+),([^,]+),([^,]+)")

reaper.ShowConsoleMsg(  "Number of pieces: " .. num_pieces .. "\n" ..
                        "Minimum length (ms): " .. min_length .. " \n" ..
                        "Skip items named: " .. skip_name)
```

Paste that in and give it a spin.

- `string.match` accepts a string (duh) and a _pattern_ to look for.

- Each set of `()`s marks a portion of the string we want to _capture_ into a variable. Notice how there are three sets?
  
- `[]` defines a _set_ - in this case, the set of everything that is not (`^`) a comma.


- We want to capture as much as we can (read: all), so we add `+` to tell it "as many characters as you can".

- Since it's a CSV, we separate each capture group by commas.

All told, our pattern says "Find me three sets of things that aren't commas, separated by commas", which works out _beautifully_ with the fact that we have a CSV to work with. Almost as if it were planned...

## Things take a turn for the worse

Justin and his friends are astounded by what you've come up with. It works perfectly for them, everyone can run it, and nobody has to get their hands dirty.

In fact, it works TOO perfectly. They start asking you for even more parameters and suddenly that dialog box starts looking REALLY questionable.

You need a different way to interact with the user. Something that offers more control. Something that lets you control exactly what they can do and how they can do it.

You need a GUI.

After some digging through the API and a few trips to the forum, you realize something. Lua doesn't have anything for a GUI. Reaper won't let you load any of the GUI frameworks that exist for standalone apps, like Corona or wxWidgets. Reaper also doesn't provide anything beyond a few basic drawing functions.

If you want a GUI... you're going to have make one yourself.

[xkcd]: https://xkcd.com/1022/