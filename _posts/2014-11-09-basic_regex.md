---
title: Basics > Regular Expressions AKA Regex
author: "Sep"
date: "Saturday, November 08, 2014"
output: html_document
layout: post
---


# What is Regex?



Regex is an abbreviation for 'regular expression'.

<div class="message">
Regular expressions are the key to powerful, flexible, and efficient text processing. Regular expressions 
themselves, with a general pattern notation almost like a mini programming language, allow you to describe 
and parse text.
</div>
 
-- From "Mastering Regular Expressions, third edition by Jeffrey E. F. Friedl. ISBN 0596528124.
 
# What you'll find in this post.

Not very much, but an introduction with a few examples using basic regex. 

I'll include R code examples along the way, however regex is so useful that regardless of what computing you do 
the chances are you'll have regex tools in your "native language". A great place to just play around with 
regex is at [www.regexr.com](www.regexr.com) - a 5 star for looks alone from me.

## The cheat sheet.

`[...]` : Match any characters within the brackets. [click.](#sqrbrackets)

` | `   : Alternation, "or" match left or right of the pipe. [click.](#pipe)

`.`     : Match any character (apart from newline \n). [click.](#dot)

`^`     : Carrot [click](#carrot), as a negation [click](#carrotnegate), as a literal carrot [click](#carrotliteral), and as start of string [click.](#carrotstart)

`[A-Za-z0-9]` : Match alphanumeric characters [click.](#alphanumeric)
 
 - `[a-z]`  : lower case alphabetical letters
 - `[A-Z]`  : upper case alphabetical letters
 - `[0-9]`  : the numeric characters
 - Note ranges can be made smaller `[a-h]` for example.
 
`$`     : Dollar is the end of a string. [click.](#dollar)

`*` and `+` : The kleene characters. [click.](#kleene)

 - `*` : match 0 or more of the preceding.
 - `+` : match 1 or more of the preceding.

`?`     : The question mark used to optionally match the preceding.[click.](#qmark)    

`{...}` : quantity to match. [click.](#curlybraces)

 - `{m}` : match m times
 - `{m,}`: match m or more times.
 - `{m,n}` : match between m and n times.



## How to use this page.

If you're looking at regex for the first time look through the page sequencially as it builds up the various basics until you arrive at the "email hunter" regex at the end.

If you just want a little reminder of a particular bit of regex then clicking a link might work out a little better!



## What does regex look like?

Your first exposure to regex:


```r
/^([A-Za-z0-9_\.-]+)@([A-Za-z\.-]+)\.([A-Za-z\.]{2,6})$/
```

I'll admit that it looks like I just started bashing my fist at the keyboard and decided "hey that's regex" - 
but I didn't. That little bit of cryptic typing is a means of extracting email addresses from a text document. By the end of this post
the hope is you'll see why.


# Using regex.

There are two things that make up a regular expression:
    - Literals: Characters that mean exactly what they look like.
	- Metacharacters/Special Characters: Characters that don't have a literal meaning but serve another purpose.


## Matching Literals.

The first thing we can do is look for a literal set of characters in a string. I'm going to pull a piece of text (from 1984 - by Orwell)


```r
text <- "Being in a minority, even in a minority of one, did not make you mad. There was truth and there was untruth, and if you clung to the truth even against the whole world, you were not mad. "

# splitting the string
stext <- unlist(strsplit(text," "))
print(stext)
```

```
##  [1] "Being"     "in"        "a"         "minority," "even"     
##  [6] "in"        "a"         "minority"  "of"        "one,"     
## [11] "did"       "not"       "make"      "you"       "mad."     
## [16] "There"     "was"       "truth"     "and"       "there"    
## [21] "was"       "untruth,"  "and"       "if"        "you"      
## [26] "clung"     "to"        "the"       "truth"     "even"     
## [31] "against"   "the"       "whole"     "world,"    "you"      
## [36] "were"      "not"       "mad."
```

Note that I've split the string so that I have a whole bunch of words to search through.

Lets look for the word "there" and see what we get, here I'm using the 'grep' (Global Regular Expression Print) command in R to look through all the words in `stext`


```r
# The location of "there"
grep(pattern = "there", x = stext)
```

```
## [1] 20
```

```r
stext[grep(pattern = "there", x = stext)]
```

```
## [1] "there"
```

So far so good! 

Notice there where in fact two instances of the word "there" in `stext`. The difference being that one of them has a capital T while the other starts with a lower case t.

<div class="message">
Regex is *case sensitive*.
</div>


## Regexs look inside of words for matches!

Even though out initial regex that we looked a was good enough to be case sensitive, 
regexs aren't "word/space" sensitive. What this means is that a regex will look for a match regardless of if there are other characters around the match or not.

As an example, let's look for the word "or" in `stext`. We know that the word "or" doesn't occur in our text and so you might expect a call to grep to return nothing


```r
grep(pattern = "or", x = stext)
```

```
## [1]  4  8 34
```

Looks like we have some matches! Looking at what these matches are


```r
stext[grep(pattern = "or", x = stext)]
```

```
## [1] "minority," "minority"  "world,"
```


## `[ ]` OR <a id="sqrbrackets"></a>. 

The first special characters that we will look are the square brackets `[ ]`. The square 
brackets of a regular expression behave like an "OR" acting on the objects we place inside them. We can use this to catch both our instances of the word there for example

Example: 

```r
stext[grep(pattern = "[Tt]here", x = stext)]
```

```
## [1] "There" "there"
```

here `[Tt]` means "T" of "t". We could have added as many characters in the square brackets as we liked `[tTzfre]`.

## `|` is an OR <a id="pipe"></a>. 

The `|` pipe serves as an OR metacharacter. for example `a|b` will match an "a" or a "b".

Example: Find the words There or there in the stext

```r
stext[grep(pattern = "There|there", x = stext)]
```

```
## [1] "There" "there"
```


### Using `[]` and `|`.

There isn't anything stopping us from using both the square brackets and the | in a single regular expression:

Example: Find any "There", "there" or "not"

```r
grep(pattern = "[Tt]here|not", x = stext)
```

```
## [1] 12 16 20 37
```

```r
stext[grep(pattern = "[Tt]here|not", x = stext)]
```

```
## [1] "not"   "There" "there" "not"
```


## `.` the dot <a id="dot"></a>.

The `.` is the place holder for any character, not to be confused with the literal fullstop/period (which requires escaping `\.`).

The following regex looks for any object which contains "a character followed by the letter a"

```r
stext[ grep(pattern = ".a", x = stext) ]
```

```
## [1] "make"    "mad."    "was"     "was"     "against" "mad."
```

Which is a little different to searching for an object which contains an a

```r
stext[ grep(pattern = "a", x= stext) ]
```

```
##  [1] "a"       "a"       "make"    "mad."    "was"     "and"     "was"    
##  [8] "and"     "against" "mad."
```

## `^` carrot and three of its uses <a id="carrot"></a> .

The three uses of the carrot symbol are

 1. As the negation operation.
 2. As the literal carrot symbol.
 3. As the "at the start of the line" metacharacter.

### `^` as the negate metacharacter <a id="carrotnegate"></a>.

Using the carrot as the first symbol within a square bracket means "not" for example `m[^a]` is the regex for 'an "m" followed by a character that is not an "a" '. 

As an example,


```r
stext[ grep( pattern = "m[^a]", x = stext) ]
```

```
## [1] "minority," "minority"
```

askes for a word containing an "m" which is followed by a character which is not an "a". The word "minority" pops up as it contains an "m" followed by a letter which is not an a.

Here's another one


```r
stext[ grep( pattern = "a[^g]", x = stext) ]
```

```
## [1] "make"    "mad."    "was"     "and"     "was"     "and"     "against"
## [8] "mad."
```

the query is for words that have within them an "a" followed by a letter that is not a "g". 

Notice: 

- that "a" itself wasn't returned. This is because the regex is asking for an "a" *followed* by a character that is not a "g", since there is no character at all after the "a" with the string "a" the regex doesn't pick it up.

- the only string that does contain an "a" followed by a "g", the string "against", is infact returned! Why? it has another "a" and that "a" is followed by a character that isn't a "g".


### `\^` as the literal carrot <a id="carrotliteral"></a>.

Using the escaped carrot `\^` sigifies the literal carrot character.


```r
# Adding more strings to stext
stext <- c(stext, "m.Y", "   str^anGe", "stri^ng?!")
print(stext)
```

```
##  [1] "Being"       "in"          "a"           "minority,"   "even"       
##  [6] "in"          "a"           "minority"    "of"          "one,"       
## [11] "did"         "not"         "make"        "you"         "mad."       
## [16] "There"       "was"         "truth"       "and"         "there"      
## [21] "was"         "untruth,"    "and"         "if"          "you"        
## [26] "clung"       "to"          "the"         "truth"       "even"       
## [31] "against"     "the"         "whole"       "world,"      "you"        
## [36] "were"        "not"         "mad."        "m.Y"         "   str^anGe"
## [41] "stri^ng?!"
```

Note: due to the string argument being passed to grep in R the escape char needs to be escaped as well (this might be the same in java as well) but that's why you'll see "\ \^" being used here


```r
stext[ grep(pattern = "\\^", x = stext)]
```

```
## [1] "   str^anGe" "stri^ng?!"
```

the metacharacter carrot can be used at the same time too! How about looking for strings which contain the letter "r" followed by a character which isn't "^"


```r
stext[ grep(pattern = "r[^\\^]", x = stext) ]
```

```
##  [1] "minority," "minority"  "There"     "truth"     "there"    
##  [6] "untruth,"  "truth"     "world,"    "were"      "stri^ng?!"
```

### `^` as the "start of the line" meta-character. <a id="carrotstart"></a>.

The carrot used before `[]` signifies find matches "at the start of the line". For example `^[Bm]` is the regex for finding words at the start of the line that start with a "B" or an "m"


```r
stext[ grep(pattern = "^[Bm]", x = stext) ]
```

```
## [1] "Being"     "minority," "minority"  "make"      "mad."      "mad."     
## [7] "m.Y"
```

## Alphanumeric character range searches. <a id="alphanumeric"></a>.

There are a few useful methods when looking for letters of the alphabet or even numbers. To find a captial letter we can of course use `[ABCDEFGHIJKLMNOPQRSTUVWXYZ]` and the same again for lower case letters `[abcdefghijklmnopqrstuvwxyz]`, there is however a shorter expression for the above

### Alphabetical characters.

- The regex `[A-Z]` stands for any captial letter.
- The regex `[a-z]` stands for any lowercase letter.
- The regex `[A-Za-z]` stands for any upper or lower case letter.

Note: The range can be reduced; for example `[a-g]` searches for a lower case letter from "a" to "g".

Example: Starting with a capital letter

```r
stext[ grep(pattern="^[A-Z]", x= stext) ]
```

```
## [1] "Being" "There"
```

Example: Starting with a lower case letter

```r
stext[ grep(pattern = "^[a-z]", x = stext) ]
```

```
##  [1] "in"        "a"         "minority," "even"      "in"       
##  [6] "a"         "minority"  "of"        "one,"      "did"      
## [11] "not"       "make"      "you"       "mad."      "was"      
## [16] "truth"     "and"       "there"     "was"       "untruth," 
## [21] "and"       "if"        "you"       "clung"     "to"       
## [26] "the"       "truth"     "even"      "against"   "the"      
## [31] "whole"     "world,"    "you"       "were"      "not"      
## [36] "mad."      "m.Y"       "stri^ng?!"
```

Example: Not starting with a letter

```r
stext[ grep(pattern = "^[^A-Za-z]", x = stext) ]
```

```
## [1] "   str^anGe"
```

### Numeric characters.

- The regex `[0-9]` looks for any number character from 0 to 9.

Example:

```r
numtext <- c("ONE", "2", "tHr33", "44", "the answer is 42", ":)")
print(numtext)
```

```
## [1] "ONE"              "2"                "tHr33"           
## [4] "44"               "the answer is 42" ":)"
```

Find strings which containg lowercase letters or numbers.

```r
numtext[ grep(pattern = "[a-z0-9]", numtext)]
```

```
## [1] "2"                "tHr33"            "44"              
## [4] "the answer is 42"
```


## `$` and characters at the end of a line <a id="dollar"></a>.

The dollar metacharacter `$` signifies matches at the end of a line.

Example:  Find all objects that end in a non letter character

```r
stext[ grep(pattern = "[^A-Za-z]$", x= stext) ]
```

```
## [1] "minority," "one,"      "mad."      "untruth,"  "world,"    "mad."     
## [7] "stri^ng?!"
```


## The Kleene `*` and the Kleene `+` <a id="kleene"></a>.

Both `*` and `+` are quantifiers. They indicate the number of occurances the characters preceding them should be examined.

- `*` signifies "zero or more instances of the preceding".
- `+` signifies "one or more instances of the preceding".

Example: ooooooh!


```r
ohtext <- c("oh", "h!", "oh!", "ooh!", "oooooh!")
ohtext
```

```
## [1] "oh"      "h!"      "oh!"     "ooh!"    "oooooh!"
```

The `+` looks for at least one instance of

```r
ohtext[ grep(pattern = "(o+)h!", x = ohtext)]
```

```
## [1] "oh!"     "ooh!"    "oooooh!"
```

The `*` looks for zero or more instances

```r
ohtext[ grep(pattern = "(o*)h!", x = ohtext)]
```

```
## [1] "h!"      "oh!"     "ooh!"    "oooooh!"
```

Looking for the "oh"s

```r
ohtext[ grep(pattern = "(o+)h(!*)", x = ohtext)]
```

```
## [1] "oh"      "oh!"     "ooh!"    "oooooh!"
```

## `?` signifies optional <a id="qmark"></a>.

The character group before the ? are treated as optional for example


```r
coltext <- c("Graham", "colour","Hunter S Thompson", "Hunter s. Thompson", "color",  "Hunter Thompson", "Graeme", "Grahame" )
coltext
```

```
## [1] "Graham"             "colour"             "Hunter S Thompson" 
## [4] "Hunter s. Thompson" "color"              "Hunter Thompson"   
## [7] "Graeme"             "Grahame"
```


Example: UK and US spelling of colo(u?)r

```r
coltext[ grep( pattern="colo(u?)r", x = coltext) ]
```

```
## [1] "colour" "color"
```

Example: Leaving out a middle initial

```r
coltext[ grep( pattern = "Hunter( [Ss]\\.*)? Thompson", x=coltext) ]
```

```
## [1] "Hunter S Thompson"  "Hunter s. Thompson" "Hunter Thompson"
```

Example: Multiple ways of spelling a name

```r
coltext[ grep(pattern = "Gra[eh](a?)m(e?)", x=coltext)]
```

```
## [1] "Graham"  "Graeme"  "Grahame"
```

## `{m,n}` repetition <a id="curlybraces"></a>.

The curly braces give control over the number of matches of the characters preceding the braces we are looking for. For example:

- `a{m}` signifies look for exactly m instances of the character "a" in a row.
- `a{m,}` signifies look for at least m instances of the character "a" in a row.
- `a{m,n}` signifies look for at least m and no more than n instanes of "a" in a row.


```r
ohtext
```

```
## [1] "oh"      "h!"      "oh!"     "ooh!"    "oooooh!"
```


Example: Match terms with between 1 and 2 "o"s at the start followed by a "h"

```r
ohtext[ grep(pattern = "^o{1,2}h", x = ohtext)]
```

```
## [1] "oh"   "oh!"  "ooh!"
```

## A final example, the email hunter.

As a final example we can breakdown the "email" finder regex which was quoted at the start of this post


```r
^([A-Za-z0-9_\.-]+)@([A-Za-z\.-]+)\.([A-Za-z\.]{2,6})$
```

1. `^([A-Za-z0-9_\.-]+)`. The `^` tells us that we are looking a the start of the line, the brackets give us the grouping. `[A-Za-z0-9_\.-]` This regex tells us we are looking for any letter (upper or lower case), any number, underscore, period or dash. Having the `+` metacharacter means we are looking for *at least one apearance of the characters. Thus in plain english we can say `^([A-Za-z0-9_\.-]+)` - Look for one of more letter, number, underscore, dash or period at the start of the line.


2. Next we have `@` which is literally look for an "@".

3. `([A-Za-z\.-]+)`. Looking for at leas one letter, dash or period.

4. `\.` Look for the literal period ".".

5. `([A-Za-z\.]{2,6})$`. The `$` tells us that whatever is in the brackets before it should be located at the end of the line. `[A-Za-z\.]` Looking for any combination of letters and periods that are repeated between `{2,6}` two and six times.




