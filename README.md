I got kinda bored one day and started playing around with writing my own "fancy" bf interpreter.
Started out by realizing that it'd be really handy in bf if every other cell described the cell to its right.
Then realized that interlacing data allowed you to basically have as many "virtual" tapes of cells as you wanted.

Then I wrote a stack implementation, this project doesn't use it but it was good practice.
- Cell 1 was whatever value you wanted to push or the last value popped.
- Cell 2 and every even cell after (4,6,8,etc) was either 1 for "you've not reached the end of the stack" or 0 for "you're past the end of the stack".
- Cell 3 and every odd cell after (5,7,9,etc) was the value on/in the stack.

As a test I made it so that it was every 3rd cell as opposed to every other so that I could have a free row to work with.
Seeing how easy this was to manage, I added 2 symboles "#" and "@" to allow switching between the rows.
Late into the project I realized I had to set an upper and lower limit for rows.
This is due to the fact that you can't, reasonably, allow arbitrary numbers of rows if you want to convert back to raw bf

To convert # and @ back into raw bf at the end you just multiply all of your > and < by the number of rows (ie: 3 rows = >>> and <<<)
Then swap the # with > and @ with <. (or the other way around, I decided # went up and @ went down)

I then wrote a pretty nice method of allowing numbers greater than 255 by using multiple cells per number with 1 extra cell to specify if the cell is 0 (as to allow looping)
This wasn't necessary for this project thankfully as it turns the single + operation into 69 steps (may upload it later), or just 47 with the # and @ being allowed

