---
layout: post
title: The Single Irresponsibility Principle
---
Sometimes I yearn for the good old days. Remember when you could write a blog
in fifteen minutes in Ruby on Rails? All you had to do was generate the
skeleton application, generate a scaffold for the posts. This covered all the
boilerplate HTML, and a `PostsController` which handled all the interactions on
the HTTP side.

The `Post` model was generated too, which handled the model's responsibilities,
persistence, that kind of thing. The backend magic of Ruby on Rails matched
everything up, so all we had to do was tweak a few things, add a sprinkling of
CSS and we were done. Fifteen minutes later, a fully fledged blogging engine.
(OK, so an MVP blog at least.)

If you listen to all the latest Object-oriented, service-oriented, rich client
application dogma, it's a little more involved now. Of course, none of the cool
kids use the scaffolding any longer, because that's just for newbies.
Experienced Ruby developers only ever create hand-crafted, bespoke, artisan
boiler plate from raw materials.

What's the minimum viable product for a responsible Rails application these
days? Well, first of all you need to install Rails. Which means you really need
to install Bundler. I hope you've got a recent Ruby installed -- Rails 4.0
works best with Ruby 2.0, which nobody distributes, so you'll have to manage an
installation of that yourself. All's not lost, though there's help at hand.
You've just got the small decision left to make: which competing Ruby
installation system do I use? `rbenv`? `rvm`? `chruby`? Or are you totally
hipster and build it from source yourself?

Well, I'm glad you've got that all sorted out. Now it's just a mere matter of
developing some software.

## Generating the Rails skeleton

But before we get to that, you've a few important decisions to make. After all,
while Rails still has *defaults*, none of the cool kids are into them any
longer. If you want to make the most of community help and support, you'll have
to guess which customised stack you're going to use:

* Which storage backend? The default is MySQL, but seriously, that's so old
  school. If you're going to insist that SQL is the way forward, then the least
  you can do is use PostgreSQL. But if you want to be down with the kids, you
  should be using some distributed, eventually consistent, NoSQL storage
  backend, which will scale with your company's hockey stick growth. Oh, we're
  just building a personal blog? Well, you never can tell, you might post
  something that winds up on Hacker News, right?

* There's no question that you *are* going to test-drive your implementation.
  After all, no responsible developer would ever spike production code without
  first writing a test, right? Of course not! But which testing framework do
  you use? `Test::Unit`? Minitest? RSpec? This is important stuff, because the
  mere words used to express your ideas will forever affect the final product.

  And are you going to write high level feature specs in Cucumber? Do you value
  the ability to communicate with stakeholders in natural language, or are the
  stakeholders theoretically into all this stuff but, in practice too lazy
  (erm, I mean 'busy doing stuff of real business value') to read the Cucumber
  specs, never mind help you write them?

* Which base JavaScript library are you going to use to abstract away all the
  niggles of less-than-modern browsers? JQuery? Prototype? Zepto? Something
  else entirely? This is important stuff, because it's going to affect your
  ability to cargo cult solutions from Stack Overflow later on.

Got all that? Phew. Now you can generate your Rails application skeleton. Now,
I know that none of the cool kids are into code generation but, sorry, Rails is
still a little opinionated about the directory layout and, if you don't
generate all the skeleton configuration, you're going to miss something. That
said, you probably want to review all the stuff it generates -- particularly
the commented out code -- and remove all the junk.

## The user interface

Before we can actually write any code, we have a moral responsibility to make
sure it looks pretty. Don't worry. While there are hundreds of choices of CSS
frameworks (nobody beyond Neanderthals hand code their CSS from scratch these
days!), this is a simple choice. Everybody uses Twitter Bootstrap. It's good
that half web sites on the planet look the same. It provides a familiar 'user
experience' to the customers.

Don't think you've got it all easy, of course. You've still got to pick the
template language for building the HTML of the views. This one is still fairly
straightforward. If you're working with somebody else who's designing the user
HTML interface, and you hate them, mandate HAML. If you don't hate them, stick
with ERB. (If you're not working with a UI designer, you have to figure out
whether you hate yourself, or not.)

## The Rails View

Now we're on to day three of "Learn Rails in 21 days": Implementing the Rails
back end. We are responsible developers, of course, so it's not like we're going
to eschew accessibility and usability by building a single page JavaScript
application with an API backend (tempting though it is, because all our friends
are doing it). We're going to build a proper Rails application, which we'll
progressively enhance. Because, of course, Turbolinks has our backs and will
make it all super speedy anyway.

