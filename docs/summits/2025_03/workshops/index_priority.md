# Workshop - Installer Index Priority

## Abstract

In this workshop, we will focus on brainstorming about the best way to deliver the
Python [Index Priority proposal](/proposals/pep766_explicit_priority_choices.md), which addresses the behavior of Python package installers when handling multiple indexes. And at the same time addresses security risk with associate with usage of `--index-url` option.

## Rapid Summary of the Solution

- **Version priority:** This behavior is characterized by the installer always getting the “best” version of a package, regardless of the index that it comes from. “Best” is defined by the installer’s algorithm for optimizing the various traits of a package, also factoring in user input (such as preferring only binaries, or no binaries). While installers may differ in their optimization criteria and user options, the general trait that all version priority installers share is that the index contents are collated prior to candidate selection.

- **Index priority:** In index priority, the resolver finds candidates for each index, one at a time. The resolver proceeds to subsequent indexes only if the current package request has no viable candidates. Index priority does not combine indexes into one global, flat namespace. Because indexes are searched in order, the package from an earlier index will be preferred over a package from a later index, regardless of whether the later index had a better match with the installer’s optimization criteria. For a given installer, the optimization criteria and selection algorithm should be the same for both index priority and version priority. It is only the treatment of multiple indexes that differs: all together for version priority, and individually for index priority.


## Points of Discussion

Several elements of the proposal require thorough analysis and discussion. We aim to keep the conversation focused on user experience while minimizing technical details.

- **[Discussion Topic 1]:**
    - [Sub-topic or Detail 1]
    - [Sub-topic or Detail 2]

- **[Discussion Topic 2]:**
    - [Sub-topic or Detail 1]
    - [Sub-topic or Detail 2]

- **[Discussion Topic 3]:**
    - [Sub-topic or Detail 1]
    - [Sub-topic or Detail 2]

- **[Discussion Topic 4]:**
    - [Sub-topic or Detail 1]
    - [Sub-topic or Detail 2]

## Expected Deliverables

By the end of the workshop, we aim to achieve the following deliverables:

- [Deliverable 1]
- [Deliverable 2]
- [Deliverable 3]
- [Deliverable 4]

## Useful Links

- [Link Title 1](https://wheelnext.dev)
- [Link Title 2](https://wheelnext.dev/)
- [Link Title 3](https://wheelnext.dev/)
- [Link Title 4](https://wheelnext.dev/)

## Organizers

- **Organizer 1:** [Name, Company, Email - if want to share]
- **Organizer 2:** [Name, Company, Email - if want to share]
- **Organizer 3:** [Name, Company, Email - if want to share]

We look forward to your participation and contributions to making [Subject of Workshop] more efficient and robust!
