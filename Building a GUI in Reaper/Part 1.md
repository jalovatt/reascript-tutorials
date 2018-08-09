# Building a GUI in Reaper (Part the First)

First, the bad news: We're pretty far off from the cool stuff. Reaper only provides a handful of drawing functions, and only a couple of ways to check what the user is doing, so any GUI is going to have to start from the very, very beginning.

## Opening a window

```txt
gfx.init("name"[,width,height,dockstate,xpos,ypos])
Initializes the graphics window with title name. Suggested width and height can be specified.

Once the graphics window is open, gfx.update() should be called periodically.
```

This is one of those not-as-common-as-we'd-like situations where everything is more-or-less self-explanatory.

- Our window needs a `name`.

- It needs `width` and `height` too, if we're going to do anything useful.

- `dockstate` is a number, determining which of Reaper's dockers the window will live in. Since we want an actual _window_ window, we'll set it to 0.

- We could leave the window floating up in the corner of the screen, or we could assign it an `x` and `y`.

**API tip**: The `[]`s up there are letting us know that those parameters are optional. We could give it a name and nothing else if we want. Isn't that nice of them?

```lua
gfx.init("Llamasplitter", 320, 240, 0, 0, 0)
```

There. Take it. Run it.

Did it work?

If you're using the Reaper's built-in script editor, you should see a blank window. Reaper's doing you a favor here - those of us using grown-up software to write our scripts saw nothing. Maybe a brief flicker, at best.

Why? Because we just told it to **open** a window - not to keep it there. It did what we asked and immediately moved on with its life.

