---
title: Dune build failure trying to make an OCaml library
created: 2023-04-01
---

# RubberDuckGPT
After asking a couple dozen questions to ChatGPT this weekend while trying to work through various issues, I have noticed a few things of interest. These aren't new insights, but writing out thoughts is helpful in many ways (more on that below), so here are mine.

ChatGPT gives answers to questions that are often surprisingly close to the right answer, but not usually EXACTLY right. Part of this is likely a learning curve on how to properly formulate thoughts in general, as well as how to do so for an AI chatbot in particular. Another part is that the technology is still evolving.

That's pretty astounding from a technological perspective, and also mildly disappointing for figuring out what's wrong. While the answers ChatGPT has been providing were almost never completely correct, they almost always gave me a thought on what to check out next.

# Enter: [rubber duck debugging](https://en.wikipedia.org/wiki/Rubber_duck_debugging). 
So, you have a problem.

It could be a bug in some code, difficulty figuring out the right way to approach a problem, general life issues, pretty much any problem will do.

The act of organizing thoughts about that problem into actual words can help your brain think about things in a new way. Saying those thoughts out loud, or in this case typing them into a chat window, can also help trigger ideas for other possible solutions. Eventually one of those ideas will often lead to a viable solution to your problem.

That's roughly the basis of rubber duck debugging as I understand it.

# Supercharging the duck
What happens when the rubber duck can respond to you? Historically that would mean you're talking to another person, and a massive step up from the 1-sided conversation that comes with talking to an inanimate object.

Today that can also mean talking to a trained machine learning model, which is often being referred to as "AI". I grew up with the idea that AI included some kind of consciousness, and find it difficult to accept machine learning as actual AI. But conciousness is a surprisingly hard concept to define, so maybe this is the early stages of it after all.

Regardless of how we define and classify tools like ChatGPT, their potential impact is undeniably **massive**.

Much like conversing **with** another person, chatting **with** an AI adds an extra layer of thought-provocation that goes above and beyond talking **at** an object. There are trade-offs between people and AI in this regard. Some reasons an AI might be preferable in certain cases include:
* They do not need to sleep or rest like people do - making them more or less always available.
* They do not feel emotions (yet?) - which can be helpful for those of us who worry too much about being an annoyance to others, among other things.

# TL;DR
What does all this mean, basically?

It means that when you don't want to (or can't) talk to another person about your problem, you can talk to a tool like ChatGPT.

You'll need to think quite a bit about the information it gives you, after also thinking about how to put the question or problem you have into words. And all that thinking is great for problem solving, even if the information given isn't quite right.

# Example: OCaml type sadness
I usually work in ReasonML, which is (oversimplified) an OCaml syntax that can be used to write type-safe JavaScript. I have been dipping my toes more into the OCaml pool, in order to get a better feel for the overall ecosystem and a stronger understanding of how the many tools within it interact.

This weekend that has included making an OCaml project from scratch. This project is a sandbox to learn, and focuses on coding challenges like Advent of Code (AoC). I started into the 2022 AoC challenge in ReasonML last year before getting sidetracked and stopping in the middle of Day 9.

Porting over the ways I solved the early problems into OCaml has already been hugely helpful for getting a better feel of some of the basics. One issue along the way was trying to figure out how to take a `list` of `option` values and convert it to a `list` of values that were wrapped in `Some`, while dropping any `None` values that were present.

My initial approach was pretty straightforward:

```ocaml
let keep_somes_ditch_nones lst =
  lst
  |> List.map Option.to_list
  |> List.flatten
```

This takes the list of options and maps over each option via `List.map`. The mapping will convert each option into a list. Any `Some` value is returned in a new list, while any `None` value produces an empty list. Since that all happens inside a `map` function the result is a list of lists of values. Here's an example of what that looks like:

```ocaml
# [Some 42; None; Some 5; None] |> List.map Option.to_list;;
- : int list list = [[42]; []; [5]; []]
```

The input is a list of options of integers: `[Some 42; None; Some 5; None]`.

That input is then piped into `List.map Option.to_list`, as described above.

The result is now a list of lists of integers: `[[42]; []; [5]; []]`.

A `list of lists of integers` isn't what's wanted in this particular case though. The goal here is to get a `list of integers`. The `List.flatten` function will take care of this nicely:

```ocaml
# [[42]; []; [5]; []] |> List.flatten;;
- : int list = [42; 5]
```

Great! Now the `list of integers` can be passed on to the next part of the program.

