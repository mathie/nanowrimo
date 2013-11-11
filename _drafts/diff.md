---
layout: post
title: Diff
---
How often do I spend time looking at the output of `diff`? Going by the shell
history on this laptop, which does some duplicate elimination, out of the last
8,800 commands I've typed, 98 of them have been to invoke a diff tool of some
variety (85 of which were `gd` -- my alias for `git diff` -- or `gdc` -- my
alias for `git diff --cached` to perform a final review before committing).
That's not counting the times I use other tools, like vim support for
manipulating diffs, or interactively adding stuff to git's index, visual diff
tools, or time spent looking at pull requests in GitHub.

I think it's safe to say I spend quite a bit of time messing around with diffs.

But you know what, I have absolutely no idea how it works. Well, yes, I can
tell you in general terms, which I will as an introduction shortly, but I
couldn't just code up a version off the top of my head in a couple of hours.

So I thought I should find out.

## What is a "diff"?

At a high level, a "diff" is a mechanism for representing the things that are
different between two different pieces of information. In my case, it's almost
always the different between two different piles of text, and most of the time
the text is structured such that it's heavily line-oriented (i.e. it makes
sense to express changes as being line by line).

"diff"s can be produced to express the differences between arbitrary binary
files, too, but it's difficult to capture *what* changed if you don't have a
deeper semantic understanding of the information represented. So, while it's
very useful for some other reasons outlined below, it's less ... interesting.
To me, at least. :)

Here's an example. I have this file that contains a simple todo list:

> * Research diff algorithms.
>
> * Buy cat food.
>
> * Write my NaNoWriMo words for today.
>
> * Cook dinner.

It gets to the start of tomorrow, and I've had a productive day yesterday. I've
successfully bought cat food and cooked dinner. As always, though, there are
more things to be added:

> * Research diff algorithms.
>
> * Write my NanoWriMo words for yesterday and today.
>
> * Find cat.
>
> * Buy human food.

How would we represent the differences between these two versions of the todo
list? That's where a diff tools comes in. There are a few different ways of
representing the differences, but they're semantically equivalent and I think
we're all agreed that unified diffs are the prettiest, so I won't confuse
things by discussing alternatives. So, a representation of the differences
between the two lists would look like:

{% highlight diff %}
  * Research diff algorithms.
- * Buy cat food.
- * Write my NaNoWriMo words for today.
+ * Write my NaNoWriMo words for yesterday and today.
- * Cook dinner.
+ * Find cat.
+ * Buy human food.
{% endhighlight %}

As you can see, there are a few things being represented here:

* Lines that haven't been changed at all appear just as they were. (These are
  called 'context', and help us understand where in the todo list the change
  happened by anchoring it in something that hasn't changed.)

* Lines that have been removed in the new version are prefixed with a `-`
  (minus) sign.

* Lines that have been added to the new version of the todo list are prefixed
  with a `+` (plus) sign.

* Lines that have had some of their content changed are treated as replacements
  -- the old line is removed and the new one is added.

## But why?

It makes for something that's definitely human readable. I can scan that diff
and understand that I removed two items, added two items and changed one item.

Better still, computer programs can read and interpret this diff format (with a
little extra metadata I've elided for now). This means that I can take the
first version of the file along with the diff, and from that I can produce the
second version of the file. Since the changes between two versions of a (set
of) files is almost always smaller than the new complete set of files, this is
a useful property. (This is how I kept up with Linux kernel development in the
mid 90s, when I was sitting on the slow end of a 9,600 baud modem.)

It's particularly useful for computer geeks to be able to see the difference
between files, and to travel back in time through a project's history. This is
why we use version control systems all the time. Amongst other things, we can:

* Review changes to a system as part of an effective peer review process. This
  means that there's another set of eyeballs looking at the changes you've
  made, making sure they make sense, thinking about things you may or may not
  have considered.

* Recreate historical versions of a system. When you can represent the changes
  between two versions of a system, you can design a repository that will
  efficiently keep track of all the changes to a system. Which means that you
  can recreate the system as it was at any checkpoint. This can be useful for
  reproducing bugs, for example.

* By tracking a little extra information, you start to be able to answer, not
  just questions about "what changed?" but about when it was changed, who
  changed it and, depending on the quality of that extra information, why that
  change was made. This can often be a powerful tool in understanding a system
  but understanding the context in which is was created/changed.

The thing is that, despite relying on this tool very heavily for a couple of
decades now, I have never considered how it works. In general, I like to know
how my tools work, so I figured it was time to learn. This was prompted by an
accidental reading of the git man pages, where it noted that there were several
different algorithms for diff:

* The basic, greedy diff algorithm, also called the Myers algorithm, named
  after Eugene W. Myers.

* The 'minimal' algorithm, which apparently spends a few extra CPU cycles to
  make sure the smallest possible diff is produced.

* The 'patience diff' algorithm.

* The 'histogram' algorithm, which extends the patience algorithm to "support
  low-occurrence common elements".

I'm basically quoting the `git diff` manpage here -- there's very little
information on what these diff algorithms are, and why I might choose one over
another. That's what piqued my interest. How does the basic, greedy, algorithm
work? What does the minimal version do to get a smaller diff? What's the
patience algorithm? And what are low-occurrence common elements?
