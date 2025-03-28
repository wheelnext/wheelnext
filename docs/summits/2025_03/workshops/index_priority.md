# Workshop - Installer Index Priority

## Abstract

In this workshop, we will focus on brainstorming about the best way to proceed with the
Python [Index Priority proposal](site:proposals/pep766_explicit_priority_choices.md), which addresses the behavior of Python
package installers when handling multiple indexes. The heart of this proposal is enabling users to choose whether trust
of indexes is more important to them than a result that is a closer match for their provided package specs. This would
also reduce security risk with dependency confusion attacks in a way that is compatible with [PEP 708](https://peps.python.org/pep-0708/).

## Rapid Summary of the Solution

- **Version priority:** This behavior is characterized by the installer always getting the “best” version of a package,
regardless of the index that it comes from. “Best” is defined by the installer’s algorithm for optimizing the various
traits of a package, also factoring in user input (such as preferring only binaries, or no binaries). While installers
may differ in their optimization criteria and user options, the general trait that all version priority installers share
is that the index contents are collated prior to candidate selection.

- **Index priority:** In index priority, the resolver finds candidates for each index, one at a time. The resolver
proceeds to subsequent indexes only if the current package request has no viable candidates. Index priority does not
combine indexes into one global, flat namespace. Because indexes are searched in order, the package from an earlier
index will be preferred over a package from a later index, regardless of whether the later index had a better match
with the installer’s optimization criteria. For a given installer, the optimization criteria and selection algorithm
should be the same for both index priority and version priority. It is only the treatment of multiple indexes that
differs: all together for version priority, and individually for index priority.

- **Mirroring:** The index priority scheme breaks the use case of more than one index url serving the same content. Such
mirrors may be used with the intent of ameliorating network issues or otherwise improving reliability. One approach that
installers could take to preserve mirroring functionality while adding index priority would be to add a notion of
user-definable index groups, where each index in the group is assumed to be equivalent.

This is currently an informational PEP, so we are not defining a standard. Instead, we are showing a pattern that works,
and recommending it as a useful pattern for installers to implement.

## Points of Discussion

- **[UX: General Questions]:**
    - What are situations where we anticipate this feature will help?
    - What are situations where it will cause problems? Are these new problems with this change, or do we just expose or
    exacerbate existing problems? Are these things that we can engineer around?
    - How can we help users discover this feature, especially in situations where it may help them?
    - How to communicate potential unintuitive behaviors with index priority, such as lack of updates from
    higher-priority indexes ?
    - How important is it that the diaspora of tools implement a common index priority scheme?
    - Are there any guidelines we can offer to package authors as far as "we think your tool would benefit your users
    because `_____` and `____`"?
    - Is it up to us to try to advertise this to package maintainers? How detailed/customized to each installer should
    our recommendations be?
    - The Version Priority scheme assumes that a given package name represents identical files on all indexes. Index
    priority allows relaxing that assumption, but should it be allowed? If not, how should we ensure that different
    indexes do not have conflicting metadata for the same package, or de-conflicting the metadata by treating the
    metadata for each index independently (caching scheme change) ?
    - If we relax the assumption that package distributions with a shared name are identical, what needs to change?
    Cache keys?
    - What is a better name for this "package distributions with a shared name are identical", since the topic comes up
    for often in this discussion.
    - Should we develop tools to validate the sanity of a set of indexes and support installers in identifying confusing
    issues ? This problem is not
      unique to index priority, and these kinds of tools would help avoid confusion with any handling of more than one index.

- **[UX: Specifying and configuring Index Priority groups]:**
    Existing configuration will continue to work exactly as-is. Users wishing to utilize the priority feature would
    need to add new configuration. Open questions regarding configuration are included below, but this topic would
    benefit from further discussion and exploration.

    - Should each index group should support the different flags that already exist on an installer?
  
    - How should specification of index groups via command line arguments look? Possibility:

    ```bash
    --index-group=group_name:index_urls=https://blah/simple,https://more/simple;find_links=...;prefer_binary=true;...
    ```

    - How should specification of index groups via flat configuration (e.g. pip.conf) look?

    ```js
    index-groups =
        red-hot-chili-peppers:index_urls=https://blah/simple,https://more/simple;find_links=...;prefer_binary=true;...
        oasis:index_urls=https://blah/simple,https://more/simple;find_links=...;prefer_binary=true;...
    index-group-priority =
        oasis
        red-hot-chili-peppers
    ```

    - How should specification of index groups via hierarchical configuration look? Is this common enough that we should
    explicitly support it?

## Expected Deliverables

By the conclusion of the workshop, our goal is to align on key ideas, strategies, and a consensus to effectively address
the questions discussed.

## Useful Links

- [Index Priority proposal](site:proposals/pep766_explicit_priority_choices/)
- [PEP 766 – Explicit Priority Choices Among Multiple Indexes](https://peps.python.org/pep-0766/)
- [Discussion on discuss.python.org](https://discuss.python.org/t/pep-766-handling-multiple-indexes-index-priority/71589)
- [Github Repository](https://github.com/wheelnext/pep_766/)
- [Partial implementation of index groups in pip](https://github.com/pypa/pip/pull/13210)

## Organizers

- **Organizer 1:** Michael Sarahan, NVIDIA, @msarahan
- **Organizer 2:** Andrey Talman, Meta, @atalman

We look forward to your participation and contributions to making the concept of index priority more well-understood and
ideally universally available!
