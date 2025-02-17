# Philosophy and Design Space

## Foreword

Any proposal, solution, or technical implementation is built upon certain foundational assumptions -
what we refer to as "design axioms". These are core ideas, principles and mantras guiding our
thinking processes, design and engineering. It helps to evaluate the feasibility of different
solutions and their respective effectiveness.

We believe that clearly stating the ideals and principles guiding our proposals helps focus our
efforts, and establishes a shared understanding of the reasoning behind our suggestions.  By framing
the discussions around these core principles, we hope that it will be clear that given the myriad of
packaging issues we *could* try to improve, why it's *these* issues we are spending our time and
energy on.

Discussions on these principles themselves is of course welcome!  We always want to ensure that our
foundational ideals are consistent and relevant, and lead to outcomes that benefit the wider Python
community of packaging producers, consumers, users, and tool makers who care deeply about the
experience.

## Design Axioms

### Evolution Not Revolution

The Python packaging ecosystem has evolved over many decades, with much good work, contributions,
healthy discussions, user studies, and improvements along the way.  We recognize that the landscape
is vast and the user population is huge.  It takes a long time for changes to percolate through the
long tail of tools, services, and workflows that form the everyday basis of people's lives.

Therefore we strongly believe in the ongoing evolution of packaging standards and solutions, not a
wholesale revolution in the way people and tools operate.

### If you don't care now, you won't care later

For *many* use cases and users, Python packaging is "good enough". Maybe they are building
libraries and applications that don't ever touch the "Accelerated Python" ecosystem.  Python is used
in so many domains and at such widely varying scales, from one-off scripts to large, long lived
application platforms, and more.  The niche of problems that WheelNext is trying to solve simply
don't affect everyone.

Therefore we strongly believe that if you are one of those users who doesn't care now about the
problems WheelNext is trying to solve, you will *still* not care when the solutions being proposed.
In other words, to you, "if it ain't broke, it ain't gonna be fixed".

### Don't focus on a single tool or service

Despite the overwhelming popularity of `pip` and `PyPI`, these aren't the only games in town.  For
example, `uv` has a `pip` [compatible interface](https://docs.astral.sh/uv/#the-pip-interface) which
is gaining popularity, and there are many alternative PyPI compatible services, some open source and
some commercial.  Whenever possible, try to propose solutions that can be adopted across all
relevant tools, and think about the level of effort to get other indexes or installers on board.

### Avoid mission creep

We all have our pet peeves and complaints about packaging.  Regardless of how you interface with the
packaging ecosystem, it's pretty much guaranteed that if you do, there are things that bug you about
it.  WheelNext cannot solve all these problems, and to attempt to will dilute the limited bandwidth
and resources of its participants.  There are also plenty of ways to improve things outside the
scope of WheelNext, and we encourage you to do so!  Let's keep WheelNext focused on the problems
we can improve and resist the temptation to fix the world.

## Principles of participation

### Be fully open, transparent, and collaborative

All work for the WheelNext solutions is done in the open, with full transparency and collaboration,
adhering to the spirit of Open Source.  We welcome participation from all stakeholders and
interested parties.  We strongly believe that solutions which represent the widest possible
consensus will yield the best results, and the most widespread and quickest adoption across the
ecosystem.  We'll rely on all participants to help improve proposals, implement prototypes and
production quality code, and perhaps most of all adopt and *spread the word* about how these
solutions improve *your* projects and the lives of your users.

### Stay positive and keep things moving

It's simply the nature of things for discussions on [discuss.python.org
(DPO)](https://discuss.python.org/c/packaging/14) to go on seemingly forever, with long threads that
are difficult to get resolution on.  This can be demoralizing, but we encourage you to stay
positive!  We are making a difference!  Try to avoid putting too much [Stop
Energy](https://radio-weblogs.com/0107584/stories/2002/05/05/stopEnergyByDaveWiner.html) into
discussions and instead focus on Forward Motion.

Most discussion threads often seem to lose steam and the longer they go on, the more difficult it is
for people to keep up on where things stand.  Plan to inject timely updates to keep the momentum
going.  Suggestions include posting periodic summaries of resolved and open issues so that people
don't have to scroll all the way to the beginning of a long thread to catch up.  Keep good notes of
the changes you're making based on the discussion, and post updates outlining the changes.  Plan on
posting regular status reports about proofs of concept, prototypes, demos, and working code that
folks can try.  Video demos are nice too!  If you've made significant changes to a proposal, don't
be afraid to start a new thread to "reboot" the topic.

### Code of Conduct

The Python Software Foundation itself publishes a [code of
conduct](https://policies.python.org/python.org/code-of-conduct/) which the Python Packaging
Authority (PyPA) [also adopts](https://www.pypa.io/en/latest/code-of-conduct/).

We strongly believe that it is of the utmost importance to treat each other with respect, and assume
each other's best intentions, *especially* when we disagree!  Disagreement can be healthy for
reaching consensus on the best solutions, given the complex and myriad constraints we're working
with.
