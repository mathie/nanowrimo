---
layout: post
title: My Favourite Interview Question
---
I have an interview question I rather like using. I can't for the life remember
who I stole it from -- I had thought it was from an Amazon employee -- but I
can't find a reference offhand. Talking with my father in law at the weekend, I
was reminded about it, and realised I'd never really answered the question
myself.

It's a pretty simple question:

> When I type 'www.bbc.co.uk' into the location bar of my favourite browser,
> and press return, what happens?

(I should probably update the question to reflect the automatic searching
nature of most browsers' location bars, now.)

It's a doozy of a technical question, I think. There are plenty of different
areas and concepts involved, so it encourages a fun and interesting
conversation.  It gives me an idea of whereabouts in the stack the interviewee
feels most comfortable (do they primarily talk about the server page
generation, the intermediate communication, or the client side browser
behaviour?) and their depth of knowledge in those areas. Finally, it tells me
something of the candidate's communication (and teaching) skills.

Plus I always learn something new. And when you're interviewing, as Valve
succinctly put it, you're looking for a 'T' shape. Somebody who has a wide
breadth of experience, but has depth of experience in one particular area.
Personally, I want to find people who can cover much of the DevOps arena, but
who has depth in something in particular (Ruby on Rails, MySQL, scaling HTTP,
networking, racking servers, etc). Either a lack of depth, or a lack of
breadth, is suboptimal.

It's good to find somebody who can answer the breadth of the question and at
least one area in some depth. (Ideally more depth than me because I want to
learn!) Given the choice, though, I'd rather manage employees with depth, than
trust autonomy to people with breadth. (*FIXME: Is that a fair way of phrasing
it?*)

So, where to start? I find it helpful to think in terms of three different
aspects:

* the passing of time;

* the entities involved; and

* the scale at which we're examining the system.

Looking at the time line at a very high scale, we get an accurate, but entirely
superficial answer: the web browser goes to the BBC web site, fetches the
information for the page and displays it to the user.

How best to break that down into something a little more detailed? Well, there
are three high level components involved in this interaction:

* the browser running on the user's computer;

* the intermediate network that enables message passing between the user's
  computer and the BBC's server; and

* the server at the other end, which generates the page in response to the
  user's request.

Each of these high level components encapsulate lower level details that enable
the system as a whole to work. So now it's all about diving into the detail of
these systems, and their interactions at smaller, and smaller scales.
Eventually, we get to electrons flowing, and light waves passing through bits
of glass. That's way deeper than I know about. ;-)

In terms of structuring an answer, this time round, I thought I'd experiment
with working along the lines of the Internet architecture defined in RFC 1122
(I think this is me showing my bias towards the network side of things). RFC
1122 defines the Internet architecture in four layers:

* The application layer, which is the top level application-specific
  communications protocol used to allow individual hosts to communicate intent.
  The main protocol in this particular part of the system for us is the
  HyperText Transfer Protocol (HTTP), though there are a few other guests
  appearance. This is a bit hand-wavy nowadays, so as we get to looking at this
  layer in more detail, we'll break it down a bit more.

* The transport layer, which provides a consistent communications channel
  between two application endpoints. The application endpoints are identified
  by a source and a destination port, which allows for bidirectional
  communication between the applications. Some transport layer protocols
  provide additional features, which we'll get on to in time. The most
  communication protocol in our conversation will be the Transmission Control
  Protocol (TCP).

* The Internet layer. This is the mechanism that all our transport layers use
  to communicate between two hosts on the Internet. The Internet layer, and its
  corresponding control protocols, are lightweight, and connectionless,
  providing no guarantees over whether information is delivered at all and,
  even if it is, what order it turns up in. The Internet layer provides us with
  the tools to communicate via intermediate hosts, called gateways, when two
  hosts aren't directly connected by the lower layers.

* The link layer, which allows two directly connected hosts to communicate over
  shared media.

Since our scope is wider than just the network, then there are two additional
layers which we could consider:

* The application itself, which interacts with the user, taking input from
  them, processing information locally, communicating using application layer
  protocols, and rendering information back as output to the user.

* The physical layer, where the electrons fire, and the waves bounce.

The OSI (FIXME: remind myself what this actually stands for!) model breaks the
network down into seven separate layers, which roughly correspond to the
description above. But there some differences between them as to where a layer
starts and ends, and we're (mostly, probably, or at least can assume we are)
living in an Internet world, so the layers defined by the Internet Engineering
Task Force (IETF) make most sense.

For the main information flow, we're going to start at the top layer -- the
application -- in the form of the web browser running on our computer. We're
then going to dive down through the layers and surface on the server, to see
what it thinks of what's been asked of it. Once the server has figured out its
response, we'll dive back down through the stack, back up the other side, and
see what the browser makes of the response. Since much of the details are
repeated, the first dive will be the most content-heavy from the network's
perspective, and I'll try not to repeat myself too much on the subsequent
journeys.

OK, let's get started.

## It all starts with the browser

Well, that's not strictly true, is it? We could start with the key switch.
Pressing the actuator which we symbolically associate with the 'w' on a
keyboard causes an electric circuit to complete, which a microcontroller inside
the keyboard interprets and converts into USB data packets, which is
communicated down the USB cable to the host computer which ... you get the
idea. There's complexity in every aspect of this answer, and there are
differing levels of scale that I'll assume at different parts of the system.
Essentially, as I indicated above, this is a feature of the question. It's a
good way to gauge interest, depth of knowledge and what you consider to be
relevant.

Since I'm primarily involved in interviewing software developers (usually web
developers), I'm less interested in the computer itself. If I was interviewing
hardware engineers, I'd be hoping for an answer slanted towards the electrical
signals and low level communications protocols (like what happens when I press
the 'w' key). And I'd be really interested in learning the answer. :)

