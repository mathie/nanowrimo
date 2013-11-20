---
layout: post
title: Personal Code Review
---
[`git`](http://www.git-scm.com/) is my distributed version control system of
choice, usually paired with [GitHub](https://www.github.com/) for collaboration
(because sometimes the conversation around code is just as important as the
code itself). It's been that way since 2007, when
[Mark](http://www.sirena.org.uk/) did a talk at the Scottish Ruby User Group
and convinced us all it was better than Subversion. (It really is.)

I've seen, and contributed to, many conversations on work flow using Git. These
tend to be focused around the collaboration amongst team members, how to manage
releases, how to build larger features. The right answer in these situations is
certainly product specific: a product where you release specific versions and
have long term support for several simultaneous versions is a very different
beast from a web application that's continuously delivered to production with
only one version in the wild.

It's often team- and culture-specific, too. And the team's culture can change
over time. What works well for an open source project might not apply well to a
corporation suffering under PCI or ISO 29001 compliance, for example.

The conversations often revolve around a model for branching (to work on larger
features away from the main, stable, production code), the length of time that
branches can successfully survive for, the maintenance overhead in branching,
how and when to merge code. These are, fundamentally, not really conversations
about git itself, but about team collaboration. (Code that's sitting on your
computer is a branch, it's just one that you haven't shared with your team yet.
Even if it's called 'master'.)

But that's not what I want to talk about. I have a micro work flow with Git
that I'm quite fond of, and I thought I'd write it up, just in case it's not
blindingly obvious.

The key result of my git micro work flow is that I want each individual commit
to represent a single, atomic, change, and for it to make as much sense as it
can in isolation. A branch turned into a patch (or GitHub pull request, when
that's how I'm collaborating) is a collection of these atomic commits that tell
a larger, cohesive, story. The key thing is that neither commits, nor pull
requests, are a jumble of unrelated stories, all interwoven.

This is how I achieve that.

I'm as scatter-brained as it comes. When I'm hacking on a bit of code, I'll
often spot something entirely unrelated that *needs* fixing. *Right now*. If
I'm being good, I resist the urge, make a note on my to do list (an 'internal'
interruption in Pomodoro parlance) and carry on with what I'm supposed to be
doing.

But often I'm not good. I'll get distracted by this yak shave and spend the
next hour fixing that instead. Of course, that takes me off to other parts of
the code, where the same thing happens, and we have a recursive rabbit hole.
Hours later, I emerge, victorious, popping my stack back to the original
problem and getting on with that.

But my working tree is a complete shambles of unrelated changes. I can't
possibly submit this as a single, cohesive, pull request. Nobody's going to
notice the two line patch that solves the problem I'd intended to solve.
Instead, it'll be hidden amongst the 3,000 line 'refactoring' I've done on a
pile of unrelated things.

How do I get out this mess? It's somewhat telling that I have three particular
git commands as aliases in my shell, and that they crop up so much in my shell
history:

* `git diff` (aliased to `gd`) which tells me the changes I've made compared to
  what I've already staged for commit. When I'm just getting started untangling
  the mess, this is the changes I've made but not yet committed.

* `git diff --cached` (aliased to `gdc`) which tells me the changes that I've
  already staged for commit. This is essentially a work-in-progress of "what
  I'm about to commit next".

* `git add --patch` (aliased to `gap`) which shows me each of the fragments of
  changes I've made (showing each change within a file separately), asking of
  each in turn, "would you like to stage this fragment for commit?"

The combination of these three, with a bit of extra fiddling, allows me to
create those atomic, cohesive, commits that I desire.

Let's say, in this situation, that I have implemented two different things,
that really ought to be in discrete commits, which are part of the current
'story' I'm telling, and I've gone down a rabbit hole, refactoring two entirely
unrelated parts of the code base. What a mess!

Let's start with the two bits that are related to the current story I'm trying
to tell. How do I extract them? `git add --patch` to the rescue. It will show
you each change that you've made, allowing you to decide whether to stage it
for commit or not. Pick one of the atomic stories you want to commit first. If
one is dependent on t'other, which is often the case, it makes sense to pick
the prerequisite first. Otherwise, pick the one that shows up first. While
working through `git add --patch`, accept the changes that are related to
that commit.

When you've accepted all the changes, review the proposed commit with
`git diff --cached`. This is your chance to cast a final eye over the change,
make sure it makes sense in isolation and doesn't have unrelated changes. This
is also your chance for a personal code review:

* Does the code look correct?

* Are there unit/integration tests covering the changes you've made?

* Does the code look pretty? Is there any trailing white space, or duplicate
  carriage returns, or missing carriage returns? Are things lined up nicely?

* Is it accurately documented?

* Have I accidentally left in any debugging code?

It often gives me good inspiration for the commit message, too, which is an
important part of the story you're telling; you've described *what* your
changing the code already; now is the time to explain *why*. Now that I'm
satisfied about what I'm committing, I commit that chunk.

Repeat the process for the other, related, change that's part of this pull
request's plot arc.

But what about the other, unrelated, changes? They don't have a place in this
plot arc -- they're more like the standalone 'filler' episodes that TV writers
bung in to fill up a season. What shall we do with them? Separate stories,
separate branches/patches/pull requests, of course! If you're lucky, then you
can create a new branch straight away, with master as the branch point, keeping
your working tree intact:

    git checkout -b shiny-refactoring master

then use `git add --patch`, `git diff --cached` and `git commit` to build up
and review atomic commits on the new branch for your shiny refactoring.

If your current working tree's changes will not cleanly apply to master, then
the easiest way to deal with it is to temporarily stash them, and pop them from
the stash afterwards.

{% highlight bash %}
git stash save --include-untracked
git checkout -b shiny-refactoring master
git stash pop
{% endhighlight %}

You'll still need to resolve the conflicts yourself, though.

When things get a bit more complicated, `git add --patch` still has your back.
When you've got an intertwined set of changes which are close to each other in
the source, but are semantically unrelated, it's a bit more effort to unpick
them into individual commits. There are two scenarios here.

If you have two unrelated changes in the same fragment, but they're only in the
same fragment because they share some context, then you can hit `s`. The
fragment will be split into smaller constituent fragments and you'll be asked
if you'd like to stage each one individually.

If the code is seriously intertwined, then you can ask git to fire up your
favourite editor with the patch fragment in question. You can then *edit* that
patch to the version you want to stage. If the patch is adding a line that you
don't want to add, delete that line from the patch. If the patch is removing a
line that you don't want removed, turn it into a context line (by turning the
leading `-` into a space). This can be a bit finickity, but when you need to do
it, it's awesome.

I've had a couple of people pushing back a little on this work flow (usually
when I'm in the driving seat while pairing) over the past few years.

The first was from a long-time eXtreme Programmer, who rightly pointed out,
"but doesn't that mean you're committing untested code?" Even if the entire
working tree has a passing test suite, the fragment that I'm staging for commit
might not. It's a fair point and one I occasionally have some angst over. If
I'm feeling that angst, then there is a workaround. At the point where I have a
set of changes staged and ready to commit, I can tweak the work flow to:

{% highlight bash %}
git stash save --keep-index --include-untracked
rake test # Or whatever your testing strategy is
git commit
git stash pop
{% endhighlight %}

This will stash away the changes that aren't staged for commit, then run my
full suite of tests, so I can be sure I've got a passing test suite for this
individual commit.

The other argument I hear is, "why bother?" Well, that's really the point of
this article. I think it's important to tell stories with my code: each commit
should tell an individual story, and each patch/pull-request should tell a
single (albeit larger) story, too.

(Not that I always listen to my own advice, of course.)
