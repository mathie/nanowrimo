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
is your chance for a personal code review. It often gives me good inspiration
for the commit message, too, which is an important part of the story you're
telling; you've described *what* your changing the code already; now is the
time to explain *why*.
