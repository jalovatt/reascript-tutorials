# Introduction to Lua (Part the First)

If I'm going to spend the time writing Lua tutorials and examples I should first, perhaps, make sure that anyone reading them is up to speed on the language.

This post is intended as a brief - and by "brief" I mean "much longer than I thought it would be when I started" - overview of Lua. If you're completely new to coding, you may reach the end feeling bewildered, lost, and unsure of what the hell you just read - in that case, have a look at [X-Raym's lovely video tutorials][xraymtuts]. If you're at least familiar with the general concepts (variables, simple logic, what functions are), you should reach the end feeling curious, eager, and as if you know far more than you actually do (like a teenager).

I should note that, while the majority of this post is applicable to any Lua environment, at some point we'll inevitably move on to [Reaper][reaper]-specific functionality. It should be fairly obvious though - any functions starting with `reaper` or `gfx`, for instance.

## What is Lua?

>Lua is a lightweight, multi-paradigm programming language designed primarily for embedded use in applications. Lua is cross-platform, since the interpreter is written in ANSI C, and has a relatively simple C API. _(Wikipedia)_

Gee, thanks. Let's try that again in plain English:
>Lua is very friendly to your processor and memory, and lets you do things pretty much whatever way is easiest for you.

It's become incredibly popular in recent years as a scripting language for other software, particularly games - World of Warcraft, Garry's Mod, and Roblox are well-known examples that use Lua for their user-made content. It can also be used to make standalone applications, but that's obviously
beyond the scope of this post.

As programming languages go, Lua is pretty fun. It's fairly readable, yet very powerful, *ridiculously* flexible, and can do most of the same things any of the more popular languages do. One tradeoff, of course, is that Lua is a little less efficient for some tasks. As far as Reaper goes, audio processing is a big one - scripts that need to work with audio might take three, or more, times as long as doing the equivalent in EEL. EEL is specifically designed for audio, though, and is native to Reaper, so the contest is a little lopsided.

## The ReaScript editor

Yes, I tricked you - we're starting with Reaper stuff. Why? Because you're going to need somewhere to type your scripts. If you aren't a Reaper user, you'll have to install a Lua release and set it up yourself, then use whatever text editor makes you feel comfortable to follow along.

I prefer using an external editor (currently [Visual Studio Code][vscode], for reasons I won't get into here), but Reaper offers a very capable editor for scripts and JS effects. To access it, open the Action List and select `ReaScript: Edit new reaScript (EEL, lua, or python)...`. It will prompt you for a name - just make something up for now; it doesn't matter.

-- PLACEHOLDER: Screenshot of the editor with points of interest marked --

1. Current Filename

    When typing a function, this area will also list the parameters it expects. Downside: Some functions have a lot of parameters, so it often can't list them all.

2. Start Button

    This will run your script. This can also be done by pressing Ctrl+S, which will save and then run it.

3. API Help

    This button generates a big, long, _extensive_, **exhaustive** list of all the available functions and variables Reaper has to offer. It's quite impressive, but I honestly don't recommend using it - for one, it's a pain to search through. [X-Raym's API page][xraymdocs] is a bit easier to read, has a search bar, and can display only Lua functions for you. Use it, bookmark it, make sweet love to it. I almost always have it open in a tab somewhere.

4. Editor

    You type in it. Depending on what you type, it makes pretty colors so you can see what's going at a glance. Shortcut keys are listed along the bottom, and can vary depending on what you're doing.

5. Right-click Menu

    This will let you quickly jump around to different functions in your script, which becomes very handy for larger projects.

6. Debugging Options

    We can ignore them for now, but make sure `Run defer() code` is checked.

7. Variable List

    Any variables created in your project will be shown here, along with their value. This list is updated as the script runs, making it incredibly useful for debugging and seeing exactly what's going on and when.

8. API Search

    It really frustrates me that there's no label for this. Typing here will show any matching API functions in the list above, similar to the (superior) search bar on X-Raym's site.

Moving on...

## Basic syntax

