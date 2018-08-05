# Introduction to Lua (Part the Second)

## Tables

Most scripts find themselves needing to store more than one piece of information about the same thing. You could do what we've been doing and declare a variable for each of them:

```lua
breakfast = "Eggs"
second_breakfast = "Bacon"
elevensies = "Steak"
luncheon = "Waffles"
afternoon_tea = "Curry"

purchase_groceries_for(breakfast, second_breakfast, elevensies, luncheon, afternoon_tea)
```

You **could** do that, but why?

```lua
meals = {
    breakfast = "Eggs",
    second_breakfast = "Bacon",
    elevensies = "Steak",
    luncheon = "Waffles",
    afternoon_tea = "Curry"
}

purchase_groceries_for(meals)
```

Shit, that was easy.

- `{}`s denote a _table_. In this case, our table is referred to as a _keyed_ table because we've used variable names as a key for each entry. The alternative is an _indexed_ table:

```lua
meals = {"Eggs", "Bacon", "Steak", "Waffles", "Curry"}
```

In this case, the entries are assigned slots (_indices_) in the order they were entered - 1, 2, 3, 4, and 5.

**Side note:** Lua is unusual in that it expects table indices to start from 1. The reasoning for this is, roughly, "we wanted it to be understandable by engineers and not just programmers". All of the built-in table functions start from `1`, which is often a point of confusion for anyone coming from another programming language. You can use 0, but it will require extra work and rewriting of functions on your part. You could use -10 if you wanted, and in some cases that might even be sensible.

Both methods have pros and cons, which would constitute a decently-sized post on their own. For now we'll just worry about how to access the entries in each.

```lua
meals = {
    breakfast = "Eggs",
    second_breakfast = "Bacon",
    elevensies = "Steak",
    luncheon = "Waffles",
    afternoon_tea = "Curry"
}

Msg(meals.breakfast)
```

```lua
meals = {"Eggs", "Bacon", "Steak", "Waffles", "Curry"}

Msg(meals[2])
```

As with everything else, these can be used in the strangest places. Such as:

```lua
if time > 10 then
    start_making(meals.elevensies)
end
```

or

```lua
for i = 1, #meals do
    find_ingredients_for(meals[i])
end
```

- `#meals` is shorthand for `table.length(meals)`, which is itself shorthand for "the number of entries in `meals`". Keep in mind that this will only work for indexed tables - Lua isn't able to count the entries in a keyed table without a little help:

```lua
function count_keys(t)
    num_keys = 0
    for key, value in pairs(t) do
        num_keys = num_keys + 1
    end
    return num_keys
end

for i = 1, count_keys(meals) do...
```

[Pretend there's a GIF here of Keanu Reeves saying "Whoa"]

- We've introduced a new way of looping, `pairs`, which simply pulls out each entry in a table (keyed or indexed), returning it as the variables `key` and `value`. You can of course use names that are more descriptive for the task at hand, such as `for meal, food in pairs(meals) do`.

- For each entry we add 1 to the value of `num_keys`.

- After the loop has finished, the function `return`s our total.

- Lua is quite content to let you shove a function into any place you can think of, as we've done with the second `for` loop here. When the script reaches that line, it will get the value returned by `count_keys` first and then use it for the loop's upper limit.

## The Reaper API

"What now? Reaper has its own [brand of tiny rack equipment][API]?"

Not quite. _Application Programming Interface_ is just a fancy term for "the things a program will let another program do to it". Sounds a lot sexier that way, right?

We've already made heavy use of one such API function, `reaper.ShowConsoleMsg`. At this point you might notice the similarity to a keyed table - any suspicions this might arouse are, in fact, correct. `reaper` is just a table. Lua, and any APIs for it, are built on tables [all the way down][turtles]. There are even hidden tables, furtive, secretive tables, lurking in strange corners, visible only from the corner of your eye and even then only briefly. No, I'm not kidding, but that's a story for another time.

As mentioned in the previous post, the ReaScript editor can generate a list of the functions and variables exposed by Reaper's API. (The same document is also available from Reaper's Help menu) Again, I would be remiss if I didn't mention [X-Raym's much friendlier API page][xraymdocs].

Exhibit A:

```lua
MediaTrack reaper.GetTrack(ReaProject proj, integer trackidx)
get a track from a project by track count (zero-based) (proj=0 for active project)
```

- `reaper.GetTrack` is the function in question.

