# PEP 771 - Default Extras for Python Packages

| Resource               | Link                                                                                                 |
| ---------------------- | ---------------------------------------------------------------------------------------------------- |
| `PEP Link`             | <https://peps.python.org/pep-0771/>                                                                  |
| `DPO Discussion`       | [PEP 771: Default Extras for Python Software Packages](https://discuss.python.org/t/pep-771-default-extras-for-python-software-packages/79706) |
| `DPO Discussion [old]` | [(pre-publish) PEP 771: Default Extras for Python Software Packages](https://discuss.python.org/t/pre-publish-pep-771-default-extras-for-python-software-packages/77892) |
| `Github Repository`    | <https://github.com/wheelnext/pep_771/>                                                              |

## Abstract

PEP 771 proposes a mechanism to allow one or more extras to be installed by default if none are provided explicitly.
This will enhance the way Python package's dependencies are managed, particularly for optional components.

## Motivation

The motivation behind this PEP is to address two main use cases:

1. **Recommended but not required dependencies:** To facilitate the installation of recommended dependencies by default
while allowing a way for users to request minimal installations.

2. **Packages supporting multiple backends or frontends:** To ensure that packages requiring at least one backend or
frontend can specify a default, improving the user experience and preventing broken installations.

## Rationale

This PEP has been discussed extensively within the community and offers an opt-in, flexible solution using the existing
syntax from PEP 508. It addresses key use cases without introducing major backward compatibility issues.

## Specification

### Default-Extra Metadata Field

A new multiple-use metadata field, **Default-Extra**, will be added to the core package metadata. Each entry must be a
string specifying an extra that will be automatically included when the package is installed without any extras specified.

Examples:

```yaml
Default-Extra: recommended
Default-Extra: backend1
Default-Extra: backend2
Default-Extra: backend3
```

### New key in `[project]` metadata table

A new key `default-optional-dependency-keys` will be added to the `[project]` section of the `pyproject.toml` file,
corresponding to the Default-Extra core metadata field.

Example:

```toml
[project]
default-optional-dependency-keys = [
    "recommended",
]

[project.optional-dependencies]
recommended = [
    "package1",
    "package2"
]
```

### Overriding default extras

If extras are explicitly given in a dependency specification, the default extra(s) are ignored.

For example:

```sh
pip install package  # installs default extra
pip install package[extra2]  # installs extra2, ignoring the default extra
```

### Installing without default extras

Package maintainers can define an empty extra (e.g., **minimal**) to allow installations without default extras.

```toml
[project.optional-dependencies]
minimal = []
recommended = [
    "package1",
    "package2"
]
```

## Backward Compatibility

### Packages not using default extras

Packages that do not use default extras will continue to work as-is, with no compatibility issues.

### Packages using default extra(s)

Packages defining default extras will remain installable with older packaging tools, although default extras will not be
installed automatically with older tools.

## Security Implications

There are no known security implications for this PEP.

## How to teach this

### Package end users

Provide clear installation instructions showing available extras and their behaviors.

### Package authors

Authors need to consider backward compatibility, avoid bloating installations with many default dependencies, and ensure
proper handling of circular dependencies.

### Packaging repository maintainers

Repackaging considerations for different distributions need to be addressed, particularly for automated repackaging processes.

## Reference Implementation

A proof-of-concept repository implementing the default extras proposal can be found at <https://github.com/wheelnext/pep_771>. This implementation
uses modified branches of `setuptools` and `pip`.

## Rejected Ideas

### Syntax for deselecting extras

Introducing a new syntax for unselecting extras would create backward compatibility issues and complicate the specification.

### Adding a special entry in extras_require

Using a special name for extras would break compatibility with existing usage and would not work with declarative files.

### Relying on tooling to deselect any default extras

Implementing this at the packaging tool level would not allow packages to specify the exclusion of default extras and
could risk breaking dependency trees.

### `package[]` disables default extras

Using `package[]` to disable default extras would break the current assumption that `package[]` is equivalent to package
and may lead to misuse.

## Open issues

### Should `package[]` disable default extras?

The PEP does not allow package[] to install the package without default extras. However, there are benefits to allowing
this, such as providing a consistent way to get a minimal install. Further investigation is required to understand if
this change would break compatibility with any tools.