OK, so we've typed 'www.bbc.co.uk' into the browser and pressed return. What
happens next? Well, it has to interpret what we've typed and figure out what we
actually mean. Then phrase 'www.bbc.co.uk' is at the very least ambiguous, and
requires some inference. Since we're typing this into the location bar of a
modern web browser, and particularly since it starts with the phrase 'www',
then it's probably reasonable to assume it's a web site. In particular, that
allows us to make some inference that it's probably the HTTP protocol we're
being asked to use, and turn that into a fully fledged URI:

    http://www.bbc.co.uk/

So, what's a URI? It's a Universal Resource Indicator, a generalisation of the
Uniform Resource Locator (URL). A generalisation? Yes, people discovered they
could use these universal beasties for more than just pointing to the location
of something. As it is, ours is a location, and is also, more specifically, a
URL.

Given the URL we have now inferred, the browser has enough information to
proceed. It now knows:

* The application layer protocol that we've requested, `http`.

* The location that we're looking to access: `www.bbc.co.uk/`.

* By the absence of other information (explicit port numbers, any
  authentication details, a path, a fragment or any query parameters) that can
  be part of a URI, we also understand what the user believes is not required.

and can parse that information from the URL according to its well defined
format.

But before going any further, the browser has the opportunity to check its own
private cache of data that it has previously retrieved. Most of the details of
caching belong to the application layer, but once a web page lands at the
browser, it usually comes with a length of time the browser can consider it to
be 'fresh' (i.e. current and substitutable as a response without checking). If
the page already exists in the local cache, and is still fresh, then the
browser is free to display that to the user with no further hassle.

In practice, this sort of work flow often happens with additional information
used to display a fully fledged web page to the user (so called 'static' assets
like JavaScript, CSS or images), but rarely happens with the initial web
request. It's unlikely that the initial request will hit that local cache. In
fact, the BBC web site explicitly forbids it through an application layer
header.

Now we've checked the browser's cache, and it doesn't have an authoritative
response. Bother, we're going to have to do this the hard way. Now we need to
break that URL down into a couple of component parts:

* The protocol, `http` which tells us the application layer protocol we're
  going to use to communicate with the remote host; and

* `www.bbc.co.uk` which gives us enough information to figure out the remote
  host we're going to talk to.

### Name resolution

Now we're onto a slight digression on turning names into something we can
actually use to communicate intent to the lower layers. At each layer (mostly),
there's a mechanism to turn the symbolic identifier we have to represent a
remote host into something the lower layer understands. At this layer, the
information we have is a host name, and the information we require to
communicate to the lower layers is an IP address.

