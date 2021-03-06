---
layout: post
title: Infrastructure Automation with Puppet
---
I'm at Puppet Camp today, mostly discovering that the ways in which I (ab)use
puppet are not in fact all that crazy after all, and that I'm doing stuff in
much the same way as other people in the community. That's a relief. So maybe
it's time to start writing up some of my puppet work flow. There are also
specific problems I've solved in my various puppet environments; perhaps some
of them are of interest to others, too.

## What is Puppet?

But let's take a step back, first of all: what is puppet anyway? It's one of
the tools that attempts to solve the problem of managing multiple computers in
an efficient manner. This problem space is often referred to as configuration
management.

So, in turn, let's take another step back: what exactly is configuration
management? It's an attempt to scale the management of computer systems better
than linearly with the number of people required to manage them. If your ops
team are configuring, computers, storage and switches by hand, then the number
team members scales linearly with the growth of the number of devices that need
to be managed. This is typically considered suboptimal for a growing company,
particularly when the input growth curves (i.e. customers) are exponential.

How do we solve this problem, increasing the power that an individual
operations team member has to manage these devices? The same pattern as has
been applied to many other industries: automation. Instead of an operations
team member bringing up a new server, installing the operating system,
installing a bunch of packages, then hand crafting the configuration files, we
develop a system to automate this process. The improvements are many-fold:

* Faster deployment. If a human being is having to respond to prompts, type
  commands in, and wait for actions to be completed, it takes time. Computers
  are good at watching for something being completed and issuing the next step
  in the process.

* Correctness. That same human being, who's hand typing commands into a console
  is, well, human. And humans make mistakes. Computers are less likely to make
  mistakes.

* Repeatability. If you can perform an automated process once, you can perform
  it 10,000 times, and it should still work. And (if designed correctly), you
  can perform these operations concurrently. So an operations team member can
  bring up 10,000 servers simultaneously, in the same time as she can
  instantiate a single one.

* Consistency. I don't know about you, but I tend to learn new stuff over time.
  And I like to apply that new information when I learn it. This means that the
  next time I hand-craft a new server, I'll deviate from the 'standard' script,
  incorporating my new knowledge into this build. But unless I update the rest
  of the infrastructure with this new knowledge -- by hand! -- then the
  configuration of systems starts to diverge over time. That server deployed in
  2009 that's still critical to the business (usually a database server) has
  diverged wildly from the current documented way to configure a database
  server...

To be fair, here, this is old news. People have been automating the deployment
and update of servers for decades now. Operations people are, naturally, lazy.
They tend to have (varying levels of) programming experience, and so reach for
one programming tool or other to automate their process.

In my personal experience (having been one of these lazy operations people),
these automations usually take the form of shell scripts. A quick-hack bash
script to take a system from being pristine to fully configured for the desired
environment. Install some packages, make some clever use of `sed` & `awk` to
modify the configuration, then start the appropriate services.

In fact, one tack that I've taken in the past is to produce custom packages
that do all this for me. Install the package, and the package's post-install
scripting hooks take it from its pristine install to one that's suitable for
the environment.

That then reduces the manual install process to include installing the
operating system, and installing a pile of packages, each of which know their
dependencies, and know how to configure themselves into our environment. Job
done, right?

Well, it's better than nowt. The next piece of the puzzle. Bootstrapping
systems and installing those packages is still manual. Another piece to the
jigsaw is to automate the provisioning and installation of these systems. Back
when I was doing this stuff in anger, Solaris had a tool called JumpStart. You
had a central JumpStart server which had an association between MAC addresses
(the unique address of the network interface in a computer) and an installation
configuration.