And, as responsible developers, we're all about object oriented design,
patterns and SOLID design. So we can't just whack together a `PostsContoller`,
a `Post` model and a handful of views. Ha, no, that would only take 15 minutes
or so. And let's face it, most of us Rails developers are consultants that
charge by the hour. We'd never make a living from that.

### Controllers

As usual, we need a hand crafted, artisinal, test- (or behaviour-) driven
controller to handle the standard HTTP actions. Things like getting a list of
posts, displaying the "new post" form, showing an individual post. The usual.
Our tests will cleverly mock out the interaction with the back end (because
that's a different, separately tested, responsibility), and ignore the
generated HTML (because that's too fragile and irritating to test anyway).

In addition to the standard controller for HTML requests, we need a separate
controller to handle API requests. After all, every responsible application has
an API and we'll need that to implement the rich client side JS application. In
fact, that's a good point. We don't really want to tie our private API and
public API together. Our internal interests might change, and we can't ever
break a commitment to our public API users.

OK, let's be "pragmatic" about it. We don't need the private API to interact with
the JavaScript rich client today, so we can ignore that today, and implement it
tomorrow. We still need the public API, though. No responsible Web 2.0
application can exist on the Internet without an API for the hordes of third
parties to mash us up with all the other shiny new applications.

Shit, we nearly forgot the iPhone client we're developing to go along with the
web application! It needs to use a private API, but it's not quite the same as
the one we'll need for the JS client (and we wouldn't want to risk one being
constrained by the other anyway), so we'll have a controller to handle it, too.

Of course, we're going to get the first version wrong. Everybody does. We'll
probably pivot the business in six months' time anyway. So we *must* version
the API.

Right, so far we've wound up with:

* `PostsController`, which handles the HTTP requests from regular browsers.

* `API::V1::PostsController`, our public API.

* `API::Mobile::PostsController` to handle the iPhone application. I hope we're
  not going to regret being so generic here ... maybe we'll paint ourselves
  into a corner and need a separate API for the Android app? That's going to be
  a hell of a refactoring!

And in the future, we just know we're going to introduce
`API::Private::V1::PostsController` to handle the rich JavaScript client. And,
of course, we've got corresponding decoupled unit tests for all these bits.
Each controller handles creation of new posts, updating existing posts, reading
single/multiple posts and deleting posts.

This all sounds like massive amounts of duplication, but don't worry.
Controllers are thin. They're practically boilerplate. In fact, there are a
dozen gems we can use which -- more or less -- abstract them away. Of course,
each of these gems have strong opinions, so I hope you can stomach one of the
opinions.

We've not yet gotten down to the actions that are performed by each of the
controllers. Let's again be pragmatic by assuming that each of the interactions
are simple; just the usual creation, reading (collections and individual
posts), updating and deletion. And, because we're simplifying things, aiming
for that minimum viable product, we can ignore complex things like
authentication and authorisation.

(Security is driven by demand, right?  Actually, total tangent, but staying
'one page ahead of the class' is exactly how you responsibly implement most
things, including security, if you consider that you've got a script kiddie in
your class. Everything has a cost, and security inhibits innovation and
process, so the later on you have to kowtow to it, the better. It's only when
you become popular/irritating that folks tend to test your security. Of course,
that assumes a minimal responsible attitude to security -- hashing passwords,
using TLS, not being an idiot -- in the first place. I've been involved in
companies where the security demand is driven by the development team, where
it's been driven by external entities (usually investors or partners), and when
it's just been driven by the owners/founders' paranoia levels. Perhaps I'm
naive because I've never dealt with a major data leak. Is that luck, or me
being reasonably good at what I do?)

### The Persistence Layer

We've successfully test-driven our controllers into existence. And, for bonus
points, we've got really fast tests because they've been implemented entirely
in isolation because we're not talking to some slow back end persistence layer
-- hell, we haven't even needed to implement it yet! After all, if you can mock
it out, you can save the implementation for another day.

First things first, who should the controller even talk to about acquiring
knowledge of the post and its persistence mechanism? Directly referencing a
`Post` would be a heinous crime. How would the test suite be able to inject an
alternative implementation? So first of all, we need to implement a registry.
Something that we can ask for stuff at runtime, because baking that sort of
thing in at compile time would be inflexible and crazy.

What does our registry return? A `PostRepository`, of course. This is the part
of the application whose single responsibility is to manage the persistence of
posts. Now I think about it, perhaps fetching and storing posts are two
responsibilities? It all depends on scale. Still, our `PostRepository` has
responsibility for fetching a post from the abstract backend storage system,
and saving posts back to it (creating new ones, or updating existing ones).

The `PostRepository` can finally give us a `PostData` model. This is the
ActiveRecord model that we'd traditionally have used, but because we're not
really into this humongous multi-purpose class, we hide it behind our more
responsible interfaces so you'll never actually interact with it. Still, we're
stuck with it because we're leveraging the power of Ruby on Rails.

Since we don't really want to expose this nasty, bloated, ActiveRecord model,
to the rest of our mature, responsible application, we need a "Plain Ol' Ruby
Object" to cover for it. Just because we hate performance and love to bust the
method caches, we base this on an `OpenStruct`. But at least we now have a home
for the actual behaviour/business logic of our model.

Naturally, the `Post` model doesn't have any view-related concerns, because,
well, they're (kinda) the responsibility of the view. So we need a
`PostDecorator` to enhance the post so that it can be displayed nicely to the
user without:

* polluting the views themselves with code; or

* having any of those crazy, global, helpers.

The `PostDecorator`'s responsibility is only to help build HTML views. Anything
else would be a responsibility too many. Now we need a `PostSerializer` which
is responsible for turning a post into something suitable for delivery to API
clients. It's just as well we're mature developers who have abstracted the what
from the how -- imagine having to duplicate all this code for each possible
representation? After all, our API users absolutely demand JSON, XML and Apple
property lists. But still, each API we've implemented above requires a
different set of data in the serialisation, so maybe each should have a
different serialiser? (And let's not get into the British English cognitive
shift of talking about serialisation but feeling obliged to follow the
convention of saying `Serializer`.)

I'm almost tempted to suggest there should be a separate `PostDeserialiser`
there too, which takes raw data submitted from each of the versions of the API,
and converts it back into a plain ol' `Post` object. But this is getting crazy.
We need a class for each object to say how it should be serialised already
(thank goodness we've abstracted the format!) and doubling that with a
deseraliser for each seems like ... fun!

We're not done yet. We've still got to deal with HTML forms. While our API
clients are relatively well behaved, submitting correctly typed data (because
we can infer the type from the XML or JSON representation), browsers aren't so
helpful. Once the abstract deserialisation of data from the browser happens,
we're left to deal with a nested hash of strings. How do we turn that into a
post, ready for passing to the `PostRepository` to be persisted? How do we
communicate the current form state to the browser, based on the current `Post`
object?

Step in the `PostForm`, which is capable of deserialising these nested hashes
of strings into the internal representation, with native data types. This could
be the place to do validation, but we might even want a separate
`PostValidator` concern mixed in to handle that -- after all, validation might
be shared across several deserialisation classes (e.g. the ones that take API
input, too).

Are we done with the Rails side of things, yet? Have we covered all the
responsibilities we need? Have we managed to build the simplest thing that
could possibly work yet? Let's just summarise what we've got so far:

* As a responsible, progress web 2.0 application, we need at least three
  different controllers:

  * `PostsController` to interact with regular web browsers.

  * `API::V1::PostsController` which is our public API so everyone can mash up
    our content.

  * `API::Private::PostsController`, which is the internal API our JavaScript
    smart client is going to need.

  * `API::Mobile::PostsController`, the mobile-optimised private API for the
    internal iPhone application.

* A representation for business logic and persistence:

  * A `Registry` where we can retrieve a handle to the object's persistence
    store at runtime, decoupling the code from the store.

  * A `PostRepository` which handles the retrieval and persistence of these
    posts.

  * A `PostData` model, our traditional `ActiveRecord` model. We're being
    pragmatic here, leveraging the power of ActiveRecord, but hiding it behind
    a more modern facade.

  * A `Post` model, which represents the post within our application.

* And then we've got the classes that handle different representations:

  * A `PostSerializer` which takes a post and knows how to render it to an
    abstract nested hash which is the lowest common denominator for JSON, XML,
    etc. Let's be pragmatic and say it's responsible for de-serialising, too.

  * A `PostDecorator`, which handles all the view-related concerns of rendering
    a post on the HTML page, so there's no complex logic in the view templates.

  * A `PostForm` which handles the logical representation of the form (a
    `FormBuilder` handles the HTML bits), and can deserialise the submitted
    form data back into native `Post` objects.

That seems about enough for now. We've got a clear separation of concerns, each
responsibility has its own place, and we don't have bloated models or
controllers. *phew*, bullet dodged!

## What about the client side?

Every responsible Web 2.0 application has a rich JavaScript client interface.
It's the only way to be sure. And, as cool kids, we really *love* distributed
systems because they make life so much more fun, so why not distribute state
across every browser that visits the site?

But, oh my, which client side JavaScript application framework does one pick?
Backbone.js pitches itself as lightweight (it's all relative). EmberJS is fully
featured, opinionated, and relatively closely aligned to the opinions of Rails.
AngularJS is ... clever?

Either way, this means we're going to need to create a 