The standard mechanism which provides this service is the local operating
system's resolver. This is a service shared amongst all the applications on a
single computer which turns a name into an IP address. (In some web browsers,
it's not so simple, because they provide their own resolver inside the
application in order to give them some performance or portability advantages,
but the principle is the same).

In theory, there are several mechanisms for converting names to IP addresses
-- static files, files distributed by a local network source, and various
network services. This is managed by the standard library on Unix-based
systems, which provides the Name Service Switch (NSS), abstracting what names
you want to resolve from how the local system administrator has said how you
can resolve them.

In practice the mechanism that supports this for most users is the Domain Name
Service (DNS) which, amongst other things, allows the owners of a name to
provide users with the IP address associated with that name. I'm going to gloss
over a few details at this point because we'll cover them well enough later,
and it kinda breaks from the narrative a bit. :)

The browser asks the local resolver to provide an IP address that the next
layer down can use to further the communication. Once again, there's caching
involved. If the local resolver has already been asked this question before,
and the result is still considered authoritative, it immediately returns that
result. Let's say that's not the case, because that would be dull.

Now we have to communicate with another system to figure it out. As I say,
we'll gloss over the details of how that happens for now, because we'll cover
it all later, during the main communication. The local resolver constructs a
User Datagram Protocol (UDP) packet, saying, "I have this name, which IP
address should I use to communicate with it?" In DNS terms, the client is
looking for an `A` resource record (RR) which tells the client the IP address.

Of course, it's never that simple (and this is where I diverge from reality
into poetic license in order to tell a good story). The "well known" name for a
service is rarely the name of the system that provides that service. So often,
in DNS terms, the server will return the "canonical name" (in DNS terms, the
`CNAME`) for the hostname. If we're lucky, and the server we've asked also
knows the answer to that question, it'll throw in the answer 'for free'. If
not, then it's wash, rinse, and repeat, to find out the address of the
canonical host name's IP address.

Hallelujah. We now have the address of the person we're looking to talk to, and
we know the application layer protocol that we want to use to talk to them. And
we know we can't answer the question from our local cache. Time to ask the next
layer down, the application layer, what to do.

## The application layer

The application layer does a lot. It's most useful to break it down from the
bottom up, really. Its responsibilities include:

* Communicating the intent from the application itself;