- It expects to see two things:

    1. A `ReaProject` (don't worry about what that is yet) that will tell it which project tab to work on. The function's documentation also mentions that we can simply tell it `0` if we just want the currently-active project, which in most cases is true.

    2. A track number. The documentation notes that this numbering starts from 0, yet another programming convention, rather than from 1 as the track numbers are displayed.

- The function returns a `MediaTrack`. This is a specific variable type, as is `ReaProject`, that allows our script to access track information directly.

    "Um, but you said last time that Lua doesn't have variable types."

    You're right, I did. And it doesn't. Functions typically expect a certain kind of data though, and will tell you as much in an error message if they don't get it. It would be difficult to find track number `"Godspeed You! Black Emperor"`, for instance, or to perform `x = "bob" / 4`.

    Thus, despite not actually having variable types, it's good manners for any API to still tell you what types it wants.

Functions are always used in the same ways we've seen:

```lua
track_num = 4
track = reaper.GetTrack(0, track_num - 1)
```

- Note the `- 1`, which allows us to use "real" track numbering for our variable while still pointing `GetTrack` to the correct number internally.

Okay, so what can we do with a `MediaTrack`? For starters, let's try printing the names of every track in the project. First, we'll need this again:

```lua
function Msg(msg)
    reaper.ShowConsoleMsg(msg .. "\n")
end
```

This example is a bit bigger than what we've looked at previously, so it seems like a good time introduce yet another convenience - comments. Any time Lua sees `--`, it will consider the remainder of that line to be a _comment_ aimed at humans and not part of our code.

```lua
-- Get every track name in the project
-- Returns an indexed table
function GetTrackNames()

    -- This will hold all of our track names
    names = {}

    -- integer reaper.CountTracks(ReaProject proj)
    -- count the number of tracks in the project (proj=0 for active project)
    for i = 1, reaper.CountTracks(0) do

        -- Subtract to match the internal track numbering
        track = reaper.GetTrack(0, i - 1)

        -- Use the MediaTrack to find its name
        -- boolean retval, string buf = reaper.GetTrackName(MediaTrack track, string buf)
        -- Returns "MASTER" for master track, "Track N" if track has no name.
        ret, name = reaper.GetTrackName(track, "")

        -- Add it to our table
        names[i] = name

    end

    -- Return the table to whatever called this function
    return names

end

names = GetTrackNames()
Msg( table.concat(names, "\n") )
```

Okay, so there's a little more going on this time.

- As before, we create a function to hold our code.

- Within this function, we create an empty table to hold all of the `names` as we compile our list.

- We create a loop that will run once for every track in the project. As with `GetTrack`, `0` here specifies the current project tab.

- On each loop, we get a `MediaTrack` and pass it along to `GetTrackName`. This function adds a couple of new wrinkles:

    1. Two returned values. In this case, the first is simply an indicator - `true`/`false` - of whether it got a name for the `MediaTrack` or not. This is fairly common, and is handy when trying to error-proof your code.

    2. A blank string. Wait, what?

        This is a Reaper quirk. Because the API is exposed for three different languages (Lua, EEL, and Python), the developers have opted to make the function calls friendly for all three when possible. Many languages, like EEL, are unable to return multiple values and instead have you provide a variable for the returned value _as function arguments_ - that is, you give them a variable and they set it themselves:

        ```C
        // bool GetTrackName(MediaTrack track, #buf)
        retval = GetTrackName(track, name);
        ```

        When dealing with functions like this, Reaper just needs to see something of the expected type - a string, in this case, so we give it a blank one: `""`.

- We add the `name` to our table.

- Once the loop finishes, the entire table is returned.

- Back at the "top" level, we have a variable hold on to that table.

- We print the contents of the table using `table.concat`, which puts all of the entries into a string separated by whatever we specify - `\n`, here, so that they're all printed to new lines.

**Note:** As with the length function in the previous post, `concat` will only work on an indexed table. For a keyed table, you'll have to write an equivalent yourself. Or Google one.

## Error Hunting

Let's face it - sometimes shit goes wrong. You make a typo, the user does something you didn't expect, Reaper behaves counterintuitively (which _never_ happens), or your script runs into a situation that falls outside your carefully-constructed web of logic.

And it crashes. HOW? WHERE?

Here's an example:

1. Open a fresh project.

2. Create a few tracks.

3. Give them names, even something as simple as A, B, C...

4. Copy/paste the following code into a blank ReaScript and give it a quick read:

```lua
-- Delete every track in the project
num_tracks = reaper.CountTracks(0)
for i = 1, num_tracks do
    reaper.ShowConsoleMsg("Deleting track " .. i .. "\n")
    track = reaper.GetTrack(0, i)
    retval, name = reaper.GetTrackName(track, "")
    reaper.ShowConsoleMsg("\tGoodbye, " .. name .. "!\n\n")
    reaper.DeleteTrack(track)
end
```

Everything appears to be on the up-and-up, right? Nothing we haven't already done. Let's see what's going on:

- Get the number of tracks in the project.

- Set up a loop to run from track 1 all the way to the end.

- On each loop:

  - Get the `MediaTrack` for the current track number.
  
  - We get the track's name so we can...

  - Say goodbye to it.
  
  - Delete the track.

We're all agreed? Looks pretty straightforward?

5. Run the script. Go ahead. I'll wait.

"**WHAT THE HELL HAPPENED?** I thought you said this script would work! I trusted you!"

Calm down. That's why we used a blank project. You... you did use a **blank** project, right? Not something for a client...? Please tell me you didn't run that script on an actual project.

Kidding, of course. _Ctrl+Z_ is always there to save your ass, like when your parents tell you "I don't care how late it is, or where you are, you can always call us for a ride home."

Before we can diagnose the problem, or problems, we should first try to identify everything that went wrong here:

- The console messages are screwy, and a bit... uh... incomplete.
  
        Deleting track 1
            Goodbye, B!

        Deleting track 2
            Goodbye, D!

        Deleting track 3

- We still have a few tracks.
  
- The ReaScript editor doesn't draw as much attention to it as I'd like, but down at the bottom you might have noticed an error message. Don't worry if you didn't - undo what the script did, run it again, and see what comes up.

        testing.lua:6: bad argument #1 to 'GetTrackName' (MediaTrack expected)

Finally, a clue. So what's that error telling us?

- `testing.lua` had a problem, specifically on line `6`.
  
- On that line, `reaper.GetTrackName` had a tantrum because it was expecting a `MediaTrack` for its first argument. (Remember when I said that functions will tell you if they don't get what they want?)

Start from what we know - it wanted a `MediaTrack` and didn't get one - and work backwards to see what could have caused it. We just passed it the information that we got from `reaper.GetTrack`, which means it didn't give _us_ a `MediaTrack` in the first place.

Again, what could have caused that? `GetTrack` was given the default project, `0`, and the current track number, `i`, and I'm extremely confident in your ability to have typed `0` correctly. We know that `i` is a valid number because it comes directly from the `for` loop, but... what if `i` isn't a valid _track_ number?

"Huh? WTF are you on about?"

Consider: You're driving to a place you've never been using a list of directions. One of them says "turn onto highway 19 and drive 3.8 miles", so off you go, but then Highway 19 ends at a T around 3 miles in. The information on your paper is valid - that is, 3.8 kilometers is a real distance you can drive, but _not on Highway 19_.

Most of us have experienced something like this, so a few solutions present themselves:

1. It's the wrong highway, and perhaps 19A is a little further along the previous road. Referring back to our script, that would mean our `for` loop was looking at the wrong set of numbers. That's silly, of course - `GetTrack` wants a track number, and the loop is counting track numbers.

2. It's the wrong distance. There _was_ a house back at **2**.8 miles, so that's a possibility. In script land, that would mean counting too many tracks... but we got the track count from Reaper so this too seems unlikely.

3. The directions are outdated, and highway 19 isn't 3.8 miles long _anymore_. Ooh, that's intriguing. Is our script doing anything that might affect the total number of tracks?

[Pause for dramatic effect]

Well, we are **deleting** them.

[Light-bulbs start turning on]

There you go - I knew you'd catch up.

It looks like we need a different way to track the, er, tracks, as we delete them. Think about what happens when you delete track 1 - is there just a hole there? A gaping wound in the project, bleeding 1s and 0s all over the place? No - all of the other tracks shuffle down by one place.

What if we just keep deleting track 1?

```lua
-- Delete every track in the project
num_tracks = reaper.CountTracks(0)
for i = 1, num_tracks do
    reaper.ShowConsoleMsg("Deleting track " .. i .. "\n")
    track = reaper.GetTrack(0, 1)
    retval, name = reaper.GetTrackName(track, "")
    reaper.ShowConsoleMsg("\tGoodbye, " .. name .. "!\n\n")
    reaper.DeleteTrack(track)
end
```

All I've done there is change the `i` in our `GetTrack` call to `1`. The script still gets the total number, loops that many times, and says goodbye, but operates on track 1 every time.

        Deleting track 1
            Goodbye, B!

        Deleting track 2
            Goodbye, C!

        Deleting track 3
            Goodbye, D!

        Deleting track 4
            Goodbye, E!

        Deleting track 5

Much better, but track A is still there and we still have the same error message.

Referring back to the driving example, some of you will have driven on roads with mile/KM markers - this is common in many rural areas. It's usually important to know what mile marker to start at if giving someone directions - we might turn onto highway 19 at mile 12, in which case we could expect (ignoring the outdated directions) to be looking for a house just prior to mile 16.

Back to Reaper - what number are we starting at? Well, the `for` loop is starting at 1, and the tracks are numbered from 1. All good, right?

```lua
MediaTrack reaper.GetTrack(ReaProject proj, integer trackidx)
get a track from a project by track count (zero-based) (proj=0 for active project)
```

Look! See how neatly I brought us right back to the same API call? That wasn't even intentional, it just happened.

I see the problem. X-Raym sees the problem. Even Justin F sees the problem and he's busy drawing llamas in his notebook.

`GetTrack` wants track numbers that start at `0`, and we're starting the loop at `1`.

```lua
-- Delete every track in the project
num_tracks = reaper.CountTracks(0)
for i = 1, num_tracks do
    reaper.ShowConsoleMsg("Deleting track " .. i .. "\n")
    track = reaper.GetTrack(0, 0)
    retval, name = reaper.GetTrackName(track, "")
    reaper.ShowConsoleMsg("\tGoodbye, " .. name .. "!\n\n")
    reaper.DeleteTrack(track)
end
```

Again, I've simply changed the `1` in our `GetTrack` call to `0`.

        Deleting track 1
            Goodbye, A!

        Deleting track 2
            Goodbye, B!

        Deleting track 3
            Goodbye, C!

        Deleting track 4
            Goodbye, D!

        Deleting track 5
            Goodbye, E!

Script is work. Much correct. Very delete.

(If you've been paying attention, you might notice that I solved this problem very differently for the example script in the Reaper API section earlier. As I said in the previous post, Lua is very flexible and generally lets you do what you want, how you want. Do whatever works for you. In general, I prefer naming my variables so it's obvious whether I'm working in "displayed" track numbers vs. internal ones, and then throw in the occasional `- 1` or `+ 1` when a conversion is necessary.)

## Error Handling

One more example and I'm done. I swear.

```lua
-- Delete the first selected track
local track = reaper.GetSelectedTrack(0, 0)
reaper.DeleteTrack(track)
```

Again, create a fresh project, add some tracks, paste the script, run it.

It works... if you have a track selected. If not, you get the same error message we were seeing above. Why? Because the script just assumes Selected Track #0 will be there.

We could solve this, as in the previous script, by getting the number of selected tracks first. That seems like overkill for such a tiny script, though. Why not just prevent the error and move on with our lives?

In the previous tutorial we saw that Lua will happily use any value/variable/function you want to make Boolean comparisons. This is the perfect situation for that:

```lua
-- Delete the first selected track
local track = reaper.GetSelectedTrack(0, 0)
if not track then return end
reaper.DeleteTrack(track)
```

- Get the first selected track.
  
- **Check if we actually got something.**

- If we didn't, `return` will immediately exit whatever function we're in. In this case, that's the script itself.

- If we did get something, continue on deleting things mercilessly.
  
Checks like this are known as _guard clauses_ in the programming world, and they're incredibly helpful. It's typically a good idea to validate things as early as you can so that later in your script, when you're four levels into a nested set of functions and loops, you can write code with the expectation that values coming in won't break anything.

## Conclusion

Today we learned:

- How to create a table.

- Several ways to access it.

- How to interpret the Reaper API.
  
- How to diagnose, isolate, and fix problems with a script.
  
- How to keep problems from happening in the first place.


[API]: http://apiaudio.com/product.php?id=109
[turtles]: https://en.wikipedia.org/wiki/Turtles_all_the_way_down
[xraymdocs]: https://www.extremraym.com/cloud/reascript-doc/