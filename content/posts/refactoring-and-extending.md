+++
title = "Refactoring and Extending RPN"
date = 2023-02-25T07:04:56-06:00
toc = true
images = []
tags = ["rust", "rpn", "devlog"]
categories = ["rust"]
projects = ["RPN"]
+++

I decided to work on this a bit more before bed. Quick synopsis of the commands in the current codebase's accepted commands:

| Name    | Description                                               | Implemented? |
|---------|-----------------------------------------------------------|--------------|
| drop    | Deletes the top element from the stack                    | yes          |
| swap    | Exchanges the top and second elements from the stack      | yes          |
| roll{n} | Moves the top element of the stack to the {n}th location  | no           |
| dup     | Duplicates the top element of the stack                   | yes          |
| sqrt    | Square root                                               | yes          |
| store   | Uses top of stack to store the literal below that in heap | no           |
| exec    | Executes a function literal, or by name on stack top      | no           |

There is a lot more to the latest push than that though...

## Breaking Out Numbers from Literals
Numbers (`Float` and `Integer`) being in the same enum as `Function`s was causing some annoying issues, mainly in math operations. I'd have to implement, for example, [`std::ops::Add`](https://doc.rust-lang.org/stable/std/ops/trait.Add.html) in four(!) different variants: `Literal` for `Literal`, `&Literal` for `Literal`, `Literal` for `&Literal`, and `&Literal` for `&Literal`. That's way too much code duplication, and would be much easier with just being able to derive `Copy` for the type. `Function` however contains a `Vec<Token>`, and as `Vec<_>` is a heap allocated type, it cannot derive `Copy`. So the only reasonable fix was to create a new, stacked enum for just number literals. I also moved all mathematic operations into this new module.

Additionally, this made it so I could do these math ops without checking to see if I was trying to add a `Function` first. Neat!

## Moving execution out of `Context`

`Context` was huge. And ugly. And honestly should have nothing to do with execution. So I moved exectuion of builtins into `token::builtin`, and literals into `tokens::literals`. This required passing `Context` into them, but is overall much neater. Builtins now handle their own execution errors, but the prior `ExectionError` type in `context` has been renamed to `ParseError` and can convert an `ExecutionError` into its own type. Importantly, I realized `ExecutionError` must pass back the `Context`, otherwise the entire `Context` would be dropped on error! That said, the executable is responsable for extracting the context from the error. This might need to be improved later.

## TODO:
I think I'm about ready for heap operations now. `store` and `exec` both rely on that, so it makes sense. The only missing function at this point that does rely on that feature is `roll{n}`, and it's mostly out of laziness and lack of overall importance to most of the times I've used RPN that I haven't even attempted to implement it...

## Example working code!
This is the most complex thing I've been able to execute -- correctly solving for the pythagrian theorem:
```
> 3 4 dup * swap dup * + sqrt
 5.0
```
Working through the syntax token by token:
1. 3 is pushed to the stack
2. 4 is pushed to the stack
3. 4 is duplicated
4. The two copies of 4 are multiplied, putting 16 on the stack
5. The 16 and 3 are swapped around
6. 3 is duplicated
7. The two copies of 3 are multiplied, replacing them with 9.
8. 9 and 16 are summed, replacing those two values with 25.
9. The square root of the top of the stack is computed, resulting in 5.0.

It's a fairly simple computation but should hopefully make my point about explicit order of operations in my last post more clear. :3