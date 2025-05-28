* `i` - inside
* `a` - around
* `c` - same as delete but goes directly into insert mode instead of keeping same model like d
* `I` - begging of line
* `A` - end of line
* `c$` or `C` - delete and replace to end of line
* `s/S` -  Delete char/line and insert
* `D` - same as delete but deletes everything from cursor on
* `!` - exclamation command mode - lets you Filter through external program
* `^`- Start of line (go to non-blank character)
* `=` - formats code
`> / <` - Shift seelected text with indent right/left
* `~` - switch casing
* `U` - uppercase
* `u` - lowercase



* `V` - Linewise visual mode
* `<C-v>` - Visual block mode for vim

* `W` - lets you jump words ingoring spaces
* `B` - lets jump words ignoring spaces

* '{' - lets you switch to previous paragraphs
* '}' -  lets you switch to next paragraphs
* `%` - lets you jump matching symbol
[m	Previous method start
[M	Previous method end


productivity motions:
* `gv` - You may re-select the last selected visual area with .
* `gM` - lets you go to middle of text
* `@@` -	Repeat last macro
* `.` -	Repeat last command
* `gi` - insert at last insert point
* `<C-o>`	Go back to previous cursor location
* `<C-i>`	Go forward to next cursor location
* `gf` -	Go to file in cursor
*`gt/gT`	- Go to next tab / previous tab
* `["x]gp `        2  put the text [from register x] after the

* `m {a-z}` - Marks current position as {a-z}
* `‘ {a-z}` -	Move to position {a-z}
* `ggyG` - Copy a whole document

`\ `` `	Back to position in current buffer where jumped from
`.	Last change in current buffer

do `ga` on a text to get the ascii enocding
`za` / `zA`	Toggle fold (`zr` reduce fold `zm` expend fold, `zo` open fold `zc` close fold) 
`]p`	Paste under the current indentation level
`:set ff=unix`	Convert Windows line endings to Unix line endings




you can do actions (d, y, v,  c) - inside/around(i, a) - text object

so 

text objects are things like 
* `p`	Paragraph
* `w`	Word
* `W`	WORD (surrounded by whitespace)
* `s`	Sentence
* `[ ( { <	A [], (), or {} block`
* `] ) } >	A [], (), or {} block`
* `' " ``	A quoted string
b	A block [(
B	A block in [{
t	A HTML tag block

so like to delete text inside quotes do - `ci"` which is going to into change mode, then inside the quotes, then deleting to quotes
you can also do `ciw` which will go into change mode, then inside, then delete a word
you can also do  `vip` which lets you go into visual mode, then include the paragraph inside the text into selection

you can also do  `vi"` which lets you go into visual mode, then get the inside of a quotes




to increment text - select text, then press `g`, then `<Ctrl + a>`

and g8 to get the utf 8 encoding
`:e` to reopen the buffer

`ggVG`	Select all text
`ggdG`	Delete a complete document
`gg=G`	Indent a complete document


counter - motion - mode

example: 5 press insert, type 123, make a new line by pressing enter(press `<Ctrl-e>` if you cmps), press `esc` to go normal mode
now you should seee 

123
123
123
123
123

`J` - join the line below the current one








`:z` - write and quit



* `<leader>fn` -    New File
* `[q` - Previous Quickfix
* `]q` - Next Quickfix
* `<leader>cf` -    Format
* `]d`    - Next Diagnostic
* `[d`    - Prev Diagnostic
* `<leader>cd`    - Line Diagnostics    
* `<leader><tab>]` -    Next Tab
* `<leader><tab>[`    - Previous Tab
* `<leader>ca` - Code Action
* `<leader>cc` -    Run Codelens
* `<leader>cl`    - Lsp Info    `n`
* default `<localleader>` is `\`



insert mode specials:

- `:help i_CTRL-N` - Cycle down autocomplete
- `:help i_CTRL-P` - Cycle up auto complete
- `:help i_CTRL-E` - closes the auto complete
- `:help i_CTRL-T` - Add tab to line
- `:help i_CTRL-D` - Remove tab from line
to add and remove tabs
- `:help i_CTRL-W` - Delete word at a time
 to remove a word at a time
 Ctrl+x Ctrl+l	Complete line
 
 
 
 Splitting Windows 
 - Split horizontally with `:split` or `CTRL-w s`  
 - vertically with `:vsplit` or `CTRL-w v`.
 - You can close with `:close` or `CTRL-w c`.
 - Change windows with `CTRL-w h`, `CTRL-w j`, `CTRL-w k` or `CTRL-w l`.


Use H and L if the buffer you want to go to is visually close to where you are
Otherwise, if the buffer is open, use <leader>,
For other files, use <leader><space>
Close buffers you no longer need with <leader>bd
<leader>ss to quickly jump to a function in the buffer you're on
<c-o>, <c-i> and gd to navigate the code
You can pin buffers with <leader>bp and delete all non pinned buffers with <leader>bP
Add TODOs in files you want to work on in future but don't need now and delete their buffers, git will track them






"*p | "+p	Paste from system clipboard
"*y | "+y	Paste to system clipboard



`:!pwd`	Execute the pwd unix command, then returns to Vi
`!!pwd`	Execute the pwd unix command and insert output in file
`:sh`	Temporary returns to Unix
`$exit`	Retourns to Vi




Combination	Description
`dd`	Delete current line
`dj`	Delete two lines
`dw`	Delete to next word
`db`	Delete to beginning of word
`dfa`	Delete until a char
`d/hello`	Delete until hello
`cc`	Change current line, synonym with S
`yy`	Copy current line
`>j`	Indent 2 lines





Ranges
`%`	Entire file
`’<,’>`	Current selection
`5`	Line 5
`5,10`	Lines 5 to 10
`$`	Last line
`2,$`	Lines 2 to Last
`.`	Current line
`,3`	Next 3 lines
`-3`,	Forward 3 lines



LazyCheck
Mason
TsUpdate