As I said earlier, Lua is very readable. You don't have to worry about semicolons. We'll start with the traditional starting point for any programming language:

```lua
reaper.ShowConsoleMsg("Hello, world!")
```

We're performing a command, `ShowConsoleMsg`, provided to us by `reaper`, and giving that command a string of text, `Hello, world!` to do... well, whatever it is the command does. In an ideal situation, such as this, the command's name will be fairly self-explanatory - it prints a message to Reaper's console. Go ahead and type it in. Or paste it, because we're all lazy these days. Press Ctrl+S to save the file and run it. You should see:

```lua
Hello, world!
```

**Programming achievement unlocked: _Third-grader_!**

For anyone using a Lua environment outside of Reaper, the above code will clearly not work - you don't have `reaper` commands to use. Most interpreters should have an equivalent though: `print("Hello, world!")`.

## Variables

As the name suggests, variables are things that can vary. We use them as placeholders for storing information, ranging from a single number to a thousand-item list that contains, in each entry, another thousand-item list, some of which do the same and eventually end up referring back to other lists at different levels of the original one.

Yes, Lua will happily do that. Lua teaches yoga on weeknights and works as a contortionist for Cirque Du Soleil because _it's so incredibly flexible_. Too flexible, some might argue. But some would be wrong.

Adapting our original example, what if we felt like spamming the message a few times?

```lua
reaper.ShowConsoleMsg("Hello, world!")
reaper.ShowConsoleMsg("Hello, world!")
reaper.ShowConsoleMsg("Hello, world!")
reaper.ShowConsoleMsg("Hello, world!")
```

- That's a lot of typing. Or pasting, because even smart people aren't going to type that out four times. I certainly didnt.

- It's all printed on a single line.

    "Did the computer mess up? I thought Lua was supposed to be cool!"

    Rest assured that it didn't, and it is. We simply didn't tell it to skip to the next line.

Let's make a couple of changes:

```lua
a = "Hello, world!\n"

reaper.ShowConsoleMsg(a)
reaper.ShowConsoleMsg(a)
reaper.ShowConsoleMsg(a)
reaper.ShowConsoleMsg(a)
```

- We've _declared_ a variable, `a`, in this case declaring that it holds a string of text. Many languages require you to specify that sort of data (string, number, big number, true/false) a variable will be used for, but Lua doesn't. It's so busy catching up on Game of Thrones that you can even change what kind of data the variable contains while the script is running and Lua won't bat an eye as long as everything else is still working.

