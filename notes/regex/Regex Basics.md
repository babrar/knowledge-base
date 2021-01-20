# Regex Basics

The format for using regex is as the following:

```
(pattern) or [pattern]
```

**OR**

```
/pattern/flag  <- This method of passing flags must be supported by the regex engine
```
```
# It might also look like the following, where 'i' is case insensitve flag
string.match("G[a-b].*", "i") 
```

## Pattern Syntax

Patterns are usually written as the following (without any spaces in between and `[]` items being optional):
```
[ZWT](token)[quantifier][ZWT]
```
ZWT's refer to zero-width tokens like `^` and `$`. More on that later.

### Token
A token to match *one* character (i.e. tokens allow the matching of a `char`).
Tokens can be chained by concatenating them without spaces in between to match a string of tokens specifically. Examples given below.
To match a token that is a reserved character in regex, we can escape the character using backslash `\`. However, this is not uniform across all regex engines. Read the docs for your engine.
Some common tokens are:

- `.` - Matches any 1 character.
  ```
  Example: 1. matches 1A or 11
  ```

- `[0-9] (or \d)` - Matches 1 numeric character
  ```
  Example: 
  A[0-9] matches A0 and A1 ... upto A9
  [1-2][0-9] matches all numbers between 10 to 29
  ```

- `[A-Z]` - Matches 1 character from A to Z


- `[a-zA-z]` - Match 1 ASCII character in the range from A to Z and in the range from a to z

### Zero-width Token
They are tokens that match without consuming any characters. I.e. they provide context and does not represent any physical character.

Some common ZWTs are:
- `^` start of line
- `$` end of line
- `\b` word boundary. **NOTE:** Not all engines recognize the backslash character as an escape sequence.

Advanced example:
```
abc(?=123)/ matches the sequence abc only if it is followed by the sequence 123, but it doesn't actually consume the 123
```


### Quantifier (a.k.a repeater)
A specifier for matching a `string` of characters. Tells the token how many times to repeat.

Some common repeaters are:

- `*` 0 or more of the preceding expression
  ```
  .* matches all strings in the universe (including the empty string)
  ^ABC.* matches all strings that start with ABC
  .*ABC$ matches all strings that end with ABC
  ```

- `+` 1 or more of the preceding expression
  ```
  .+ matches all non-empty string in the universe
  [0-9]+ represents all non-empty numbers in the universe
  [0-9]+-[0-9]+ represents all numbers in the format numbers-numbers
  ```