# Making it point-free (tacit programming)
Sometimes in functional programming it can be nice to [express functions without their input arguments](https://en.wikipedia.org/wiki/Tacit_programming). This is known as `point-free style` or `tacit programming`.

This can, in some cases, be as simple as dropping the arguments to the function and switching from using pipes (`|>` in OCaml/ReasonML) to function composition (sometimes expressed as `>>`).

A `pipe` is used to "pipe" data through functions. Here's an example:

```ocaml
(* point-free *)
let add_1 = (+) 1

(* with function parameters *)
let greater_than_5 x = x > 5

(* piping data through functions *)
[8; 6; 7; 5; 3; 0; 9]
|> List.map add_1
|> List.filter greater_than_5
|> List.sort Int.compare
```

Running the above results in `[6; 7; 8; 9; 10]`. The original list had 1 added to each value in it, any values less than or equal to 5 are then filtered out, and everything is then sorted.

Function composition (`>>`) can be thought of as gluing functions together in preparation to be used on some data later. This is a common way to combine small reusable chunks of logic/work into larger and more specialized chunks.

Here's what the above code could also look like:

```ocaml
(* point-free *)
let add_1 = (+) 1

(* with function parameters *)
let greater_than_5 x = x > 5

(* reusable logic chunks glued together *)
let add_filter_sort =
  List.map add_1
  >> List.filter greater_than_5
  >> List.sort Int.compare

(* using the larger chunk *)
[8; 6; 7; 5; 3; 0; 9]
|> add_filter_sort
```

This yields the same result: `[6; 7; 8; 9; 10]`

# Point-free style on the `keep_somes_ditch_nones` function
At first glance it seemed logical that this:

```ocaml
let keep_somes_ditch_nones lst =
  lst
  |> List.map Option.to_list
  |> List.flatten
```

could be replaced with this:

```ocaml
let keep_somes_ditch_nones_bad =
  List.map Option.to_list
  >> List.flatten
```

But that results in an interesting error instead:

```
File "src/utils/lst.ml", line 50, characters 4-30:
50 | let keep_somes_ditch_nones_bad = List.map Option.to_list >> List.flatten
         ^^^^^^^^^^^^^^^^^^^^^^^^^^
Error: The type of this expression, '_weak1 option list -> '_weak1 list,
       contains type variables that cannot be generalized
```

After a fair bit of googling and [reading](https://stackoverflow.com/questions/32268203/ocaml-contains-type-variables-that-cannot-be-generalized) about [weak types](https://caml.inria.fr/pub/old_caml_site/FAQ/FAQ_EXPERT-eng.html#variables_de_types_faibles) in OCaml I wasn't feeling any closer to understanding the problem.

# Asking ChatGPT for help
So I asked ChatGPT this:

```
Why does this function work in OCaml:

let keep_somes_ditch_nones lst = lst |> List.map Option.to_list |> List.flatten

But this one doesn't?

let keep_somes_ditch_nones_bad = List.map Option.to_list >> List.flatten
```

And it replied with:

```
The reason why keep_somes_ditch_nones_bad doesn't work is due to the order of operations when using the forward composition operator >>.

In OCaml, the >> operator is used to compose two functions, where the output of the first function is passed as the input to the second function. In the case of keep_somes_ditch_nones_bad, List.map Option.to_list is the first function, which takes a list of option values and returns a list of list values. The second function is List.flatten, which takes a list of list values and flattens it into a single list.

The problem with this composition is that the output of the first function is a list of list values, not a list of values that can be flattened. Therefore, when we try to apply List.flatten to the output of List.map Option.to_list, we get a type error, since List.flatten expects a list of list values, not a list of option values.

On the other hand, in keep_somes_ditch_nones, the |> operator is used to pipe the input list lst into List.map Option.to_list, which returns a list of list values. Then, the result is piped into List.flatten, which takes a list of list values and flattens it into a single list. The order of operations in this function is correct, which is why it works as expected.
```

It's not quite correct in it's assessment of the issue. While the first couple paragraphs seem mostly right, this part seems blatantly wrong:

```
The problem with this composition is that the output of the first function is a list of list values, not a list of values that can be flattened. Therefore, when we try to apply List.flatten to the output of List.map Option.to_list, we get a type error, since List.flatten expects a list of list values, not a list of option values.
```

Calling `List.map Option.to_list` on a `list of options` does indeed return a `list of lists`, but we **can** then use `List.flatten` to flatten that `list of lists`. The non-point-free version of the function does exactly that!

# Eureka
The first sentence of ChatGPT's reply ended up being the key to realizing what was wrong:

```
The reason why keep_somes_ditch_nones_bad doesn't work is due to the order of operations when using the forward composition operator >>.
```
The thought process went something like this:

Order of operations? I should try writing out the equivalent code without the `>>` operator:

```ocaml
let keep_somes_ditch_nones_bad =
  List.flatten (List.map Option.to_list)
```

All `>>` is really doing is reordering things after all. So what about trying to add the parameter back in now? Well that would look like this:

```ocaml
let keep_somes_ditch_nones_bad lst =
  List.flatten (List.map Option.to_list) lst
```

Well that doesn't seem right! I wonder what the original function would look like without its pipe operators? Like this:

```ocaml
let keep_somes_ditch_nones lst =
  List.flatten (List.map Option.to_list lst)
```

Oh, I see now! Trying to compose the functions together in this particular case is going to be more trouble than it is worth.

# Conclusion
Keen observers who are also familiar with this style of programming probably wonder why I didn't try re-writing things in the above manner sooner.

Simply put: I was stuck focusing on the wrong parts of the problem.

We've all been there, it happens, especially when looking at a problem while tired and late in the evening.

And while the OCaml discord server (or StackOverflow or whatever else) is full of very helpful folks, this case is a great example of a situation where a tool like ChatGPT could serve as the perfect supercharged rubber duck to bounce ideas off of.