(If you still have a window open, you'll have to close it by clicking the big X or red dot or whatever is there.)

## Keeping it there

Our GUI will need to run until either script decides it's done, or the user closes it. To do that, we need three things:

1. A way to keep the script running.

    `reaper.defer(func)`

    This is Reaper's equivalent of "hey, remind me to get groceries later" - the next time Reaper is processing scripts, defer() tells it to run your function again. If a function uses defer() to call itself again then, obviously, it will just keep running endlessly.

2. A way to keep the window open.

    `gfx.update()`

    This tells the window we opened to quit falling asleep, making sure it stays open as long as our `defer` loop keeps calling `update` again.

3. A way to determine when the script should end.

    `gfx.getchar([char])`

    This function gets the current keyboard state, which doesn't seem especially helpful at first glance. However, the Reaper devs were nice enough to include a small treat with it - if the window has been closed `getchar` will return `-1`, giving us an easy way to break the loop. Since we're civilized people, it's also customary to let the `Escape` key close the window too.

```lua
gfx.init("Llamasplitter", 320, 240, 0, 0, 0)

local function Main()

    local char = gfx.getchar()

    --  Closed         Escape
    if char == -1 or char == 27 then return end

    gfx.update()
    reaper.defer(Main)
end

Main()
```

Again, those using the built-in script editor will see things a little differently - or rather, exactly the same. It opens a window and the window sits there. If you watch the list of variables in the script editor, though, you can see that they stop updating as soon as you press `Escape` - the script IS finished. It just left the window there.

A couple of notes, before we actually make it do something:

- The call to `gfx.init` needs to be outside our main function because we only want to open the window once.
  
- `defer` only wants the _name_ of the function you want to be run - it doesn't want to immediately run it, so you can't include `()`s like so: `reaper.defer( Main() )`. To Lua that says you're giving `defer` whatever value `Main` returns, which in this case is a big fat _nil_.

## Drawing stuff

A black void on the screen is all well and good, but shouldn't it maybe have something it?

```txt
gfx.line(x,y,x2,y2[,aa])
```

Draws a boring ol' line, from the first pair of `x,y` coordinates to the second. The last parameter, `aa`, controls antialiasing and should pretty much always be left alone unless you have a reason to want it off.

```txt
gfx.circle(x,y,r[,fill,antialias])
```

This time we only have one coordinate - the center. After that is the `radius`, which is sure to give you chills if you didn't enjoy high school trigonometry. `Fill`, as the name cleverly suggests, will fill in the circle (`1`) or leave it empty (`0`). Antialiasing, again, is best left on.

```txt
gfx.rect(x,y,w,h[,filled])
```

It's a rectangle. Do I really have to explain this one?

```txt
gfx.roundrect(x,y,w,h,radius[,antialias])
```

"Ooh, you sneaky bastard. First a circle and then a rectangle and now a circular rect... wait, what?"

Calm down. It's a rectangle with rounded corners - effectively, the corners are quarters of a circle with the given `radius`.

Throwing this and some random numbers into our script, we get:

```lua
gfx.init("Llamasplitter", 320, 240, 0, 0, 0)

local function Main()

    local char = gfx.getchar()
    if char == -1 or char == 27 then return end

    gfx.line(10, 20, 80, 120)
    gfx.circle(40, 100, 20, 1)
    gfx.rect(120, 120, 60, 80, 0)
    gfx.roundrect(170, 32, 100, 80, 10)

    gfx.update()
    reaper.defer(Main)
end

Main()
```

Well, it's... it's, uh... it's certainly something. I'm sure there's a museum out there we could sellit to and say it was drawn by a monkey or something, but I'd feel bad about that. Monkeys can draw better than that.

Pop quiz: What parameter do `circle` and `rect` have that `roundrect` doesn't?

That's right - `fill`. If we want to color that lil' guy in, we're going to have to it ourselves. All it takes is a bit of effort and some ...dun dun DUNNNN... **Math**. Yup. You must have known it was coming sooner or later.

(I can't claim to have thought of this trick. **mwe** did, in [this thread here][mwe])

We already know that `roundrect` is making a rectangle with circles instead of corners. We also know how to draw circles and rectangles ourselves, and that _those_ functions _do_ have `fill` parameters. By combining four circles and three rectangles we can fill that roundrect ourselves:

```lua
-- Improved roundrect() function with fill, adapted from mwe's EEL example.
local function roundrect(x, y, w, h, r, aa, fill)

    local aa = aa or 1
    local fill = fill or 0

    if fill == 0 or false then
        gfx.roundrect(x, y, w, h, r, aa)
    elseif h >= 2 * r then

        -- Corners
        gfx.circle(x + r, y + r, r, 1, aa)		-- top-left
        gfx.circle(x + w - r, y + r, r, 1, aa)		-- top-right
        gfx.circle(x + w - r, y + h - r, r , 1, aa)	-- bottom-right
        gfx.circle(x + r, y + h - r, r, 1, aa)		-- bottom-left

        -- Ends
        gfx.rect(x, y + r, r, h - r * 2)
        gfx.rect(x + w - r, y + r, r + 1, h - r * 2)

        -- Body + sides
        gfx.rect(x + r, y, w - r * 2, h + 1)

    else

        r = h / 2 - 1

        -- Ends
        gfx.circle(x + r, y + r, r, 1, aa)
        gfx.circle(x + w - r, y + r, r, 1, aa)

        -- Body
        gfx.rect(x + r, y, w - r * 2, h)

    end

end
```

Trust me, it looks worse than it is.

- See if `antialias` or `fill` were given. If they weren't, use the default values. It's also worth noting the trick used here to save a bit of space:
  
    `aa = aa or 1` is the same as this:

    ```lua
    if not aa then
        aa = 1
    end
    ```

    But that's a trick that we can look at in a different, less graphics-oriented post.

- If `fill` was specified as 0, we don't actually need to be here. We can just pass all of the function's parameters right over to the normal `roundrect` and call it a night.
  
- If we're given a height that's less than twice the radius, we can get by with a smaller set of shapes. Less drawing = less CPU usage = your script doesn't bog down someone's computer while they're tracking Paul McCartney.

I won't go into the details of how it's working. You get one rectangle for the middle of the roundrect, smaller ones for each end, then four circles.

Presto! Let's add that to the script above and check out our masterpiece.

```lua
-- Improved roundrect() function with fill, adapted from mwe's EEL example.
local function roundrect(x, y, w, h, r, aa, fill)

    local aa = aa or 1
    local fill = fill or 0

    if fill == 0 or false then
        gfx.roundrect(x, y, w, h, r, aa)
    elseif h >= 2 * r then

        -- Corners
        gfx.circle(x + r, y + r, r, 1, aa)    -- top-left
        gfx.circle(x + w - r, y + r, r, 1, aa)    -- top-right
        gfx.circle(x + w - r, y + h - r, r , 1, aa)  -- bottom-right
        gfx.circle(x + r, y + h - r, r, 1, aa)    -- bottom-left

        -- Ends
        gfx.rect(x, y + r, r, h - r * 2)
        gfx.rect(x + w - r, y + r, r + 1, h - r * 2)

        -- Body + sides
        gfx.rect(x + r, y, w - r * 2, h + 1)

    else

        r = h / 2 - 1

        -- Ends
        gfx.circle(x + r, y + r, r, 1, aa)
        gfx.circle(x + w - r, y + r, r, 1, aa)

        -- Body
        gfx.rect(x + r, y, w - r * 2, h)

    end

end



gfx.init("Llamasplitter", 320, 240, 0, 0, 0)

local function Main()

    local char = gfx.getchar()
    if char == -1 or char == 27 then return end

    gfx.line(10, 20, 80, 120)
    gfx.circle(40, 100, 20, 1)
    gfx.rect(120, 120, 60, 80, 0)

    -- Changed to use our command, and being filled in
    roundrect(170, 32, 100, 80, 10, 1, 1)

    gfx.update()
    reaper.defer(Main)
end

Main()
```

And there you go. Let's see a monkey do that.

Although... it _is_ a little, uh... white.

[mwe]: http://forum.cockos.com/showthread.php?t=150499