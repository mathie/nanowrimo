---
layout: post
title: Learning through refactoring
excerpt: |
  FIXME:
---
I enjoy learning new programming languages. There are always new things to
learn from a different perspective of looking at a problem. Some languages suit
particular problems better than others. Every language is a trade off, whether
it's between programmer productivity and raw performance, or formally provable
correctness vs flexibility, or a dozen other conflicting goals. There is no
'perfect' programming language, no perfect deployment platform, not even a
single programming paradigm that's best.

## Why Learn a new language?

It's possible to be as good as you can possibly be in a single programming
language. And that's a laudible goal: in order to be proficient at doing what
you want to achieve, it's good to have depth of knowledge and experience in
your tools. However, if you only know one language, even if you are as adept as
it's possible to be with that language, you still lack different skills and
expertise acquired from learning other languages. In social sciences, I believe
this is called a local optima versus a global optima. (I've seen it used to
argue that people always perform better in teams rather than working
independently. I believe the same thing applies to breadth of knowledge.)

I've labelled myself as a Rubyist for the past 8 years (is it really that
long?!). In principle, that's been a commercial/marketing decision in order to
develop a reputation in what I saw as a new market niche. In reality, I started
out believing the magic that Rails provided, struggling for the first 6 months
with Ruby itself, and still writing all the stuff I needed to get done quickly
(scripts, hacks, demos) in Python.

Through my career (which hasn't been *that* long), I've delivered code to
customers in C, C++, Java, Javascript, Ruby, Python, Perl and Go. And I've
developed chunks of code (for personal use or for school/University courses) in
BASIC, Comal, COBOL, Pascal, Visual Basic, Delphi, Scheme, Emacs Lisp, ML, and
Tcl. I'm not really counting all the shell scripting stuff like sh, csh, sed
and awk, or Domain Specific Languages like Chef and Puppet, or, for that
matter, the Domain Specific Lanuages that comprise various configuration file
syntaxes (yes, I was proficient at writing Sendmail rules).

I'm really keen to learn something new, too. Lately, I've been toying with
Clojure, Erlang, Go, Haskell and Scala.

So much for being a Rubyist, eh?

## The Dreyfus model of learning a language

Let's take it as a given that learning new programming languages is a good
thing. So, how do you learn a new language? In my case, so far, it's largely
been brute force and ignorance. I've had a problem I needed to solve (either a
commercial or an academic one), I've mandated a programming language
(either through my own choice, or based on external decisions) and I've had to
build something, most often to a deadline.

Wash, rinse, repeat.

How do the stages of this approach look? For me, it's been:

* Like wading through glue. I have to look up bits of the syntax, and I have to
  look up most of the standard library. It takes several goes to even make code
  get to runtime, fixing up silly errors, scratching my head to try and make it
  even compile. Any time I want to get something done quickly, I switch
  straight back to a more familiar programming language, get it done and think,
  "I'll port it across later."

* Transliteration. I've gotten reasonably good at writing code that will
  compile and run, but I'm basically writing code using the idioms of a more
  familiar programming language. When you're reading code as a developer more
  seasoned in a language, you'll spot this all the time. How many times have
  you seen code that might run in a Ruby interpreter, but it's clearly written
  by a Java developer? (Or whatever your equivalent area of expertise currently
  is.)

  At this point, while starting to get reasonably productive, the language
  still feels like a pain, because it still doesn't feel natural. And what
  you're trying to do with the language *isn't* natural, because you're working
  at cross purposes to it. Trying to write imperative code in a functional
  language, for example. It's usually possible (most functional languages wuss
  out on trying to implement side-effects and state in a pure functional
  manner), but it's really painful to write.

  And I'm still writing scripts, hacks and side projects in a more familiar
  language, just because sometimes I need to get stuff done and sometimes I
  need to remember that code isn't all pain.

* Cargo culting idioms. Finally, the natural flow of the language is starting
  to emerge, but only because I'm taking patterns that other people have used,
  and applying those patterns to my own code. I'm proficient and productive,
  and I've even started using the language for my own side projects.

  I'm happy using third party libraries for implementing functionality, but I
  couldn't really write my own libraries, and I don't always (want to) know
  what's going on inside them. I'm not particularly discriminatory about the
  libraries I choose to use -- I don't evaluate their code quality or
  correctness, because I'm not yet sure what constitutes good code.

* Proficient in using the programming language's natural idioms and applying
  them correctly. I'm able to design and build software which takes advantage
  of the language. I don't rely on the structure of existing third party
  libraries. I can take generic patterns from programming books (e.g. the Gang
  of Four Patterns book) and implement them idiomatically in the language.

  At this point, I'm able to take most problems and carve out a half decent
  solution. It's become my familiar language, and the one I default to for all
  the scripts, hacks and side projects I write, in addition to the bits where
  it's proscribed.

* And then, finally, I get frustrated about what's missing in the language,
  what makes it a poor fit for particular problems, where I'm conscious that my
  tools aren't the right solution to the problem. And I start shopping around
  for something new.

