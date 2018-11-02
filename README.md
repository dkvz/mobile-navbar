## Analysing
WIth Materialize they use a `<ul>` element with this style:
```
backface-visibility: hidden;
background-color: rgb(255, 255, 255);
box-shadow: rgba(0, 0, 0, 0.14) 0px 2px 2px 0px, rgba(0, 0, 0, 0.12) 0px 1px 5px 0px, rgba(0, 0, 0, 0.2) 0px 3px 1px -2px;
box-sizing: border-box;
color: rgba(0, 0, 0, 0.87);
font-family: Roboto, sans-serif;
font-size: 14px;
font-weight: 400;
height: 948px;
left: 0px;
line-height: 21px;
list-style-type: none;
margin-bottom: 0px;
margin-left: 0px;
margin-right: 0px;
margin-top: 0px;
overflow-y: auto;
padding-bottom: 60px;
padding-left: 0px;
position: fixed;
top: 0px;
transform: matrix(1, 0, 0, 1, 0, 0);
width: 300px;
will-change: transform;
z-index: 999;
```

## The animation
The animation can be done with transform (or hiding the block can be done with transform).

It looks like they use an initial transformX equal to the width, and then set it to 0.

Is it better to use that or the width: 0 to the actual width?

I'm going to go for transform this time because I suppose the materialize guys know optimization stuff that I don't (I hope they do).

### Perfect timing
The default timing feels a little weird. Maybe I should try the cubic bezier proposed by VSCode or the one in that Paul Lewis video.

## Will-change
There is a CSS prop called will-change which indicated what you're going to do with a block that could be expensive.

As shown in the investigation above, Materialize uses `will-change: transform`.

I'm confused however because on the MDN page they're saying you shouldn't use this unless absolutely necessary or something (see here: https://developer.mozilla.org/en-US/docs/Web/CSS/will-change).

## Gestures
There are 3 gesture events to register:
* touchstart
* touchmove
* touchend

For each of these the event has a changedTouches array, we need to access the first element of that array to get the coordinates. Which I think are called clientX and clientY. These are strings with a trailing "px", so bear that in mind if you need to do calculation.

I think parseInt works, maybe Number does as well.

-> You need to add preventDefault for touchstart and touchmove (and maybe touchend), or at the very least touchmove so that these events do not register a click as well.

I should address this message:
```
Ignoring ‘preventDefault()’ call on event of type ‘touchstart’ from a listener registered as ‘passive’.
```
Not sure what they mean here.

I removed all the preventDefault at some point. I don't think it's needed at all.

## Menu height
The easiest is to use viewport units and set it to 100vh.

It's possible to use height 100% but then body has to have height 100% as well.

On my site I have body min-height set to 100vh and a flex layout to display the header, main and footer sections. So I might as well use viewport units anyway.

## The overlay
There's more than one way to do the mobile menu, Materialize creates an overlay div to have the transparent grey background.

Problem: on mobile, when both sliding in the menu and scrolling, the overlay stays at the top. I think we might have to position: fixed the thing.

## Drop shadow
You can use a drop-shadow but then the translate has to go further to the left.
OR: you could just display: none the thing.

I think I'm going to try display: none, and showing it in the script.
-> For some reason it doesn't stack well together, the transform transition does not proc.

I could use a setImmediate or something but that's getting too complicated. Let's translate a little more. Since I'm using percentages I can use 110% instead of 100%. 10% of my 300px width is 30px.

## Events
If using an overlay, closing the menu is easy: just bind a click event to the overlay (also probably work something  out with gestures).

Without overlay something has to be done with body. But a body click event will always fire as far as I know. Which is annoying.

Clicking any link inside the menu should also close the menu, so we need to bind an event listener for that on every single link at initialization.

### Issues with my links
Clicking a li element in the nav list acutally closes the menu without doing anything. We should have a click on the li proc a click on the child link... Or maybe not close the men once we click inside the menu thing.

Solved this by not applying styles on li (only on a).

The link click event still isn't firing right when using gestures. Don't know if it's just on the Firefox inspector, I'll assume it's a general issue with the click not happening when handling gestures.

This may be linked to how taps are captured? Clicking the link always works unless we are on mobile mode in the Firefox inspector. It works fine on the Chrome inspector, haven't tried on a mobile.

OK it works even for Firefox on mobile. I think we're fine like this.

-> Link issue resolved.

## Media queries
I think we only need media queries to hide the button that shows the menu.

## To investigate
* What is backface-visibility?
* Do I really need line-height if I'm using paddings?