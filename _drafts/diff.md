---
layout: post
title: Diff
---
How often do I spend time looking at the output of `diff`? Going by the shell
history on this laptop, which does some duplicate elimination, out of the last
8,800 commands I've typed, 98 of them have been to invoke a diff tool of some
variety. These broke down into three main divisions:

* `gd` which is my shortcut for `git diff`. This is most often when I'm
  reviewing in-progress code to see what I've changed so far, sometimes as a
  reminder of what I've still got left to do.

* `gdc` which is my short cut for `git diff --cached`. This is me reviewing the
  changes that I've staged in git's index, almost always as a final code review
  before committing it.

* Others, like looking at the differences between branches, tags, individual
  commits, or using `diff` entirely outside the scope of git.

The majority were associated with git, since that's the version control system
I put most of my life into.  That's not counting the times I use other tools,
like vim support for manipulating diffs, or interactively adding stuff to git's
index, visual diff tools, or time spent looking at pull requests in GitHub.

I think it's safe to say I spend quite a bit of time messing around with diffs.

But you know what, I have absolutely no idea how it works. Well, yes, I can
tell you in general terms, which I will as an introduction shortly, but I
couldn't just code up a version off the top of my head in a couple of hours.

So I thought I should find out.

## What is a "diff"?

At a high level, a "diff" is a mechanism for representing the things that are
different between two different pieces of information. In my case, it's almost
always the changes between two different piles of text, and most of the time
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
  by understanding the context in which is was created/changed.

The thing is that, despite relying on this tool very heavily for nearly two
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

(Of course, in the meantime I've selected the 'histogram' algorithm, since it
sounds like the one with all the bells and whistles. It's not like my CPU is
busy doing anything else!)

## Longest common subsequence

All the diff algorithms I've seen so far are variations, tweaks and
optimisations on the basic issue: searching for the longest common subsequence
between two pieces of information. It's probably a good idea to start there,
then.

Wikipedia tells us that the Longest common subsequence problem is:

> The longest common subsequence (LCS) problem is to find the longest
> subsequence common to all sequences in a set of sequences (often just two).

I'm happy to accept that it's just two subsequences we're dealing with for
this, since my use case is usually to find the differences between two versions
of a file. So let's try and restate it slightly:

> The longest common subsequence (LCS) problem is to find the longest
> subsequence which is common to two sequences.

The word 'sequence' suggests to me that ordering is significant. So we're
looking for the longest common ordered list when is contained in both ordered
lists.

(I often approach Wikipedia articles like this; taking what they say in some
precise language and try to break it down into something I understand, which
has a close enough meaning to my understanding.)

The article notes that this is different from the longest common substring
problem, which is finding the longest common sequence of consecutive terms. So
a subsequence needn't consist of consecutive symbols. I think we've enough
information here to come up with an example. Take the degenerate case:

* The first subsequence is `{ A, B, C }`.

* The second subsequence is `{ A, B, C }`.

I think we can agree that the longest common subsequence is also `{ A, B, C }`
since they're identical. How about:

* `{ A, B, C }`; vs

* `{ B, C, D }`?

Again not controversial, the common subsequence is `{ B, C }`. Here's where
substrings and subsequences start to differ, though. Take:

* `{ A, B, C, D, E }`; and

* `{ A, C, E }`.

The longest common subsequence is `{ A, C, E }`. Of course, there can be
several subsequences that have the same (longest) length:

* `{ A, B, C }`; and

* `{ A, C, B }`

have a longest common subsequence of both `{ A, B }` and `{ A, C }`. In terms
of correctness for representing the differences between two versions of a file,
it doesn't matter which subsequence is chosen. However, semantically it can
make a huge difference to representing the human-readable meaning behind the
changes, as we'll see later. So often the problem is really to find *all* the
longest common subsequences for a set of sequences.

You might be getting the idea that this is **hard** to compute. And you're
right. In the general case, for an arbitrary number of input sequences, the
problem is NP-complete. However, if we reduce it to a known set of sequences
(and we're happy with two sequences, as we've said above), and we've just
searching for a single solution, then it gets a bit more manageable.

In terms of building a solution to the problem, there are there are three
useful things to note:

* The degenerate case: if either of the subsequences is empty, then the common
  subsequence is obviously empty.

