![[Pasted image 20260815200446.png]]

`[a-z]` - matches 1 character
`[a-m]` - matches a to m
`[abcd]` - matches a,b,c,d
`[a-e][a-e][a-e][a-e]` - matches a 4 letter word with all letters from a to e only

what if you need to repeat `[a-e]` infinite times?

`[a-e]*` - means repeat / match the pattern `[a-e]` 0 or more times
`[a-e]`+ - match `[a-e]` one or more times 
`[abc]?` - 0/1 times

but these will also match substrings within words. What if you wanted to match just words?

`\b[abcd]\b` - word boundary

ex-

`\b[abcde]\b` - keep matching `[abcde]` until you hit a word boundary (space).

In above example, only 'ace' is matched. 'decade' matches but 'decades' doesn't.

Match numbers:

`[0-9]+` - matches `[0-9]` >=1 times

Caps and more:

`[A-Za-z0-9]` -  match 1 alphanumeric character

Negation:

`[^aeiou]` - match words with no vowels

but in the example, you will basically match all consonants. Use:

`\b[^aeiou]\b`

But you match spaces. To avoid spaces, put `\s` in negation:

`\b[^aeiou\s]\b`

Match all words of length 3:

`\b[a-z][a-z][a-z]\b` 

orr,

`\b[a-z]{3}\b`

Match all words of length 3 or 4:

`\b[a-z]{3,4}\b`

3-5:

`\b[a-z]{3,5}\b`

3 or more:

`\b[a-z]{3,}\b`

But writing `[a-z]`, `[A-Z]` is tedious, lets use `\w` instead:

`\b\w{3,}\b`

`\w` matches letters, numbers and underscores.

Match mulitple words:

`bat|cat|nigga` <-- matches 'bat' and 'cat' and 'nigga' exactly
`(b|c)at` <--- matches 'bat' and 'cat'


Find valid sentences:

valid sentence: starts with capital letter, ends with a .

`[A-Z].*\.` 

. -> means any character, `.*` means any character any number of times.
To end with a period, we can't use . ! as it already means something in regex. use `\.`

### ^ and $

### 1. `^` (Caret) – Start of String

The `^` anchor forces the matching pattern to start at the very first position (index 0) of the string.

- **Pattern:** `^cat`
    
- **Matches:** `"cat in a hat"` (starts with "cat")
    
- **Fails to match:** `"the cat"` (contains "cat", but not at index 0)
    

### 2. `$` (Dollar Sign) – End of String

The `$` anchor forces the matching pattern to occur right before the string terminates.

- **Pattern:** `cat$`
    
- **Matches:** `"the cat"` (ends with "cat")
    
- **Fails to match:** `"cat in a hat"` (contains "cat", but not at the end)
    

### Combining Both: Exact Matching

When both anchors enclose a pattern, the regex requires the entire string to match the pattern exactly from start to finish, with no extra characters allowed before or after.

- **Pattern:** `^cat$`
    
- **Matches ONLY:** `"cat"`
    
- **Fails to match:** `"cat "` (has a trailing space) or `"the cat"`

### Multiline Mode Note (`m` flag)

By default (single-line mode):

- `^` matches the start of the entire input string.
    
- `$` matches the end of the entire input string.
    

If multiline mode (`m`) is enabled:

- `^` matches the start of the string **or** immediately after any newline character (`\n`).
    
- `$` matches the end of the string **or** immediately before any newline character (`\n`).


Negation of `\w` - `\W` (not `\w)

(This is how these backslash negations work, just capitalise)


#### Backslashes:

`\[a-z\]` - what it do?? First it disables `[]` so it tries to match the string "[a-z]" exactly.



![[Pasted image 20260815205007.png]]

Match wavy text:

```
[A-Z]?([a-z]\s?[A-Z]\s?)+[a-z]? <--- this will match all sentences
```
Match dates:

MM-DD-YYYY or MM.DD.YYYY or MM/DD/YYYY

```
[0-1][0-9][\.-/][0-3][0-9][\.-/][0-9]{4} (not bad)
```
```
\d\d-\d\d-(19|20)\d\d (match dates only in last ~100 yrs)
```






