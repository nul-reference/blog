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
series = ["RPN"]
+++

Firstly, let me preface this with [the repository](https://git.sr.ht/~nul/rpn). It builds, it more or less functions, and it isn't the worst code in the world. Refactoring is needed before "release" (assuming I do on crates.io.)

## What is RPN?
For those who haven't seen it before, RPN, or Reverse Polish Notation, is a notation of code that is heavily stack oriented. As an example, to divide 40 by 2, you would _push_ 40, then 2, onto the stack, and finally use the divide operation on the stack. Divide will _pop_ the first two elements from the stack, run its operation, and _push_ the result back onto the stack. The stack is shown between each operation in most RPN calculators that have the screen estate to do so.

### Example
```
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
```
> 40 2 /
 20
```

## Other keywords (so far)
| Name    | Description                                              | Implemented? |
|---------|----------------------------------------------------------|--------------|
| drop    | Deletes the top element from the stack                   | yes          |
| swap    | Exchanges the top and second elements from the stack     | yes          |
| roll{n} | Moves the top element of the stack to the {n}th location | no           |
| dup     | Duplicates the top element of the stack                  | no           |
| sqrt    | Square root                                              | no           |

## But what's the point?
The biggest benefit is to make operations explicit. Unlike the standard algebraic (also known as infix) format, in which order of operations is based on (largely arbitrary, don't at me mathematicians) a set of rules we all have to memorize, operations are done in the order stated, from left to right. This makes things much more explicit. Also, the astute among you may realize this is very much how a computer typically operates on a low level, hence its use in one of the first languages, [FORTH](https://en.wikipedia.org/wiki/Forth_(programming_language)).

## Enough lessons, what are you doing?
One of my friends is working on an Lisp interpretor in Rust, to help them learn the language. While trying to help them learn it, I decided to try and make my own interpretor as an example. I don't really know Lisp, but I have a deep love of RPN -- to the point I purchased one of the last HP 50g I could get new when they were discontinued. So I started with a basic calculator. However, that was a very simple thing to write... and I got bored. So I decided to add first class function support, which instantly made it much more complicated.

Functions can be defined and added to the stack using `<`...`>` tags. They implicitly use the stack, so it's required to "know" what the tag does before using it. While the following syntax isn't quite working yet, the intent is the following.

```
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
    let mut input_buffer = String::new();
    let stdin = std::io::stdin();
    let mut stdout = std::io::stdout();
    let mut ctx = rpn::Context::default();
    loop {
        print!("> ");
        stdout.flush().unwrap();
        input_buffer.drain(..);
        stdin.read_line(&mut input_buffer).unwrap();
        input_buffer = input_buffer.trim().to_owned();
        let stack = ctx.execute(&input_buffer).unwrap();
        for entry in stack.iter() {
            println!(" {entry}");
        }
    }
}
```

This isn't intended to be the "best" way of having a REPL, I just wanted to hack something up quickly to try it out... and it works shockingly well.