* Maintaining state between individual communications because, over the long
  term, all the lower layers are stateless (though some maintain state across a
  single connection, as we'll see later).

* Making the caching, cache-testing and expiry semantics clear to the upper
  layers.

* Providing authentication, authorisation and encryption facilities.

Depending on the application semantics, the application layer protocol may or
may not need to implement all these things, and communicate such things to the
upper layers.

So far, we've tripped across two high level protocols: HTTP and DNS. They're
pretty different, so let's take each in turn, starting with DNS.

### The Domain Name System

The goal of the Domain Name System is to turn symbolic names into IP addresses.
These addresses are usable by the lower layers of the network stack to find out
who they're supposed to be talking to. There are a few ancillary operations --
still communicating some data associated with a symbolic name -- but it mostly
boils down to finding an IP address through one or more layers of indirection.

This sounds like a simple problem. It's one of those key/value stores, right?
The slight complication comes from two aspects:

* Decentralised control. Every DNS domain, or subdomain, on the Internet can
  have delegated control. Some random computer in somebody's student flat, on
  the back of a dodgy 14k4 modem, can be the authoritative source for
  `example.com`. When I change a value on that computer, I expect that new
  value to be available across the entire Internet (eventually), even if that
  dodgy modem is having a bad day.

* It's a popular service. I mean really popular. Every single web request on
  the planet requires somebody to convert a symbolic name to an IP address.
  Every single email sent requires several names to be (indirectly, even)
  converted to IP addresses. Just imagine.

Imagine some crazy Web 2.0 Software as a Service name resolution system that
was handling millions of users making hundreds of requests per second, while
hundreds of thousands of users were providing dozens of updates at a time.
Sounds like we need a NoSQL solution, right?

This is mostly a sidetrack to the matter at hand, so I'll keep it brief.
Essentially, the reasons that DNS doesn't crash around our ears every day is
because of three things:

* Delegation. At each level in the system, an entity is able to say, "I
  delegate control of this sub entity to somebody else. Go talk to them." So,
  for example, the root name servers (of which there are "8", if I recall
  correctly -- '8' because each of the IPs these resolve to are globally load
  balanced, a very clever topic we'll get onto later) can say, "If you want to
  find a `.com` domain, you should talk to one of these servers." In turn the
  servers that are authoritative for `.com` can take your request and say,
  "well, if you're looking for `google.com`, you should go talk to these
  servers." And so on, all the way down the chain.

* Extensive caching. Every single element in DNS comes with a 'time to live'.
  This is the length of time that somebody can consider that response to be
  absolutely correct. If the record is still within its time to live, it's
  valid. This works even better because the DNS system encourages layering
  these caches. There's one on (almost) every computer. There's usually one on
  your local network. There's almost certainly one at your ISP. And either
  there will be one at their peer, or they're directly querying the roots. If
  any of these intermediates have a valid cached answer, they can give it to
  you without violating their agreement.

* Recursion. This is how the layering becomes effective. When you make a DNS
  request to your local server where it doesn't have the answer in its local
  cache already, you have two options:

  * ask it to point you in the direction of a name server that is more likely
    to have an answer; or

  * have it ask the question of the next name server up the chain, find out the
    answer for you, and pass that answer back to you.

  The latter is recursion and it's what makes the caching so effective. If one
  person using a recursive name server asks for `www.example.com`, then the
  server finds it out on their behalf and caches the answer. Next time a
  different client asks the same question, the answer is already in the cache.

This does, of course, result in 'eventual consistency', and often the
consistency is in terms of hours. It's not uncommon for the time to live to be a
day. (Which is, incidentally, how I've internalised that a day is 86,400
seconds.)

A little more detail is necessary about the Domain Name System; the data that's
being served. There are two concepts in DNS:

* Resource Records (RRs). These are the individual records that are returned as
  the result of a query. There can be many resource records returned for a
  single query (i.e. it's common for a single DNS name to be served by multiple
  IP addresses).

* Zones, which are collections of some resources records, and optionally, some
  other zones (referred to as 'subdomains').

This gives us a tree like structure:

    com <--zone
      |
      +-> example <-- zone
          |
          +-> www <-- resource record
          |
          +-> ftp <-- resource record
          |
          +-> foo <-- zone
              |
              + www <-- resource record

which gives us the ability to resolve things like `ftp.example.com` and
`www.foo.example.com`. Resource records are the leaves, zones are the
intermediate nodes.

In terms of resource records, there are a few options:

* The main one, and the whole point of DNS, really, is the `A` record. This is
  the record that turns one of our symbolic names into an IP address.

* `CNAME` records, which are essentially aliases. If I say that
  `www.example.com` is a `CNAME` for `ghs.google.com`, I'm essentially saying,
  "go ask `ghs.google.com` for its resource record and keep following the trail
  until you get an IP address." (I'm also saying, "I use Google hosted sites
  for my web site." Make of that what you will.)

* `MX` records, which return the symbolic names of the entities who will
  receive email for a domain.

* `SRV` records, which are basically a generalisation of `MX` records when it
  was realised that having specialised records for each type of server resource
  wouldn't scale terribly well.

* `NS` records which allow for delegation of a sub-domain from one zone to
  another. They're the 'glue' (and often involve glue records, which is a
  digression too many right now).

* `SOA` records, called 'start of authority' records, which publish some
  information about each of these delegations, from the delegated authority
  (mostly caching policies).

There are dozens of other resource records, and there's even another namespace
for resource record types, which people have used for distributing
configuration management information in the past. (Google `HESIOD`, see if it
still turns up anything useful.)

All told, though, essentially our application layer understanding is that DNS
asks for a symbolic name to be resolved to an IP address. This is typically a
service provided by the local operating system's name service switch. While its
entirely normal that host names will be resolved through DNS (and in reality,
other than specialised circumstances, they are), there's an abstraction in Unix
called the Name Service Switch. This is a central point where the computer's
administrator gets to set policy about how symbolic names are resolved into
their underlying representation.

This abstraction is not just restricted to host names either. The same
mechanism is used to look up users, groups, network names, net groups,
services, protocol names, and others. This, again, is one of these useful
abstractions, separating *what* you want to retrieve from *how* you want to
retrieve it, and providing the operations people with the ability to specify
*how*.

The Name Service Switch has a pluggable back end (usually, these days, in the
form of Pluggable Authentication Modules (PAM)) which allows an admin to say,
"try this resource, then this resource, etc, to resolve the name you're looking
for." Restricting the conversation to DNS, the name service switch can try
(amongst any pluggable other), a local file (generally `/etc/hosts`) or the
local resolver library. Often, on a larger network, it can be asked to consult
a local LDAP directory, too.

It really is turtles all the way down. This is a good thing, and something
useful to remember. Mature systems that survive have points of abstraction,
where the what, and the how, are kept separate, and where a third party gets to
set the policy of how the what happens. Of course, this pattern is only
*necessary* in software that has stood the test of time, and has a disparate
user community, so I suppose it's not necessarily true that one implies
t'other.

So, in practice in our case, the local resolver is first going to check the
contents of `/etc/hosts` to see if it knows what the IP address should be. If
not, it's going to use the Domain Name System to figure it out. When it comes
to the resolver to figure it out, if it already knows the answer (i.e. has it
in its local cache, and that cached entry is still valid -- or in the case of
some versions of Windows, it doesn't matter if it's still valid!), then it can
return that value. This local cache is operating system-wide, so each
application benefits from others having looked up an address. (Mostly. I hear
rumours that some web browsers have their own private resolver -- and cache --
but I haven't tested that.)

Finally, we're actually getting to the application layer, and some protocol
action. (This is typical -- the application layer has richer semantics, and a
lot more knowledge, so can make more intelligent inferences. The lower layers
are way more dumb and way less complex.)

We've got to the stage where we have a host name (`www.example.com`) that we
want to resolve into an IP address (`127.0.0.1`), and our operating system
knows the IP address of who we should ask for that information (usually
acquired by the Dynamic Host Control Protocol (DHCP), which is way out of
scope, but suffice to say it provides a node with the configuration information
is needs to be a part of the local network.)

Having discovered the IP address of the local name server, the last remaining
piece of the jigsaw we need is the port number it listens on. The protocols at
the transport layer have a concept of a source port and a destination port so
that the transport layer knows which application to pass data back up to when
it receives it from the network. For the client side of the communication, a
random port is assigned by the transport layer (usually in the 'dynamic' range
of 49,152 - 65,535), unless the client requests otherwise.

But how does the client know which port to use as the destination? The Internet
Assigned Names Authority (IANA) maintains a list of well known services. This
is a mapping from a well known name for a service to the port that service
normally listens on. This mapping is distributed as standard with all operating
systems -- on all Unix-alikes, you'll find the mapping in a text file in
`/etc/services`. The well known name (as defined by the RFC which defines the
Domain Name System protocol) is "domain". You can find out the port number for
this well known name yourself:

    > grep '^domain\b' /etc/services
    domain           53/udp     # Domain Name Server
    domain           53/tcp     # Domain Name Server

Now we know the name server listens on port 53 by default, so that's the port
we should connect to. In practice, this is standard and well known (which is
why the ports in the range 0 - 1,023 are known as the 'well known ports'), so
the mapping from name to number seems superfluous. However, the decoupling of
service names and their associated port numbers is still useful. Service lookup
is done through the Name Service Switch, the same mechanism that handles host
name resolution. This allows a system administrator to, for example, provide a
network-based look up of service mappings, perhaps through LDAP. This is most
useful when providing custom services on a network. The software developers
give the protocol a registered name, and the operations people deploying it can
decide on the port number it uses in the deployment environment.

So. At last. We have the IP address of the name server we can use to resolve
the name into an IP address, and we have the port number we want to connect to.
This gives us enough information to open a 'socket' to the name server and
communicate using our application layer protocol.

### The BSD Socket Library

There's a standard interface between the application layer and the transport
layer, and it's called the BSD socket library. It's a standard part of every
Unix operating system (and by 'standard', I mean there are several subtly
different standards that each Unix implements in its own special way) and there
is a moral equivalent of it in Windows, too, I believe. Almost every
programming language provides a low level API which is similar to the
underlying socket library, usually trying to abstract away the crazy
differences amongst their supported platforms.

But the principles are the same. There is a mechanism to initiate a connection
to a remote host (`connect()`), to send data to that host (`send()`), to wait
for data to come back from that host (`select()`) and to read data from that
remote host (`recv()`). There are corresponding server-side mechanisms to
announce that you're prepared to deal with connections to a particular
application-specific port (`listen()`) and to wait for connections on that port
(`accept()`).

While I've mentioned the vagaries of this particular interface (across dozens
of operating systems, which have matured over the course of decades), it's a
powerful thing. It provides a clean interface between the application level
saying *what* it wants to achieve, and the underlying operating system
promising that it will achieve it, without leaking the details of *how*. This
sort of abstraction is powerful, and pervasive across all the successful (ops,
at least) projects I work with.

### The DNS protocol

Once we've got this open socket, we can start to communicate using the
application layer DNS protocol. It's a datagram protocol, which for our
purposes just now at least, means it's completely stateless. Each packet of
data being sent is independent of every other. This is generally useful if
we're just trying to communicate a small amount of data (the DNS protocol
limits the size of an individual packet to 512 bytes, which is almost always
small enough to fit in a single packet on the underlying layers).

