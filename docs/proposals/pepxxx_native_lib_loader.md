# PEP ### - Loading shared library dependencies of Python extension modules

| Resource            | Link                                                                                                                                     |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `PEP Link`          | `To Be Published`                                                                                                                        |
| `DPO Discussion`    | `To Be Published`                                                                                                                        |
| `Github Repository` | `To Be Published`                                                                                                                        |

*Note: It is not yet clear whether we will be able to develop a sufficiently general solution to this problem to be PEP-worthy, or if it will be an external solution that works in a documented subset of cases. For now, this document is structured as a potential PEP, but if necessary we will convert it to a less general proposal.

## Abstract

This PEP addresses the challenges and presents solutions for distributing Python extension modules with dependencies on compiled libraries, specifically focusing on the standalone distribution of those dependencies. It aims to enhance current library handling by ensuring that installed binaries can locate their dependencies and that symbols can be resolved safely without the various concerns around library load order and symbol collisions that various solutions currently in use often run into.

## Motivation

Python's ability to leverage existing compiled libraries via extension modules is a critical aspect of its success. However, distributing these libraries with pip is challenging because pip lacks sufficient information to fully describe the compatibility of libraries at the binary level and because pip-distributed libraries can conflict with system-installed ones. This PEP aims to fill these gaps, improving the user experience and reliability of Python package distribution.

## Rationale

Pip needs to satisfy the following criteria to ensure smooth distribution of Python extension modules:
1. Ensuring packages are compatible with low-level system libraries and architecture.
2. Ensuring all packages know how to find each other.
3. Ensuring all installed binaries know how to find their dependencies.
4. Ensuring all packages have compatible application programming interfaces (APIs).
5. Ensuring all installed binaries have compatible application binary interfaces (ABIs).

Currently, pip only satisfies criteria 2 and 4 for Python packages. It lacks the ability to query the system for properties needed to satisfy criterion 1, it does not install packages into system locations where the loader can find them as needed for criterion 3, and it does not fully describe library ABIs in its metadata as would be required for criterion 5. This PEP focuses primarily on addressing criterion 3, with the possibility of extending to address criterion 5 in the future.

To address criterion 3, it is crucial to understand:
- How the loader finds libraries.
- How symbols are resolved in a binary.

This PEP will aim to leverage the answers to these questions to propose a common interface for loading libraries. We will attempt homogeneous solutions across all platforms, but if necessary, we will hide platform-specific solutions behind a common interface. If that is not possible, we will error loudly and provide informational errors on platforms that we cannot support.

## Specification

This PEP proposes the following solutions for ensuring that installed binaries can find their dependencies and that symbols are resolved:

1. **Leveraging Loaded-Libraries List**: Ensure that libraries are loaded with the same key as in the loaded-modules list (install name on OS X, SONAME on Linux). This allows the loader to find dependencies without needing to locate them on the filesystem.
2. **Avoiding Symbol Collisions**: Avoid populating symbols into the global namespace. This minimizes the risk of symbol collisions and ensures that binaries can safely share libraries.
3. **Mangling**: When necessary, mangle library and/or symbol names to avoid conflicts with system-installed libraries.

### Proposed API

*Note: This API is provisional and may be completely changed at any point.*

For packages that contain shared libraries, the following API can be used:

```python
# __init__.py
import shared_lib_manager
import os

root = os.path.dirname(os.path.abspath(__file__))
loader = shared_lib_manager.LibraryLoader(
    {
        "foo": shared_lib_manager.PlatformLibrary(
            Darwin=os.path.join(root, "lib", "libfoo.dylib"),
            Windows=os.path.join(root, "lib", "foo.dll"),
        ),
    },
    mode=shared_lib_manager.LoadMode.{{ load_mode }},
)
```

For packages containing extension modules with shared library dependencies, the following API can be used:

```python
import shared_lib_consumer
shared_lib_consumer.load_library_module("foo")
```

## Backward Compatibility

This PEP proposes changes that are backward-compatible with existing Python packages. It introduces new methods for handling dependencies without altering the current functionality of pip or Python's import system. Existing packages will continue to work as before, with new packages benefiting from enhanced dependency management.

## Security Implications

N/A

## How to Teach This

TBD pending the finalization of the interface

## Reference Implementation

The reference implementation can be found at [native_lib_loader](https://github.com/wheelnext/native_lib_loader/).

## Rejected Ideas

1. **Requiring users to manually install dependencies**: This approach is user-unfriendly and prone to errors.
2. **Bundling all dependencies within each wheel**: This increases binary size and may lead to conflicts with global state or objects in multiple versions of the same library.
3. **Relying solely on environment variables or dynamic loader APIs**: These methods are not portable or universally supported.

## Open Issues

1. **Platform-Specific Behaviors**: Testing and addressing platform-specific behaviors and edge cases to ensure the proposed solutions work consistently across different environments.
2. **Load Order**: Given the load order dependencies of shared libraries, we likely need to augment the proposed solutions with some mechanism for forcing load of libraries on Python startup, e.g. via pth files or similar mechanisms.

## Conclusion

This PEP proposes a comprehensive approach to improving the distribution compiled libraries with pip for use as dependencies of extension modules. By enhancing pip's ability to manage these dependencies, we can improve the user experience, reliability, and security of Python packages.
