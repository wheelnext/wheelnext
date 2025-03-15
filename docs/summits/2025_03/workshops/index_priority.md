# Workshop - Installer Index Priority

## Abstract

In this workshop, we will focus on brainstorming about the best way to deliver the
Python [Index Priority proposal](/proposals/pep766_explicit_priority_choices.md), which addresses the behavior of Python package installers when handling multiple indexes. And at the same time addresses security risk with associate with usage of `--index-url` option.

## Rapid Summary of the Solution

- **Version priority:** This behavior is characterized by the installer always getting the “best” version of a package, regardless of the index that it comes from. “Best” is defined by the installer’s algorithm for optimizing the various traits of a package, also factoring in user input (such as preferring only binaries, or no binaries). While installers may differ in their optimization criteria and user options, the general trait that all version priority installers share is that the index contents are collated prior to candidate selection.

- **Index priority:** In index priority, the resolver finds candidates for each index, one at a time. The resolver proceeds to subsequent indexes only if the current package request has no viable candidates. Index priority does not combine indexes into one global, flat namespace. Because indexes are searched in order, the package from an earlier index will be preferred over a package from a later index, regardless of whether the later index had a better match with the installer’s optimization criteria. For a given installer, the optimization criteria and selection algorithm should be the same for both index priority and version priority. It is only the treatment of multiple indexes that differs: all together for version priority, and individually for index priority.

- **Mirroring:** The index priority scheme breaks the use case of more than one index url serving the same content. Such mirrors may be used with the intent of ameliorating network issues or otherwise improving reliability. One approach that installers could take to preserve mirroring functionality while adding index priority would be to add a notion of user-definable index groups, where each index in the group is assumed to be equivalent.


## Points of Discussion

Several elements of the proposal require thorough analysis and discussion. We aim to keep the conversation focused on user experience, however we want to discuss some technical details as well.

- **[UX: General Questions]:**
    - How do users encounter this feature, especially in situations where it may help them?
    - What are situations where this feature will help, and also situations where it will cause problems which we can hopefully engineer around.
    - How to make sure this does not break any existing pip functionality, is there a scenario this can fall apart ? (Example conflicts between conda-forge and anaconda channels)
    - How important is it that tools implement a common index priority scheme? Are there any guidelines we can offer to package authors as far as "we think your tool would benefit your users because _____ and ____"

- **[UX: Open issues]:**
    - How to ensure that different indexes do not have conflicting metadata for the same package, or de-conflicting the metadata by treating the metadata for each index independently (caching scheme change) ?
    - How to communicate potential unintuitive behaviors with index priority, such as lack of updates from higher-priority indexes ?
    - Should we develop tools to validate the sanity of a set of indexes and support installers in identifying confusing issues ? This problem is not unique to index priority, and these kinds of tools would help avoid confusion with any handling of more than one index.

- **[UX: Specifying and configuring Index Priority groups]:**
    Existing configuration will continue to work exactly as-is. Users wishing to utilize the priority feature would need to add new configuration.
    - Specification of index groups via command line arguments:
    ```
        --index-group=group_name:index_urls=https://blah/simple,https://more/simple;find_links=...;prefer_binary=true;...
    ```
    - Specification of index groups via pip.conf:
    ```
    index-groups =
        red-hot-chili-peppers:index_urls=https://blah/simple,https://more/simple;find_links=...;prefer_binary=true;...
        oasis:index_urls=https://blah/simple,https://more/simple;find_links=...;prefer_binary=true;...
    index-group-priority =
        oasis
        red-hot-chili-peppers
    ```

## Expected Deliverables

By the conclusion of the workshop, our goal is to align on key ideas, strategies, and a consensus to effectively address the questions discussed.

## Useful Links

- [Index Priority proposal](/proposals/pep766_explicit_priority_choices/)
- [PEP 766 – Explicit Priority Choices Among Multiple Indexes](https://peps.python.org/pep-0766/)
- [Discussion on discuss.python.org](https://discuss.python.org/t/pep-766-handling-multiple-indexes-index-priority/71589)
- [Github Repository](https://github.com/wheelnext/pep_766/)

## Organizers

- **Organizer 1:** [Name, Company, Email - if want to share]
- **Organizer 2:** [Name, Company, Email - if want to share]
- **Organizer 3:** [Name, Company, Email - if want to share]

We look forward to your participation and contributions to making [Subject of Workshop] more efficient and robust!
