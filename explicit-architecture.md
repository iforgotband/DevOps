# What is Explicit Architecture?

Explicit Architecture was proposed by [Herbeto Graca](https://herbetograca.com)
as a method and structure to work as efficiently as possible. The setup
and idea are new, but the tools and concepts used in it are production
worthy and thoroughly tested.

## The Pieces

The code that is internal, the code that is external, and the code linking
the two together are viewed as three entities with _explicit_ separation. Three fundamental blocks of code are identified:

1. The user interface, which includes anything using the core.
2. The logical core of the application, used by the UX.
3. The code connecting the first to the second, or "Adapters".

The part we want to be focusing on for the most part is that second one. The
engine that drives the car that is the application. The application itself
should be agnostic and explicitly separated from whatever UX is interfacing
with it, and whatever additional outside tools (database, apis, etc) are
used in the chain.

Because the potential of the human mind to conflate similar, but different,
ideas, it's useful to think of the UX as containing all that asks the
core application to work. The external tools can be thought of as those
things that our application asks for work from.

`UX <=> Application Core <=> Infrastructure`

## Adapters and Ports

To accomplish our goal of a neatly structured development environment,
we need to use the right tools for each job in the first place. If we're
connecting to a database, it needs to be the best database to fit the
project, and the same of course for the rest. After that's been done,
we're ready to look at connecting everything together.

Adapters ([Ports and Adapters Architecture](https://herbertograca.com/2017/09/14/ports-adapters-architecture/)) are the units of code
that impliment the code that communicates between the application
core and the external tools it uses. The adapters that are telling
our application what to do are called Driving Adapters, while the
ones that are told to act by our application are Driven Adapters.

The adapters are built to fit a specific entry point of the core,
called a port. A port is a specification of how a tool can either
use, or be used by, the application core. Ports are generally an
interface, and they are grouped in the logical core of the
application. Ports have to be tailor-made to fit the core and
its specific needs.

### Driving Adapters

Also called a Primary adapter in the original documents, a driving
adapter interfaces with a port, and uses it to direct the core. 
A driving adapter will translate whatever is thrown at it from
a delivery mechanism into a specific method in the core. 

GUIs, APIs, Admin GUIs, CLIs - anything that interfaces betwen the
user and the application is going to be a driving adapter.

### Driven Adapters

Conversely, Driven adapters are also called Secondary adapters.
I've chosen to write this adaptation using the Driven/Driving
terminology because the mental image seems more accurate. Whereas
a driving adapter wraps around a port, a driven adapter 
_implements_ that port interface, and are injected into the core,
wherever that port is supposed to be injected.

A database-specific driven adapter would implement the needed
interface to save an array, or change a table. Let's say we've 
chosen to use MariaSQL in the past and built its adapter. The 
landscape has changed to where PostgreSQL is more advantageous. 
We then need to write an adapter to implement the interfaces
specific to PostgreSQL and drop it in place. The separation of
the three building blocks makes this possible.