* Two recursive cases:

  * If both sequences end with the same set of terms, then the LCS is that of
    the LCS of the two subsequences without the common terms, plus the common
    terms.

  * If the sequences don't end with the same terms, then one of those terms
    doesn't exist in the longest common subsequence. So the LCS is either that
    of the LCS of the two strings with the last term of one or t'other removed.

It's sometimes more complicated to express an algorithm in words than it is in
code. Here's a naive implementation in Ruby:

{% highlight ruby %}
def longest(a, b)
  a.size > b.size ? a : b
end

def lcs(a, b)
  if a.empty? || b.empty?
    []
  elsif a[-1] == b[-1]
    lcs(a[0..-2], b[0..-2]) + [ a[-1] ]
  else
    longest(lcs(a[0..-2], b), lcs(a, b[0..-2]))
  end
end
{% endhighlight %}

That's clearer, isn't it? :) I'm sure in other functional languages you'd get a
relatively performant implementation by reversing the inputs, allowing you to
take the `car` and `cdr` of the sequences as you worked through the input. What
can I say? Ruby is my hammer.

So, that's all terribly interesting, but what's the longest *common*
subsequence got to do with representing the differences? It's exactly the
opposite solution to what we're looking for; we want to find all the bits that
are different, not all the bits that are the same! For that, let's try a
slightly different tack. First of all, create a table that contains the lengths
of all the common subsequences between two sequences:

{% highlight ruby %}
def common_subsequence_lengths(as, bs)
  lengths = Array.new(as.length + 1) { Array.new(bs.length + 1, 0) }

  as.each_with_index do |a, i|
    bs.each_with_index do |b, j|
      if a == b
        lengths[i + 1][j + 1] = lengths[i][j] + 1
      else
        lengths[i + 1][j + 1] = [lengths[i + 1][j], lengths [i][j + 1]].max
      end
    end
  end

  lengths
end
{% endhighlight %}

**FIXME: This is pretty much a transliteration of the Wikipedia article's
algorithm. I can't help feeling there's a more idiomatic way of representing it
in Ruby.**

Now we have a note of all the lengths of all the common subsequences of two
strings, we can find the longest common subsequence's length by looking at the
bottom right of the table:

{% highlight ruby %}
def lcs_length(as, bs)
  common_subsequence_lengths(as, bs)[-1][-1]
end
{% endhighlight %}

We can use the table to find a longest common subsequence, still. First of all,
let's define a couple of handy methods on `Array` to make it clear(ish) what
we're doing:

{% highlight ruby %}
class Array
  def car
    self[-1]
  end

  def cdr
    self[0..-2]
  end
end
{% endhighlight %}


{% highlight ruby %}
def lcs(as, bs, lengths = lcs_lengths(as, bs))
  if as.empty? || bs.empty?
    []
  elsif as.car == bs.car
    lcs(as.cdr, bs.cdr, lengths) + [ as.car ]
  else
    if lengths[-1][-2] > lengths[-2][-1]
      lcs(as, bs.cdr, lengths.cdr)
    else
      lcs(as.cdr, bs, lengths.map { |xs| xs.cdr })
    end
  end
end
{% endhighlight %}

**FIXME: Still not much like the Ruby I know and love.**

This will happily return one of the longest common subsequences, which puts us
back where we were. But with this table of lengths of subsequences, we've got
enough information to trivially construct a diff:

{% highlight ruby %}
def diff(as, bs, lengths = lcs_lengths(as, bs))
end
{% endhighlight %}

**FIXME: Irritatingly, I've had enough beer that I can't make this one work
just now!**

## Hunt-McIlroy Algorithm

It turns out that the first version of the `diff` program was was written by
James W. Hunt, along with Douglas McIlroy in 1976, and was the subject of a
paper entitled, "An algorithm for differential file comparison". It's an
improvement on the longest common subsequence solution so it's faster and uses
less memory.

## Resources

* [Longest common substring problem](http://en.wikipedia.org/wiki/Longest_common_substring_problem)
  on Wikipedia.

* [Patience Diff, a brief summary](http://alfedenzo.livejournal.com/170301.html).

* [Patience Diff advantages](http://bramcohen.livejournal.com/73318.html) from
  the 'inventor' of the algorithm.