- "Wait, what's with the slashy thing? Why didn't it print that `n`? How did you add line breaks?

    Characters preceded by a backslash (`\`) form what's known as an _escape sequence_, which I can only assume is because someone wanted to _escape_ the boring task they had of giving names to programming tricks.

    Escape sequences are used, as you might have guessed, for typing operations that don't involve a written character. Return (`\n`) and tab (`\t`) are the most common.

Better, but still fairly repetitive.

## Loops

Why not let Lua handle the repetition? After all, that's why we love computers - they don't get bored.

```lua
a = "Hello, world!\n"
b = 4

for i = 1, b do
    reaper.ShowConsoleMsg(a)
end
```

Okay, so we've fixed the repetition at the expense of... well, a whole bunch of gibberish if I'm being honest.

- We create another variable, `b`, for the number of times we want to print our message.

- We tell Lua to start a loop using `for`, as in "for each sprinkle I find on my ice cream, I shall kill you".

- In this case, we just want it to count from `1` to `b`.

- On each repetition, we want it to `do` everything up until the `end` of that section.

## Functions

What if we want to print a different message? What if we want to print it a different number of times? We could edit `a` and `b` each time, certainly, but that might become tedious. And what if we wanted to do that multiple times in the same script? Smells like more repetition to me. If only we could create a more generic way to do it...

```lua
function repeat_msg(msg, num_times)
    for i = 1, num_times do
        reaper.ShowConsoleMsg(msg)
    end
end

repeat_msg("Hello, world!\n", 4)
repeat_msg("No, seriously - hello!\n", 3)
repeat_msg("Are you even listening to me, world?\n", 5)
repeat_msg("Fine, be a jerk. I'm switching to C++.\n", 2)
```

The first thing you'll probably notice is that I've given everything more descriptive names. This can be a _very_ critical step when writing, debugging, or simply reading code - especially if it's not yours. Short variable names and abbreviations are all well and good, as long as you can understand them, but anybody else looking at the script probably won't be able to.

Some languages force you to use short names, for whatever reason, but Lua doesn't. It doesn't care at all. You could write `this_variable_contains_a_single_integer_which_is_5 = 5` if you really wanted to, you psychopath.

As always, conciseness (concissitude? concissiery? concision?) is a virtue.

- We create a `function` called `repeat_msg`.

- The `()`s denote the function's _arguments_, a fancy word for "information it wants to see" and often used synonymously with _parameters_. They aren't quite the same thing, but only if you want to be pedantic.

- In this case, the function will expect to see some text, `msg`, and a number for the loop, `num_times`.

- The loop's code is identical, aside from changes to the variable names. I should note that, unlike the previous examples, we never specifically declared the variables - that's done automatically for all of the function arguments. So we've got that going for us, which is nice.

- As with the loop, we have to specify the `end` of our function.

"But wait!", you must surely be asking. "How does it know which `end` is for what?"

Lua is thankfully smart enough to match the inner `end` with the loop and the outer one with the function. Likewise, it will figure out the appropriate matches for whatever convoluted set of `(({[([])]}))` your code ends up having.

We can take this process one step further and create a _wrapper_ for the message command, just to save a bit more typing:

```lua
function Msg(msg)
    reaper.ShowConsoleMsg(msg)
end

Msg("Hello, world!")
```

- Yes, Lua is case sensitive, so no, it won't be confused by `Msg` and `msg`.

- Here we've created another function that simply takes a `msg` and passes it straight along to the original command.

Functions can be as short or as long as you want, and are a great place for doing fancy (I use the term "fancy" loosely, since we're still just typing stuff and looking at pretty colors in a text editor inside of a DAW) stuff so you don't have to remember it elsewhere.

```lua
function Msg(msg)
    reaper.ShowConsoleMsg(msg .. "\n")
end
```

- `..` tells Lua to put two strings together, giving us a really convenient way to automatically drop down to a new line every time we print something.

I hope we're all okay with keeping `Msg` around in order to save our fingers a bit of work - the remaining examples will assume that `Msg` is part of the script already.

## Logic

Lua is also, as it turns out, pretty good at problem-solving. Math? No sweat.

```lua
a = 2
b = 3
x = a + b
y = a * b

if x > y then
    Msg("x is greater")
else
    Msg("x is smaller")
end
```

- We've created two variables, and then used them to calculate two more. `*` multiplies, `/` divides, and if I have to explain `+` or `-` then you should probably stop reading now.

- We compare our results, the mysteriously-codenamed `x` and `y`, and see which one is bigger (`>` is the mathematical symbol for "greater than". You can probably figure out "less than").

- `if` `x` is larger than `y`, `then` we want to print the first message, or `else` print the second.

- Once again, and I can probably stop pointing it out after this, we declare our complex logical puzzle to be at an `end`.

Word problems? Easy peasy.

```lua
x = is_today_Thursday()
y = is_it_lunchtime()

if x == true and y == true then
    Msg("I never could get the hang of Thursdays.")
end
```

Assume, for the moment, that those functions do what their names suggest and _return_ a yes or no answer. "Yes" and "no", in computer-speak, are the same as "true" and "false" - the logic involved is known as a _Boolean operation_ because of course we have to name everything after a famous dead person. To be fair, though, he wrote the first book about it. So:

- We declare two variables, `x` and `y` again because programmers suffer from a lack of imagination. Actually, you know what? I'm going to take a [Mulligan][mulliganwiki] here.

```lua
isThurs = is_today_Thursday()
isLunch = is_it_lunchtime()

if isThurs and isLunch then
    Msg("I never could get the hang of Thursdays.")
end
```

There now, that's a little nicer.

- Referring to the original version, `==` is the comparative form of "equals". The distinction from `=`, used to **set** a variable, helps Lua keep track of what's going on because there are many situations in which either could be happening. Again, many other languages use the same convention.

- One of Lua's numerous, er, friendlinesses, allows us to remove `== true` altogether. `if isThurs` is much more readable, but syntactically the same as,`if isThurs == true`. Likewise, we can trim `if x == false` down to `if not x`.

- In this case, the answer is yes (that is, the statement is true) only if it's Thursday `and` it's lunchtime. The complement, `or`, works exactly like you would expect. Well, unless you don't expect it to work the way that it does in which case, well, it won't work like you would expect.

The example above can even be condensed and renamed one step further, if you want:

```lua
if today_is_Thursday() and it_is_lunchtime() then
    Msg("I never could get the hang of Thursdays.")
end
```

- Lua will see the `()`s and realise that there are two functions here, so it will run them and substitute any values they return in their place.

- Any variable can be treated as Boolean. If it exists, it's `true`. If it doesn't exist (`nil`), or is specifically false, it's `false`. Note that while some languages also consider `0` to be false, Lua does not. Zero is a perfectly cromulent number.

We can also combine mathematical and Boolean comparisons. I know, it's so exciting I almost peed myself when I first saw it. But only almost.

```lua
x = 4
user = "Justin F"

if x > 3 and user ~= "Justin G" then
   Msg("I don't have a witty message to put here.")
end
```

- The Boolean operator `~=` translates as "does not equal".

More importantly, for pretty much any script doing something worth using a script to do, we can chain these statements together in all sorts of diabolical ways, using `()`s to mark phrases that should be read as a whole.

```lua
time = 6
user = "Justin F"
day = "Friday"
released_Reaper_update = true
beer_in_fridge = 3

if user == "Justin F" and beer_in_fridge >= 1 and ((time > 5 or released_Reaper_update) or (day == "Friday" or day == "Saturday" or day == "Sunday")) then
    get_beer(1)
end
```

"...what the hell did I just read?"

Yet another thing I love, _love_, **love** about Lua is how it simply doesn't give a shit about spaces, indenting, or empty lines. You can use as many of them as you want more-or-less wherever you want.

Let's use that to our advantage here:

```lua
if  user == "Justin F"
and beer_in_fridge >= 1
and (   (time > 5 or released_Reaper_update)
    or  (day == "Friday" or day == "Saturday" or day == "Sunday"))
then
    get_beer(1)
end
```

Slightly better. Many programmers would have my head on a stake for doing that a) splitting an _if_ statement across multiple lines, b) indending the _or_, and c) making such a complicated statement in the first place, but I find it straightforward and easy to read. Do whatever works for you.

- If `Justin F` is the current `user`,

- `and` if there's at least `1` beer in the fridge,

- `and` if **either** it's after `5` or Justin has `released_Reaper_update`, `or` the current `day` is Friday, Saturday, or Sunday because even Justin deserves some time off,

- Justin is allowed to have a beer. But only one. **THE UPDATES MUST FLOW**.

## Conclusion

In conclusion, because this concludes our first post, and I conclude that this is a very poor way to write a conclusory statement, we've now learned:

- How to find and open Reaper's script editor.
- How to write a very simple Lua script.
- How to write a slightly more difficult Lua script.
- How to turn parts of a script into functions so they're easy to reuse.
- How to have the script do a bit of thinking for itself.

Not too shabby, I think.

## Further Reading

[Programming in Lua][pil]
[Lua 5.3 Reference Manual][luamanual]

[xraymtuts]: https://www.extremraym.com/en/reascript-video-console/
[reaper]: http://reaper.fm/
[vscode]: https://code.visualstudio.com/
[xraymdocs]: https://www.extremraym.com/cloud/reascript-doc/
[mulliganwiki]: https://en.wikipedia.org/wiki/Mulligan_(games)
[pil]: https://www.lua.org/pil/contents.html
[luamanual]: http://www.lua.org/manual/5.3/contents.html#contents
