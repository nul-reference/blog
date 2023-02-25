+++
title = "RPN Calculator"
description = "How did we get here? Where are we going?"
date = "2023-02-24T19:27:31-06:00"
tags = [
    "rust",
    "rpn",
    "devlog"
]
categories = ["rust"]
projects = ["RPN"]
+++

Firstly, let me preface this with [the repository](https://git.sr.ht/~nul/rpn). It builds, it more or less functions, and it isn't the worst code in the world. Refactoring is needed before "release" (assuming I do on crates.io.)

## What is RPN?
For those who haven't seen it before, RPN, or Reverse Polish Notation, is a notation of code that is heavily stack oriented. As an example, to divide 40 by 2, you would _push_ 40, then 2, onto the stack, and finally use the divide operation on the stack. Divide will _pop_ the first two elements from the stack, run its operation, and _push_ the result back onto the stack. The stack is shown between each operation in most RPN calculators that have the screen estate to do so.

### Example
```sh
> 40
 40
> 2
 40
 2
> /
 20
```

## That's too damn verbose...
It's pretty common to allow seperating entries with just a space, and mine does the same thing.

### Example
```sh
> 40 2 /
 20
```

## Other keywords (so far)
| Name    | Description                                               | Implemented? |
|---------|-----------------------------------------------------------|--------------|
| drop    | Deletes the top element from the stack                    | yes          |
| swap    | Exchanges the top and second elements from the stack      | yes          |
| roll{n} | Moves the top element of the stack to the {n}th location  | no           |
| dup     | Duplicates the top element of the stack                   | no           |
| sqrt    | Square root                                               | no           |
| store   | Uses top of stack to store the literal below that in heap | no           |
| exec    | Executes a function literal, or by name on stack top      | no           |

## But what's the point?
The biggest benefit is to make operations explicit. Unlike the standard algebraic (also known as infix) format, in which order of operations is based on (largely arbitrary, don't at me mathematicians) a set of rules we all have to memorize, operations are done in the order stated, from left to right. This makes things much more explicit. Also, the astute among you may realize this is very much how a computer typically operates on a low level, hence its use in one of the first languages, [FORTH](https://en.wikipedia.org/wiki/Forth_(programming_language)).

## Enough lessons, what are you doing?
One of my friends is working on an Lisp interpretor in Rust, to help them learn the language. While trying to help them learn it, I decided to try and make my own interpretor as an example. I don't really know Lisp, but I have a deep love of RPN -- to the point I purchased one of the last HP 50g I could get new when they were discontinued. So I started with a basic calculator. However, that was a very simple thing to write... and I got bored. So I decided to add first class function support, which instantly made it much more complicated.

Functions can be defined and added to the stack using `<`...`>` tags. They implicitly use the stack, so it's required to "know" what the tag does before using it. While the following syntax isn't quite working yet, the intent is the following.

```sh
> <dup * swap dup * + sqrt>
 <dup * swap dup * + sqrt>
> pathag store
> 3 4 pathag
 5
> 
```

## What next?
Mostly, just polish. I don't really have any large asperations. The nice thing is my super ugly hacky main.rs code shows how nice the library API really is.

```rust
use std::io::Write;

fn main() {
    // Create string buffer for user input
    let mut input_buffer = String::new();

    // Create handles to stdin and stdout
    let stdin = std::io::stdin();
    let mut stdout = std::io::stdout();

    // Create new context for the interpreter
    let mut ctx = rpn::Context::default();

    // Start the loop part of Read-Eval-Print-Loop.
    loop {
        // Ensure the input buffer is empty.
        // This is noop in the first loop but critical thereafter.
        input_buffer.drain(..);

        // "Read" portion of the loop.
        print!("> ");
        stdout.flush().unwrap();
        stdin.read_line(&mut input_buffer).unwrap();
        
        // Get rid of trailing newlines, and any other
        // whitespace before and after.
        input_buffer = input_buffer.trim().to_owned();

        // Call the context to Eval the line.
        // This returns the updated stack state.
        let stack = ctx.execute(&input_buffer).unwrap();

        // Print the stack
        for entry in stack.iter() {
            println!(" {entry}");
        }

        // Aaaand loop it, until Ctrl-C breaks program flow.
    }
}
```

This isn't intended to be the "best" way of having a REPL, I just wanted to hack something up quickly to try it out... and it works shockingly well.

## Where next?
I want to get parsing of functions actually working. I haven't even tested it yet but most of the code should be there. I also want to integrate in the heap values I was talking about before. Once again, most of the work is done, it's just "wiring in" the values, and handling more commands. As you can see from the implemented commands, there's quite a few I need to work in there. I want to refactor the code handling that to elsewhere too, as well as offer better interop of floating point and integer types -- as of right now, floating point types and integer cannot be used together mathematically.

In other words, there's a lot of things that need doing, but at the same time, the groundwork is mostly laid out.