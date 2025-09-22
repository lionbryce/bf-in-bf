Apologies if this readme is a mess, I wrote it write before bed a few days ago and have not reviewed it since.
The actual program is reasonably commented (at least well enough that I could look at it a week later and diagnose a bug)

## Background

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

## On to the interpreter:
Thankfully, all bf operations are 1 character (some argument in regards to [ and ] being part of the same operation making it 2 characters),
so all you need to do is take in a character, see if it's any of the 8 valid operations, and repeat.

### Data structure

For this explanation I will be referring to multiple, interlaced rows. The structure being:
1. input
2. input pointer track / one row
3. zero row
4. zero row
5. memory
6. memory pointer track
7. loop track        [ if there is a spot to return to
8. loop/skip ahead byte

### reading and interpreting characters

Every character in stdin is read 1 character at a time and slotted into the last available "input" slot (row 1). At face value this is as easy as `,[>,]` where you read a byte, move over, and just keep reading until you run out. I'm sure it's possible to just do it that way however I ran into some mental blocks when figuring out how that'd play with reading and looping (ie: , and []).
Therefore, we now have a row of just 1s to indicate where the end of the input is (much like the stack implementation).

After a character is read into input (or whenever we get back to it if we're in a loop), we need to see if it's one of the 8 valid operations (+-><[].,).
We can only compare if a value is or isn't 0, so the only way I figured out that worked was to just subtract the various ascii values (ie: `+=43; ,=44; -=45; .=46; [=91; ]=93`)
and see if we ended up at 0 or not. We end up doing so by subtracted the value, jumping up a row if we're at a nonzero value and then jumping up 1 more row regardless (`[#]#` or `[>]>`).
At this point we'll either be in row 2 if it the value matched (ie: subtracting the compared value made it 0) or row 4 if it didn't. Row 4 is all 0s by as the structure suggests and Row 2 is a chain of 1s followed by all 0s. Strategically, the value of row 2 is always 1 when we switch to it from row 1.

After all of that we now have a way to jump into a loop if a value matches or not. Why a loop? It's the only form of flow control we have. Convert a loop into an if statement is reasonably easy though, you just need to make sure it only runs once and reallign the pointer with where it'd be prior to the if statement running. In this case, we can just go back to the end of the input by following the chain of ones (`[>]<`) and jumping back up into row 4 prior to the loop ending.

For various reasons, we read prior to the loop starting (loops need a nonzero value to start), and at the very end. This allows it to immediately stop if you don't put any input in and allows us to do some skipping as needed when we get to implementing `[` and `]`

### the calm before the storm

To keep the next instructions short: the memory pointer track works the same way as the input pointer track except it's adjusted with `>` and `<`. The memory row is updated/read by looking at the end of the memory track row and doing the appropriate action (either `+`,`-`, or `,` to read and `.` to output, as expected). After we finish checking all 8 operations we need to add 93 to the input that we're on to undo all of the subtractions (93 being the highest value that we end up subtracting)

### looping

With how easy setting up the 1st 6 instructions `+-><.,` was at this point, I figured I had gotten a majority of the work done, but then I remembered how painful setting up the loops was likely to be.
Eventually I narrowed down that we'd need both a track (which was reasonably obvious) and a single byte to say that we're looping ahead. The track is set whenever we get to a `[` if the current value (found by looking at the end of the memory pointer track and reading the same cell in the memory row) is nonzero. All we have to end up doing is starting a loop, go to the end of the input, and work our way backwards incrementing the loop track until we hit the start of the input (ie: cell 1, though we can start on either end in theory, I may have ended up doing it the other way, actually...).

Unlike the other tracks, this one actually uses values greater than 1. In this implementation, you can have up to 255 subloops. I can think of 2 ways I'd go about making that either 256^n or even unlimited (memory allowing) but I don't think that'll be necessary (not that any of this really was, I did it for fun).

Regardless of if the current value is nonzero, whenever we hit a `]` we subtract 1 from every value in the loop track. If the current value was nonzero we need to also update the input track by subtracting every value after the current cell from it. If the current value was 0 then we just leave it alone.

Only other thing to handle is the `[` which we do with "row" 8. Why the ""? Well, it's only 1 byte (from here out: "the skip byte"), in theory you could optimize this by moving it to cell -1 or something along those lines. I didn't have the energy to do that, but it would shorten up all the >s and <s by 1 (from 8 to 7) giving a pretty substantial speed boost. If we end up on `[` and the current value is 0 then we need to increment the skip byte. When we get to the end of the main program loop, we check if the byte is nonzero and we start a much smaller version of this interpreter where we only care about `[`, `]`, and `,` to make sure that we stop skipping as needed (the `,` as we read from stdin while this program is also ran from stdin hence we don't read the next value)

Since we're moving the input pointer we also need to make sure that we add/subtract 93 as needed to make sure values all end up the same (ie: add 93 to the old pointer spot and subtract 93 from the new one)

## TODO

Check a couple other edge cases
