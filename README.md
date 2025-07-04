# Rust Dot Env Parser

This repository contains a relatively simple dotenv file parser. This project is being developed for educational purposes and should not be used in production.

## dotenv Files

According to [dotenv.org](dotenv.org), `.env` files were introduced in 2012 and popularized in 2013 as a way for developers to store important environment variables / secrets / keys outside of source control (like Git).

An example file looks like the following:
```env
# example .env file
STRIPE_API_KEY=scr_12345
TWILIO_API_KEY=abcd1234
```
The `.env` syntax consists of:
- a set of keys and values, where
- the keys are assigned to values via the `=` assignment operator, such that
- there should be no spaces between the key, the assignment operator, and the value, and
- each key-value pair is separated by a new line,
- and comments are indicated by a `#` sign and end at the end of a line, and
- if there are spaces, equal signs, or newlines (`' '` or `=` or `\n`) used in the key or value, the entire key or value must be enclosed in quotation marks,
- and other typical escape character rules apply within quotation marked keys and values

# Implementation

This module breaks down the process of reading the `.env` file into two steps: (1) lexing the `.env` file and (2) parsing the lexed content.

When most people say 'parsing', they mean transforming some raw input into a well-defined, often pre-defined, structure--like a struct, array, or other similar thing. Another step in that transformation, though, is lexing, the process of transforming raw input into meaningful chunks. This is also known as tokenizing, although lexing is subtly different. Tokenizing involves breaking raw input into tokens, and lexing does the same but adds some additional content to those tokens.

Once the input is lexed, it can be parsed. This involves making sure that the tokens follow the pre-defined syntax rules of the structure that the content is being parsed into. For example, above I outlined the basic syntax of `.env` files.

In my lexing function, I read the contents into the following tokens, using Rust's enums:
```Rust
enum EnvToken {
    Character(String),
    AssignmentOperator,
    NewLine,
    EOF,
    Comment,
    Whitespace,
}
```

Supposing we had an `.env` file like this:
```env
HELLO=WORLD

```
The array of lexed tokens would look like this:
```
[Character("H"), Character("E"), Character("L"), Character("L"), Character("O"), AssignmentOperator, Character("W"), Character("O"), Character("R"), Character("L"), Character("D"), NewLine, EOF]
```
From there, the parsing function takes in this array and uses the syntax rules to determine if the tokens follow the right rules. The parser ensures that the ordering of the tokens goes something like:
```h
Line Start -> Key
Line Start -> Comment
Line Start -> Line Start
Key -> AssignmentOperator
AssignmentOperator -> Value
Value -> NewLine
Value -> Comment
Value -> Whitespace
Value -> EOF
```
Where any sequence that breaks these rules returns an error.

The following code is an example of use of my module:
```Rust
use std::collections::HashMap;
use std::fs;

fn main() {
    let contents = fs::read_to_string("Test.env").expect("unable to read file");
    let new_env_map: HashMap<String, String> = process_dot_env(contents).expect("unable to parse env file");
    for (k, v) in new_env_map.iter() {
        println!("{} : {}", k, v);
    }
}
```

Only after all that lexing and parsing does the resulting key-value contents get returned to the programmer in the form of a regular hash map.

This repository's `main.rs` file opens, parses, and prints the variables in the `Test.env` file at the directory's root.

## Limitations

The module cannot parse multi-line or quoted keys/values. I may add this to complete the project but my main task of learning more basic Rust concepts was completed.

# Why parse with Rust?

This is the first real parser I've written. I have written regular expressions before, and some rudementary parsing, but nothing that fully tokenized and then parsed content in this way. Despite the fact that I did not choose Rust because I wanted to build a `.env` parser (I chose to write an `.env` parser because I wanted to write a project in Rust), it ended up being the right language for the job.

Rust's enums were an excellent language feature that allowed me to lex the `.env` content. Enums are basically exhaustive lists of possible values of a given type. For example, enums for the state of whether a door is open might be `Open` or `Closed`. Similarly, `.env` files can only have `Keys`, `Values`, or `Comments` (forgetting, for a moment, whitespaces and newline). Rust's enums not only allowed me to turn `a` into the enum `EnvToken::Character` but also attach the value `a` to it (like `EnvToken::Character("a")`)

This is much different than, for example, Go which (1) doesn't have proper enums at all and (2) has no way of attaching values to enums. `a` would have turned into `Character` and I would have needed to use a different strategy for lexing the content.

(Instead, I would have had to lex the content into full keys and full values, rather than characters. Right now, the lexer generates many `Character()` tokens and the parser puts the keys back together from the tokens).

# Conclusion

I enjoyed this project and may try to build other lexers and parsers with Rust down the road. I also have not set up this project as a proper module yet, so I may learn more about the Rust toolchain by publishing it as a crate.

As I am learning, if I've said anything incorrect, please feel free to make an issue of the issue tracker.

### Artificial Intelligence

Although I hand-wrote all of the code, as I am still learning, I used large language models to help me understand some of the syntax of Rust.

### License

All source code in this repository is licensed under the Apache 2.0 License.