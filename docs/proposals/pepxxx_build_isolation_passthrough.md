# PEP ### - Build Isolation Passthrough

| Resource            | Link                                                                |
| ------------------- | ------------------------------------------------------------------- |
| `PEP Link`          | `To Be Published`                                                   |
| `DPO Discussion`    | [\[Build Isolation Passthrough\] Allowing a "package” to passthrough the build isolation](<https://discuss.python.org/t/build-isolation-passthrough-allowing-a-specific-package-to-always-passthrough-the-build-isolation/77284/>) |
| `Github Repository` |                                                                     |

## Abstract

In modern software development, build isolation is a critical practice that ensures the consistency and reliability of
builds. It involves creating an isolated environment where dependencies and build tools are controlled and do not
interfere with the system or other projects. However, there are scenarios where certain tools or dependencies need to
be accessed directly from the host environment.

This proposal outlines a strategy to enhance the build isolation mechanism introduced by
[PEP 518](http://peps.python.org/pep-0518/). Build isolation passthrough is a mechanism that allows
selective access to host resources while maintaining the integrity of the isolated environment.
This feature aims to address the challenges faced when a package depends on the ABI of another package, particularly in
scenarios involving C/C++/Rust/CUDA/etc. dependencies.

## Motivation

With the adoption of `pyproject.toml` as specified in [PEP 518](http://peps.python.org/pep-0518/), build isolation has
become a standard practice in Python packaging. However, there are cases where build isolation can be counterproductive,
especially when dealing with packages that depend on the ABI of other packages. This proposal aims to provide a solution
for such cases, improving the efficiency and flexibility of the build process.

## Rationale

### Current Challenges

1. **Version Mismatch**: During the build phase, pip may download a different version of a dependency than the one
present on the user's machine, leading to potential compatibility issues.
2. **Unreleased/Local Dependencies**: Some dependencies may not be publicly released or may exist only as development versions
on a developer's machine.
3. **Large Dependencies**: Downloading large dependencies repeatedly can significantly increase build times, especially
for packages with large binary dependencies like Artificial Intelligence libraries and frameworks.

### Proposed Solution

Allowing specific packages to opt-in to bypass build isolation for a one or many packages already present on the user's
machine can mitigate these issues. This approach would reduce unnecessary downloads, ensure compatibility with the
user's environment, and decrease build times.

## Specification

### pyproject.toml Configuration

The proposed feature would allow specifying packages that should bypass build isolation in the `pyproject.toml` file.

For example:

```toml
[tool.uv]
no-build-isolation-package = ["package1", "package2"]
```

If the specified packages are present on the user's machine, they will be used directly without downloading them during
the build phase. If they are not present, the build process will proceed as usual, downloading the necessary packages.

## Implementation Details

- **Detection:** The build system should detect the presence of the specified packages on the user's machine.
- **Fallback:** If the packages are not found, the build system should fall back to the standard behavior of downloading
the packages.
- **Compatibility:** This feature should be compatible with existing build isolation mechanisms and should not require
significant changes to the current build process.

## Backward Compatibility

The proposed feature is fully backward compatible, as it introduces an optional configuration in `pyproject.toml`. Existing
projects that do not use this feature will continue to operate as before.

## Security Implications

There is no known security implication associated with this proposal as this proposal is about using packages already
present on the user's machine.

## How to teach this

### Documentation

Comprehensive documentation should be provided, including:

1. An overview of the feature and its benefits.
2. Detailed instructions on configuring pyproject.toml to use the feature.

### Tutorials and Examples

Providing tutorials and examples will help users understand how to effectively use this feature in their projects.

### Reference Implementation

A reference implementation can be found in the `uv` tool, which already supports a similar feature:

```toml
[tool.uv]
no-build-isolation-package = ["package1", "package2"]
```

This implementation can serve as a basis for integrating the feature into `pip`.

## Rejected Ideas

### Disabling Build Isolation Entirely

Disabling build isolation entirely (`--no-isolation`) is not a viable solution, as it negates the benefits of build
isolation, such as ensuring a clean build environment and avoiding conflicts with the user's installed packages.

### Manual Dependency Management

Requiring users to manually manage dependencies is error-prone and can lead to inconsistent build environments.
Automating the process through `pyproject.toml` configuration is a more reliable and user-friendly approach.

## References

- [PEP 518 – Specifying Minimum Build System Requirements for Python Projects](https://peps.python.org/pep-0518/)
- [`uv` Documentation ~ `no-build-isolation-package`](https://docs.astral.sh/uv/reference/settings/#no-build-isolation-package)

## Open issues

- No known issue