A new server had just enough 'intelligence' baked into the hardware to say
(through the magic of RARP and TFTP if you're interested!) "I have this MAC
address. Tell me what to do next!" The JumpStart server would respond and say,
"Here, have this IP address for now so that you can communicate with the
network. Now, have this install image with this parameters. Go ahead and run
that, you'll have enough information to provision yourself from there."

Every other major operating system (or distribution, in the case of Linux) has
a similar mechanism. RedHat calls is Kickstart, giving away the influence of
its roots. Debian (and Ubuntu) call it **I can't remember**. FreeBSD, AIX, and
even Microsoft Windows have similar mechanisms.

These days, when most of us are primarily concerned about clouds of virtual
machines, the process is a little different. Instead of the new system being
told, "here's some install media and a configuration file", it gets booted with
some gold master installation, customised slightly on first boot by
configuration meta-data provided by the cloud and then goes on its merry way. If
the base image is still a pristine installation, then the principle is roughly
the same. It's just a little quicker. :)

Better still, we can preconfigure these golden master images so they already
have the correct set of packages installed, already have most of the bootstrap
scripts run on them, just require a smaller set of bootstrap tweak scripts to
run when they're first instantiated.

So we've now got a system whereby we can spawn a new server, automatically
install the operating system, install our custom packages, and run some
scripts. Now it's a fully fledged, configured, member of our fleet of servers.
Job done, right?

If you consider your server infrastructure to be immutable (i.e. once it's
instantiated, it cannot ever be changed), then yes, you might be right.

The real world tends not to be quite so neat. Servers have side effects
(otherwise, what are they doing?). Even in the trivial example of an HTTP load
balancer in the form of Nginx. I might be able to neatly model its primary
purpose as a pure function:

    response = forward(request)

but that still produces a number of side effects:

* logging information about the request it received and the response it made;

* metrics about request and response times; and

* discovering that a back end service is actually down after timing out,
  forwarding the request onto another backend to get the desired response, but
  also marking that backend as 'down' internally so that other requests don't
  get sent to it.

And a load balancer is a trivial example (which I'm sure could be represented
with the IO monad in Haskell if one wanted to be particularly picky). What about
database servers that persist real data, or parts of the system that
asynchronously defer operations to other parts of the infrastructure?

That's just the main function, too. Servers have other state that they need to
take care of in order to be a responsible part of the Internet. Things like the
IP address and interfaces that they use to respond to Internet traffic. What
the current date and time is. Their capabilities relative to their peers. The
peers that they can communicate with, and how they communicate with those
peers. The operations that particular users, groups and applications are
allowed to perform. The versions of installed packages.

In principle, it is possible to take this golden master approach. Any time
there's a configuration change, you produce a new gold master for affected
systems. Then you spin up replacement systems that implement the new
configuration, and tear down the old ones. This is readily doable for some
parts of the infrastructure (the traditionally 'stateless' parts like
application servers, load balancers, etc. but it's not necessarily a good fit
for other parts.

Plus it seems wasteful to me, to be constantly recycling systems down to their
raw materials and building them back up from scratch. Could we not make do and
mend existing systems instead?

Of course, there are advantages to this approach. Systems are then typically
not long lived, so there are fewer opportunities for them to deviate from the
golden master. Any genetic mutations are going to be killed off next time the
golden master is updated. Instead of evolution and natural selection, we get
known good clones.

Let's make the assumption that we want to jump from immutable systems to ones
which change over time. (I'm not entirely sure I've convinced you, dear reader,
that this the right jump to make, but I'll work on that, and the rest of this
is basically pointless without that jump.)

Even when we've achieved this state of automated deployment, we still need some
mechanism to update the configuration of systems while they're running. Right
now, in the narrative, we're back to manual configuration.

When we decide that we need to change the Nginx configuration on all the
application servers -- due to some new knowledge, or an update to the
application we're serving, or to mitigate a security vulnerability -- it's back
to applying it by hand. Either ssh-ing in to each relevant host (let's hope we
didn't forget one!) and manually changing the configuration files or, if we're
able to, copying the new configuration file to each host. Oh, and restarting
the necessary services (or reloading their configuration files, in some
service-dependent fashion) so the new configuration takes effect.

It's particularly easy for human error to creep in here; forgetting to restart
a service so it takes on the new configuration, or fat-fingering a
configuration change causing the service to fail to restart in the first place.
Been there, done it, gotten the "Argh", *fix*, *hope my boss/the monitoring
systems didn't notice* t-shirt. (There needs to be a t-shirt for that.)

Or when we need to roll out some security updates. We're all responsible
security professionals, and have a measured response to security
vulnerabilities, right? My general process is to:

* Review the security vulnerability. If I'm confident that it doesn't impact
  any of our systems, then I step down and let it be taken care of in our
  regular package update process. There is a cost and a risk to updating
  infrastructure, so unless it's urgent, I like to defer it.

* If it could affect our production infrastructure (or, to turn it on its head,
  if I can't convince myself that it *couldn't* affect our infrastructure),
  then there are two further things to think about:

  * Risk. What are the chances of this being exploited? If it's an internal
    system with other layers of defence in front of it, then the risk is
    relatively low. However, if it's something on the public Internet,
    responding to requests that come in from the great unwashed, then the risk
    is high. (This is a good argument for keeping the plane of attack as small
    as possible.)

  * Severity. If this vulnerability was exploited, what are the consequences?
    If somebody's gained access to personally identifiable information (which
    the Data Protection Act gives me a legal responsibility to protect), then
    the severity is high. If it can affect the operations of the business, then
    the severity is high. If it allows somebody to prevent our developers from
    posting LOLcats on the company chat system, I'm less concerned.

* If the risk and severity are low, then it's probably back to relying on our
  regular package update process.

* If the risk and/or severity are high, then enact the regular package update
  process *right now*.

We'll get to the package update process later because without some sort of
automation, it boils down to:

     for each server:
       update packages on that server

The point is that, unless we've got something else in place, this is a manual,
error-prone, time consuming process.

What if there was a way to automate the update of systems? To be able to apply
a policy to servers, storage and switches, then allow the machines themselves
to converge on that desired state? That's essentially what configuration
management gives us.

## The standard solution

There is the standard solution, which is adopted by the operations team in most
organisations: identify the pain points, and write some code to automate them.
Each individual organisation has its own peculiarities, or preferences, or
constraints, so each organisation winds up with their own, custom, solution.
This has its advantages:

* in house solutions tend to *exactly* solve the business problem, or at least
  the current understanding of the business problem.

* they tend to be relatively simple, since they apply to a single problem
  domain and a relatively small set of solution domains.

* the team that builds the solution completely understands it.

This is awesome. It's the minimum viable product, and the team fully
understands what's been built. However, there are a couple of disadvantages:

* It's not leveraging existing expertise. There's a huge wealth of expertise
  and experience in the software engineering community. And the particular
  advantage of open source is that much of that knowledge and experience has
  been codified into open source software that you can reuse in your
  organisation. If you're reinventing the wheel internally, then you're
  typically making the same R&D investment as somebody else has, and you're not
  learning from their mistakes.

* There's team-specific knowledge. This makes life difficult for two reasons.
  Bob, who wrote the scripts in the first place becomes important and
  irreplaceable. If you're Bob, that might be a good thing. It's job security.
  It's an indication of authority, of seniority. If you're the business, then
  it's definitely a bad thing. You've got a single point of failure in Bob. If
  he gets hit by a bus, the company has a major problem.

  It's also difficult to hire people. People need much more domain knowledge
  before they can become useful, because not only do they need to learn the
  problem domain, they need to learn Bob's solution domain.

How do you solve these problems? Look to an existing product, of course. There
are several people who have already discovered they've got these problems, and
shared their solutions. And, for bonus points, they're in the open source
world, so we can leverage their solutions, too.

## The landscape

Of course, as with any different software problem domain, there are a dozen
solutions to the problem. There are two relative grandfathers in the open
source space:

* LCFG, the Local ConFiGuration system, developed at Edinburgh University in
  the early nineties by Paul Anderson to manage the growing collection of
  Solaris servers and desktops kicking around the computer science department.
  It was ported to Linux by Alistair Scobie later on, and became the system
  used to manage a heterogeneous network of Solaris, Linux (RedHat, Debian and
  Scientific Linux), SGI and Mac OS X systems used by students, staff and
  researchers. Computing environments in Universities tend to be difficult to
  manage, due to rogue agents (students, including me, having discovered
  tcpdump at one point) and the unique needs of researchers.

* CFEngine, developed by Mark Burgess. It came from a similar background; a
  physics graduate tasked with managing a bunch of servers and figuring there
  was a better way to do it. Since then, it's been an academic playground for
  the underpinning ideas of configuration management, introducing the concepts
  of Computer Immunology and promise theory (which, as it turns out, has many
  more general applications).

Personally, I have a soft spot for LCFG; it pre-dates my attendance at
Edinburgh University by a couple of years, but we used it heavily to power the
TARDIS project, and I was far more interested in it than the more academic side
of the Computer Science curriculum. (In retrospect, I really should have
realised that configuration management was a 'proper' branch of CS and devoted
my final year project to pitching in with LCFG, DICE (the 'Department of
Informatics Computing Environment') and what would become LCFG 2, instead of
hanging around on the edge of the system-onna-chip group, messing around with
Bluetooth and the Linux kernel.)

Since then, there has been a family tree of descendants from these two projects:

* Puppet was an ideological fork of CFEngine, when Luke Kanies, its founder,
  and still the most prolific contributor, got fed up trying to get new
  features into CFEngine. He'd made some major contributions to CFEngine, and
  had strong opinions on the direction in which it should go. Frustrated by the
  slow pace of academia, he instead built puppet and open sourced those ideas.
  Puppet is now a healthy commercial ecosystem, with Puppet Labs still funding
  much of the development of the open source project, while maintaining its
  commercial needs with an enterprise edition.

* Chef, which was an idealogical fork of Puppet. Adam Jacobs had a consultancy
  company building automated infrastructure for start-up companies with Puppet.
  He got frustrated with some of the design decisions of puppet and wrote Chef
  to alleviate those problems. While appearing similar on the surface, Chef and
  Puppet really do have differing underlying core beliefs, particularly about
  the split of responsibilities between the server and client.

Then there are the new kids on the block, like Ansible and SaltStack and Serf.
I should spend some time with them at some point, figure out where they differ
and where they're aligned.

## My personal history

Personally, I've played around with a number of these tools. I've helped manage
the TARDIS project with LCFG. I also had a rack of SparcStation 2 computers in
my flat as a student, running Solaris 2.6, all managed with LCFG, NIS and
optdepot for packages. (It was awesome, though my girlfriend at the time was
never impressed with the noise of whirring fans. On the plus side, she's now my
wife, but we no longer have half an ISP in the corner of the bedroom. No,
instead we have kids, which are much louder than server fans!)

In my first graduate job, having at the time, temporarily, decided I wasn't cut
out for software development (I spent the first two years in that job buggering
around with low level details of hardware to enable DMA transfer of voice data
from DSPs to the network card, and writing NDIS drivers for Windows NT; you may
have had the same thoughts), I built and deployed a cluster of machines with
CFEngine to support a development team, including:

* user authentication, including LDAP for distributing user information and
  Kerberos for managing authorisation;

* file sharing, with NFS & Samba;

* email services (IMAP for reading, SMTP for sending), back in the good old
  days when you did email by yourself;

* the usual responsible office network stuff, like DNS, NTP, a transparent web
  proxy/cache; and

* a safe place for the developers to test their network protocol stuff without
  interfering with anyone else.

It was a fairly small team, but I got the opportunity to experiment with a
whole pile of stuff, especially network protocols, routing, network management
and, of course, CFEngine. In retrospect, I should probably have been more
grateful to my team leader at the time for indulging all my experiments... ;-)

Since then, I jumped back to development, and kind of forgot about my
operations background. I even caught myself configuring servers by hand (who
cares, there were only two of them, and I'm a developer, I've got more
important things to think about). Then I got involved in a project in 2009
where I had the opportunity to manage a few servers, and the prospect of this
project growing at the sort of rate where efficiently scaling would be
desirable.

The cool kids were talking about Chef at the time, and having been a bit rusty
about my configuration management chops (as well as being heavily involved in
the Ruby ecosystem at the time), I picked on it as the answer to all my
problems. So I picked Chef as being the solution and built an infrastructure
out with it.

Since then, I've built a Chef infrastructure out for one other company, which
sadly didn't scale in the way we'd hoped. And I've tried it out with another
couple of projects.

The last of these was a disaster recovery project. The idea was to be able to
tick a box which was worrying one of the investors. If the entire
infrastructure crashed -- say the single provider we were with, in a single
data centre, was nuked from orbit (it's the only way to be sure) -- how could
we recover? The answer to my mind was to quickly spin up replacement
infrastructure on our cloud provider of choice. The choice happened to be
Amazon Web Services. So I spent a few weeks building up a chef architecture
that would allow us to run the application from AWS, restoring data from point
in time backups.

It worked, eventually, but then we hired in folks with explicit operations
experience, and the tool of choice was Puppet.

My general take on these things is that most tools will solve the business
problem, and most of them will do a good job of it. And it's hard work keeping
a bunch of alternative methodologies in your head when trying to do different
projects. (That's why Ruby is often my hammer. Maybe another language would be
slightly more elegant, but then I'd have the cognitive shift to the solution
domain for the project, as well as the cognitive shift to the problem domain.)

So I shifted to Puppet. I swear I'd used puppet in the past, but for the life
of my I can't remember what the project was. Still, it felt familiar, but was a
sight easier to install (i.e. bootstrap) than it used to be.

Since then, I've built Puppet infrastructure for several different projects.
It's great that once you get the 'base' repository -- the capability to
bootstrap new nodes into the cluster, and the basic set of services that a
responsible Internet server citizen should have -- then it become easy to make
changes and add new services and build out the custom infrastructure that a
particular client needs.

## Features

All these configuration management systems have a common set of goals:

* A convergent configuration for systems. This is the fundamentally important
  idea. The architect of the system defines the ideal configuration state for
  the system, and each of the component parts of that system attempt to
  converge towards that ideal. It naturally follows that these systems are
  idempotent. Applying the same policy to these components should converge them
  towards the same ideal goal.

* Abstraction of the 'how' from the 'what'. Computers are funny things. While
  we have a fairly common concept of the what -- users, packages, running
  services, files -- how each operating system fulfils those is a bit wacky. To
  install a package on Ubuntu or Debian, you'll want to execute `apt-get
  install <package name>`. For RedHat and derivatives, you'll want to use `rpm`
  to install a subtly differently named package. On Solaris, well, Solaris is
  always special. If your package on Debian and RedHat is 'postgresql-server',
  then it might be something along the lines of `SUNWpsql` instead. And the
  command required to install the package might be `pkgin`, `blastwave` or
  something else entirely depending on your environment.

* Separation of the 'what' from the 'how'. Conversely, individual packages, and
  sets of packages that provide a service, have configuration files. It can be
  useful to separate the meaning of some configuration (i.e. why this
  particular high level policy decision is being made in this environment) from
  how that's implemented across a pile of configuration files.

* Introspection of the running environment. Almost all of the configuration
  management tools I've experienced have the ability to inspect a bunch of
  aspects of the machine they're running on, and make decisions based on that.
  For example, if you've got a database server with 4GB of RAM, then you'll
  want to configure it differently from a database server with 16GB RAM and a
  stash of SSD on the side.

* Most systems have the idea of separating code and data. This is common tack
  in development, but less pervasive in operations. Fundamentally, what
  separating code from data allows is reuse. While my environment is specific
  to me -- and so the data, the versions of software, the IP addresses, the
  constraints, the availability requirements -- are specific to me, the code
  that drives these can be common, can be shared. Particularly since the
  implementations of particular resources are already abstracted, so we're able
  to reuse these abstract concepts along with data that's specific to the
  environment.

Each have their own specialisations, too, but they are all intent on the main
goal: converging your system's configuration towards the policy ideal that
you've declared.

## Benefits

Why is this a good idea? What benefits will it actually bring to your
organisation? (Other than you having this warm, fuzzy glow that you're doing
the 'right thing'?)

## Impact

That's all well and good, but what difference do these benefits actually make
to my bottom line?

## Evidence

Who else has had their bottom line impacted by this sort of evidence?

## So, Puppet?

Now we've got a broad idea of what configuration management is, and how it fits
in (to my personal history, as well as the overall ecosystem -- thanks for
humouring me there), what about Puppet? Puppet is one of the relatively mature
solutions in the space. It's got the hereditary benefits of Luke Kanies having
been involved in the configuration management space for so long. It's built in
the beautifully expressive language of Ruby (OK, I'm biased), has its own
declarative language, can manipulate dozens of resources, supports dozens of
platforms, and is pretty easy to extend.

It's also evolved massively over the years, so the Puppet of three years ago
(0.26.x series) is only superficially like the Puppet of today (as I'm writing,
3.2.x). There has been a strong ethos of maintaining backwards compatibility,
so it still has the occasional interesting behaviour (see 'the anchor pattern'
when I get to it), but for the most part, it's a mature and expressive
configuration management system, with rich introspection across the cluster of
nodes.

Roughly speaking, a modern Puppet infrastructure has a number of key
components:

* `facter`, which will introspect a node and provides local configuration
  information back to the puppet master so that it can construct a catalogue.

* The puppet agent. This takes a compiled catalogue from the puppet master and
  applies it to the local node. It then reports the results of having applied
  the catalogue to the node back to the puppet master.

* The puppet master, which takes local configuration information and other
  cluster state, then compiles a catalogue for each node which it can then
  apply. It also provides authentication and authorisation services, defining
  what nodes can be trusted to take part in the cluster.

* PuppetDB, which is the back-end persistent store for the cluster. It collects
  `facter` information from each node, reports on agent runs from each node,
  and any exported resources from each node. It then shares that information
  with the rest of the cluster, and provides an API to access this information.
  Essentially, it's the oracle of how a puppet cluster is performing, how it's
  currently configured and how the state of the cluster has changed over time.

* `mcollective` is a separate, but related, component which allows for command
  and control of the running cluster.

* Hiera is a mechanism to separate out the configuration from the code of the
  puppet modules. This way you can have shared modules that apply desired state
  throughout the cluster, but tweak the individual configuration based on
  different criteria.

Let's examine each in turn a little more.

### Facter

Facter's role is very simple. It provides an abstract mechanism to report a
number of standard 'facts' about a node. Things like the IP address, the kernel
version, how much memory the system has. Information about the hardware, the
software and the node's surrounding environment. This is the first of the
abstraction layers we'll come across and here it's on the side of reading
information. It separate the 'what' from the 'how'.

The 'what' is some standard, abstract piece of information (like, for example,
the IP address of the primary network interface). How one actually retrieves
that information is definitely operating system specific. On Linux, you can
probably get at it by parsing the output of `ifconfig` or, on more modern
distributions, the `iptools` package is probably more flexible, if the `ip`
command is available.  On Solaris ... OK, on modern 'Solaris', you're more
likely to get the information from parsing the output of `ipadm`. On Windows?
Haven't a scoobie, but somebody does, and can provide an implementation that
provides the right information.

So facter is a powerful tool that feeds into the heterogeneous nature of
configuration management tools. The higher level tools only need to care about
the 'what' of the information that's being collected about a host, not 'how'
its collected. This is pretty cool.

It's exactly the same sort of pattern as we see with Board Support Packages
(BSPs) in the embedded platform world. The operating system/compiler has a set
of primitive data that it relies on from the underlying hardware. For the most
part, it doesn't care how that data is supplied. The BSP on the other hand
knows that it needs to implement a particular set of interfaces in order to
supply the operating system with the right information. It implements the
primitives required by its esoteric hardware to make that happen.

We see the same pattern all over. (In fact, we'll see the same pattern with
commands as well as queries, shortly. Suffice to say it's a good pattern to
work with.) Coding to interfaces is useful, primarily because it turns the
implementation required from the order of `m x n` to `m + n` (where `m` is the
number of variables we need to introspect, and `n` is the number of
heterogeneous system types we'd like to support).

Facter gives us useful information about the running node, and its current
state, in order to help us converge on a sensible set of configuration for that
node. The way that Puppet is designed, the node is not permitted to make its
own decisions about the configuration that should be applied to it, so the
facter information is communicated back upstream to the puppet master.

### The puppet master

The Puppet Master sounds like a terribly important role, and sounds a lot like
a central point of failure, doesn't it? It used to be this way -- there was
typically one master, which was the gatekeeper to the knowledge of the entire
cluster's state.

It's less that way now, particularly since PuppetDB came on the scene. The role
of the puppet master now is two fold:

* Authentication and authorisation; and

* given information from facter on the client, and the holistic knowledge of
  the cluster, compiling the minimal catalogue that the node needs to know in
  order to converge its local configuration to the global optimum of the
  cluster.

#### Authentication and authorisation

Let's take authentication first because, unusually, it's the easy bit.

Puppet helpfully provides us with a mechanism to manage a public key
infrastructure (PKI). If you trust, and can scale with, Puppet's PKI mechanism
then it can be the PKI for your entire organisation, and power the secure,
trusted communication amongst other service, too. Which is pretty cool.

But let's take a look at how puppet authenticates and authorises members of
that puppet cluster.

The puppet master, by default, creates its own certificate authority. This is
the centre of a web of trust. Any client certificate signed by the puppet
master's root certificate is trusted as being part of this cluster and the
puppet master will supply catalogues of the current desired state to it.

When a node first attempts to join a puppet cluster, it will present its public
key to the puppet master. The puppet master needs to sign, and return, this
public key before the client can actually retrieve a catalogue from the puppet
master. So there's no way for an untrusted node to infiltrate the cluster
without explicit permission.

When a new node is presented to the puppet master, it has to make a decision:
can I trust this node, or do I need a human being to verify it on my behalf?
This is controlled by a configuration file on the puppet master. Essentially,
in my experience, there are a couple of options:

* The puppet master is already protected from rogue clients by another
  mechanism (usually a firewall) and so anyone who can communicate with the
  puppet master is OK. This is trading off defence in depth against
  convenience, but in some environments (e.g. development) can be OK.

* Manually approving every new node at the puppet master. This involves logging
  into the puppet master, running `sudo puppet cert list` to see the new node,
  verifying the fingerprint, then signing it with `sudo puppet sign <node>`.

When you're managing a large number of nodes, I'm sure there's a certain
scalability vs convenience trade off that one has to make, but I've (sadly) not
yet been in that situation.

As with any respectable public key authorisation system, Puppet is also
perfectly happy revoking permission to be part of the cluster. This is achieved
through a standard mechanism with PKI, providing a mechanism to revoke
certificates, and maintaining a certificate revocation list. When we're decided
that a particular node should no longer be a responsible part of the cluster,
then we tell that to the puppet master, by revoking its key. Once that's
happened, the node no longer has permission to talk to the cluster, so any
requests from its agent to get a current catalogue are refused.

Of course, any application-specific permissions need to be managed
independently, but if you happen to use the Puppet PKI infrastructure for other
purposes, then revoking a node's access to the Puppet master also has the neat
side effect of preventing it from having access to other services too. This can
be pretty powerful. Revoking a puppet certificate automatically revoking a
node's access to the shared OpenVPN network, and preventing it from logging to
the shared log server, for example, can be very useful.

#### Creating Catalogues

The most important role of the puppet master is to create a catalogue (which is
the stuff the client needs to apply to make sure it's converging towards its
ideal configuration.

There are a few things taken into account when providing this canonical
catalogue of state:

* The facts from the node itself, having just been reported by `facter` and
  passed up to the puppet master from the agent.

* The overall desired configuration, as specified by the current state of the
  puppet configuration. This is, most often, resolved from the current state of
  some branch/environment of the puppet configuration's source control
  repository.

* The current state of the rest of the cluster, as persisted by puppet's stored
  configuration back end (if it exists).

The combination of this information gives the puppet master the ability to
provide a canonical catalogue of state that the client should currently want to
converge to.

There's a distinct advantage to making the puppet master responsible for
compiling and delivering the catalogue. We might trust the client a bit. But
that's not to say we trust it entirely. The client is still treated as suspect.
It only gets as much information as it needs to do its job. That's why we
compile and minimise the information it needs to know on the puppet master. If
an application server doesn't need to know about the bits of the cluster that
configure the database servers, then why should we leak that information to
them?

As well as being a security concern, it helps with performance on the client
nodes. If a node only has to consider the components that apply to it, then it
can verify its desired state more quickly.

On the other hand, it makes the puppet master work so much harder, because it
has to deliver a bespoke catalogue to each node. Which often, in practice,
makes the puppet master the slowest node, because it's computing a rather
complicated catalogue for each node, and it's doing that on a frequent basis.

On the plus side, when we have a stable infrastructure, the changes to that
infrastructure are infrequent (relative to the poll for changes, at least). We
get to take advantage of lower level problems (for example, when the puppet
master sends back a 304 saying 'nothing has changed since last time you asked',
then it's simpler to check most recently cached catalogue and see what's
changed, without putting load on the puppet master).

But that's fundamentally the role of the puppet master: to figure put who is in
which silo, who is allowed to know what, and supply them with that information.
It also has a secondary role of receiving reports about how a catalogue has
been applied, and stashing that information appropriately.

It's funny, as I write about it, I realise that the puppet master serves two
different paradigms:

* One is delightfully pure-functional. It takes the current cluster state
  (exported resources), the current node state (from facter), and the overall
  declared state (from configuration) and produces some new state for the
  clients. It's not the canonical source of any of thsee, so it's entirely
  stateless in that regard.

* The authentication and authorisation stage. Irritatingly, it is providing,
  and storing, state here. Which is often the trouble with scaling puppet
  masters; how do you distribute and scale the PKI bits? Perhaps its a sign
  that the cluster needs to delegate that to a third party service?

# Notes on what I want to talk about

Before I get too carried away with writing, here's, roughly, some of the things
I want to talk about in this piece:


* Features, Benefits, Impact & Evidence

  * Benefits:

    * Security

    * Safety

    * Control

    * Auditing

  * Impact:

    * Reduce downtime (both planned and unplanned)

    * Increase observability and compliance.

    * Reduce staffing costs.

  * Evidence:

    * Well, that's harder from my personal perspective. ;-)

* Component parts of a puppet infrastructure:

  * Puppet itself:

    * The puppet language.

    * The puppet agent: a transactional engine to apply policy to a node.

    * The puppet master: compiling and distributing catalogues to the puppet
      agents.

  * PuppetDB:

    * a store and API for information about your running cluster

    * Drives UIs for reporting on the state of the system.

  * mcollective for command and control.

  * Heira for separating code and configuration.

  * Facter: data collection.

* Puppet Development environment with Vagrant

* Bootstrapping

* More on how Puppet works:

  * Nodes

  * Classes

  * Modules

  * Resources

  * Providers

* Puppet & Git work flow

* Rolling out updates with Puppet and mcollective.

* Nginx upstream configuration with puppet exported resources.

* My pattern for dealing with additional upstream package repositories.

* Puppet annoyances

  * Performance

  * Language niggles


