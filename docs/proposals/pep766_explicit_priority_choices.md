# PEP 766 - Explicit Priority Choices Among Indexes

| Resource            | Link                                                                                  |
| ------------------- | ------------------------------------------------------------------------------------- |
| `PEP Link`          | <https://peps.python.org/pep-0766/>                                                   |
| `DPO Discussion`    | [PEP 766: handling multiple indexes (Index Priority)](https://discuss.python.org/t/pep-766-handling-multiple-indexes-index-priority/71589) |
| `Github Repository` | <https://github.com/wheelnext/pep_766/>                                               |

## Abstract

This proposal addresses the behavior of Python package installers when handling multiple indexes. It introduces the concepts
of "version priority" and "index priority" to create a common vocabulary for community discussions and tool
implementations. "Version priority" combines indexes before resolving distributions, while "index priority" handles
each index individually in order.

## Motivation

Python package users often need to specify indexes other than PyPI due to various reasons such as file size limitations,
implementation variants, corporate policies, and local builds. Current installer behavior with multiple indexes,
particularly in pip, can lead to unexpected outcomes. This proposal aims to provide clarity and predictability in
handling multiple indexes by defining explicit priority choices.

## Rationale

The divergence in handling multiple indexes among Python installers, such as pip and uv, necessitates a standardized
approach. This proposal seeks to create a common vocabulary and predictable behavior for package resolution, addressing
issues of trust, security, and user expectations.

## Specification

### Version Priority

- Combines indexes before selecting the best package version.
- Suitable when all indexes are equally trusted and well-behaved.
- Can lead to unexpected outcomes if indexes have different content or trust levels.

### Index Priority

- Resolves packages one index at a time, preferring packages from higher-priority indexes.
- Provides stability and predictability, avoiding "surprise" updates.
- Requires modification of caching and lockfile schemes to include the index origin of a package.
- Relaxes the assumption that all files with the same filename will be identical.

### Mirroring

- Index priority can break the use case of identical content across multiple indexes.
- User-definable index groups can help preserve mirroring functionality.

## Backward Compatibility

This proposal does not mandate changes for any installer but introduces new terminology and behaviors that tools can adopt.
Tools with existing index priority schemes may choose to migrate to these proposed behaviors, or keep on their current paths.

## Security Implications

Index priority allows users to specify a trust hierarchy among indexes, limiting dependency confusion attacks. It
complements but does not replace PEP 708, which addresses dependency confusion attacks from a different angle.

## How to teach this

Promotion of the concepts should occur through message boards, GitHub issue trackers, and chat channels. Documentation
should be updated across various platforms, including PyPUG and pip's documentation, to explain the behaviors and their implications.

## Reference Implementation

The uv project demonstrates index priority with its default behavior. A reference implementation for a Python-based
tool, such as pip, will be provided if necessary. The plan includes opt-in settings, enhanced output, and tracking of
index usage.

## Rejected Ideas

- Setting up a proxy/mirror server, as it requires hosting and may not be accessible in all environments.
- Using build tags and local version specifiers, which are not always viable and have limitations.
- Relying solely on PEP 708, which does not address implementation variants among indexes.
- Namespacing, which does not improve trust expression among indexes as effectively as this proposal.

## Open issues

- Ensuring that different indexes do not have conflicting metadata for the same package.
- Communicating potential issues with index priority, such as lack of updates from higher-priority indexes.
- Developing tools to validate the sanity of a set of indexes and support installers in identifying confusing issues.