I think there's some alignment between these stages and the Dreyfus model,
which may well be why I split it out into five stages too. :) Our beginner is
wading through glue. The advanced beginner is transliterating their more
familiar mental models onto the new solution domain. The proficient developer
is able to cargo cult idioms and write new code from them. When they reach the
advanced stage, they're natural at writing code in the language. And, finally,
the expert can see the language's shortcomings, where it's not a good fit.

## Idiomatic programming

I think it's probably clear by now that I stronly believe in idiomatic use of a
programming language. But what do I mean by that? The Apple dictionary
helpfully defines 'idiom' as:

> 1. a group of words established by usage as having a meaning not deducible
>    from those of the individual words (e.g. *over the moon*, *see the light*).
>
>    * [ mass noun ] a form of expression natural to a language, person, or
>      group of people: *he had a feeling for phrase and idiom*.
>
>    *  the dialect of a people or part of a country.
>
> 2. a characteristic mode of expression in music or art: *they were both
>    working in a neo-Impressionist idiom*.

In programming terms, to me it means using the language in the natural way it
was designed to be used, and in the way that the language developers intended
you to use it.

In its simplest form, it's about following the coding style of the language.
Are method names `CamelCased` or `snake_cased`? How do you express constants?
Is there a particular sigil for global, local or instance variables?

Much as there's plenty of conflict and argument about formatting code, there is
just one guideline you need to follow: Follow the lead of the existing
community. If the community prefixes all their instance variables with `_i` or
uses reverse Polish notation to communicate the type of a variable, your code
should, too. If you think that's a stupid decision, you probably ought to
suspect some of the other decisions about the core language that the community
have settled on.

I love how the Go community have solved this problem, right at the start, once
and for all. The way you should format your code is the way that `go fmt`
formats your code. End of story, no argument. I've got this as a save hook in
my text editor, so every time I hit save, I know it's formatted exactly as the
community demands, even if it's not *entirely* to my tastes.

But there's a lot more to idiomatic programming than that. A gross way of
looking at the idioms of a language is to look at the overall paradigm the
programming language aligns with. Languages will typically support writing code
in multiple idioms (most languages will let you write 'structured' programs --
sequences of code organised into procedures), but they'll align more strongly
with one in particular. For example, while you can write plain structured
programs in Ruby, and it does support functional expression well, it's
predominantly an object-oriented language. So what sort of paradigms do we find
in mainstream languages?

* **Structured** programming, where you have sequential sets of instructions
  (including conditions and loops), but instead of having a single sequence of
  these instructions, you break them down into named procedures. This allow a
  form of higher level abstraction, because instead of spelling out in assembly
  language how one makes particular bits of phosphor glow on the screen, one
  can call the `print` method. This is characteristic of 'high level'
  programming languages of the 80s when I started progrmaming, like C, BASIC
  and Pascal.

* **Object-oriented** programming, where in addition to organising the sets of
  instructions into functions, we can also group together related functions --
  with the data they manage -- into a single object. This allows for much
  higher level abstraction, and is a powerful way to be able to express
  software in terms of the problem domain. It promotes good structure for
  large, complex programs. This is the most popular paradigm currently, with
  Smalltalk, Java, C++, Ruby and Python all aligning themselves in this camp.

* **Functional** programming has more of a mathematical background. In its
  purest form, you express an entire program as a function `f(x)`. The result
  of the program is the result of calling `f` with a value for `x`. There are
  no side effects, so `f()` is idempotent -- every time you call it with the
  same value, you'll get the same result, and the world will be in the same
  state afterwards. This makes it relatively easy to reason about programs and
  to prove their correctness. Since the functions called aren't side effecting,
  it's also really easy to reason about when it's safe to run code
  concurrently.

  This is a popular paradigm in the academic community, but (except in specific
  niches), it hasn't yet found a home in the mainstream commercial world. It
  includes languages like Erlang, Haskell, ML and Lisp.

* **Logic** programming (I think that's the name for it). It's a little more
  esoteric. Instead of expressing a sequence of instructions, of defining a
  particular function, you define rules. Things that you know to be logically
  true. The rules engine can then take that information and make fresh
  inferences for new statements. For the problems it suits, it really suits
  them. The only language I've used in this style is Prolog.

From that overview, functional programming sounds like a no brainer. Why would
you use anything else? The trouble is that the world is full of side effects.
You want to print something to the screen? While in principle it's possible to
reason about the output on the screen as being a function of the input, in
reality, changing the state of those LEDs to 'on' is changing the state of the
world -- so creating a side effect.

There are two ways that most functional languages cope with this. One is to
implement some sort of structured, side-effecting part to the language, where
it's possible to express the messy bits of reality. The other is monads which
are, roughly speaking, a mechanism by which programmers confuse each other
through analogy, simile and metaphors. They're like bad poetry.

The most interesting programming languages, in my personal opinion, are the
ones that combine functional and object-oriented paradigms. I attribute the
phrase, "Object oriented in the large, functional in the small" to Michael
Feathers -- I hope that's right! -- and that's exactly what I've been aspiring
to in the past few years. Object orientation is a great way to structure the
high level of a program, so that it's easy to talk about and easy to express
high level business concepts. But it's a damn site easier to test functional
code, where there are no side effects (nothing unrelated that needs mocking
out), and the code under test does an entirely predictable, consistent thing.