That's plenty of room for a DNS request, and ought to be enough for any reply,
too (there's a mechanism to cope with a response that's longer than 512 bytes,
but it's relatively expensive in terms of performance, so any sane sysadmin
makes sure their resource records are smaller!). Let's figure out how to
formulate a query. We need to put together the following information in a
correctly formatted packet:

* an identifier -- essentially a random number -- so that the resolver library
  can match up the request with the response. This is a 16 bit field, so
  something in range 0 - 65,535.

* the query name -- the name we want to resolve -- formatted as a sequence of
  labels. If we're looking to resolve `www.example.com`, then the labels are
  `www`, `example` and `com`. The labels are arranged in the packet as a single
  byte containing the label length, followed by each of the characters of the
  label. The sequence is terminated with a null byte. So our query for
  `www.example.com` would turn into:

      [
        3, 'w', 'w', 'w',
        7, 'e', 'x', 'a', 'm', 'p', 'l', 'e',
        3, 'c', 'o', 'm',
        0
      ]

* The query type (i.e. the type of resource record we're looking for). In our
  case, we're looking to resolve a name into an IP address, so we're looking
  for an 'A' record.

* The query class. This is mostly historical, but allows for the domain name
  system to provide address resolution for other types of system. In our case,
  we're after Internet addresses, so the query class is 'IN'.

That's it. The rest of the packet is boilerplate, indicating that it's a query,
rather than a response. Typically, a resolver will indicate to the server that
recursion is desired, too.

### A point of order

Things have been going quite nicely so far, haven't they? Let's complicate it a
little. Numbers are encoded as a sequence of bits grouped into 8-bit bytes
(interchangeably referred to as 'octets', though I dimly recall there's a
subtle difference). A single byte number is nice and easy, since it fits in a
single unit. (It's helpful here to think in terms of hexadecimal numbers,
because they align neatly with the byte boundaries.) A single byte can store
values `0x00` through `0xff` (255).

But what about longer numbers? The identifier above, for example, is a 16 bit
number, which is two bytes. This allows us to represent all the integers from
`0x0000` through `0xffff` (65,535). This apparently atomic value at our level
of understanding doesn't fit into the atomic 'byte' at the transport layer, so
we need some way of encoding it.

It's pretty straightforward. Mostly. We can decompose the number into two
atomic bytes. So, if we choose an identifier value of `0xabcd`, we can
decompose that into two bytes: `0xab` and `0xcd`. The trouble is: in which
order do we transmit the sequence of bytes? Obviously, there needs to be an
agreed order, so that the system writing the bytes and the system reading the
bytes understand the same semantic value at the higher level.

This is referred to as 'endianness'. There are two real options:

* Little endian, where the least significant byte is transmitted first and the
  most significant byte is transmitted last. So our identifier would be
  transmitted as `0xcd` followed by `0xab`.

* Big endian, where most significant byte is transmitted first, so our
  identifier would be transmitted as `0xab` followed by `0xcd`.

* There's also middle endian. Don't ask.

(FIXME: I guarantee I've gotten that the wrong way round somewhere.)

The convention for all Internet protocols is to use big endian.

Guess what? The Intel PC architecture that we know and love is little endian
(which is why it's sometimes referred to as the 'Intel convention'). Each layer
in the protocol stack presents an opaque sequence of bytes to the lower layer,
so it's the responsibility of that layer to apply the correct conventions for
ordering those bytes. Fortunately, the Unix standard library provides a set of
functions to help us out:

* `htons()` (for 'short' or 16-bit numbers) and `htonl()` (for 'long' or 32-bit
  numbers) which convert a value from 'host' byte order to 'network' byte
  order.

* `ntohs()` and `ntohl()` which convert from network byte order back into host
  byte order.

The standard library is clever enough to make these null operations on big
endian computers and perform the correct byte swapping on little endian
computers.

The particularly insidious problem with endianness is that you won't notice
that you've got it wrong when you're talking between two hosts of the same
architecture. The problem only occurs when you're talking to a host who's
endianness is different. (This is why, once upon a time, when I was developing
a custom network protocol, I made sure to do interoperability tests between the
Linux PC I was using, and a dusty old DEC Alpha (with a big endian PowerPC chip
-- the Motorola convention) running Linux in the corner of the office.)

### A question and an answer

Now that we've composed our DNS query, we send it off down through the network
stack and await a response. There are a couple of possibilities here:

* The remote name server gets back to us in a timely manner with a response; or

* ... nothing happens.

The latter case is surprisingly common -- UDP is an unreliable protocol, as
we'll get to later on. Packets get lost, servers fail, networks can be slow.
The resolver library waits for a configurable amount of time before retrying
and, ultimately, giving up, returning an error back to the application.

On a good day, with a fair wind, though, we'll get a reply back. The reply is
sent from the name server's IP address and well known port back to our IP
address and the source port the query originated from. The response contains:

* The identifier we sent, so we can correlate it with the response. (This
  allows the resolver to reuse source ports for multiple queries.)

* A response code, indicating success, along with an indicator of whether the
  response can be considered authoritative (rather than cached).

* A copy of the question we asked.

* The answer(s) to the query in the form of resource records.

* If the answer is not authoritative, then a pointer to the name servers which
  should be able to provide a more authoritative answer in the form of resource
  records.

* Additional resource records which may be useful for further resolution.

Each of these are in a standard format, called the 'resource record'. It contains:

* The name of the record. When we're querying for IP address of
  `www.example.com` the name is the first label, `www` in the `example.com`
  zone.

* The type of record being returned. In our case, we're looking for an `A`
  record to come back.

* The class of the resource record. As indicated above, this will always be
  `IN`.

* The time to live for the resource record. This is the length of time that the
  answer should be considered valid, and cacheable, for.

* The resource data itself. (In the packet format, it's a 16-bit number
  representing the number of octets in the resource data, followed by that
  number of octets of data.)

In our case, we're going to get an answer along the lines of:

* Name: `www`

* Type: `A`

* Class: `IN`

* Time to live: `86400` (1 day)

* Resource data: `127.0.0.1`, encoded as a 32-bit integer.

If a name resolves to multiple IP addresses -- in other words, there are
multiple IPs which will perform the function for that name -- then multiple
resource records are returned; one for each host. The name server will
typically randomise the order of these resource records on the assumption that
the client will select the first one, as this provides a basic way of balancing
load amongst the nodes.

Awesome. At last. We have an IP address to talk to in order to request our web
page. Fortunately, in reality, this operation takes a single network round trip
to the nearest host that has an answer, so usually only takes a few
milliseconds.

## HyperText Transfer Protocol

## Keywords I want to incorporate

* Proxy caches
* DNS resolution.
* HTTP
* TLS
* Next protocol negotiation and SPDY.
* Gateway nodes, internet routing, routing protocols.
* Ethernet, Wifi, CSMA-CA and CSMA-CD.
* TCP for HTTP, UDP for DNS. Pros/cons.
* Name resolution at each layer: ARP.
* Packet formats, encapsulation.
* Fragmentation and reassembly. Ordering guarantees.
* Server side architecture -- composed services, communication with back end
  services, providing resilience, redundancy, floating IPs.
* Browser rendering, JavaScript, CSS, rendering HTML.
* NAT.
