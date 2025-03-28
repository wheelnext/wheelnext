# Workshop - Python Packaging Governance

| Resource            | Link                                                                                                             |
| ------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `PEP Link`          | <https://peps.python.org/pep-0772/>                                                                              |
| `DPO Discussion`    | [PEP 772, Packaging governance process](https://discuss.python.org/t/pep-772-packaging-governance-process/79724) |
| `Github Repository` |                                                                                                                  |

## Abstract

[PEP 772](https://peps.python.org/pep-0772/) proposes to formalize Python packaging governance.
Packaging is extremely important to Python's past, current, and future success, and needs a formal,
elected decision-making body in order to prioritize and evolve packaging standards, tools, and
services.

PEP 772 is largely modeled on [PEP 13](https://peps.python.org/pep-0013/), the governing document
establishing the Python Steering Council, which is responsible for the evolution of the Python
language, the CPython reference implementation, and the standard library.  The Python Package
Authority (PyPA) was established via [PEP 609](https://peps.python.org/pep-0609/), which does not
have a community elected decision making body, with ad hoc meeting.  The PyPA provides opinions,
insights, and feedback, on packaging standards and decisions, but doesn't itself make binding
decisions.  The PyPA is defined by the group of projects under its umbrella, rather than elected
individuals with mandates to make such binding decisions.

There is also a [Packaging Working Group](https://wiki.python.org/psf/PackagingWG) operating under
the authority of the Python Software Foundation, tasked to support the larger efforts of improving
and maintaining the packaging ecosystem in Python through fundraising and disbursement of raised
funds.

Meanwhile, the Steering Council has established [standing
delegations](https://github.com/python/steering-council/blob/main/process/standing-delegations.md#pypa-delegations)
for decisions over aspects of the packaging ecosystem:

* Package Distribution Metadata PEPs: Paul Moore
* Package Index Interface PEPs: Donald Stufft

These mostly informal arrangements lead to several problems, which PEP 772 attempts to address:

* While the individuals with standing delegates are clearly highly experienced experts, the
  packaging ecosystem is much too diverse to burden any single person with all the decision making
  responsibilities.  A larger elected body reduces the [bus
  factor](https://en.wikipedia.org/wiki/Bus_factor) and ensures a more robust and resilient
  governance structure.
* As with any single apex for decision making, biases, time constraints, and a narrow focus may lead
  to delays in critical decision making or decisions that miss important requirements from areas
  outside their area of expertise.  A larger elected body will naturally have a more global view of
  the ecosystem.
* With only standing delegations from the Steering Council as the basis for their authority, the
  decision making is less accountable to the vast Python packaging community.  While the current
  standing delegates are all well-known and respected leaders in Python packaging, their priorities
  may not always reflect the priorities of the wider community.  A larger elected body is, by
  definition, accountable to the voting members by way of the democratic election process, and the
  voting membership is by proxy accountable to all stakeholders in the Python packaging community.

To address these and other problems, PEP 772 proposes to formalize packaging governance under a
Packaging Council, with a structure largely modeled on PEP 13's establishment of the Python Steering
Council.  The Packaging Council will consist of five elected members (in two rotating cohorts) which
will have broad authority over packaging standards, and the [Python Packaging User
Guide](https://packaging.python.org/), as well as a mandate to establish processes for making
binding decisions over packaging tools and services.  The Python Steering Council would then change
its standing delegations to give authority to the Package Council for decisions within the packaging
ecosystem, and the PSF Board would be encouraged to formally deprecate the Packaging Working Group.
While the PyPA would -- at least initially -- continue to exist, both the Packaging Council and the
PyPA would be tasked with establishing decision making processes for projects under the PyPA
umbrella.

## Workshop Goals

During the [DPO
discussions](https://discuss.python.org/t/pep-772-packaging-governance-process/79724) a number of
important issues were identified. These are currently under discussion among the PEP authors, with
an update expected to be posted in the next few weeks.  The intent of the PEP 772 authors is to have
a PEP ready for Steering Council pronouncement and PSF adoption by the time of the [2025 US
PyCon](https://us.pycon.org/2025/) conference.

While the Wheel Next summit isn't directly tasked with providing input into these issues,
because of the nature of its participants, the summit provides a unique opportunity to gather
important feedback on these topics.  It's anticipated that many of the participants of the Wheel
Next summit may eventually become voting members of packaging community, and possibly Packaging
Council members themselves.

## Summary of the Issues

During this workshop, we will explore and discuss several key points:

* **Initial Seeding of the Voting Membership:** This is probably the most critical issue to resolve
  before the PEP can be pronounced upon.  Much of the evolving responsibility of the Packaging
  Council will be determined over time, including how the voting membership will grow to be fully
  representative of the Python packaging community, while still maintaining a functional voting membership.
  However, the initial seeding of that membership is critically important.
* **Communication and Inclusion of the Wider Community:** The landscape of packaging stakeholders is
  vast and likely unknowable.  Included are individual package maintainers and producers, large
  corporate users, a multitude of package consumers, packaging tool builders, maintainers of systems
  and services such as PyPI and other third party indexes, etc.  Very likely, most of these
  stakeholders aren't involved in discussion on [DPO](https://discuss.python.org), and yet they will
  be directly affected by the decisions of the Packaging Council.  How does the Packaging Council
  ensure that their voices are heard, that its work is widely disseminated, and that communication
  is both effective and widespread?  Are there other forums that should be included in the
  discussions, or at least informed regularly to encourage their participation on DPO?

## Organizers

* **Organizer 1:** Barry Warsaw, NVIDIA / Python Core Developer / Python Steering Council member; barry@python.org
* **Organizer 2:** [Name, Company, Email - if want to share]
* **Organizer 3:** [Name, Company, Email - if want to share]

We look forward to your participation and contributions to making the Python Packaging Council
discussions more efficient and robust!
